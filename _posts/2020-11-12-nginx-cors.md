---
title: "[筆記]Nginx CORS"
key: 20201112
layout: article
tags: cors nginx header preflight
---
這東西在以前應該是不太可能看到，但是在現在的環境，微服務的興起，這東西也就變成了日常

<!--more-->

## 前言
最早遇到這東西的時候，我用的是Golang，所以直接用套件就做掉了，第二次遇到是Python，然後也是用套件，我想說的是~~套件很方便~~有時候別太依賴套件，如果是快速開發的情況下就算了，求快就沒啥好說的，但是要記得回頭看看，有時候只是因為沒注意到細節而在原地繞了很久。

這次我的角色不是開發人員而是SRE，所以很多東西就算我知道要怎麼做比較快，但在以後的使用上我不可能一直要RD用套件做掉，所以就趁這個機會來試試~~然後就直接中伏~~

## 起因
又是公司的案子，在開發的初步突然跟我說他們想走前後分離，然後對方也是首次挑戰這個技術。可以這樣做有很多好處，第一不用再靠server render，第二這一切直接乾乾淨淨，我不用在包一大包的前端檔案在後端的CICD裡，這樣在流程上就單純很多。


## 第一步
開始做就直接各種CORS，最後是因為手上有東西要先趕，所以RD先用套件做掉，雖然這樣就真的通過了，但是對於沒有解開問題就是不太爽。

## 找出原因
理論上這東西的設定檔大同小異，都是先送一次OPTIONS跟server說這邊有資料要取or送，並且需要什麼樣的header，然後server回應ok後就可以發送對的請求過來。

~~然後就錯了~~

在運作之前有幾個動作要先確認的就是你的API服務會用到什麼`method`？`OPTIONS`、`GET`以及`POST`是基本三個一定會用的，後面還有`PUT`、`PATCH`和`DELETE`，這個如果你的framework有用到才會有用，不然以後端來說是用`GET/POST`最直接。

再來就是你用到的header，這邊要注意的是這個header基本上都是瀏覽器自動要求的，所以建議是看過一遍自己瀏覽器的需求再把對的header加上去，不然你真的不會知道自己加了這麼多header是幹嘛用的~~而且這些header不是你加了就有用~~

## 解法
我這邊試了三個API後發現其實加這兩個就可以了，現在想想套件真的很方便，但是你不會知道他加了多少的header，有時候你得自己下去看看才知道，有些套件預設就開了87%，自己要注意啊。

{% highlight shell %}
    location /api/ {
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Content-Type,token';
            add_header 'Access-Control-Max-Age' '86400';
            return 204;
        }

        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Content-Type,token';
    }
{% endhighlight %}

一定會有人說`Origin`設`*`有跟沒有一樣，因為我們還在開發中，為了測試方便先這樣，之後改成自己的domain就可以了，從上面可以看我的header只有兩個`Content-Type`和`token`，這兩個都是直接看瀏覽器的console那邊得來的，只看我的不一定準。

## 結語
後來翻了一些資料，以及相關的設定檔，加上不斷的`try & error`之後，大致上可以知道錯在哪，以及要如何debug，我個人覺得debug這邊應該是最麻煩的，因為CORS的報錯基本上都是瀏覽器不給你看網頁產生的問題，再加上這個用postman測也是相對麻煩，所以只好乖乖的開瀏覽器慢慢抓問題。

## 相關連結
[CORS on Nginx](https://enable-cors.org/server_nginx.html)

[Module ngx_http_headers_module](http://nginx.org/en/docs/http/ngx_http_headers_module.html)

[Nginx PHP API CORS](https://stackoverflow.com/questions/52479618/nginx-php-api-cors)

[provisional headers are shown 知多少](https://juejin.im/post/6844903727770370062)

[Request header field * is not allowed by Access-Control-Allow-Headers in preflight response](https://blog.csdn.net/u014481096/article/details/79792583)

[Http跨域时的Option请求](https://www.cnblogs.com/cc299/p/7339583.html)