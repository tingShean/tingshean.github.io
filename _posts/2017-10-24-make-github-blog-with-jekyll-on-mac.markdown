---
layout: post
title:  "[Mac]使用jekyll製作github blog"
date:   2017-10-24 12:28:00 +0800
categories: jekyll update
---
網路上已經有很多jekyll的各類教學，這邊主要是記錄在安裝上遇到的一些問題。

因為使用的平台是Mac OS，所以跟linux的安裝還是有些許的不同，我在執行`$gem install jekyll`時就有遇到問題
{% highlight ruby %}
~/Documents/work » gem install jekyll
Fetching: public_suffix-3.0.0.gem (100%)
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.0.0 directory.
{% endhighlight %}

看上去是權限的問題，所以我就直接加上`sudo`，接著直接回我
{% highlight ruby %}
Fetching: public_suffix-3.0.0.gem (100%)
ERROR:  Error installing jekyll:
	public_suffix requires Ruby version >= 2.1.
{% endhighlight %}

好的，看來內建的版本沒辦法直接安裝，在這邊我就直接下`brew install ruby`


{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
