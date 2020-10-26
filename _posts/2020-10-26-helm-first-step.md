---
title: "[筆記]Helm的第一步"
key: 20201026
layout: article
tags: k8s kubernetes helm3
---

自動化的世界隨著技術棧的提倡感覺上也開始加速了起來，雖然也有可能是我才剛進這圈子沒有多久

隨著Helm的出現到現在可以跟老爺爺或是相關CD軟體的結合，感覺如果不摸好像真的會被這波潮流淹死

<!--more-->

## 前言

**在往下繼續閱讀之前最好能對K8s有一定的熟悉或是對指令上的操作已經很習慣再往下看比較好**

因為我是用mac，所以直接用brew裝就好了 ~~對，我超懶~~，我是直接拿現有的K8s來跑，換言之就是我不是直接從零開始；最後我用的是Helm 3


## 開始之前

其實從零到一往往是最累的，K8s也是、Helm也是，難就難在你不知道怎麼樣才是真的開始，就算看了一堆說明文件、一堆大神的操作推薦，還是得自己下去跑跑看你才知道你的理解對不對，慶興的是這次不像K8s摸了這麼久，這大概就是真的懂K8s運作後的好處吧。

各式各樣的template在helm hub上都有，但是請先好好的看過官方文件(如果你已經看過了的話)，因為啟動就那麼一行而已。


## 開始你的第一步

`helm create <helm_file_name>`

其實真的就這行，我一開始就有下過，然後看著他產出來的一堆template，我以為我走錯路了，是不是不應該這麼快就進這步。但是實際上就是這樣，人家一開始就把全部的東西都給你了，剩下就是你的事 ~~法克~~

拿到這些檔後先別急著想怎麼改，直接先run起來看看是否正確，不正確的話有可能是你的K8s config有問題，好好檢查一下


`helm package <helm_pkg_name>`

這動作就是打包而已，沒什麼太多的說明，直接往下一步走


`helm install <k8s_deployment_name> <helm_pkg_name>`

上一步的package name是啥，這邊就是啥，然後注意的是k8s deployment name，沒有錯就是k8s裡部署用的名字；這邊啟動後就可以下`kubectl get all`看看有沒有成功的跑起來你設定的服務了，官方給的範例是很小的nginx，也就是剩下的自己改 ~~很合理~~


`helm uninstall <k8s_deployment_name>`

這邊要注意的就是移除的時候用的是k8s裡deployment的名字，一下忘記也可以用`helm ls`看，這邊看到的東西會比`kubectl get all`少


## 相關設定說明

`helm show values <k8s_deployment_name>`

這邊可以看到你該服務起來的相關參數，建議後面加個`> <helm_values_yaml>`，這樣就可以直接把可用的參數存成檔案直接拿來用，雖然這個檔案跟`create`後目錄下的`values.yaml`一模一樣，不過個人覺得還是讓`values.yaml`保持乾淨比較好。


`helm upgrade -f <helm_values_yaml> <k8s_deployment_name> <helm_pkg_name>`

這個指令裡`-f`後接的檔案就是上個指令輸出的那個檔案 ~~真繞口~~，作用就跟K8s rolling up動作是一樣的，同時也有儲存次數的功能


`helm rollback <k8s_deployment_name> <revision>`

這個指令有用過K8s rolling up的應該就不用多了，有up就會有back，用法是一樣的後面接版號


## 後記

我真的是看了說明、各大神和各相關的論壇兩天，想想這樣挺白痴的，就直接看著官方上寫的續作下去，途中還有問了一下同事，他是建議我直接跑官方上講的那個就可以了；有時候真的不能考慮太多，不然我就不用多花這幾天的時間。

這次的學習沒有花很多時間在繞路也是好事，也可能這部份真的太淺就沒人分享，不過這邊還是記錄一下比較安心點，之後就要來改寫template

--
[Helm 官方文件](https://helm.sh/docs/intro/using_helm/)
