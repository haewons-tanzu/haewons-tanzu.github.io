---
layout: post
title: TKG Troubleshooting Guide
categories: [TKG, Troubleshooting]
---

## Environment
- VMWare Tanzu Kubernetes Grid 1.3.1
- Azure Cloud

## Preparation

### 1. Create Spring Cloud Gateway for VMware Tanzu service instance using cf cli or Apps Manager.
You have to create service instance on TAS to use Spring Cloud Gateway for VMware Tanzu on your application. If you use cf cli, execute as below:

```shell
$ cf create-service p.gateway standard my-gateway -c '{ "host": "my-gateway", "domain": "cfapps.haas-xxx.pez.pivotal.io" }'
```

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
155  docker exec -it 39c2e206dab5 /bin/bash
  156  cd
  157  ls -la
  158  cd .tanzu
  159  cd ../
  160  cd .kube-tkg
  161  ls -la
  162  cd tmp
  163  ls -la
  164  pwd
  165  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get all -A
  166  source ~/.profile
  167  source . ~/.profile
  168  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get all -A
  169  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system get po
  170  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system describe po capi-controller-manager-c4f5f9c76-sgg2h
  171  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system logs capi-controller-manager-c4f5f9c76-sgg2h --tail --follow
  172  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system logs capi-controller-manager-c4f5f9c76-sgg2h --tail --follow manager
  173  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system logs capi-controller-manager-c4f5f9c76-sgg2h --tail 1000 --follow manager
  174  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system logs capi-controller-manager-c4f5f9c76-sgg2h manager --tail 1000 --follow
  175  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system get po
  176  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get po -A
  177  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get po,deploy,cluster,kubeadmcontrolplane,machine,machinedeployment -A
  178  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-kubeadm-control-plane-system logs pod/capi-kubeadm-control-plane-controller-manager-7f89b8594d-sp8hz
  179  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-kubeadm-control-plane-system logs pod/capi-kubeadm-control-plane-controller-manager-7f89b8594d-sp8hz manager
  180  tanzu management-cluster get
  181  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get po,deploy,cluster,kubeadmcontrolplane,machine,machinedeployment -A
  182  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n tkg-system cluster.cluster.x-k8s.io/mgmt-cluster
  183  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n tkg-system describe cluster.cluster.x-k8s.io/mgmt-cluster
  184  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get po,deploy,cluster,kubeadmcontrolplane,machine,machinedeployment -A
  185  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n tkg-system describe kubeadmcontrolplane.controlplane.cluster.x-k8s.io/mgmt-cluster-control-plane
  186  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get event -A
  187  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get event -A --sort-by=.lastTimestamp
  188  cd tanzu
  189  ls -la
  190  history
  191  cd /home/azureuser/.kube-tkg/tmp/
  192  ls -la
  193  k --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_RojJKF4q get event -A --sort-by=.lastTimestamp
  194  ls -la
  195  rm -rf config_RojJKF4q config_YzSGF3ua
  196  ls -;a
  197  ls -la
  198  cd tanzu
  199  ls -la
  200  cd ../.tanzu
  201  ls -la
  202  cd tkg
  203  ls -la
  204  cd cl*s
  205  ls -la
  206  vi aw*
  207  exit
  208  cd tanzu
  209  vi mc.yaml
  210  ls -la
  211  cd .tanzu
  212  ls -la
  213  cd tkg
  214  ls -la
  215  cd pr*
  216  ls -la
  217  cd in*azure
  218  ls -la
  219  cd ytt
  220  ls -la
  221  vi a*
  222  cd .kube
  223  cd ../
  224  cd .kube-tkg
  225  ls -la
  226  cd tmp
  227  cd config
  228  ls -la
  229  k --kubeconfig=./config_BvSodLNh get all -A
  230  k --kubeconfig=./config_BvSodLNh -n capi-kubeadm-control-plane-system describe replicaset.apps/capi-kubeadm-control-plane-controller-manager-7f89b8594d
  231  k --kubeconfig=./config_BvSodLNh get event -A --sort-by=.lastTimestamp
  232  docker ps
  233  ls -la
  234  rm -rf config_BvSodLNh
  235  k --kubeconfig=./config_GKhfOfvB get event -A
  236  k --kubeconfig=./config_GKhfOfvB get event -A --sort-by=.lastTimestamp
  237  docker ps
  238  ls -la
  239  rm -rf config_GKhfOfvB
  240  ls -la
  241  k --kubeconfig-./config_zIqpsvIa get po -A
  242  ls -la
  243  k --kubeconfig=./config_zIqpsvIa get po -A
  244  k --kubeconfig=./config_zIqpsvIa -n capi-kubeadm-control-plane-system get event
  245  k --kubeconfig=./config_zIqpsvIa -n capi-kubeadm-control-plane-system get event --sort-by=.lastTimestamp
  246  k --kubeconfig=./config_zIqpsvIa get po -A
  247  ls -la
  248  tanzu management-cluster get
  249  tanzu cluster list
  250  tanzu management-cluster get tkc-1
  251  tanzu management-cluster list
  252  tanzu cluster list
  253  tanzu cluster get tkc-1
  254  tanzu cluster list
  255  history
  256  tanzu management-cluster create -f mc.yaml -v 9
  257  ls -la
  258  cd tanzu
  259  tanzu management-cluster create -f mc.yaml -v 9
  260  tanzu management-cluster get
  261  vi mc.yaml
  262  tanzu management-cluster create -f mc.yaml -v 9
  263  tanzu get  lusters
  264  tanzu get clusters
  265  kind get clusters
  266  kind delete cluster --name=
  267  tkg-kind-c3r9kt5r0ngc99sp7770
  268  kind delete cluster --name=
  269  kind delete cluster --name=tkg-kind-c3r8evlr0ng9hjqthtp0
  270  kind delete cluster --name=tkg-kind-c3r9kt5r0ngc99sp7770
  271  kind delete cluster --name=tkg-kind-c3ra4u5r0ng5kpktik10
  272  vi mc.yaml
```
