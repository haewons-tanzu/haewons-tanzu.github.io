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

Once it is created successfully, it should have dashboard with "https://my-gateway.cfapps.haas-xxx.pez.pivotal.io/scg-dashboard".

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
$ cf bind-service app1 my-gateway -c '{ "routes":[ {"path":"/session1/**", "uri":"https://app1.cfapps.haas-xxx.pez.pivotal.io" } ] }'
$ cf bind-service app2 my-gateway -c '{ "routes":[ {"path":"/session2/**", "uri":"https://app2.cfapps.haas-xxx.pez.pivotal.io" } ] }'
```


#### 2. gradle build file

{% highlight text linenos %}
group = 'com.vmware.tanzu.gemfire'

}
{% endhighlight %}


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
