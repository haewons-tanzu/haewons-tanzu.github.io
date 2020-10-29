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

#### Disabling near cache



#### @EnableClusterAware
[@EnableClusterAware](https://docs.pivotal.io/cloud-cache-dev/spring-boot/basic-cache){:target="_blank"} allows the application to seamlessly switch between local-only (application running on local machine) and client/server (application running on TAS). This annotation includes the @EnableClusterConfiguration annotation, which dynamically creates regions if they do not exist already. Note that the @EnableClusterConfiguration annotation will only create Regions, it will not delete or update existing regions.

If you are using [SBDG](https://github.com/spring-projects/spring-boot-data-geode#spring-boot-for-apache-geode--pivotal-gemfire){:target="_blank"}, then you would annotate your main Spring Boot application class with @EnableClusterAware annotation.

However, this does not push Expiration policy metadata to the server yet as that is still a WIP. So you would need to alter the region after it is created by the Spring Boot application, using:

```shell
gfsh> connect --use-http=true --url=http://cloudcache-8761e54e-1bc0-4855-b96c-819eda347073.xxx.xxx.io/gemfire/v1
--use-ssl=false --skip-ssl-validation --user=cluster_operator_AvK614osQTWzysB8Pd1M4g --password=7eg52Fp8EM3eqpIXH0uRg

gfsh> alter region --name=ClusteredSpringSessions --entry-idle-time-expiration=1800 --entry-idle-time-expiration-action=INVALIDATE
```
I used gfsh (GemFire SHell) cli to alter region attributes. If you don't have gfsh cli in your environment, you can install [here](2020-10-29-gemfire-installation-on-mac){:target="_blank"}. 

{: .box-note}
**Note:** You should install GemFire with same version of "Tanzu GemFire for VMs" on TAS.

#### Data Serialization


For more information, please refer as below.
- [Disable Near Caching Within the App](https://docs.pivotal.io/p-cloud-cache/1-12/session-caching.html){:target="_blank"}
- [@EnableClusterAware limitation](https://docs.spring.io/spring-boot-data-geode-build/1.4.x/reference/html5/#geode-session-pcc){:target="_blank"}
