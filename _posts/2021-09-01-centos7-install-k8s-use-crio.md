---
title: "[心得]Centos7 + kubeadm + CRI-O"
key: 20210901
layout: article
tags: k8s kubeadm kubelet crio no-docker
---
# K8S(自架) + CRI-O抽換

最近遇到一些問題，用供應商的K8S沒辦法好好的debug，剛好公司最近給了我們一台規格不錯的機器，就想說順便也來挑戰一下真的自架K8s，以及直接不安裝docker的方式把CRI-O塞進去，過程中真的踩了不少雷，不過大致上可以理解底層的運作，以及相關的啟動方式是怎麼樣一個互相~~牽扯~~溝通

<!--more-->

## 事前準備
1. centos 7
2. 2core/4G
3. ansible(版本不限)
4. 安裝host切sudo免密碼設定(對ansible使用上會簡單點)
5. ***沒有安裝docker***


## 注意事項
kubernetes在`1.14.2`左右的時候有更動設定檔的位置 [出處](https://github.com/kubernetes/kubeadm/issues/1575)

    1.9
    /etc/systemd/system/kubelet.service.d/
    1.14.2
    /usr/lib/systemd/system/kubelet.service.d/


## 先行安裝crictl、containerd
1. 使用官方的ansible-playbook

        git clone https://github.com/containerd/cri
        cd ./cri/contrib/ansible

2. ansible-playbook -i <hosts_file> cri-containerd.yaml

        hosts_file填上需要安裝的機器位置設定檔


## 安裝CRI-O之前
1. 設定啟動CRI-O的config檔

    ```shell
    cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
    overlay
    br_netfilter
    EOF
    ```

2. 載入`overlay`和`br_netfilter`

        sudo modprobe overlay
        sudo modprobe br_netfilter

3. 設置sysctl所需參數，這些在重新啟動後仍然存在

    ```shell
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF
    ```

4. 重載sysctl設定

        sudo sysctl --system


## 安裝CRI-O
1. 先進到需要安裝的主機
2. 設定環境變數，這邊可以去官方的git查，不過這邊是用centos 7，所以直接輸入`VERSION=1.18`及`OS=CentOS_7`即可
3. 下載repo並安裝

        curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
        curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo
        yum install cri-o


## 安裝k8s
1. 先進到`master`主機
2. 加入`repo`

    ```shell
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg    https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF
    ```

3. `sudo yum install -y kubelet kubeadm kubectl`

        sudo systemctl start kubelet
        sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=/var/run/crio/crio.sock 

4. 設定`hostname`為`master-node`

        sudo hostnamectl set-hostname master-node

5. 更新host

        sudo vim /etc/hosts

6. 啟動`kubeadm`這邊一定會出錯，但是要靠他產幾個檔出來並更改，所以接著到後面再修正就好


## 更改相關設定 containerd.sock > crio.sock
1. 修改`kubelet`趨動

        # vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
        找到 `--container-runtime-endpoint=unix:///run/containerd/containerd.sock`
        換成 `--container-runtime-endpoint=unix:///var/run/crio.crio.sock'

2. 修改`crictl.yaml`

        # vim /etc/crictl.yaml
        換成`unix:///var/run/crio.crio.sock`


## 更改相關設定 cgroup-driver
1. 新增`kubelet`趨動

        # vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
        加上 `--cgroup-driver=cgroupfs`

2. 新增`kubeadm`環境變數

        # vim /var/lib/kubelet/kubeadm-flags.env
        最後面加上 `--cgroup-driver=cgroupfs`

3. 修改`cri-o`的配置檔

        # vim /etc/crio/crio.conf
        找到 `cgroup_manager` 把 "systemd" 改成 "cgroupfs"
        找到 `conmon_manager` 改成 "pod"

4. 重新載入`systemd daemon`並重啟服務

        # systemctl daemon-reload
        # systemctl restart crio
        # systemctl restart kubelet


## 收尾
1. 把config複製到user的home資料夾下

    ```shell
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

2. `kubectl get all`看到下面這就代表K8s的master起來了

    ```shell
    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   82m
    ```


## 附註：設定開啟NET_RAW(如果要使用busybox的ping相關功能就要先開)
1. 修改config

        # vim /etc/crio/crio.conf
        找到`default_capabilities`在列表內加入`NET_RAW`

2. 重啟cri-o，sudo systemctl restart crio


## 參考資料
[Container runtimes#CRI-O](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)

[Install CRI-O](https://github.com/cri-o/cri-o/blob/master/install.md)

[Container using ping without CAP\_NET\_RAW](https://www.antitree.com/2019/01/containers-using-ping-without-cap_net_raw/)

[10-kubeadm.conf located under different folder after upgrade to 1.14](https://github.com/kubernetes/kubeadm/issues/1575)
