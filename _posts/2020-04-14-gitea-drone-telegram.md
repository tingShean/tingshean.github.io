---
title: "[筆記]Gitea+Drone+Telegram on Docker心得"
key: 20200414
layout: article
tags: gitea drone telegram docker
---
最近在弄新的Flow弄的有點昏頭，這邊稍微記錄一下過程

各種踩坑的心得
<!--more-->

一開始會用這個Flow主要是為了脫離Jenkins老爺爺的環境，因為公司有蠻多案子都不是用Java，而且有些雲端的套件都太久了，不見得可以用。


後來想用新的架構則是因為剛好都可以用docker搞定，雖然後來也踩了一堆坑，不過在前面是用Caddy做反向，不過這東西不是用docker就不特別提了。

以下有幾點要注意

### Drone對postgres有特別優化
基於這一點，所以原本我用MySQL架，但是調整到Drone之後，就直接砍掉換postgres了，docker的好處就在這，說換就換很直接。


### Drone使用的網路名稱要一致
網路上很多相關的docker-compose檔可以看，但是drone-server跟drone-agent對接使用的網路在參數上要一致，docker-agent起來沒接到drone-server也不會重啟，就算你有設定`restart: always`；在設定上如果是用新版本的docker-compose架設的話(比方說3.7之類的)，可以先用`docker networks ls`看一下跟參數裡的`DRONE_RUNNER_NETWROKS`要一樣，然後drone-server跟drone-agent都要有`DRONE_RUNNER_NETWROKS`，而且內容要一樣。


### Gitea在設定給Drone OAuth2的時候，callback的位置要注意
我在這邊卡了一段時間，後來看了一下一定要在最後面加上`/login`，比方說`https://yourdomain.com/login`這樣，沒設定對的話在call back就會直接噴URL不符合。


### Gitea的`ROOT_URL、SSH_DOMAIN、DOMAIN`要設對
這邊會影響的是Drone到時候取到資料的問題，最快的方式就是以Drone的角度去看比較準，所以像是我前面有用Caddy架設SSL和反向代理的關係，所以我在`ROOL_URL`這邊就是直接用`https`，然後後面就直接接外網連進來的domain最後不加port。


### Drone在pull git的時候要記得三個參數要設對
`DRONE_GIT_ALWAYS_AUTH`我沒記錯預設是關閉，如果要拉的repo是公開的就沒差，但是如果是自己的私有repo就要打開，不然不會登入，然後這個參數打開後，`DRONE_GIT_USERNAME`、`DRONE_GIT_PASSWORD`這兩個就要記得填了。


### Telegram的telegram_user_id幾種取法
一種是[機器人](https://telegram.me/userinfobot)，另一種是用URL的方式取得`https://api.telegram.org/bot{$token}/getUpdates`，因為Drone的telegram只有講用BOT取得，所以找到了第二個方法可以使用。
