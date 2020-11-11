---
title: "[筆記]當火狐遇到 MOZILLA\_PKIX\_ERROR\_REQUIRED\_TLS\_FEATURE\_MISSING"
key: 20201111
layout: article
tags: firefox MOZILLA_PKIX_ERROR_REQUIRED_TLS_FEATURE_MISSING tls letsencrypt nginx
---

最近在忙別的專案設定伺服器上遇到很特別的問題，所以在這邊好好的筆記一下，該情境使用環境為nginx

<!--more-->

## 前序
這個問題主要只發生在火狐 Firefox 使用者瀏覽某些網站會發生的問題，用別的瀏覽器不會。

## 情境
大多數遇到的問題網頁都是使用了Let's Encrypt的TLS，我個人一開始推測應該是因為Firefox的安全性要求比較高產生的影響。

## 找出原因
[HTTPS connection fails with "MOZILLA_PKIX_ERROR_REQUIRED_TLS_FEATURE_MISSING"](https://support.mozilla.org/en-US/questions/1149911)

雖然是距今4年前的討論，不過這代表是以前就發生這問題了，不是現在才有，然後下面有回文叫人把`enable_ocsp_must_staple`這個關掉後強制重整 ~~你乾脆叫使用者重灌好了~~

不過由這個討論可以看的出來應該是OCSP的問題，所以我們直接從這部份找解法

## 解決方法
在出現這個問題的nginx configure裡加上以下這四行基本上就解掉了

{% highlight shell %}
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 1.1.1.1 valid=300s;
resolver_timeout 5s;
{% endhighlight %}

有興趣看裡面的設定可以看看最底下的nginx說明；這邊大概說一下

`ssl_stapling on/off` 就是字面上的意思，`stapling of OCSP response`的開關，預設是關閉的 [註1](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling)

`resolver 8.8.8.8 1.1.1.1 valid=300s` 這個先說，因為上面的`ssl_stapling`打開的時候就一定要設定這個參數。主要是在解析地址用，預設的情況下解析會同時查找IPv4和IPv6(IPv6可以關閉查詢)。 [註2](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver)

`ssl_stapling_verify on` 驗證OCSP response的開關；為了驗證生效會使用 `ssl_trusted_certificate`將伺服器證書頒發的證書和所有中間證書配置為受信任的。 [註3](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling_verify)

`resolver_timeout 5s;`設定解析的timeout時間 [註4](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver_timeout)


## 相關出處
[HTTPS connection fails with "MOZILLA_PKIX_ERROR_REQUIRED_TLS_FEATURE_MISSING"](https://support.mozilla.org/en-US/questions/1149911)

[ssl_stapling](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling)

[ssl_stapling_verify](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling_verify)

[resolver](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver)

[resolver_timeout](http://nginx.org/en/docs/http/ngx_http_core_module.html#resolver_timeout)