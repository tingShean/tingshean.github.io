---
layout: post
title:  "[Docker]使用network"
date:   2018-10-13 20:00:00 +0800
category: blog
key: 20181013
tags: docker 
---
在一開始學習Docker時，那時候如果要架一個Web server加上DB的開發環境。
順序是這樣的`DB > web server(back-end + front-end) > nginx or apache`

而各container的橋接方式就是用`--link`的方式橋接

最近因為手上的專案有些東西用docker比較方便快速，所以在架設的時候有遇到問題，就是新版docker的`--link`已經算是不支援的狀態，請大家改用network。

後來玩了一下發現這個做法跟k8s的cluster有些類似，在啟動任何image之前，先讓docker建立自己的network，然後在把image掛在這個network底下就可以

<!--more-->
## step 1
建立docker network

`$ docker network create -d <network_type> <network_name>`

## step 2
把要link的image加入network

`$ docker run -d --rm --network=<network_name> --name <container_name> <docker_image>:<version>`


這樣就可以把環境架起來了


---

參考資料 

[Legacy container links](https://docs.docker.com/network/links/)

[Docker network](https://docs.docker.com/network/#networking-tutorials)