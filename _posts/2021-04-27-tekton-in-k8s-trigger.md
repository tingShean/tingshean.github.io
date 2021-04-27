---
title: "[CICD]Tekton in k8s trigger"
key: 20210427
layout: article
tags: cicd tekton k8s trigger eventlist triggertemplate triggerbinding trigger
---

## 前言

上一篇只是個開頭，其實我花了最多時間的地方就在這邊(~~各種撞牆~~)，雖然說網路上資源很多、範例也多(~~但是大多年代久遠~~)，也許有些部份還可以拿來用，但是這種情況會讓人挫折感大增。

<!--more-->

## 了解工作流程

基本上到官方的`GitHub`都有相關說明，不過我這邊簡單帶一下就好，流程基本上就是在建立`Eventlistener`後會自動起個`Service`，該`Service`的前綴自動加上`el-`很容易分辨，這個`Service`主要目的是準備讓你接`Webhook`可以使用，如果你用`Ingress`或是`Traefik`這類的工具可以很輕鬆的把`Webhook`串起來。

`Eventlistener`會至少指定一個`TriggerTemplate`和`TriggerBinding`，以這邊的擴充性來看感覺上應該是可以一個`Template`跟多個`Binding`，但是本人真的沒有多的時間一個一個試，我只有試過串接兩個`Template`成功的例子，如果要細分資料的話也許真的可以一對多個`Binding`；而在`Eventlistener`裡要注意的地方是`interceptors`，他的執行順序在`TriggerBinding`之前，也就是`Webhook`過來後，就會被`interceptors`攔截到，並開始處理你指定的相關動作，然後再執行`TriggerBinding`的變數設定，最後才執行`TriggerTemplate`的腳本流程。

流程如下所示

Eventlistener > TriggerBinding > TriggerTemplate.
{:.info}


## 啟動之前

這部份是我個人會額外多做的一個步驟，就是驗證我的`task`和`pipeline`可不可以正確的運作，以及真的出問題的話可以順便了解是不是自己並不是真的懂這套工具了。

跟原生的手動部署方式一樣，我先分別跑`TaskRun`和`PipelineRun`來確認功能是否都可以正常的運作，


## 結語

這篇主要著重的點是在`trigger`的流程上是怎麼運作的以及需要注意的地方是什麼，把東西順一順後才發現自己當初卡很久的地方反而是在工具的理解不夠。


## 參考資料

[pipeline](https://github.com/tektoncd/pipeline)

[tekton catalog](https://github.com/tektoncd/catalog/tree/main/task)

[tekton trigger](https://github.com/tektoncd/triggers/blob/main/docs/eventlisteners.md#multiple-eventlisteners-one-eventlistener-per-namespace)
