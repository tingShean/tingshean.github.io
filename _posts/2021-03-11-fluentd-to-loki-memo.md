---
title: "[筆記]Fluentd to Loki 二三事"
key: 20210311
layout: article
tags: fluentd loki td-agent
---

## 前言
最近在研究幾個`logging`的軟體，主要是可以讓我快速的一鍵安裝到各`legacy`的`server`上，然後又可以直接無痛設定

講到這邊應該就會有各種派系戰起來了吧，~~比方說`ELK`、`PLG`之類的~~
<!--more-->
## 遇到的困難
其實重點不在於為什麼不使用已經有前人或是各大神舖好的路走，我只是想挑戰一下自己在`ansible`和自動化設定上的功力是不是如自己預期的水準；但是該踩到的雷還是會在那邊等著踩，而且這次很哭的地方是這東西不是我的領域

一開始我是直接看`promtail`，因為之前也有相關部署的經驗，不過那次是用`docker`起來的，所以並不知道比較細節的地方，雖然這個並不是`docker`的問題。總之我看了一下怎麼安裝或是怎麼設定之後就放棄了 ~~對，很直接~~。我在`GitHub`上並沒有看到是用`yum`或是相關的安裝方式，而是直接抓編好的binary用就可以了。看起來很棒但是當下的反應就是沒辦法直接使用`systemd`去管控的話，在啟動上就會比較麻煩一些；結果後來繞圈子回來的時候，果然用編好的東西沒辦法直接使用，突然開始討厭這種直覺 (無誤)。

後來是看`fluentd`，這東西也是最近很火紅的軟體，我一開始會考慮的時候是因為他是輕量化的軟體，也可以放在`docker`、`k8s`上，但是因為有些`server`要monitor的方式不能透過`docker`，所以研究了老半天發現這東西要安裝`ruby`和`rubygem` ~~差點直接放棄~~；後來在試試用`ansible`安裝的時候，發現怎麼裝都跟我說`git`有衝突，然後怎麼裝都裝不了，就算我直接進`server`把`git`升上去後安裝`ruby`也安裝不了，然後才會回去看`promtail`怎麼裝。

結果最後在官網看到`fluentd`可以直接用`curl`的方式安裝好 ~~浪費了我一整天啊~~


## 開始不是真的開始
安裝好後把腳本都寫好了，開始測試的時候，怎樣都沒辦法跟官網範例上教的一樣，在`grafana`上看不到`fluentd`的`log`可以選。因為我是用`systemd`啟動的，所以直接下`status`就可以看到啟動狀態，但是看完後也是正常的。

本來以為是`grafana`的設定跑掉，不過先排除這個可能性，因為`fluentd`也有相關的測試腳本參數可以執行，所以我直接使用`sudo td-agent --dry-run -c /path/to/your/config`，然後發現沒什麼錯誤後，直接執行`sudo td-agent -c /path/to/your/config`，然後就成功看到`log`倒進`grafana`了 ~~到底是我在平行時空是吧~~


## 繼續踩雷
後來不斷的在`systemd`和手動啟動兩邊找問題點，然後參閱各網站和看了好幾遍的官網，都沒有什麼相關的討論。不過這邊有發現`systemd`起來的指令位置跟手動啟動的指令位置不一樣，雖然我後來有改掉`systemd`的配置，不過因為找到這邊才發現了主要的問題點在哪。


## 解決方法
就是`systemd`啟動的`service`設定檔產生的問題，在`td-agent.service`的設定檔裡有設定使用者和群組，就是因為這個原因造成讀不到我指定的`log`檔。我要讀取的`log`權限是`nginx:root`，但是`td-agent.service`裡自帶`td-agent:td-agent` ~~到底在衝三小~~，所以就算是用`sudo systemctl start td-agent`去啟動`fluentd`，也會因為這個設定檔的關係讓程序從`root`變成`td-agent`。所以這裡把要讀取的`log`檔群組權限改成`td-agent`就可以了，會改群組權限的原因就是不想去更動`nginx`相關權限，所以直接把群組改成`td-agent`(有更好的方法可以提供給我)。


## 後記
這東西真的讓我繞一大圈，後來想想應該是為了避免單一服務權限過大的原因而這樣改的，但是麻煩官網可以註記一下嗎？我一度查到以為自己在平行時空，至於官網上說的`service`檔案位置也是跟實際運作不太一樣，不想動到原設定檔的話可以下`sudo systemctl edit td-agent`，這會直接開啟`overwrite`的`service`檔，然後就會發現跟官網上寫的完全不一樣。


## 參考連結
[How to ship logs to Grafana Loki with Promtail, FluentD & Fluent-bit](https://grafana.com/go/webinar/getting-started-shipping-logs-to-grafana-loki/)

[Install by RPM Package #2](https://docs.fluentd.org/installation/install-by-rpm#step-2-launch-daemon)
