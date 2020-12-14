---
title: "[筆記]Traefik TLS in k8s with helm"
key: 20201214
layout: article
tags: traefik v2 k8s helm tls
---

最近剛好在做一些細部的更新，沒想到直接踩爆這個坑，卡了一小段時間，結果這個解法在官方的`github`裡的`values.yml`可以找到 ~~但是沒先下載的情況下哪可能看到~~

<!--more-->

## 前言
自從弄好一個簡單的內部使用的服務後，就會想把很多東西都放進去使用，這一南是k8s的陰謀~~什麼都想塞~~


## 事前準備
本來也是要弄個簡單的gitlab給外包商push使用，結果直接跑helm差點炸掉整個k8s，雖然我的規格開的不低，但是一安裝下去一堆pending~~什麼鬼官方helm~~，不過重點是我原本以為我新建一個`namespace`，就要新建一個對應的`traefik`，沒想到是我誤會了，所以後來去找了簡單的`docker`版本後就自己改一改放進`k8s`~~然後就出事了~~

## 起因
因為我原本以為一個`namespace`要建一個`traefik`，所以搞的兩個`traefik`都差點掛掉，接著我全砍掉後就出事了，依照官方設定的配置裡，少了一行配置導致於如果今天同一份設定檔在重覆安裝的時候會起衝突

### 原本設定檔
``` yml
persistence:
  enabled: true
  path: /certs
  size: 128Mi
```
### 加上accessMode
``` yml
persistence:
  enabled: true
  accessMode: ReadWriteOnce
  path: /certs
  size: 128Mi
```

## 事情沒這麼容易解決
加上剛剛那行之後，只是代表你有權限可以讀取這份檔案了，根據tls的相關設定來說，他需要更嚴格的`600`檔案權限，不然你在裝完`helm`之後就會發現不能用，然後看log會噴`Unable to get ACME account: permissions 666 for /data/acme.json are too open`，我只擷取其中一段，不然這log有夠長，資料位置自己替換就是了，總之就是跟你說你檔案的權限出問題了~~不然不要用~~。

### 直接加上deployment.initContainer設定，
``` yml
deployment:
  initContainers:
    - name: volume-permissions
      image: busybox:1.31.1
      command: ["sh", "-c", "chmod -Rv 600 /certs/*"] # **注意位置換成自己的**
      volumeMounts:
        - name: data
          mountPath: /certs/ # **注意位置換成自己的**
```

### 補上podSecurityContext
``` yml
podSecurityContext:
  fsGroup: null
```

## 後記
查了一些官方的回報和相關討論，一開始是先看到說把`fsGroup`改成`null`就不會有`660`，但是改了還是沒用，結果官方直接請大家在`initContainers`裡用指令把資料夾權限直接改成`600`，我個人是覺得這樣的方式怪怪的，不過既然是官方說的就那樣唄，事情解決了就好。

## 參考資料
[Unable to get ACME account: permissions 666 for /data/acme.json are too open #6972](https://github.com/traefik/traefik/issues/6972)

[values.yaml第22行](https://github.com/traefik/traefik-helm-chart/blob/401c8cdf690cbbc765a935c7279566a13b79a082/traefik/values.yaml#L22)

[permissions 660 for /data/acme.json are too open, please use 600 #164](https://github.com/traefik/traefik-helm-chart/issues/164)

以下letsencrypt官方討論，可以看一下怎麼查自己domain申請的數量
[letsencrypt官方討論](https://community.letsencrypt.org/t/too-many-certificates-already-issued-for-domain/79066/3)

[Let's Debug Toolkit](https://tools.letsdebug.net/cert-search)

[HTTPS encryption on the web](https://transparencyreport.google.com/https/certificates)
