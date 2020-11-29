---
title: K8s的总结
date: 2020-11-29 19:17:34
categories: 运维
tags:
    - K8s
    - Docker
---

### k8s安装

- 设置yum源
```cmd
vi /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```
- 临时关闭swap
```cmd
swapoff -a
swapon -a
```
- 安装工具
```cmd
yum install -y kubelet kubeadm kubectl
systemctl start kubelet
```
- 启动k8s集群
```cmd
kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.16.0 （--ignore-preflight-errors=all一核报错）
```
- 修改端口号
```cmd
vim /etc/kubernetes/manifests/kube-apiserver.yaml
设置insecure-port=8080
重启 docker restart kube-apiserver_kube-apiserver这个容器
```
- 安装网络插件
```cmd
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
或者
wget https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml
```
- 从节点部署
```cmd
安装kubeadm、kubelet
systemctl start kubelet然后加入集群
kubeadm token create --print-join-command查看加入集群命令
```
- rancher导入k8s
```cmd
wget --no-check-certificate https://10.0.1.186/v3/import/8xhq4r95ptgghqbwx2sgf8t8vlvt5sg6wcqmvspwmn72dh4r7mp9lg.yaml
```
- 一些常用命令
```cmd
kubectl get service
kubectl get deploy
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide
kubectl get deployments --all-namespaces
kubectl delete deployments kubernetes-dashboard -n kube-system
kubectl describe pod -n kube-system 
kubectl describe pod name -o yaml
允许master节点部署pod，使用命令如下:
kubectl taint nodes --all node-role.kubernetes.io/master-
禁止master部署pod
kubectl taint nodes dvbd06 node-role.kubernetes.io/master=true:NoSchedule
列出所有的kind
kubectl api-resources -o wide --namespaced=true
```
- k8s彻底清除脚本
```cmd
docker rm -f $(sudo docker ps -aq);
docker volume rm $(sudo docker volume ls -q);
rm -rf /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico
for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
rm -f /var/lib/containerd/io.containerd.metadata.v1.bolt/meta.db
sudo systemctl restart containerd
sudo systemctl restart docker
```
