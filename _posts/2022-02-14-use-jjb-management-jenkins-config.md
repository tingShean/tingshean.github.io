---
title: "[筆記]用JenkinsJobBuilder管理Jenkins"
key: 20220214
layout: article
tags: jjb jenkinsjobbuilder jenkins
---
# 前言

因為最近又回去使用`Jenkins`的關係，對於一些配置上的整合似乎沒什麼工具可以直接好好的管理，除了自己寫`script`去呼叫`Jenkins`自有的`API`以外。後來想想也許設定都是差不多的關係，好像也沒什麼必要性一定得把每個`project`都列管。

因為之前聽過社群神人的分享，一直對`JenkinsJobBuilder`(以下簡稱`jjb`)倍感興趣，加上那時候是中途想到有課要聽，進去其實已經接近尾聲，所以只大概記了個名稱就算了。沒想到用了之後才發現這根本就是把你的`project`一一列管的工具，這工具讓你的`project`設定配置更貼進工程師那麼一點(~~嗨！YAML檔工程師~~)。

<!--more-->


# 事前安裝

對對對，為啥要拉出來講呢，因為現在雲端`vm`真心好用，所以不管是用`IaC`開機器還是你進`console`進去點一點就可以有一台小玩具可以玩了，然後照著官網上的安裝指令直接一秒撞牆。

原因在於現在所有`linux`自帶的`python`還在`2.7`版，所以不管是否裝了`pip`，他也會跟你說目前使用的`python`版本已經棄置了，然後你的`pip`太舊請更新~~哭啊！2.7的根本升不上去~~。

所以直接升上`3.x`版就可以，我記得我安裝的時候好像已經是`3.9`了。


# 開始之前

## 設定Jenkins API Token

打開`https://jenkins.url/me/configure`，直接產生一組新的`token`，這邊要注意的是你登入的`username`，是接下來要用`jjb`的使用者

## 設定JenkinsJobBuilder config

雖然`jjb`沒有什麼安裝上的難度，但是有一點我個人覺得還不夠友善，就是使用的`config`沒辦法下參數去指定我要切哪個環境使用，頂多讓你用`--conf`指定你想用的`config`位置，個人覺得有點可惜。

這邊直接輸入以下的`key`和`value`即可

官網上說`~/.config/jenkins_jobs/jenkins_jobs.ini`和`/etc/jenkins_jobs/jenkins_jobs.ini`都可以直接吃到，不需要額外下`--conf`指定。


```
[job_builder]
ignore_cache=True
keep_descriptions=False
include_path=.:scripts:~/git/
recursive=False
exclude=.*:manual:./development
allow_duplicates=False
update=all
allow_empty_variables = True

[jenkins]
user=<user_name>
password=<user_api_token>
url=<jenkins_url>
query_plugins_info=False
##### This is deprecated, use job_builder section instead
ignore_cache=True

[plugin "<plugin_name>"]
arg_key=value
```


# 第一步

## 只有一小步

先建立你要工作的檔案位置，使用預設的`~/.config/jenkins_jobs/`也行，不過要記住運作的目標都是以資料夾為單位，首先先建立以下範例至`jobs`

```YAML
- job:
    name: 'my_job'
    description: 'Automatically generated test'
    project-type: freestyle
    builders:
      - shell: echo 'hello world'
```

接著下`jenkins-jobs update /path/to/defs`，這裡的`/path/to/defs`替換成你的檔案位置，如果是我的話就會是`jenkins-jobs update jobs`，接著就會開始把這個`job`塞進`jenkins`裡了。

記著一點，這只是個管理工具，任何的運作都要自己進`jenkins`操作或是你寫好`trigger`的`git`執行相對應的動作。

## 擴展

在進行到下一個階段前，有個東西要先講一下，就是所謂的模板化，也就是說把大致上的配置和流程都定下來之後，接下來你只需要把對應的值填好就可以爽爽用了，直接看以下範例

```YAML
- job-template:
    name: '{name}_job'
    description: 'Automatically generated test'
    project-type: freestyle
    builders:
      - shell: '{command}'

- project:
    name: project-example
    jobs:
      - '{name}_job':
           name: getspace
           command: df -h
```

我直接把剛剛的腳本改成`job-template`，然後在`project`裡取用名為`{name}_job`的模板，接著把對應的變數塞上值後，一樣使用`jenkins-jobs update jobs`就可以在我的`jenkins`裡看到這些東西了。


## 變數

直接使用剛剛的範例，但是這次我做了一些些的改變，變數之後加上`|`接著的值就會是`default`

```YAML
- job-template:
    name: 'my_job'
    description: '{dest|Automatically generated test}'
    project-type: '{ptype|freestyle}'
    builders:
      - shell: "{cmd|echo 'hello world'}"
```

變數化後我個人覺得能塞`default`就塞一下，免的真的在運行測試的時候出問題，我個人覺得對於`Jenkins`的除錯訊息不是那麼的好懂，所以我是能塞就塞了


# 結尾

科技始終來自於惰性，雖然身為`SRE`這樣講好像很嘲諷，不過我覺得可以讓我少動些手指的事我還是會去做，像`jjb`這類的工具是很方便讓我可以一下就把全部負責的`project`一一列管完畢，也讓我省去了很多建置和部署上的時間；不過要記得一件事，前期的測試運作不會比你一個一個做來的快多少，但不能因為這樣就不去做這些看起來很繁索又吃力不討好的事，有時候只是一秒的操作才能顯得這工具的價值。
