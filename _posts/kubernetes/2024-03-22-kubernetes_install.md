---
title: kubernetes install by kuberspray
categories: [kubernetes,kuberspray, cri-o]
tags: [kubernetes,kuberspray,cri-o]     # TAG names should always be lowercase
published : true
---
# 개요

본 문서는 kuberspray를 활용하여 kubernetes를 구축하고, rook을 사용하여 ceph을 올리기까지의 작업순서와 명령어들을 정리해둔 것임.

## 버전정보

OS : xUbuntu_22.04
CRI-O : 1.26
kubernetes : v1.25
kuberspray : 2.24

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

이제부터 매우 중요한 내용이며, 필자가 삽질을 매우 길게 한 부분임.

지금 실행할 것은 아니지만, 있다가 최종적으로 세팅이 완료된 후에 실행할 명령어는

`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root cluster.yml` 임.

이 명령어가 정상적으로 동작하는 조건은 다음과 같음

1. 명령어를 실행하는 master node에서, inventory.ini에 적어둔 모든 노드의 ip들에 ssh로 비밀번호 없이, ssh-key를 통해 접근이 가능할 것
2. 그렇게 접근한 유저가 sudo 명령어를 비밀번호 입력 없이 실행할 수 있을 것

따라서 1번부터 하나씩 차근차근 설정해보자.

### 명령어를 실행하는 master node에서, inventory.ini에 적어둔 모든 노드의 ip들에 ssh로 비밀번호 없이, ssh-key를 통해 접근이 가능하도록 설정

master node에서 ssh-key를 생성하고 대상 노드들에 복사한다.

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

### 각 노드의 해당 user가 sudo 명령어를 비밀번호 입력 없이 실행할 수 있도록 설정

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


## kubespray 설정(inventory.ini 혹은 hosts.yaml 사용)

본 예시에서는 inventory.ini를 사용하겠다. kubespray 디렉토리 위치에서, 다음 명령들을 순서대로 실행한다.

```
cp -rfp inventory/sample/ inventory/mycluster
cd inventory/mycluster
declare -a IPS=(워커노드1ip 워커노드2ip 워커노드3ip)
CONFIG_FILE=inventory/test-cluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

```

물론 위 워커노드 ip들은 실제 자신의 워커노드 ip에 맞게 설정해야 한다.

이러고 나면 `inventory/test-cluster/inventory.ini` 파일을 열어보면 해당 사항들이 반영되어있는 것을 확인할 수 있을 것이다.

## ansible-playbook 명령어로 7에서 설정된 내용대로 클러스터 생성

위 설정에 혹시 예전에 이미 클러스터를 만든게 있다면, 그걸 삭제하기 위해 다음 명령어를 실행한다.

`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root reset.yml`

이때 reset 혹은 을 했다면, 필자의 경우 이유는 모르지만 master node와 workder node중 하나의 도메인 서버 정보가 삭제되어 nslookup naver.com 등이 작동되지 않게 되는 현상이 있었다. 그런 경우,

`sudo vi /etc/resolv.conf`

를 열어, `nameserver 127.0.0.53` 의 다음에 두 줄을 추가하면 문제가 해결된다.

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

계속 진행하자.

이제 클러스터를 생성하는 명령어를 실행한다.(물론 아까와 동일하게 kubespray 디렉토리에서 입력)


`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root cluster.yml`

### 정상적으로 설치되었는지 확인

```
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
kubectl get nodes
```

이때, 다음과 같이 출력되면 정상적으로 설치 된 것이다.

```
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   4h53m   v1.25.16
node1    Ready    <none>          4h53m   v1.25.16
node2    Ready    <none>          4h53m   v1.25.16
```

### Node를 더 추가하려면?

kubernetes를 사용하는 것이니 당연히 Node를 추가하거나 줄일 일이 있을 수 있다. 이 경우, 다음과 같이 한다.

1. 추가할 node에 ssh-key-copy
1. inventory,hosts.yaml 수정
2. (추가하는 경우)추가할 Node에도 해당 사용자가 sudo 명령어를 비밀번호 없이 사용할 수 있도록 설정
3. ansible로 inventory의 수정사항을 반영하여 scale하기

하나씩 보자

#### inventory 수정

```
declare -a IPS=(ip1 ip2 ip3 ip4)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

```

#### (추가하는 경우)추가할 Node에도 해당 사용자가 sudo 명령어를 비밀번호 없이 사용할 수 있도록 설정

알아서 위에 작성된 가이드를 참고하여 설정하자.

#### ansible로 inventory의 수정사항을 반영하여 scale하기

inventory.ini파일도 수정한 후, 다음 명령어를 통해 scale할 수 있다.

`ansible-playbook -i inventory/test-cluster/inventory.ini  --become --become-user=root scale.yml`

다만 필자의 경우 여기서 에러가 발생하며 정상적으로 동작하지 않았는데, 그 이유는 필자가 사용한 main branch의 kuberspray에 버그가 있었다.

오류내용 : 

```
fatal: [iworker6]: FAILED! => {"msg": "{{ skip_kubeadm_images | ternary({}, _kubeadm_images) }}: {{ dict(names | map('regex_replace', '^(.*)', 'kubeadm_\\1') | zip( repos | zip(_tags, _groups) | map('zip', keys) | map('map', 'reverse') | map('community.general.dict') | map('combine', defaults))) | dict2items | rejectattr('key', 'in', excluded) | items2dict }}: {{ repos | map('split', '/') | map(attribute=-1) }}: {{ images | map(attribute=0) }}: {{ kubeadm_images_raw.stdout_lines | map('split', ':') }}: 'kubeadm_images_raw' is undefined. 'kubeadm_images_raw' is undefined. {{ kubeadm_images_raw.stdout_lines | map('split', ':') }}: 'kubeadm_images_raw' is undefined. 'kubeadm_images_raw' is undefined 

... 이하 생략
```

만약 필자와 동일한 오류가 있다면(아직은 main에 merge되지 않아 해당 버그가 존재하나, 이후 버전에서는 해결될 것으로 보인다)[다음과 같이](https://github.com/kubernetes-sigs/kubespray/commit/c8e343ac619aa5deb47a95ddbfe271b844bdbc81) 수정해주면 된다.






## Rook 설치, 설정
## ceph 설정
## ceph 올리기

