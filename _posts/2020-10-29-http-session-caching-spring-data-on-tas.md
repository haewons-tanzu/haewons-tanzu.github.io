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

In this document, I'll use this environment.
- Spring Boot 2.3.1
- VMWare Tanzu GemFire for VMs 1.12
- Running on TAS (Tanzu Application Service) 2.10.3

### Preparation

#### 1. Create Tanzu GemFire service using cf cli.
You have to create service instance on TAS to use Tanzu GemFire service on your application. We'll store session data, and need to tag "session-replication" in service instance.

```shell
cf create-service p-cloudcache extra-small pcc-session-cache -t session-replication
```

#### 2. Create a region for storing session objects.
Default 


Implement a session API for demonstrating http session offloading. This sample API creates a session object and increments no. of page hits.

#### Step 1: Create a region for storing session objects. Default region name is ClusteredSpringSessions

### HTTP Session Caching

https://docs.pivotal.io/p-cloud-cache/1-12/session-caching.html


```
create region --name=ClusteredSpringSessions --type=PARTITION_HEAP_LRU
```

#### Step 2: Update the pom.xml to include the spring-session-data-gemfire dependency

```
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-geode</artifactId>
</dependency>
```

#### Step 3: navigate to configuration file and enable session caching using @EnableGemFireHttpSession

```
...
@EnableGemFireHttpSession(poolName = "DEFAULT",regionName = "ClusteredSpringSessions")
@Configuration
public class CloudCacheConfig {
}
```

#### Step 4: Implement a controller for demonstrating session caching.

```
package io.pivotal.data.controller;

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
