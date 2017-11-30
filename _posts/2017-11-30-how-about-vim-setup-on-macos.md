---
layout: post
title:  "[IDE] vim setup on macos"
date:   2017-11-30 22:00:00 +0800
category: blog
key: 20171130
tags: vim macos blog 
---
其實最近是因為在學習Golang，因為之前都是用sublime和GoLand這兩套IDE

但用久了還是懷念vim，所以想說把Mac上的vim搞一搞

雖然當初好用的.vimrc沒有備份到，想說也趁這個機會好好的把相關的設定看一看

<!--more-->

在macos上內建的vim版本是7.4.8056

而brew上安裝的版本是8.0.1300


我在調整和安裝設定的時候，一直都是直接用brew的vim下去做設定

但不知道為啥這沒辦法取代原本mac上的vim

當然，這樣也沒辦法對本機的vim做升級的動作 (問號)


所以那時候我既用brew重灌了兩次才想到應該是原生的vim不給取代

期間也下了很多次的brew link，但怎麼下就是沒用

後來受不了我就直接進.zshrc(原生的應該是.bashrc，因為我有換)

加上

{% highlight shell %}
alias vim="/usr/local/Cellar/vim/8.0.1300/bin/vim"
{% endhighlight %}


然後我才發現我的設定其實都裝好了


相關參考網站

[\[Golang\]將開發環境從sublime text切換到vim][vim]

[vim]: http://www.evanlin.com/switch-ide-to-vim/
