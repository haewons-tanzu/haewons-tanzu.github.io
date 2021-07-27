---
layout: post
title: TKG Troubleshooting Guide
categories: [tkg, troubleshooting]
---

## Environment
- VMWare Tanzu Kubernetes Grid 1.3.1
- Azure Cloud
- kind cli 

While you're installing TKG, you may meet some error messages. This document is about how to troubleshoot when you meet such errors.

Once you create management cluster with using CLI command "tanzu management-cluster create", tanzu CLI creates kind cluster in your machine which you executed it. During this process, you only see tanzu related log. However, it's creating kind cluster in your machine.
Let's check the kind cluster log. You can check it in kind cluster or docker container.

#### 1. Checking from the kind cluster
You can check if there exists kind cluster with this command.
```shell
$ kind get clusters
```

Once it exists, then you can connect kind cluster with kunernetes config file. It's located in ~/.kube-tkg/tmp 
directory. Here's useful commands to check logs.
```shell
$ kubectl --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get po,deploy,cluster,kubeadmcontrolplane,machine,machinedeployment –A

$ kubectl --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua get event -A --sort-by=.lastTimestamp

$ kubectl --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-system logs 
capi-controller-manager-c4f5f9c76-sgg2h manager --tail 1000 --follow 

$ kubectl --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n tkg-system describe cluster.cluster.x-k8s.
io/mgmt-cluster

$ kubectl --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n tkg-system describe kubeadmcontrolplane.
controlplane.cluster.x-k8s.io/mgmt-cluster-control-plane

$ kubectl --kubeconfig=/home/azureuser/.kube-tkg/tmp/config_YzSGF3ua -n capi-kubeadm-control-plane-system logs 
pod/capi-kubeadm-control-plane-controller-manager-7f89b8594d-sp8hz manager --tail 1000 –follow
```

If you want to delete failed kind cluster while installing TKG, please execute this command as below.
```shell
$ kind delete cluster --name=tkg-kind-c3r8evlr0ng9hjqthtp0
```

#### 2. Checking from the docker container
You can check running docker process and its container id.
```shell
$ docker ps
```

Go into the docker container.
```shell
$ docker exec -it 39c2e206dab5 /bin/bash
```

Connect kubernetes cluster in docker container and check related resources. Here's useful commands to check logs. 
```shell
$ root@tkg-kind-c3qdk636r0qvq58bti80-control-plane:/# kubectl --kubeconfig=/etc/kubernetes/admin.conf get po -A

$ root@tkg-kind-c3qdk636r0qvq58bti80-control-plane:/# kubectl --kubeconfig=/etc/kubernetes/admin.conf -n kube-system get event --sort-by='.lastTimestamp'

$ kubectl --kubeconfig=/etc/kubernetes/admin.conf -n cert-manager describe po cert-manager-64466f564b-bqhb2

$ kubectl --kubeconfig=/etc/kubernetes/admin.conf -n capi-system logs capi-controller-manager-c4f5f9c76-sgg2h manager --tail 1000 –follow
```

