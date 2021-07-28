---
layout: post
title: TKG Troubleshooting Guide
categories: [tkg, troubleshooting]
---

## Environment
- VMWare Tanzu Kubernetes Grid 1.3.1
- Azure Cloud
- kind cli 0.11.1

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

Now, let's start to troubleshoot for real!!
I have already management cluster named mgmt-cluster, and try to create TKC (Tanzu Kubernetes Cluster). I executed tanzu command as below.
```shell
$ tanzu cluster create -f tkc.yaml -v 9
```

Here, I'm gonna to connect to my management cluster named mgmt-cluster. There is my managent cluster context in ~/.kube/config and I can check the context info using the command as below. Default context name is <MANAGEMENT-CLUSTER-NAME>-admin@<MANAGEMENT-CLUSTER-NAME>.
```shell
$ kubectl config view
```
or 
```shell
$ kubectl config get-contexts
```
Default context name is <MANAGEMENT-CLUSTER-NAME>-admin@<MANAGEMENT-CLUSTER-NAME>. Let's change this context to use it.
```shell
$ kubectl config use-contet mgmt-cluster-admin@mgmt-cluster
```

Check all pods in this management cluster.
```shell
$ kubectl get po -A
```

Then I can see as below.
![tkg-troubleshooting 1](https://raw.githubusercontent.com/haewons-tanzu/haewons-tanzu.github.io/master/static/img/_posts/2021-07-26-tkg-troubleshooting/1.png)

I can see a pod named capz-controller-manager in capz-system namespace. This is the pod which shows logs during 
being executed APIs in Azure.
```shell
k -n capz-system logs capz-controller-manager-7f59fd4bf8-z5wjk manager --tail 1000 --follow
```

I found this error during installing TKC. This error means that I don't have enough vCPU resource. It can be checked 
in Azure portal > Subscription > Usage + quotas. Azure has resource limit per region, and user has to request to increase limits. I requested to increase quotas and re-run to create TKC.
```text
E0727 22:10:45.518001       1 controller.go:257] controller-runtime/controller "msg"="Reconciler error" "error"="failed to reconcile AzureMachine: failed to create virtual machine: failed to create VM tkc-1-control-plane-d2g6s in resource group tkc-1: compute.VirtualMachinesClient#CreateOrUpdate: Failure sending request: StatusCode=0 -- Original Error: autorest/azure: Service returned an error. Status=\u003cnil\u003e Code=\"OperationNotAllowed\" Message=\"Operation could not be completed as it results in exceeding approved standardDSv3Family Cores quota. Additional details - Deployment Model: Resource Manager, Location: KoreaCentral, Current Limit: 10, Current Usage: 8, Additional Required: 4, (Minimum) New Limit Required: 12. Submit a request for Quota increase at https://aka.ms/ProdportalCRP/?#create/Microsoft.Support/Parameters/%7B%22subId%22:%220b504234-3ba1-4543-b3da-640021e6ebab%22,%22pesId%22:%2206bfd9d3-516b-d5c6-5802-169c800dec89%22,%22supportTopicId%22:%22e12e3d1d-7fa0-af33-c6d0-3c50df9658a3%22%7D by specifying parameters listed in the ‘Details’ section for deployment to succeed. Please read more about quota limits at https://docs.microsoft.com/en-us/azure/azure-supportability/per-vm-quota-requests.\"" "controller"="azuremachine" "name"="tkc-1-control-plane-d2g6s" "namespace"="default"
```

Now, my new TKC cluster was created!!
![tkg-troubleshooting 2](https://raw.githubusercontent.com/haewons-tanzu/haewons-tanzu.github.io/master/static/img/_posts/2021-07-26-tkg-troubleshooting/2.png)

For more information, please refer [this](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.3/vmware-tanzu-kubernetes-grid-13/GUID-troubleshooting-tkg-tips.html).