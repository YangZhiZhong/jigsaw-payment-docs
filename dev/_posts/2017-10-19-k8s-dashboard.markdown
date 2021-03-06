---
layout: post 
title: "Kubernetes Dashboard安装指南"  
subtitle: "infra文档"  
date: 2017-10-20 12:00:00  
author: "shamphone"  
header-img: "img/home-bg-post.jpg"  
catalog: true  
tag: [infra]  
---

> 注意，这个文件必须以UTF-8无BOM格式编码。 

Dashboard用来监控k8s集群负载。 [官网](https://github.com/kubernetes/dashboard)、[安装指南](https://github.com/kubernetes/dashboard/wiki/Installation)

## 一、下载镜像

老规矩，把安装中需要的镜像下载到服务器上。 
```bash
docker pull  registry.cn-hangzhou.aliyuncs.com/outman_google_containers/kubernetes-dashboard-init-amd64:v1.0.1
docker tag registry.cn-hangzhou.aliyuncs.com/outman_google_containers/kubernetes-dashboard-init-amd64:v1.0.1   gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.1
docker rmi -f registry.cn-hangzhou.aliyuncs.com/outman_google_containers/kubernetes-dashboard-init-amd64:v1.0.1

docker pull  registry.cn-hangzhou.aliyuncs.com/outman_google_containers/kubernetes-dashboard-amd64:v1.7.1
docker tag registry.cn-hangzhou.aliyuncs.com/outman_google_containers/kubernetes-dashboard-amd64:v1.7.1  gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.1
docker rmi -f registry.cn-hangzhou.aliyuncs.com/outman_google_containers/kubernetes-dashboard-amd64:v1.7.1
```

## 二、安装


安装很简单，首先生成证书：

```bash
$ openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
...
$ openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
# Writing RSA key
$ rm dashboard.pass.key
$ openssl req -new -key dashboard.key -out dashboard.csr
# 一路回车即可，特别是设置密码的步骤，直接回车，不要密码
...
Country Name (2 letter code) [AU]: US
...
A challenge password []: 
...
$ openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
```
将生成的dashboard.csr, dashboard.crt, dashboard.key转移到 $HOME/certs 目录下，结果如下

```bash
[jigsaw@kube-master ~]$ ls certs
dashboard.crt  dashboard.csr  dashboard.key
```

然后执行：

```bash
kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

确认节点正常运行

```bash
[jigsaw@kube-master ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                         READY     STATUS    RESTARTS   AGE
kube-system   etcd-kube-master.jigsaw                      1/1       Running   0          1h
kube-system   kube-apiserver-kube-master.jigsaw            1/1       Running   0          1h
kube-system   kube-controller-manager-kube-master.jigsaw   1/1       Running   0          1h
kube-system   kube-dns-2425271678-jpn04                    3/3       Running   0          1h
kube-system   kube-flannel-ds-r03d4                        1/1       Running   0          1h
kube-system   kube-flannel-ds-wfmx0                        1/1       Running   0          1h
kube-system   kube-proxy-fx6pf                             1/1       Running   0          1h
kube-system   kube-proxy-s7ndf                             1/1       Running   0          1h
kube-system   kube-scheduler-kube-master.jigsaw            1/1       Running   0          1h
kube-system   kubernetes-dashboard-1592587111-lz8cx        1/1       Running   0          17m

```bash

注意kubernetes-dashboard-1592587111-lz8cx   是running的状态。 

## 二、本机和远程访问

默认的，dashboard只允许本机访问，需要修改为可以从其他机器来访问。 [官方指导](https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above)

```bash
 kubectl -n kube-system edit service kubernetes-dashboard
```
将type: ClusterIP 修改为 type: NodePort
```bash
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
...
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "343478"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard-head
  uid: 8e48f478-993d-11e7-87e0-901b0e532516
spec:
  clusterIP: 10.100.124.90
  externalTrafficPolicy: Cluster
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

查看分配的端口
```bash
$ kubectl -n kube-system get service kubernetes-dashboard
NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes-dashboard   10.100.124.90   <nodes>       443:32530/TCP   21h
```

在浏览器中访问： https://192.168.1.112:32530/

## 三、登录Token

登录页面，需要Token。 
执行
```bash
 kubectl get secret -n kube-system
 NAME                                     TYPE                                  DATA      AGE
attachdetach-controller-token-5bxjx      kubernetes.io/service-account-token   3         1d
bootstrap-signer-token-740cj             kubernetes.io/service-account-token   3         1d
bootstrap-token-eabf28                   bootstrap.kubernetes.io/token         5         1d
certificate-controller-token-x93wb       kubernetes.io/service-account-token   3         1d
daemon-set-controller-token-nhpwt        kubernetes.io/service-account-token   3         1d
default-token-qpvrp                      kubernetes.io/service-account-token   3         1d
deployment-controller-token-915lt        kubernetes.io/service-account-token   3         1d
disruption-controller-token-j82bb        kubernetes.io/service-account-token   3         1d
endpoint-controller-token-j6mwq          kubernetes.io/service-account-token   3         1d
flannel-token-671bz                      kubernetes.io/service-account-token   3         1d
generic-garbage-collector-token-vddlx    kubernetes.io/service-account-token   3         1d
horizontal-pod-autoscaler-token-fxc2j    kubernetes.io/service-account-token   3         1d
job-controller-token-q4lvm               kubernetes.io/service-account-token   3         1d
kube-dns-token-9lcmg                     kubernetes.io/service-account-token   3         1d
kube-proxy-token-0vzpd                   kubernetes.io/service-account-token   3         1d
kubernetes-dashboard-certs               Opaque                                0         23h
kubernetes-dashboard-key-holder          Opaque                                2         23h
kubernetes-dashboard-token-3qd9d         kubernetes.io/service-account-token   3         23h
kubernetes-dashboard-token-k17mr         kubernetes.io/service-account-token   3         23h
kubernetes-dashboard-token-r4dqm         kubernetes.io/service-account-token   3         23h
kubernetes-dashboard-token-xv2xp         kubernetes.io/service-account-token   3         23h
kubernetes-dashboard-token-zpbmn         kubernetes.io/service-account-token   3         23h
namespace-controller-token-r09l6         kubernetes.io/service-account-token   3         1d
node-controller-token-j6pmv              kubernetes.io/service-account-token   3         1d
persistent-volume-binder-token-jgj6v     kubernetes.io/service-account-token   3         1d
pod-garbage-collector-token-pmxr7        kubernetes.io/service-account-token   3         1d
replicaset-controller-token-c0dmb        kubernetes.io/service-account-token   3         1d
replication-controller-token-4f1q9       kubernetes.io/service-account-token   3         1d
resourcequota-controller-token-jb51f     kubernetes.io/service-account-token   3         1d
service-account-controller-token-mj9pk   kubernetes.io/service-account-token   3         1d
service-controller-token-vlvz9           kubernetes.io/service-account-token   3         1d
statefulset-controller-token-ct4t6       kubernetes.io/service-account-token   3         1d
token-cleaner-token-kff9k                kubernetes.io/service-account-token   3         1d
ttl-controller-token-r47hb               kubernetes.io/service-account-token   3         1d

```
挑一个你喜欢的，比如 kube-proxy-token-0vzpd ， 执行
```bash
[jigsaw@kube-master ~]$ kubectl describe secret/kube-proxy-token-0vzpd -n kube-system
Name:           kube-proxy-token-0vzpd
Namespace:      kube-system
Labels:         <none>
Annotations:    kubernetes.io/service-account.name=kube-proxy
                kubernetes.io/service-account.uid=9063e1eb-bc9e-11e7-92ea-f0def12142f8

Type:   kubernetes.io/service-account-token

Data
====
ca.crt:         1025 bytes
namespace:      11 bytes
token:          eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlLXByb3h5LXRva2VuLTB2enBkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt1YmUtcHJveHkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI5MDYzZTFlYi1iYzllLTExZTctOTJlYS1mMGRlZjEyMTQyZjgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06a3ViZS1wcm94eSJ9.CJXU-Cctqvwdd3Cdw4sWCqLtV1d29DnozcED5AZLqVgvXfSAj-bmSzYrUopSqhMCmaN32KS17FaFhB8cG0eJNrV5y9KhLPm875tNHHlU31FYCunb_j2gEXX3_-xs6wQBu3OHqA3ISgqQNkajFSlYXanm6LCRwi70zkXXQxagVcPPGiNgc871cyJNxY7UKgn5K-m-wtliDjPIGeChR_pUaI5759pwmY1cD41ZcZiTLJILojJBeYiTzbFrI00jM_HysHNr_ZypAZRK9CfNnlPbcMemfPHLZsI-Hd9LZgRgJ1MNrjSvChJEcM6E1nPOHDxYJhmOJkcYhdDfoRsB1x6C9g
```
将这里的token复制进来即可。 你会发现，大量的功能访问不了。这和k8s的权限设置有关。 

有必要弄一个admin权限的用户， [参考这篇文章：使用kubeadm安装Kubernetes 1.8](http://blog.frognew.com/2017/09/kubeadm-install-kubernetes-1.8.html#8dashboard%E6%8F%92%E4%BB%B6%E9%83%A8%E7%BD%B2)
创建 kubernetes-dashboard-admin.rbac.yaml， 内容如下
```bash
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-admin
  namespace: kube-system
  
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```

执行

```bash
[jigsaw@kube-master ~]$ kubectl create -f kubernetes-dashboard-admin.rbac.yaml
serviceaccount "kubernetes-dashboard-admin" created
clusterrolebinding "kubernetes-dashboard-admin" created
```

获取 kubernetes-dashboard-admin 的 token

```bash

[jigsaw@kube-master ~]$ kubectl -n kube-system get secret | grep kubernetes-dashboard-admin
kubernetes-dashboard-admin-token-r5ptk   kubernetes.io/service-account-token   3         18s
[jigsaw@kube-master ~]$ kubectl describe secret kubernetes-dashboard-admin-token-8b3zs -n kube-system > kubernetes-dashboard-admin.token
Error from server (NotFound): secrets "kubernetes-dashboard-admin-token-8b3zs" not found
[jigsaw@kube-master ~]$ kubectl describe secret kubernetes-dashboard-admin-token-r5ptk -n kube-system > kubernetes-dashboard-admin.token
[jigsaw@kube-master ~]$ cat kubernetes-dashboard-admin.token
Name:           kubernetes-dashboard-admin-token-r5ptk
Namespace:      kube-system
Labels:         <none>
Annotations:    kubernetes.io/service-account.name=kubernetes-dashboard-admin
                kubernetes.io/service-account.uid=48aa6ae4-bd82-11e7-92ea-f0def12142f8

Type:   kubernetes.io/service-account-token

Data
====
ca.crt:         1025 bytes
namespace:      11 bytes
token:          eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1yNXB0ayIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQ4YWE2YWU0LWJkODItMTFlNy05MmVhLWYwZGVmMTIxNDJmOCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.uAG9VtE-gdm6RU8ItuxTHIVG8o8UH4EQen-cJq50lShbt5fnslOV_UVpFFJhol8tZEhymCYVYtKGjW_wjWcqPJ_eTCSKFcPrQHNaCRyeHfYtjyUui01OfRVQiNzZ3L0W5B_Zhdyq8YELRUkvrdkM96uGCPMySmgwkcOvvYxuVxtr-9sJs7CqkD8P6cpZDe6_0UdknsaZN8MRTXU0vzZmPVeEx1PJQKp0KDGTnYyKz6xUidlqUoaPx4nBuYceRNTsBaSoSWZsw_QPaykUlGbTeumR3W_znp0Vfxen6AWTSgw95h00f-rhfUT1ZTbc-vTmpp4hYsFOKQQvMIPMTUKj5w
```

将Token复制到登录框中，即可登录。 
