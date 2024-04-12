<!-- ---
title: kubernetes install by kubespray
categories: [kubernetes,kubespray, cri-o]
tags: [kubernetes,kubespray,cri-o]     # TAG names should always be lowercase
published : true
--- -->
***kubernetes install by kubespray***

# 개요

kubespray를 활용하여 kubernetes를 구축하고, rook을 사용하여 ceph을 올리기까지의 작업순서와 에러 대처방법 정리

## 버전정보

OS : xUbuntu_22.04
CRI-O : 1.25
kubernetes : v1.25.6
kubespray : 2.21
rook-ceph : v1.13.7

# 순서
1. 설치할 각종 소프트웨어의 호환성 조사
2. memory swap off
3. kubespray 설치
4. ssh key 생성 및 다른 node들에 copy
5. 각 노드의 해당 user가 sudo 명령어를 비밀번호 입력 없이 실행할 수 있도록 설정
6. kubespray 설정(inventory.ini 혹은 host.yaml 사용)
7. crio 사용 설정
8. ansible-playbook 명령어로 클러스터 생성
9. 정상적으로 설치되었는지 확인


kubespray를 통해 클러스터를 만드는 명령어가 정상적으로 동작할 조건은 다음과 같다.

1. 모든 node의 swap 설정이 꺼져있을 것

2. kubespray를 실행하는 pc에서, 모든 노드의 에 ssh로 비밀번호 없이, ssh-key를 통해 접근이 가능할 것

3. 그렇게 접근한 유저가 sudo 명령어를 비밀번호 입력 없이 실행할 수 있을 것

따라서 하나씩 차근차근 설정해보자.
<!-- 2. cri-o 설치
3. kubelet, kubeadm, kubectl 설치
4. cri-o k8s 설정 -->
# 설치할 각종 소프트웨어 호환성 조사

가장 중요한 것은, 각 소프트웨어의 버전이 서로 호환되어야 한다는 것이다.

필자의 경우 이를 확인하지 않고 설치하였는데, kubespray 2.2에서 설치하여 사용되는 calico의 버전이 kubernetes 1.25버전과 호환되지 않아 오랜 시간 헤맸다.

이런 문제가 발생할 경우, 에러 로그가 엉뚱히 나와 직접적인 호환성과의 연관성을 찾기 매우 어려울 수 있으므로, 

이를 방지하기 위해서 각 홈페이지에서 각 소프트웨어가 서로 어떤 버전이 호환되는지를 반드시 먼저 파악하고 정리한 후 시작해야 한다.

그 예시로, [calico홈페이지](https://docs.tigera.io/calico/latest/getting-started/kubernetes/requirements)의 `Kubernetes requirements` 탭에서, 호환되는 kubernetes의 버전을 확인할 수 있다.

# memory swap off

쿠버네티스를 사용하려면 가장 기본적으로 모든 노드의 memory swap이 off 되어있어야 한다. 각 노드에서 아래 명령어로 swap을 끈다

```
sudo swapoff -a
```

## kubespray 설치

kubespray가 알아서 kubectl, kubelet, kubeadm, cri-o, calico 등 Kubernetes 클러스터 구성에 필요한 소프트웨어를 설치하므로, 워커노드나 master node에 따로 각 소프트웨어를 다운받을 필요가 없다. 아래 명령어를 통해 kubespray를 원하는 pc에 설치한다. 필자의 경우 master node 중 하나에 설치하여 배포하였다.

```
sudo apt install python3
sudo apt update
sudo apt install -y python3-pip
sudo apt install -y git

git clone --branch release-2.21 https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/
pip install -r requirements.txt
```

# ssh key 생성 및 다른 node들에 copy

kubespray를 실행할 pc에서 ssh-key를 생성하고 대상 노드들에 복사한다.

```
ssh-keygen
ssh-copy-id 마스터노드1ip
ssh-copy-id 마스터노드2ip
ssh-copy-id 마스터노드3ip
ssh-copy-id 워커노드1ip
ssh-copy-id 워커노드2ip
ssh-copy-id 워커노드3ip
```

ssh로 node들에 비밀번호 없이 접속되는지 확인

```
ssh 마스터노드1ip
ssh 마스터노드2ip
ssh 마스터노드3ip
ssh 워커노드1ip
ssh 워커노드2ip
ssh 워커노드3ip
```

# 각 노드의 해당 user가 sudo 명령어를 비밀번호 입력 없이 실행할 수 있도록 설정

각 노드에서, 다음 명령어로 파일을 연다

`sudo visudo /etc/sudoers`

그러면 다음과 같이 파일 내용이 나온다.

```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        use_pty

# This preserves proxy settings from user environments of root
# equivalent users (group sudo)
#Defaults:%sudo env_keep += "http_proxy https_proxy ftp_proxy all_proxy no_proxy"

# This allows running arbitrary commands, but so does ALL, and it means
# different sudoers have their choice of editor respected.
#Defaults:%sudo env_keep += "EDITOR"

# Completely harmless preservation of a user preference.
#Defaults:%sudo env_keep += "GREP_COLOR"

# While you shouldn't normally run git as root, you need to with etckeeper
#Defaults:%sudo env_keep += "GIT_AUTHOR_* GIT_COMMITTER_*"

# Per-user preferences; root won't have sensible values for them.
#Defaults:%sudo env_keep += "EMAIL DEBEMAIL DEBFULLNAME"

# "sudo scp" or "sudo rsync" should be able to use your SSH agent.
#Defaults:%sudo env_keep += "SSH_AGENT_PID SSH_AUTH_SOCK"

# Ditto for GPG agent
#Defaults:%sudo env_keep += "GPG_AGENT_INFO"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) NOPASSWD:ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
# See sudoers(5) for more information on "@include" directives:

@includedir /etc/sudoers.d
```

이 중 중요한 내용은 밑에있는 두 줄,

`%sudo   ALL=(ALL:ALL) ALL` 와 `@includedir /etc/sudoers.d` 이다.

이 부분에서 `%sudo   ALL=(ALL:ALL) ALL` 다음 줄에, 다음 내용을 추가해야 한다.

`유저이름 ALL=(ALL) NOPASSWD:ALL`

`%sudo   ALL=(ALL:ALL) ALL` 보다 아래에 위치에 추가하는 이유는, 아마도 이 유저가 이미 sudo group 소속일텐데, `%sudo   ALL=(ALL:ALL) ALL` 보다 위에 저 내용을 추가해버리면, 더 아래줄에 적힌 내용이 덮어씌워져버리므로 의미가 없어져버리기 때문이다.

하지만 위 방법처럼 추가하지 말고, 왠만하면 `@includedir /etc/sudoers.d` 부분을 활용하는 것이 좋다. 

sudoers.d 디렉토리는 유저가 다양할 때 파일로 나누어 관리하기 위한 것으로, 유저가 많을 시 본 가이드에서 아까 제시한 방식 대신 이 방법으로 관리하는것이 적극 권장된다.

따라서 해당 디렉토리 내에 해당 유저명 혹은 그룹명으로 파일을 만들어 그 안에 관련 설정들을 넣어 관리하는것이 좋다.

이번 예시의 경우라면, 필자의 유저명은 `test`이므로,

test 라는 파일을 만들고, 내부에 `test ALL=(ALL) NOPASSWD:ALL` 라는 내용을 추가하면 된다.

물론 파일명은 `~` 혹은 `.` 으로 시작하지않는 모든 파일을 포함하기때문에 편하게 만들어도된다.


그리고 가능은 되지만 하지 말아야 할 방법을 소개하겠다.

아마 이미 해당 유저가 sudo 그룹에 소속되어으니, `%sudo   ALL=(ALL:ALL) ALL` 부분을

`%sudo   ALL=(ALL:ALL) NOPASSWD:ALL`

로 수정한다면 sudo group에 포함된 모든 사용자가 비밀번호 없이 사용이 가능할 것이다. 하지만 모든 sudo 그룹 사용자가 이렇게 되는 것 보단 위 방법이 좋으니, 그렇게 하자.


# kubespray 설정(inventory.ini 혹은 hosts.yaml 사용)

본 예시에서는 inventory.ini를 사용하겠다. kubespray 디렉토리 위치에서, 다음 명령들을 순서대로 실행한다.

```
cp -rfp inventory/sample/ inventory/test-cluster
declare -a IPS=(마스터노드1ip 마스터노드2ip 마스터노드3ip 워커노드1ip 워커노드2ip 워커노드3ip)
CONFIG_FILE=inventory/test-cluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

이후 `inventory/test-cluster/hosts.yaml` 파일을 열어보면 해당 사항들이 반영되어있는 것을 확인할 수 있을 것이다.

이 내용에 맞춰, `inventory/test-cluster/inventory.ini` 파일을 열고 아래와 같은 부분들을 확인한다.

```
[kube_control_plane]
[etcd]
```
위 두 그룹안에 각각 마스터노드와 etcd로 사용할 노드를 배치하고,
`[kube_node]` 안에 워커노드로 사용할 노드를 배치해야 한다.

아래는 그 구체적 예시이다.

```
[all]
master1 ansible_host=192.168.***.***
master2 ansible_host=192.168.***.***
master3 ansible_host=192.168.***.***
node1 ansible_host=192.168.***.***
node2 ansible_host=192.168.***.***
node3 ansible_host=192.168.***.***


[kube_control_plane]
master1 ansible_host=192.168.***.***
master2 ansible_host=192.168.***.***
master3 ansible_host=192.168.***.***

[etcd]
master1 ansible_host=192.168.***.***
master2 ansible_host=192.168.***.***
master3 ansible_host=192.168.***.***

[kube_node]
node1 ansible_host=192.168.***.***
node2 ansible_host=192.168.***.***
node3 ansible_host=192.168.***.***

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

**중요** 그 후 본 kubespray v2.21 만의 오류로 수정해야 하는 부분이 하나 더 있다.

`vi inventory/test-cluster/group_vars/k8s_cluster/k8s-cluster.yml` 를 열어서,

`remove_default_searchdomains: false` 이 부분을 찾아 주석을 해제해야 한다.

그 이유는, 다음 [사이트](https://github.com/kubernetes-sigs/kubespray/issues/9948)에서 확인할 수 있는데, 

단순히 kubespray 개발자들이 저 부분의 기본값을 실수로 true로 설정해두었기 때문이다.

# crio 사용 설정

kubespray v2.21은 기본적으로 containerd를 사용하게 되어있다. 이를 cri-o로 수정하기 위해서는 다음과 같이 파일을 수정해야 한다.

[공식문서](https://github.com/kubernetes-sigs/kubespray/blob/release-2.21/docs/cri-o.md)가 존재하지만, 내용이 다수 잘못되어 있어 본 가이드를 따라가면 된다.

`inventory/test-cluster/group_vars/all/all.yml` 파일에서,

```
download_container: false
skip_downloads: false
etcd_deployment_type: host # optionally kubeadm
```

로 수정 혹은 내용 추가를 해야 한다.

또한, `inventory/test-cluster/group_vars/k8s_cluster/k8s-cluster.yml` 파일에서,

`container_manager: crio` 로 수정해야한다. 기본값으로는 `container_manager: containerd` 로 되어있을 것이다.

`inventory/test-cluster/group_vars/all/cri-o.yml` 파일의 내용을 다음과 같이 수정해야 한다.

```
crio_registries:
  - prefix: docker.io
    insecure: false
    blocked: false
    location: registry-1.docker.io
    unqualified: true # registry 주소의 기본값을 docker.io로 사용하겠다는 설정
    mirrors:
      - location: {사내 내부 레지스트리 주소}
        insecure: false
      - location: mirror.gcr.io
        insecure: false
crio_registry_auth:
  - registry: {사내 내부 레지스트리 주소}
    username: {사내 내부 레지스트리 id}
    password: {사내 내부 레지스트리 pw}
```

필요한 경우 pids_limit을 올리려면 `roles/container-engine/cri-o/defaults/main.yml` 에 있는 `crio_pids_limit` 값을 수정하면 된다.

# ansible-playbook 명령어로 클러스터 생성

클러스터를 생성하는 명령어를 실행한다.(아까와 동일하게 kubespray 디렉토리 안에서 입력)

`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root cluster.yml`

## Truble shooting

이때 ansible이 각 노드의 dns 설정을 건드리며 아래와 같은 오류가 발생할 수 있다. 증상과 해결방법은 다음과 같다.

1. 일정 확률로, 도메인 서버가 고장남

이 경우를 확인하는 것은, 다음 명령어로 확인이 가능하다.

`nslookup google.com`
`ping 8.8.8.8`
`dig +dnssec google.com`

nslookup으로 정상적으로 구글의 ip가 반환되지 않고 에러가 나면서, `ping 8.8.8.8` 은 제대로 수행되고, 

`dig +dnssec google.com` 에서 status가 SERVFAIL이 나온다면, ansible가 dns 설정을 중간에 잘못 건드려 발생한 오류이다.

이 경우 다음과 같이 다시 고칠 수 있다.

1. `sudo nano /etc/systemd/resolved.conf` 로 파일을 열고, `DNS=` 의 주석을 해제하고 해당 줄을 `DNS=8.8.8.8`로 수정한다.

2. `sudo systemctl restart systemd-resolved`를 입력하여 해당사항을 반영하고, `nslookup google.com` 을 입력하여 정상적으로 값이 나오는 것을 확인한다.

3. 1번에서 주석을 해제했던 부분을 원상복귀(주석처리)하고, 다시 `sudo systemctl restart systemd-resolved` 를 입력하여 해당 사항을 적용한다.

이러고 나면 정상적으로 다시 nslookup google.com이 수행되는 것을 확인할 수 있다.


# 정상적으로 설치되었는지 확인

```
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config

# k9s에서 접근할 수 있도록 권한 설정
sudo chmod -R 555 ~/.kube

kubectl get nodes
```

이때, 다음과 같이 출력되면 정상적으로 설치 된 것이다.

```
NAME     STATUS   ROLES           AGE     VERSION
master1  Ready    control-plane   4h53m   v1.25.6
master2  Ready    control-plane   4h53m   v1.25.6
master3  Ready    control-plane   4h53m   v1.25.6
node1    Ready    <none>          4h53m   v1.25.6
node2    Ready    <none>          4h53m   v1.25.6
node3    Ready    <none>          4h53m   v1.25.6

```

하지만 클러스터가 정상적으로 만들어졌다 하더라도, 아직은 안심할 수 없다. 각종 kubernetes의 시스템을 구성하는 요소들이 잘 초기화되고 실행되는지 확인해야 한다.

따라서 k9s를 설치하여 나머지 Kube-system 항목들이 잘 초기화되고있는지 확인하자


# 각 노드별 필수 수정사항

k9s 설치 후 모든 pod들이 잘 돌아가고 있는 것을 확인했다면, kubespray 2.21 버전에서 cri-o를 사용할 때만 존재하는 버그를 수정하기 위해,

각 노드 별 한가지 수정이 필요하다.

kubespray 2.21 버전은 이미지의 주소가 docker.io 와 같은 레포지토리 주소를 포함하지 않는 짧은 주소라면, cri-o를 사용했을 때 해당 주소를 찾지 못하는 문제가 존재한다.

이는 kubespray 2.21 버전이 crio를 사용하여 배포할 때, 위 crio 사용 설정 부분에서 설정한 `cri-o.yml` 파일의 내용에 따라

짧은 이미지 주소에 대해 레포지토리를 자동으로 연결해주는 `/etc/containers/registries.conf.d/01-unqualified.conf` 파일을 생성하는데, 이 파일의 내용이 잘못되기 때문이다.

따라서 각 노드에서 `/etc/containers/registries.conf.d/01-unqualified.conf` 파일을 열고, 다음 내용을 확인해야 한다.

`unqualified-search-registries = ['docker.io']`

위 내용에서, 문제의 원인은 `'docker.io'` 이다. 이 부분을 `"docker.io"` 로 수정해야 한다.

## Node를 더 추가하려면?

kubernetes를 사용하는 것이니 당연히 Node를 추가하거나 줄일 일이 있을 수 있다. 이 경우, 다음과 같이 한다.

1. 추가할 node에 ssh-key-copy
1. inventory,hosts.yaml 수정
2. (추가하는 경우)추가할 Node에도 해당 사용자가 sudo 명령어를 비밀번호 없이 사용할 수 있도록 설정
3. ansible로 inventory의 수정사항을 반영하여 scale하기

하나씩 보자

### inventory 수정

```
declare -a IPS=(ip1 ip2 ip3 ip4)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

```

### (추가하는 경우)추가할 Node의 필수사항 구성

ssh로 비밀번호 없이 접속 가능하도록 설정하고, sudo 명령어를 비밀번호 입력 없이 수행할 수 있도록 설정해야 한다.(방법은 위와 동일)

### ansible로 inventory의 수정사항을 반영하여 scale하기

inventory.ini파일도 수정한 후, 다음 명령어를 통해 scale할 수 있다.

`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root scale.yml`


#### Truble shooting

여기서 아래와 같은 증상의 에러가 발생하며 정상적으로 동작하지 않는다면, [다음 방법](https://github.com/kubernetes-sigs/kubespray/commit/c8e343ac619aa5deb47a95ddbfe271b844bdbc81)을 참고하면 된다.

```
'kubeadm_images_raw' is undefined
... 이하 생략
```

## 클러스터 구성을 해제하고 삭제하려면??

모든 클러스터 구성을 해제하고 삭제하려면 다음 명령어를 실행한다.

`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root reset.yml`

이때 클러스터 구성 시 발생하는 dns 관련 문제가 발생할 확률이 높다. 해당 문제가 발생한다면 위 해당부분 가이드를 참고하여 문제를 해결하면 된다.









<!-- ## cri-o 설치

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

아마 이렇게 나올것이다.

```
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.16", GitCommit:"c5f43560a4f98f2af3743a59299fb79f07924373", GitTreeState:"clean", BuildDate:"2023-11-15T22:39:12Z", GoVersion:"go1.20.10", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

여기서 버전정보를 확인할 수 있고, `The connection to the server localhost:8080 was refused - did you specify the right host or port?` 부분은, kubectl을 설치만 했지 아무것도 구성한 것이 없기때문에 이렇게 나온다.


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

``` -->
