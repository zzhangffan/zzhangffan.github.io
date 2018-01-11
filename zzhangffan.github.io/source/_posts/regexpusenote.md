---
title: 正则表达式使用笔记
date: 2018-01-09 10:54:22
categoreies:
tags: RegExp
---
> 项目过程中有很多用到正则表达式的地方，如：手机验证、邮箱验证等。总是在网上百度一条，扔进去运行，咦！没问题，行吧~就这样了  我想大部分人跟正则的关系都在这个层次了

其实之前就去看过正则了，一直也没有总结，今天在这里总结下正则用法！

## 话不多说，先上“栗子” -
需求：匹配字符串 ` [Rat, ox, tiger, rabbit, dragon, snake, horse, sheep, monkey, chicken, dog, pig] ` 十二生肖中的 ` dog `
正则：` dog `
匹配结果： [Rat, ox, tiger, rabbit, dragon, snake, horse, sheep, monkey, chicken, <span style="color: red;">dog</span>, pig]

<span style="color: red;">PS:上面我们使用正则` dog ` 匹配了字符示例字符串中的 dog，这是最简单的正则了，其实生活中我们也用到过类似场景，还记得Windows系统中的文件搜索吗</span>
![Windows系统中的文件搜索例图](/images/regexpusenote/Windows_find.png)
<center>（在当前磁盘中搜索包含 Git 的文件[夹]）</center>

是不是有那么点感觉了，没有感觉也没有关系，相信深入了解，你会对正则有感觉的！看过例子后，让我们来了解下正则↓

## 正则 -

先来几个问题，引人深思↓
* 正则是什么？用来干什么？
* 怎么用？
* 正则语法？

随着问题走进正则使用笔记：
正则是什么？用来干什么？
---------------------
&emsp;&emsp;[正则表达式，又称规则表达式。（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表达式通常被用来检索、替换那些符合某个模式(规则)的文本。](https://baike.baidu.com/item/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/1700215?fr=aladdin)
怎么用？
-------
<span style="color: red;">各大编程语言都对正则表达式进行了支持，如：Java、JavaScript、Prel等。这需要我们去了解相应的API。这里我们使用[在线测试工具](http://regex.zjmainstay.cn/)进行学习。
正则语法？
-------
我们以试题来学习正则语法：
需求：匹配字符串 ` [Rat, ox, tiger, rabbit, dragon, snake, horse, sheep, monkey, chicken, dog, pig] ` 十二生肖中的所有动物
正则：` \w+ `
匹配结果： <span style="color: red;">Rat ox tiger rabbit dragon snake horse sheep monkey chicken dog pig</span>

> PS:上面我们使用了正则元字符 ` \w ` 和 表重复的 ` + ` 。
&emsp;&emsp; ` \w ` ： 匹配字母、数字或下划线
&emsp;&emsp; ` + ` ： 匹配前面的字符一次或多次