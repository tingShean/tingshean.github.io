---
title: "[筆記]換掉 K8s 裡的 docker-shim"
key: 20201217
layout: article
tags: kubernetes k8s docker-shim cri-o containerd
---

最近這件事真的是在圈子裡引起很多的討論，連台灣這邊幾個大神都有出來講大致上的狀況以及幫大家釐清自家的服務有沒有需要擔心這件事。所以這篇只是我自己練習後~~踩坑~~的心得筆記。

<!--more-->

## 前言

聽完各路大神們的講解和分析後，想想還是得找時間親自了解一下或是自己試著弄弄看，雖然以前玩過minikube，不過那時候的`k8s`才`1.8`不到，很多細部的設定都跟現在各雲端公司提供的版本有些出入；然後也跟大神提供自己的文章有些出入，簡單講就是`k8s`版本一更新就跟世界脫離了一樣。


## 前置作業

我在自架的時候用到的套件版本
1. k8s 1.20
2. kube-flannel commit版本`d943e2c8b3d26f22e95940db28000933d5d140e0`
3. ansible 2.9.10
4. cri-o-1.15

`kube-flannel`在教學文件上的commit版本是距今一年多以前，所以有些`api`因為版本不對`k8s`不吃，然後`ansible`的影響在於`playbook`的格式也會造成一些問題


## 開始踩坑 - containerd

在照著文章一步一步走的同時就可以知道有些元件是可以抽換的，不過今天是在練習換`oci`，照著文章走第一個會先遇到的就是`ansible`我的版本過新而`playbook`的寫法過舊的問題，其實照著上面的`error`就可以知道了，改成新的格式就可以了，這邊的問題不大。接著安裝好後進機器內下指令就會踩到第二個坑，在安裝`kube-flannel`的時候因為版本的問題，所以我換成`https://raw.githubusercontent.com/coreos/flannel/d943e2c8b3d26f22e95940db28000933d5d140e0/Documentation/kube-flannel.yml`，到這邊就結束了，可能因為是新的安裝所以相對問題不大。

後來想想目前`cncf`比較推的流程以及也是大神的解釋，我想說來試試替換掉這個`containerd`會有多困難~~然後就踩了更大的坑~~


## 開始踩坑 - CRI-O

一開始也是直接照著文章做，雖然這次面向不同的是在機器內，`ansible`安裝套件這段就可以跳過了因為剛剛已經做過，不過記得要裝`CRI-O`，然後在裝完啟動的時候就出事了，如文章所說的少了個link，但事情沒這麼簡單，這邊我踩了另一個坑，[相關討論](https://github.com/cri-o/cri-o/issues/2903)在裡面，我不知道原來安裝套件也會跟別人安裝的結果不一樣，照著討論裡的指令下`sudo sed 's|/usr/libexec/crio/conmon|/usr/bin/conmon|' -i /etc/crio/crio.conf`就可以，不過我還是有先進去這個檔案看是不是真的有這行，以及要替換的`cmd`是不是真的在`/usr/bin/`，加完link再替換掉這行後就可以啟動了，但是我的k8s現在是啟動的狀態，所以我先下了`kubaadm reset`，然後在重啟的時候因為你的`cri`有兩個，所以在啟動的時候他會跟你說你得選擇使用哪一個`cri`。

`sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket /var/run/crio/crio.sock`

這樣就是指定使用`CRI-O`啟動了，接著他上面會有提示你該下哪幾行指令，然後一樣要安裝`kube-flannel`，接著下`crictl ps`就出事了~~活在平行時空~~，這邊使用`kubectl`還看的到服務，然後用`sudo crictl images`也看的到有多少`image`在機器內，但是其他指令怎麼下都是空的；最後是想到我已經替換掉`cri`了。

``` shell
$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
```

看到這個就知道問題在哪了，你的`cri`還是使用`containerd`在運作，所以下指令就等於是去這邊要資料，難怪都空的，所以直接換成`runtime-endpoint: unix:///var/run/crio/crio.sock`就可以了


## 結尾

目前在替換這邊大致上有個底，不知道有沒有辦法在不砍掉`k8s`的情況下直接替換掉`cri`，這是個值得好好研究的地方。


## 參考資料

[[Day6] Kubernetes & CRI (Container Runtime Interface)(II)](https://ithelp.ithome.com.tw/articles/10218570)

[[Day7] Container Runtime - CRI-O](https://ithelp.ithome.com.tw/articles/10219102)

[1.15.3-1~dev~ubuntu18.04~ppa4 systemctl crio.service failed missed /usr/libexec/crio/conmon directory](https://github.com/cri-o/cri-o/issues/2903)

[kube-flannel](https://github.com/coreos/flannel)
