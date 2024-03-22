---
title: kubernetes install by kuberspray
categories: [kubernetes,kuberspray, cri-o]
tags: [kubernetes,kuberspray,cri-o]     # TAG names should always be lowercase
published : true
---
# 개요

본 문서는 kuberspray를 활용하여 kubernetes를 구축하고, rook을 사용하여 ceph을 올리기까지의 작업순서와 명령어들을 정리해둔 것임.

## 버전정보

TODO

# 순서
1. memory swap off
2. cri-o 설치
3. kubelet, kubeadm, kubectl 설치
4. cri-o k8s 설정
5. master node에 kubespray 설치
6. master node의 ssh key 생성 및 다른 node들에 copy
7. kubespray 설정(inventory.ini 혹은 host.yaml 사용)
8. ansible-playbook 명령어로 7에서 설정된 내용대로 클러스터 생성
9. Rook 설치, 설정
10. ceph 설정
11. ceph 올리기

## memory swap off

```
sudo swapoff -a
```

## cri-o 설치

먼저, curl 설치 확인

`sudo apt-get install -y apt-transport-https ca-certificates curl`

```


echo "deb [signed-by=/usr/share/keyrings/libcontainers-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_22.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

echo "deb [signed-by=/usr/share/keyrings/libcontainers-crio-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.26/xUbuntu_22.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:1.26.list

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_22.04/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/libcontainers-archive-keyring.gpg

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.26/xUbuntu_22.04/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/libcontainers-crio-archive-keyring.gpg

sudo apt-get update

sudo apt-get install cri-o cri-o-runc
```

## kubelet, kubeadm, kubectl 설치

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.25/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.25/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl
```

버전 고정

`sudo apt-mark hold kubelet kubeadm kubectl`

설치되었는지 확인

`kubectl version`

## cri-o k8s 설정

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

```

## master node에 kuberspray 설치

```
sudo apt install python3
sudo apt update
sudo apt install -y python3-pip
sudo apt install -y git


git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/
sudo pip install -r requirements.txt
```


## master node의 ssh key 생성 및 다른 node들에 copy

```
ssh-keygen
ssh-copy-id 워커노드1ip
ssh-copy-id 워커노드2ip
ssh-copy-id 워커노드3ip


```


master에서 ssh로 타 node들에 비밀번호 없이 접속되는지 확인

```
ssh 워커노드1ip
ssh 워커노드2ip
ssh 워커노드3ip

```

## kubespray 설정(inventory.ini 혹은 host.yaml 사용)

```
cp -rfp inventory/sample/ inventory/mycluster
cd inventory/mycluster
declare -a IPS=(워커노드1ip 워커노드2ip 워커노드3ip)
CONFIG_FILE=inventory/test-cluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

```

## ansible-playbook 명령어로 7에서 설정된 내용대로 클러스터 생성

`ansible-playbook -i inventory/test-cluster/hosts.yaml  --become --become-user=root cluster.yml
`

## Rook 설치, 설정
## ceph 설정
## ceph 올리기





