---
title: "[Centos]Nginx網頁讀取出現403"
key: 20200211
layout: article
tags: nginx centos 403
---
最近在幫忙弄其他專案的時候終於碰到連docker都沒辦法使用的情況

所以這下能用的就只有很hardcode的作法

一個一個裝...

<!--more-->

在我把全部的檔案權限都設定好之後~~事情就這樣發生了~~

不管我怎麼連網頁都連不到，好歹給個403、404之類的

雖然403有給，但是都是在log裡才看到

後來只好往權限這個方向試試，才想到以前教我的前輩是叫我直接關掉

原因就是很難用~~明明就是不會用~~


### Step1 先看權限功能是否開著
`getenforce`

如果回應是Enforcing，繼續往下

### Step2 修改目錄檔案 content_type
`chcon -Rt httpd_sys_content_t /path/to/www`

### Step3 重啟nginx
`sudo systemctl restart nginx`


基本上這樣就可以了


相關參考
[403 Forbidden nginx (13) permission denied](https://www.digitalocean.com/community/questions/403-forbidden-nginx-13-permission-denied)
[限制網頁根目錄外的存取](https://dywang.csie.cyut.edu.tw/dywang/rhel7/node53.html)
