---
layout: post
title: "Q Replication - 一些 PD 技巧"
date: 2014-03-17 21:41:54 +0800
comments: true
categories: 
- Q replication
---

最近不知為何 Q Replication 一直發爐，發發發發你媽的爐啊啊啊啊啊。

紀錄 & 整理一些釐清 / 解決問題的小技巧。

<!-- more -->

## Find the Problem ##

### log ###

大部分時候 Replication 的問題會記在他的 log 裡面 (沒有記在裡面的分開討論)，
所以有狀況的時候通常可以在 log 中找出到底發生了什麼事情。

如果 log 顯示 Replication 正常，但是資料卻遲遲一直沒有往下抄，可能發生

1. 下游資料庫 performance 很差 / 有 deadlock
2. Replication 做得很慢


通常下游效能很差時會所有的 Queue 一起停擺，如果只有一個 / 數個 table 做得很慢則可以合理懷疑是 deadlock 或者在 replication 端卡住。

### Q Depth ###

Q Depth 指的是在 Queue 中等待被抄寫的 msg 深度。

如果 Q Depth 一直成長而沒有消掉，則表示該 Queue 的第一個 msg 做得非常慢。

### asnqmfmt ###

`asnqmfmt` 可以把 Queue 裡面的 msg dump 出來看。

我通常用的是:
    asnqmfmt *queue_name* *queue_manager_name*  -mqmd -l 1

意為 dump Queue 中的第一筆資料，看這個 queue 到底卡在哪一筆做得很慢。


## Solve the Problem ##

### spill ###

spill 是把有問題的 Table 從 queue 中挑出來讓他單獨往下做，
不要讓這個壞掉的 subs 影響到其他正常的抄寫。
    asnqacmd apply_server=*apply_server* apply_schema=*schema*
    spillsub=*queue_name*:*sub_name*
    
spill 成功之後 IBMQREP_SUBS 中該 subname 的 STATE 會變成 S ，表示該 subname 進入 spill mode。

如果沒有變成 S 則表示 spill 失敗。

spill 出來檢查問題，之後把 table 從 spill 狀態復原的指令
    asnqacmd apply_server=*apply_server* apply_schema=*schema*
    resumesub=*queue_name*:*sub_name*

spill 指令在 Receive Queue Inactive(check IBMQREP_RECVQUEUES `STATE` column) ，
或者 replication 正在處理該 table 的時候是無效的。

    SELECT STATE FROM SCHEMA.IBMQREP_RECVQUEUES WHERE SUBNAME='subname'

若是 Receive Queue Inactive 時想要 spill，那... 就快速啟動該 Queue ，
在還沒有 Inactive 之前丟 spill 指令即可。

### 強制停止 ###

如果在 replication 正在處理該 table 的時候要強制停止對該 table 的 Apply 的話，
直接把 IBMQREP_SUBS 與 IBMQREP_TARGETS 中的 `STATE` 改成 'I'，
接著重新啟動 Q Apply 即可。

    # connect to Capture server
    UPDATE SCHEMA.IBMQREP_SUBS SET STATE='I' WHERE ...

    # connect to Apply server
    UPDATE SCHEMA.IBMQREP_TARGETS SET STATE='I' WHERE ...

    # stop Apply
    asnqacmd apply_server=*apply_server* apply_schema=*schema* stop
    # start Apply
    asnqacmd apply_server=*apply_server* apply_schema=*schema* 
    startq=*queue_name*


## Reference ##

1. [Information Center: asnqmfmt](http://pic.dhe.ibm.com/infocenter/db2luw/v9r7/topic/com.ibm.swg.im.iis.repl.qrepl.doc/topics/iiyrqfmtrasnqmfm.html?resultof=%22%61%73%6e%71%6d%66%6d%74%22%20)
