---
layout: article
title:  "[Python][學習]function? method?"
key: 20180927
tags: python 
---

學東西就是要越學越深入，才會知道自己以前理解的觀點是對是錯，也可以慢慢了解設計者的想法以及定義。

先看看這兩個東西的定義

`
function —— 
A series of statements which returns some value to a caller. It can also be passed zero or more arguments which may be used in the execution of the body.
`

`
method —— A function which is defined inside a class body. If called as an attribute of an instance of that class, the method will get the instance object as its first argument (which is usually called self).
`

function的定義比較廣泛
***「會返回一些值，然後可以傳遞零或多個參數，而且可以用於執行主體」***

method則是基於function的定義在更嚴謹的定義其範圍
***「在類別主體內定義的函數，如果作為該類別實體的屬性使用，該方法將取得實體對像的第一個參數(通常是self)***

其實有時候會覺得這些定義文謅謅到我常常~~懷疑人生~~

不過整體上來說每個程式語言都大同小異，由上述可以理解的是method的定義更明確了，只要是在class產生的instance裡的function有帶第一個參數的，都歸類為method，就算第一個參數不是叫self，他還是會被當作self使用；而function定義比較廣，能被呼叫的通通都算。

-

參考資料 

[Python里method和function的区别](https://blog.csdn.net/peachpi/article/details/6047826)

[Python Classes](https://docs.python.org/3/tutorial/classes.html#instance-objects)