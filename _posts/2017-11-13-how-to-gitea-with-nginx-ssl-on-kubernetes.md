---
title: [Kubernetes]gitea+Nginx+certbot
tags: kubernetes kubectl gitea nginx ssl certbot blog 
layout: article
---
記錄一下遇過的問題以及解決的方法，如果還沒辦法成功的透過nginx當入口，不建議直接看這篇

網路上其實有很多gitea+nginx+docker+certbot

kubernetes運作的底層是docker，所以架構也是一樣

我在這遇到的問題是使用certbot產生key了之後，把key掛載上去會一直出現

{% highlight shell %}
2017/11/12 10:25:49 [emerg] 9#9: BIO_new_file("/etc/letsencrypt/live/yourdomain.com/fullchain.pem") failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/letsencrypt/live/yourdomain.com/fullchain.pem','r') error:2006D080:BIO routines:BIO_new_file:no such file)
nginx: [emerg] BIO_new_file("/etc/letsencrypt/live/yourdomain.com/fullchain.pem") failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/letsencrypt/live/yourdomain.com/fullchain.pem','r') error:2006D080:BIO routines:BIO_new_file:no such file)
{% endhighlight %}

<!--more-->
(黑人問號)

這邊不管怎麼把實體位置掛上去，都會產生這個問題，就算是把權限群組改掉以及更換權限(read only)

本來一度想把產好的key包進image裡，不過想了想這樣手續更多，而且更費工


而且網路上不管是哪國的教學都是跟你說直接在nginx的config裡設定好目錄，並把對應的目錄掛進docker就可以了...(黑人問號)

後來我直接把產好的key放到我想要的目錄，並掛載上去，就解決了...(滿滿的黑人問號)

雖然我知道是目錄權限的問題，不過對於網路上的教學文章還真的是諸多疑問


以下是我的做法

1. 先把產好的key cp到我要的目錄位置

{% highlight shell %}
fantasy@ubuntu:~/kube$ sudo cp /etc/letsencrypt/live/yourdomain.com/chain.pem /etc/nginx/yourdomain.com/
fantasy@ubuntu:~/kube$ sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem /etc/nginx/yourdomain.com/
fantasy@ubuntu:~/kube$ sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem /etc/nginx/yourdomain.com/
{% endhighlight %}

2. 修改要掛載的位置並對應到nginx config
3. 啟動pods後連入，可以使用https連到自己的網頁並且看到gitea畫面
4. 製作root crontab以達到長期的ssl

{% highlight crontab %}
30 2 * * 1 /usr/bin/certbot renew  >> /var/log/le-renew.log
30 2 * * 1 /usr/bin/cp /etc/letsencrypt/live/yourdomain.com/chain.pem /etc/nginx/yourdomain.com/
30 2 * * 1 /usr/bin/cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem /etc/nginx/yourdomain.com/
30 2 * * 1 /usr/bin/cp /etc/letsencrypt/live/yourdomain.com/privkey.pem /etc/nginx/yourdomain.com/
{% endhighlight %}

最後的步驟因為都要有root的權限才能從letsencrypt搬檔，所以就直接放在root的crontab囉


相關參考網站

[Certbot官網][certbot-offical]

[Gitea with Nginx Reverse Proxy][focaabys-note]

[手把手教你在Nginx上使用CertBot][nginx-certbot]

[Gitea wiki][gitea-wiki]

[certbot-offical]: https://certbot.eff.org/#ubuntuxenial-other
[focaabys-note]: https://focaaby.github.io/2017/10/22/Gitea-with-Nginx-Reverse-Proxy/
[nginx-certbot]: https://segmentfault.com/a/1190000005797776
[gitea-wiki]: https://wiki.archlinux.org/index.php/Gitea