---
layout: post
title: HTTP Session Caching with Spring Data on TAS (Tanzu Application Service)
#subtitle: Each post also has a subtitle
gh-repo: hshin-pivotal/hshin-pivotal.github.io
gh-badge: [star, fork, follow]
tags: [Architect, GemFire]
comments: true
---

## What is Tanzu GemFire & Apache Geode?

Apache Geode is a data management platform that provides real-time, consistent access to data-intensive applications throughout widely distributed cloud architectures. Commercially available as Tanzu GemFire by VMware.

There are 3 ways to use GemFire or Apache Geode. 
- Apache Geode: OSS In-Memory Data Grid
- VMware Tanzu GemFire: Commercial version of Apache Geode, VMware supports it. It can be installed on bare metal or VMs.
- VMware Tanzu GemFire for VMs: Commercial version of Apache Geode for TAS(Tanzu Application Service, PaaS Platform)


## HTTP Session Caching

### Environment
- Spring Boot 2.3.1
- VMWare Tanzu GemFire for VMs 1.12
- Running on TAS (Tanzu Application Service) 2.10.3

### Preparation

#### 1. Create Tanzu GemFire service using cf cli.
You have to create service instance on TAS to use Tanzu GemFire service on your application. We need to tag "session-replication" in service instance to store session data in GemFire.

```shell
$ cf create-service p-cloudcache extra-small pcc-session-cache -t session-replication
```

#### 2. Create a region for storing session objects.
Default region name for session storage is ClusteredSpringSessions in SDG (Spring Data Grid). We'll use gfsh (GemFire SHell) cli to create region. 
```shell
gfsh> gfsh> connect --use-http=true --url=http://cloudcache-8761e54e-1bc0-4855-b96c-819eda347073.xxx.xxx.io/gemfire/v1
--use-ssl=false --skip-ssl-validation --user=cluster_operator_AvK614osQTWzysB8Pd1M4g --password=7eg52Fp8EM3eqpIXH0uRg
```
If you don't have gfsh in your environment, you can install [here](2020-10-29-gemfire-installation-on-mac){:target="_blank"}. You should install GemFire with same version of "Tanzu GemFire for VMs" on TAS.

Let's create region named "ClusteredSpringSessions".

```shell
gfsh> create region --name=ClusteredSpringSessions --type=PARTITION_HEAP_LRU
```

### HTTP Session Caching

Now, let's prepare sample session app based Srring Boot.

#### 3. gradle build file

```text
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
```

#### 4. GemFire configuration file in Spring Boot App to enable session caching using @EnableGemFireHttpSession

We set the region name here.

```java
package com.vmware.tanzu.gemfire.session;

import org.springframework.context.annotation.Configuration;

import org.springframework.data.gemfire.config.annotation.EnableLogging;
import org.springframework.session.data.gemfire.config.annotation.web.http.EnableGemFireHttpSession;

@EnableGemFireHttpSession(poolName = "DEFAULT",regionName = "ClusteredSpringSessions")
@EnableLogging(logLevel = "info")
@Configuration
  public class CloudCacheConfig {
}
```

#### 5. Implement Controller for retrieving session value

```java
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
```

#### 6. Create manifest file (manifest.yml)

```yml
---
applications:
  - name: spring-session-demo
    path: ./build/libs/session-0.0.1-SNAPSHOT.jar
    buildpacks: [java_buildpack_offline]
    services: [my-sessioncache]
```

#### 7. Build and push
```shell
$ ./gradlew clean assemble

$ cf push
```


Implement a session API for demonstrating http session offloading. This sample API creates a session object and increments no. of page hits.

https://docs.pivotal.io/p-cloud-cache/1-12/session-caching.html








#### Step 5: Rebuild and push the app

#### Step 6: session API

http://pizza-store-pcc-client.apps.xyz.com/session

```
Session Id [0056EFC36B06C14619B3F14A4ED66272] 
No. of Hits [4]
```








Refered to [the link](https://github.com/Tanzu-Solutions-Engineering/PivotalCloudCache-Workshop/){:target="_blank"} for sampe codes.





## Here is a secondary heading



~~~
var foo = function(x) {
  return(x + 5);
}
foo(3)
~~~

And here is the same code with syntax highlighting:

```javascript
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

And here is the same code yet again but with line numbers:

{% highlight javascript linenos %}
var foo = function(x) {
  return(x + 5);
}
foo(3)
{% endhighlight %}

## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.
