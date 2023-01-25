---
layout: post
title: When you can't see Pull Config and Delivery of Run Cluster in your supply chain - as of TAP 1.3.2

categories: [troubleshooting, tap]
---

Affected version: As of TAP 1.3.2 (Latest: TAP 1.4.0, not fixed)

If you're a user who was [using TAP on multi cluster](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.4/tap/multicluster-about.html) since the initial version(I belive the version is 1.1 :)), you'll be familiar with the supply chain in the screenshot as below. 
![tap-supply-chian 1](https://raw.githubusercontent.com/haewons-tanzu/haewons-tanzu.github.io/master/static/img/_posts/2023-01-25-tap-deliverables/1.png)

However, unfortunately you may not see the Pull Config and Delivery box in the right side located in this supply chain. I found this sympton since TAP 1.3.2. Pull Config and Delivery is configured in run cluster with ["run" profile](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.4/tap/multicluster-reference-tap-values-run-sample.html).

After many times of trial and errors, I fount this is related to [this](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.3/tap/GUID-multicluster-getting-started.html). In a multi cluster environment, we have to copy deliverable.yaml to "run cluster" from "build cluster". Before TAP 1.3.2, we did it using the command as below.
```bash
$ kubectl get deliverable --namespace ${DEVELOPER_NAMESPACE}
```
As of TAP 1.3.2, we do it with this command.
```bash
$ kubectl get configmap tanzu-java-web-app --namespace ${DEVELOPER_NAMESPACE} -o go-template='{ {.data.deliverable}}'
```
According to the documentation, above 2 commands give the same result as below even they're different commands.
```yaml
apiVersion: carto.run/v1alpha1
kind: Deliverable
metadata:
  name: tanzu-java-web-app
  labels:
    apis.apps.tanzu.vmware.com/register-api: "true"
    app.kubernetes.io/part-of: tanzu-java-web-app
    apps.tanzu.vmware.com/workload-type: web
    app.kubernetes.io/component: deliverable
    app.tanzu.vmware.com/deliverable-type: web
spec:
  params:
  - name: gitops_ssh_secret
    value: "git-creds"
  source:
    git:
      url: http://git-server.default.svc.cluster.local/app-namespace/tanzu-java-web-app
      ref:
        branch: main
```

I created it on "run cluster" from copying it "build cluster" following the documentation. However, I could only see the screenshot as below, instead of the expected one.
![tap-supply-chian 2](https://raw.githubusercontent.com/haewons-tanzu/haewons-tanzu.github.io/master/static/img/_posts/2023-01-25-tap-deliverables/2.png)

So, I started digging what configurations might have changed and found that the results of versions prior to TAP 1.3.2 were as follows.
```yaml
apiVersion: carto.run/v1alpha1
kind: Deliverable
metadata:
  name: tanzu-java-web-app
  labels:
    app.kubernetes.io/part-of: tanzu-java-web-app
    apps.tanzu.vmware.com/has-tests: "true"
    apps.tanzu.vmware.com/workload-type: web
    app.kubernetes.io/component: deliverable
    app.tanzu.vmware.com/deliverable-type: web
    carto.run/cluster-template-name: deliverable-template
    carto.run/resource-name: deliverable
    carto.run/supply-chain-name: source-test-scan-to-url
    carto.run/template-kind: ClusterTemplate
    carto.run/workload-name: simple-demo
    carto.run/workload-namespace: dev-team-01
spec:
  params:
  - name: gitops_ssh_secret
    value: git-creds
  source:
    git:
      url: https://github.com/haewons-tanzu/tap-gitops-repo.git
      ref:
        branch: master
    subPath: config/dev-team-01/tanzu-java-web-app
```

Please note the part of labels in metadata. It was added 6 more line starting with carto.run.

I added these 6 lines starting with carto.run in deliverables in "run cluster", and it worked normally. :)

I believe something is happening in the new features of TAP. In fact, "View Approval" button for approval process disappeared in TAP 1.4. Instead, new arrow mark in Config Writer box appeared in the right corner.

TAP is improving and getting better and better. I will continue to keep an eye on these features.

