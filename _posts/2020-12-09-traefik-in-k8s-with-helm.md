---
title: "[筆記]Traefik in k8s with helm"
key: 20201209
layout: article
tags: traefik v2 k8s helm
---

最近在幫公司找有沒有好用的專案管理軟體之餘，想想也很久沒碰這個之前一直想放進k8s的東西，加上k8s的tls要自動化難度不低，就只好來挑戰一下這個東西了 ~~發懶是一切的原動力~~

<!--more-->

## 前言
其實透過官方的教學文章就可以很快的把server建立起來~~所以這篇就這樣了~~，不過有幾個地方還是要注意一下

## 概念
新版的Traefik不用`ingress-controller`，而是自定義的`IngressRoute`去做轉發，這部份是拜`k8s`的高彈性之賜，不過要注意的是這邊跟`docker`的設定方式完全不一樣；不像`docker`使用了一堆的`labels`來做設定。

## Helm的設定
這邊真的是照官方給的文件就可以完成，不過這邊我還是沒辦法進dashboard，應該是有沒設定到的地方。

因為我家的`k8s`在`aws`上，然後使用的`provider`也不是`cloudflare`，所以我在這部份卡了一段時間

不囉嗦直接上code，我只貼我改的部份
``` yml
additionalArguments:
  - --entrypoints.websecure.http.tls.certresolver=route53
  - --certificatesresolvers.route53.acme.dnschallenge.provider=route53
  - --certificatesresolvers.route53.acme.email=your@email
...
env:
  - name: AWS_ACCESS_KEY_ID
    valueFrom:
      secretKeyRef:
        key: user
        name: aws-credentials
  - name: AWS_SECRET_ACCESS_KEY
    valueFrom:
      secretKeyRef:
        key: token
        name: aws-credentials
```

原本範例裡的cloudflare都被我換成route53，[出處](https://doc.traefik.io/traefik/https/acme/)可以從這裡查，不過官方沒特別講這部份我覺得很不ok，花了一些時間在這


再來是token跟key的部份，一樣是參照上面的連結可以知道使用route53後用的key就會不一樣，而主要的參數就是`AWS_ACCESS_KEY_ID`以及`AWS_SECRET_ACCESS_KEY`。而在aws裡設定的部份可以參照[這裡](https://go-acme.github.io/lego/dns/route53/)，這裡主要是講IAM的設定。


再來是`Secret`的部份
``` yml
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
  namespace: traefik
type: Opaque
stringData:
  user: AWS_ACCESS_KEY_ID
  token: AWS_SECRET_ACCESS_KEY
```

記得這邊這樣改的話，上面那個`helm`檔的`env`也要改成對應的key和value，這邊我就不再多貼了


## 結語
要改的東西真的不多，主要是`provider`和`aws IAM`對應的部份要對，~~再來就是這個東西不能跨`namespace`，也就是你一個`namespace`就要建一次traefik，所以我才會只拿來放小東西就好。~~如果要新增網域之類的，只需新增`IngressRoute`就可以了，很意外的`traefik`可以跨`namespace` **update 20201214**

--
## 參考資料
[traefikLabs](https://traefik.io/blog/install-and-configure-traefik-with-helm/)

[traefik provider list](https://doc.traefik.io/traefik/https/acme/)

[aws route53設定](https://go-acme.github.io/lego/dns/route53/)
