---
layout: post
title:  "[Mac]使用jekyll製作github blog"
date:   2017-10-24 12:28:00 +0800
categories: blog
tags: mac github.io jekyll
---
網路上已經有很多jekyll的各類教學，這邊主要是記錄在安裝上遇到的一些問題.


因為使用的平台是Mac OS，所以跟linux的安裝還是有些許的不同，我在執行`$gem install jekyll`時就有遇到問題


{% highlight shell %}
~/Documents/work » gem install jekyll
Fetching: public_suffix-3.0.0.gem (100%)
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
{% endhighlight %}


看上去是權限的問題，所以我就直接加上`sudo`，接著直接回我:

{% highlight shell %}
Fetching: public_suffix-3.0.0.gem (100%)
ERROR:  Error installing jekyll:
	public_suffix requires Ruby version >= 2.1.
{% endhighlight %}

好的，看來內建的版本沒辦法直接安裝，在這邊我就直接下`brew install ruby`

因為下了`brew upgrade ruby`直接回我沒有安裝(?!)，不過這邊應該是指我個人用戶下沒有這個東西.

該裝的都裝好，總該沒問題了吧？

但是現實是很殘酷的，該出錯的地方就是會錯，工程師本命就是在解決問題，自然也會吸引一堆問題(x)


當`jekyll`裝好後要啟動的時候就出現問題:

{% highlight shell %}
$ jekyll serve
/usr/local/Cellar/ruby/2.4.2_1/lib/ruby/2.4.0/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- bundler (LoadError)
	from /usr/local/Cellar/ruby/2.4.2_1/lib/ruby/2.4.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /usr/local/lib/ruby/gems/2.4.0/gems/jekyll-3.6.2/lib/jekyll/plugin_manager.rb:48:in `require_from_bundler'
	from /usr/local/lib/ruby/gems/2.4.0/gems/jekyll-3.6.2/exe/jekyll:11:in `<top (required)>'
	from /usr/local/bin/jekyll:23:in `load'
	from /usr/local/bin/jekyll:23:in `<main>'
{% endhighlight %}


ok，果不其然，事情就是這樣發生了，還好也有人發生類似的問題，一樣要用gem安裝bundler:

{% highlight shell%}
$ sudo gem install bundler
{% endhighlight %}

{% highlight shell %}
# 這個執行完會問密碼
$ bundle install
Password: 
Installing jekyll-feed 0.9.2
Fetching minima 2.1.1
Installing minima 2.1.1
Bundle complete! 4 Gemfile dependencies, 22 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
{% endhighlight %}

最後就是啟動啦
{% highlight shell %}
$ bundle exec jekyll serv
{% endhighlight %}

[Jekyll運行問題出處][jekyll-git]

[Jekyll安裝參考網站][jekyll-url]

[jekyll-git]: https://github.com/jekyll/jekyll/issues/5165
[jekyll-url]: http://seans.tw/2016/make-own-blog-with-jekyll-and-github-page/
