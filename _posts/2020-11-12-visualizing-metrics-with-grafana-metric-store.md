---
layout: post
title: Visualizing metrics with Grafana and Metric Store on TAS
#subtitle: Each post also has a subtitle
gh-repo: hshin-pivotal/hshin-pivotal.github.io
gh-badge: [star, fork, follow]
tags: [TAS, Monitoring]
comments: true
---

## Environment
- Used sample code [here](https://github.com/resilience4j/resilience4j-spring-boot2-demo){:target="_blank"}.
- Spring Boot 2.3.1: changed Spring Boot version to 2.3.1 from sample code
- Healthwatch 2.0.4
- Metric Store 1.4.4
- Running on TAS (Tanzu Application Service) 2.10.3

## Preparation

### 1. Grafana Dashboard
After installing Healthwatch and Metric Store, you'll have grafana dashboard from Healthwatch created. 

### 2. Metric Store authorization
#### For Metric Store admins
As a Metric Store admin, you will have access to all recorded metrics (platform and application) and can execute Label and Series queries. You will need to add either `doppler.firehose` or `logs.admin` scope to your admin account.

```shell
$ cf login
$ curl -vvv -H "Authorization: $(cf oauth-token)" -G https://metric-store.run.haas-203.pez.pivotal.io/api/v1/label/source_id/values -k
```

#### For PAS developers
As a PAS developer, you can query metrics for applications you have access to. When querying with PromQL, you must specify the source_id label with the application guid as the label value.

```shell
$ curl -H "Authorization: $(cf oauth-token)" -G "https://metric-store.run.haas-203.pez.pivotal.io/api/v1/query" --data-urlencode "query=cpu{source_id=\"$(cf app --guid your-app-name)\"}"
```

$ curl -H "Authorization: $(cf oauth-token)" -G "https://metric-store.run.haas-203.pez.pivotal.io/api/v1/query" --data-urlencode 'query=metric_name_0{source_id="source_id_0"}' -k





Once it is created successfully, it should have dashboard with link "https://my-gateway.cfapps.haas-xxx.pez.pivotal.io/scg-dashboard".

{: .box-error}
**Error:** If you met the error with "<b>Service broker error: env cannot be null</b>" using Spring Cloud Gateway for VMware Tanzu 1.0.11, please check environment values in your app. It seems bug. :(

You can check it with cf cli as below:
```shell
$ cf curl /v2/apps/$(cf app app1 --guid)/summary | jq -r .environment_json
```

In this case, you can add mock value in Apps Manager > Application Info > Settings > User Provided Environment Variables. (For example, Key=APP_NAME, Value=app1) And then, you can try to bind application again. It will work. 


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


## Test

Test is simple. We just invoke 2 applications with the link via SCG, refresh it and check if sessions are kept. "No. of Hits" will be increasing.

```text
Session Id [40cc2617-5c40-4072-8e9c-b6c3373f87d8]
No. of Hits [3]
```
\

For more information about Spring Cloud Gateway for VMware Tanzu, please refer as below:
- https://dev.to/silviobuss/resilience-for-java-microservices-circuit-breaker-with-resilience4j-5c81
- https://docs.google.com/document/d/1EEv4JgTtBcXPri3lpBai4G1yjt2XXtgOXl7EqJlB_wk
- [Using Metric Store](https://docs.pivotal.io/metric-store/1-4/using.html){:target="_blank"}.
