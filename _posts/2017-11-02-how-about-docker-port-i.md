---
layout: article
title:  "[Docker]粗略淺談 part I"
key: 20171102
tags: docker blog 
---
`Docker`相關的技術文章網路上也是多到炸掉，這裡主要是記錄一些檢測方式以及曾經遇過的問題。


一個docker container運行的成功與否除了最直接的看到期望運作的內容，我們可以運用幾個內建的功能

<!--more-->

`$ docker run -ti --rm --name <container_name> <images> env`

`$ docker run -ti --rm --name <container_name> <images> /bin/bash`

這兩段指令是我用來看我包好的image有沒有把我要的設定吃進去，比方說php-fpm的設定是否為9000 port，第二段指令會讓你啟動一個服務並進入該container內，這邊可以看到更細的東西，基本上這裡面就跟平常server裡docker正常運作是同樣的環境


`docker inspect <container_name>`

這段指令是讓你看看這個container裡所有公開的設定，像nginx在container內部的ip，php-fpm沒有對外的9000 port，或是想知道這個container實際上的位置都可以從這個指令得知


`docker logs <container_name>`

如果你的服務一直都沒有起來，以上的幾個指令是無法使用的，而對於這種情況，就只剩這個直接看docker logs的指令了


其實docker發展到現在，真的是會越用越上癮，但是除了會用以外，也要練習處理一些相關的問題，有時候錯誤本身並不在docker，而是根本你設定就是錯了的時候，你只會看到docker的服務狀態顯示Exit之類的，試著去解決發生的問題，以後你就會避免這個問題產生。