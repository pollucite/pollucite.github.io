---
layout: post
title: "backdoor CTF Web 10 and 30"
date: 2014-03-23 22:59:03 +0800
comments: true
categories:
- CTF
- backdoor 2014
- Web
---


暖身題★不過好久沒碰，想想還是稍微紀錄一下♥

<!-- more -->

## Web 10 ##

上題目 :)

![backdoor CTF web 10 question](/images/2014-03-23-backdoor-web-10-and-30/web-10-question.PNG)

「仔細看看四周！」 source code 裡面沒藏東西，翻一下 Header ♪ 

Wow! get it ♪

![backdoor CTF web 10 answer](/images/2014-03-23-backdoor-web-10-and-30/web-10-answer.PNG)


## Web 30 ##

一個 submit 按鈕，按下去以後會說你沒有權限 :O

![backdoor CTF web 30 question](/images/2014-03-23-backdoor-web-10-and-30/web-30-question.PNG)
![backdoor CTF web 30 question](/images/2014-03-23-backdoor-web-10-and-30/web-30-question-1.PNG)

直接看 Header... Cookie: auth=false; 

![backdoor CTF web 30 question](/images/2014-03-23-backdoor-web-10-and-30/web-30-answer-1.PNG)

把 auth 改成 true... get it!

![backdoor CTF web 30 question](/images/2014-03-23-backdoor-web-10-and-30/web-30-answer-2.PNG)
