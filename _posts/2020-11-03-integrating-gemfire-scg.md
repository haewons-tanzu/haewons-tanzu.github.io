---
layout: post
title: Integrating SCG with HTTP Session Caching with Spring Data on TAS
#subtitle: Each post also has a subtitle
gh-repo: hshin-pivotal/hshin-pivotal.github.io
gh-badge: [star, fork, follow]
tags: [Architect, GemFire, SCG]
comments: true
---

## Environment
- Used test code [here](/2020-10-29-http-session-caching-spring-data-on-tas/){:target="_blank"}.
- Spring Boot 2.3.1
- VMWare Tanzu GemFire for VMs 1.12
- Spring Cloud Gateway for VMware Tanzu 1.0.11
- Running on TAS (Tanzu Application Service) 2.10.3

## Preparation

#### 1. Create Spring Cloud Gateway for VMware Tanzu service using cf cli or Apps Manager.
You have to create service instance on TAS to use Spring Cloud Gateway for VMware Tanzu on your application. If you use cf cli, execute as below:

```shell
$ cf create-service p.gateway standard my-gateway -c '{ "host": "my-gateway", "domain": "cfapps.haas-xxx.pez.pivotal.io" }'
```

### 2. Deploy applications

For testing, we need 2 applications. We'll use applications with 

Now, let's prepare sample session app based Srring Boot.

#### 2. gradle build file

{% highlight text linenos %}
group = 'com.vmware.tanzu.gemfire'
version = '0.0.1-SNAPSHOT'

buildscript {
    ext {
        springBootVersion = '2.3.1.RELEASE'
    }
    repositories {
        mavenCentral()
        maven { url "https://repo.spring.io/snapshot" }
        maven { url "https://repo.spring.io/milestone" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'idea'

idea{
    module{
        downloadSources = true
        downloadJavadoc = true
    }
}

sourceCompatibility = '1.8'
targetCompatibility = '1.8'

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-milestone" }
    maven { url "https://repo.spring.io/milestone" }
    maven { url "http://repo.springsource.org/simple/ext-release-local" }
    maven { url "http://repo.spring.io/libs-release/" }
    maven { url "https://repository.apache.org/content/repositories/snapshots" }
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    compile("org.springframework.boot:spring-boot-starter-web:2.3.1.RELEASE")
    compile("org.springframework.session:spring-session-data-gemfire:2.3.1.RELEASE")
    compile("org.springframework.geode:spring-gemfire-starter:1.2.0.RELEASE")
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}

test {
    useJUnitPlatform()
}
{% endhighlight %}

#### 3. Configiration to create cache region using @EnableClusterAware

We set the region name here.

{% highlight java linenos %}
package com.vmware.tanzu.gemfire.session;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.geode.config.annotation.EnableClusterAware;

@SpringBootApplication
@EnableClusterAware
public class SessionApplication {

    public static void main(String[] args) {
        SpringApplication.run(SessionApplication.class, args);
    }

}
{% endhighlight %}

{: .box-note}
**Note:** This does not push Expiration policy metadata to the server, so you would need to alter the region after region is created by the Spring Boot application. For more information, please refer [here](2020-10-30-consideration-for-using-tanzu-gemfire-vms){:target="_blank"}. 

#### 4. GemFire configuration file in Spring Boot App to enable session caching using @EnableGemFireHttpSession

{% highlight java linenos %}
package com.vmware.tanzu.gemfire.session;

import org.springframework.context.annotation.Configuration;

import org.springframework.data.gemfire.config.annotation.EnableLogging;
import org.springframework.session.data.gemfire.config.annotation.web.http.EnableGemFireHttpSession;

@EnableGemFireHttpSession(poolName = "DEFAULT",regionName = "ClusteredSpringSessions")
@EnableLogging(logLevel = "info")
@Configuration
  public class CloudCacheConfig {
}
{% endhighlight %}

Default region name for session storage is ClusteredSpringSessions in [SSDG (Spring Session Data Grid)](https://spring.io/projects/spring-session-data-geode){:target="_blank"}.

{: .box-note}
**Note:** Default region name for session storage is gemfire_modules_sessions in VMware Tanzu GemFire. For more its default values, please refer [here](https://gemfire.docs.pivotal.io/910/geode/tools_modules/http_session_mgmt/tomcat_changing_gf_default_cfg.html){:target="_blank"}. 

Here, we enable GemFire session region using EnableGemFireHttpSession. If we skip this step, we'll see this error.

```console
2020-10-29T08:13:31.808-07:00 [APP/PROC/WEB/0] [OUT] 2020-10-29 15:13:31.808 ERROR 25 --- [ main] o.s.b.web.embedded.tomcat.TomcatStarter : Error starting Tomcat context. Exception: org.springframework.beans.factory.UnsatisfiedDependencyException. Message: Error creating bean with name 'springSessionRepositoryFilter' defined in class path resource [org/springframework/session/data/gemfire/config/annotation/web/http/GemFireHttpSessionConfiguration.class]: Unsatisfied dependency expressed through method 'springSessionRepositoryFilter' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'sessionRepository' defined in class path resource [org/springframework/session/data/gemfire/config/annotation/web/http/GemFireHttpSessionConfiguration.class]: Unsatisfied dependency expressed through method 'sessionRepository' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'ClusteredSpringSessions' defined in class path resource [org/springframework/session/data/gemfire/config/annotation/web/http/GemFireHttpSessionConfiguration.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: [gemfirePool] is not resolvable as a Pool in the application context
```

#### 5. Implement Controller for retrieving session value

{% highlight java linenos %}
package com.vmware.tanzu.gemfire.session;

import javax.servlet.http.HttpSession;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HttpSessionController {
    private static final Logger LOGGER = LoggerFactory.getLogger(HttpSessionController.class);

    @RequestMapping(value = "/session")
    public String index(HttpSession httpSession) {

        Integer hits = (Integer) httpSession.getAttribute("hits");

        LOGGER.info("No. of Hits: '{}', session id '{}'", hits, httpSession.getId());

        if (hits == null) {
            hits = 0;
        }

        httpSession.setAttribute("hits", ++hits);
        return String.format("Session Id [<b>%1$s</b>] <br/>"
                + "No. of Hits [<b>%2$s</b>]%n", httpSession.getId(), httpSession.getAttribute("hits"));
    }

}
{% endhighlight %}

#### 6. Create manifest file (manifest.yml)
Once configuring service here, you don't need to bind GemFire service to Spring Boot app. Enough pushing app!

{% highlight yml linenos %}
---
applications:
  - name: spring-session-demo
    path: ./build/libs/session-0.0.1-SNAPSHOT.jar
    buildpacks: [java_buildpack_offline]
    services: [my-sessioncache]

{% endhighlight %}

#### 7. Build and push

```shell
$ ./gradlew clean assemble

$ cf push
```

#### 8. Test

Let's invoke app url we've pushed. 

```text
https://spring-session-demo.xxx.xxx.io/session
```

Web page will be shown as below. When we refresh it, "No. of Hits" will be increasing.
```text
Session Id [604f75f7-8ec8-4607-8373-56f054f25653]
No. of Hits [2]
```

Now, we restart this app on Apps Manager, which is a console for developers in Tanzu Application Service. Just click "restart" button. If session is stored in app container itself, then session is initialized, session id will be changed and value of "No. of Hits" is 1. 

However, we stored session value in GemFire service instance, there's no affect!. That's why we use IMDG (In-Memory Data Grid). After restarting app, let's refresh the browser.

```text
Session Id [604f75f7-8ec8-4607-8373-56f054f25653]
No. of Hits [7]
```

Session is not lost. Good job!

I'll share codes [here](https://github.com/hshin-pivotal/ssdg-gemfire-demo){:target="_blank"}.

Refered to [the link for sample codes](https://github.com/Tanzu-Solutions-Engineering/PivotalCloudCache-Workshop/){:target="_blank"} and document for [Connecting a Spring Boot App to VMware Tanzu GemFire with Session State Caching](https://docs.pivotal.io/p-cloud-cache/1-12/Spring-SessionState.html){:target="_blank"}.
