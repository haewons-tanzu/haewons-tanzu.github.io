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
- Used sample code [here](https://github.com/hshin-pivotal/ssdg-gemfire-demo){:target="_blank"}.
- Spring Boot 2.3.1
- VMWare Tanzu GemFire for VMs 1.12
- Spring Cloud Gateway for VMware Tanzu 1.0.11
- Running on TAS (Tanzu Application Service) 2.10.3

## Preparation

### 1. Create Spring Cloud Gateway for VMware Tanzu service instance using cf cli or Apps Manager.
You have to create service instance on TAS to use Spring Cloud Gateway for VMware Tanzu on your application. If you use cf cli, execute as below:

```shell
$ cf create-service p.gateway standard my-gateway -c '{ "host": "my-gateway", "domain": "cfapps.haas-xxx.pez.pivotal.io" }'
```

Once it is created successfully, it should have dashboard with link "https://my-gateway.cfapps.haas-xxx.pez.pivotal.io/scg-dashboard".

{: .box-error}
**Error:** This does not push Expiration policy metadata to the server, so you would need to alter the region after region is created by the Spring Boot application. For more information, please refer [here](2020-10-30-consideration-for-using-tanzu-gemfire-vms){:target="_blank"}. 

### 2. Deploy applications

For testing, we need 2 applications. We'll use sample code in Environment section. Build the source code and deploy 2 applications with different names.
```shell
$ cf push app1
$ cf push app2
```

### 3. Create VMWare Tanzu GemFire service service instance using cf cli for session sharing.

You can refer [here](/2020-10-29-http-session-caching-spring-data-on-tas/){:target="_blank"} to create GemFire service instance.

### 4. Bind SCG service instance to 2 applications separately.

By HttpSessionController class in the source code, it has context path "/session". When we deploy 2 different applications with same source, url would be as below.
- https://app1.cfapps.haas-xxx.pez.pivotal.io/session
- https://app2.cfapps.haas-xxx.pez.pivotal.io/session

```shell
$ cf bind-service app1 my-gateway -c '{ "routes":[ {"path":"/app1/**", "uri":"https://app1.cfapps.haas-xxx.pez.pivotal.io" } ] }'
$ cf bind-service app2 my-gateway -c '{ "routes":[ {"path":"/app2/**", "uri":"https://app2.cfapps.haas-xxx.pez.pivotal.io" } ] }'
```

After binding applications, you should restage these applications.
```shell
$ cf restage app1
$ cf restage app2
```

Now, we can access these 2 applications via SCG (Spring Cloud Gateway).
- https://my-gateway.cfapps.haas-xxx.pez.pivotal.io/app1/session
- https://my-gateway.cfapps.haas-xxx.pez.pivotal.io/app2/session


## Test

Test is simple. We just invoke 2 applications with the link via SCG, refresh it and check if sessions are kept. "No. of Hits" will be increasing.

```text
Session Id [40cc2617-5c40-4072-8e9c-b6c3373f87d8]
No. of Hits [7
```

Session is not lost. Good job!
