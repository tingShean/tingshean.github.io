---
title: "[筆記]使用 Ansible 的 docker_compose 二三事"
key: 20200514
layout: article
tags: ansible docker_compose
---

**注意：這篇並不是在說明ansible執行docker-compose.yml這件事，而是指ansible裡docker_compose的使用記錄**

<!--more-->
最近一樣是繼續踩坑填坑無限LOOP，好處是踩的越多，知道的也就越多，能融會貫通的地方也就更多 ~~(公三小)~~


首先，這篇不是在說用ansible執行docker-compose.yml(不管是yml還是yaml)，而是使用`docker_compose`這個`module`，因為我用的是`Centos 7`，所以這坑的深度雖然沒這麼深，但他就跟滿地坑一樣，你不會只踩到一次，如果他是地雷，我大概已經死無全屍了~~連環爆看過嗎~~


曾經一度查到懷疑人生，懷疑是不是自己根本就誤會了


## 回到正題

第一個要注意的是，使用ansible內docker和docker_compose這兩個module的時候，要安裝的東西不是`yum install`的`docker`，而是`pip install docker docker-compose` ~~(然後就會踩到一連串的坑，如果你也用Centos7)~~
	會這樣做的原因只是為了讓ansible可以接到CI/CD的變數，所以直接使用內建的module，才會接到外部進來的變數

接下來當你改成
``` yml
- name: pip install docker docker-compose
  pip:
    - docker
    - docker-compose
  state: latest
```

~~就會跟你說，不好意思不給你裝喔~~
會直接跟你說無法安裝，因為沒有pip

很好，沒有pip那就直接裝`python2-pip`~~(直接中第二次陷井)~~，開始懷疑自己曾經是不是這樣亂裝了，關於這邊不用python3的原因後面再說
``` yml
- name: yum install python-pip
  become: yes
  yum:
    name: 
      - python-setuptools
      - python2-pip
    state: present
```


執行到這直接也跟你說不給裝，後來去爬文是少了兩個套件，直接加上去吧，我不知道為啥沒有`gcc`...
``` yml
- name: yum install python-pip
  become: yes
  yum:
    name: 
      - gcc
      - python-devel
      - python-setuptools
      - python2-pip
    state: present
```


接著這邊，我一開始是用python3的方式然後到這踩到下面的坑後，這坑就像地雷一路把我炸回去；在執行docker_compose的時候，他會要求用pip裡的套件，所以前面沒灌就沒救了，而這時候我是先後灌了`docker`和`docker-py`，以經驗上來說應該沒什麼太大的問題，所以當我灌好`docker-compose`後，他就跟我說，`docker`和`docker-py`請挑一個使用~~很難伺候~~，好啊，所以我直接進server砍掉`docker-py`(官方文件比較推docker)，先說這邊為啥會裝這兩個，因為我看了github上的討論(docker_compose的問題真的很大)，砍完後繼續試，又噴錯，直接跟你說要用`docker or docker-py`...(公三小)，最後去找官方文件說，如果你兩個都裝了請移除一個，然後移除完後，記得重裝另一個。


因為是用腳本跑，所以記得在`pip`裡加上`state: latest`，他就會重跑了。


以為沒事了嗎？接著我想把`templates`加進去，省得一些設定的東西我得每個server都一份，然後又出事了；使用python3的時候會噴權限問題，但問題是我已經把資料夾的權限改好了~~你他○的是有三小問題~~，他上面會顯示
`"msg": "Aborting, target uses selinux but python bindings (libselinux-python) aren't installed!`
然後你跟著裝就完了，這個是在使用python3才會遇到的問題，你會想說那我就直接安裝就好啦，抱歉，python3無法使用`yum`跑腳本，所以請你改成`dnf`，然後你改成`dnf`後他就會跟你說無法使用`dnf`~~(無限公三小)~~，接著我想說先用`python2`裝好這完意行不行；抱歉，一樣不行。最後我是找到開發人員在github上的回覆，有想了解的可以點下面的連結過去。


大致上的意思是說這個是`Centos7`底層的問題，不可能也無法改成`python3`，建議最好的做法是直接改`ansible`的設定檔，或是在執行腳本的時候加上`ansible_python_interpreter: "/usr/bin/python"`即可，然後就沒有然後了。


其實上面這些經驗就解掉`docker_compose`的問題，接下來的是使用`docker_container`如果要掛進`docker_compose`起好的`network`時，記得在networks裡加上`aliases`，才會掛載成功，這個參數是要讓同一個`networks`的別的容器可以使用這個名稱呼叫到該容器，我也覺得很怪，但是加上後就成功啟動了。



## 後記
其實最近踩坑踩的有點累，但是經驗上來說還是賺飽飽，有空的話我再寫一篇用ansible安裝php7遇到的問題；至於為什麼堅持要用腳本跑完整套流程，沒什麼特別的，就只是因為我懶，我希望可以在固定的環境下，我直接下指令讓他跑完整包腳本就可以交差，何樂不為？以前光是LAMP或LNMP裝起來都是一小時上下，有時候插個單起來直接兩小時起跳，直接用腳本跑，被插單就插單，我還不用煩惱等等回神還要找我做到哪，多好？


[安裝pip](https://ansible.tw/#!docs/installation.md)
[Ansible dnf python3 is not working with Centos 7](https://github.com/ansible/ansible/issues/67083)
[Docker fails to create network bridge on start up](https://github.com/moby/moby/issues/18113)