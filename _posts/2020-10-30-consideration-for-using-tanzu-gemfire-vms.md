---
layout: post
title: Consideration for using "Tanzu GemFire for VMs" as session storage
#subtitle: Each post also has a subtitle
gh-repo: hshin-pivotal/hshin-pivotal.github.io
gh-badge: [star, fork, follow]
tags: [Architect, GemFire]
comments: true
---

"Tanzu GemFire for VMs" is a good solution for storing session data on top of TAS (Tanzu Application Service), but there are some consideration to use it. 

## Disabling near cache



## @EnableClusterAware

If you are using [SBDG](https://github.com/spring-projects/spring-boot-data-geode#spring-boot-for-apache-geode--pivotal-gemfire){:target="_blank"}, then you would annotate your main Spring Boot application class with @EnableClusterAware annotation.

However, this does not push Expiration policy metadata to the server yet as that is still a WIP. So you would need to alter the region after it is created by the Spring Boot application, using:

```shell
gfsh> alter region --name=ClusteredSpringSessions --entry-idle-time-expiration=1800 --entry-idle-time-expiration-action=INVALIDATE
```


For more information, please refer as below.
- [Disable Near Caching Within the App](https://docs.pivotal.io/p-cloud-cache/1-12/session-caching.html){:target="_blank"}
- [@EnableClusterAware limitation](https://docs.spring.io/spring-boot-data-geode-build/1.4.x/reference/html5/#geode-session-pcc){:target="_blank"}
