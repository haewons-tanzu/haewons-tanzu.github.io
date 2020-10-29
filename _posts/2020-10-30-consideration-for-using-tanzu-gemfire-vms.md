---
layout: post
title: Consideration for using Tanzu GemFire for VMs
#subtitle: Each post also has a subtitle
gh-repo: hshin-pivotal/hshin-pivotal.github.io
gh-badge: [star, fork, follow]
tags: [Architect, GemFire]
comments: true
---

Apache Geode is a data management platform that provides real-time, consistent access to data-intensive applications throughout widely distributed cloud architectures. Commercially available as Tanzu GemFire by VMware.

There are 3 ways to use GemFire or Apache Geode. 

- Disabling near cache
https://docs.pivotal.io/p-cloud-cache/1-12/session-caching.html


- @EnableClusterAware

https://docs.spring.io/spring-boot-data-geode-build/1.4.x/reference/html5/#geode-session-pcc


However, this does not push Expiration policy metadata to the server yet as that is still a WIP.


So you would need to alter the region after it is created by the Spring Boot application, using:
alter region --name=ClusteredSpringSessions --entry-idle-time-expiration=1800 --entry-idle-time-expiration-action=INVALIDATE


Again make sure to adjust the Region name if you used a different name for the Region when you specified it.


