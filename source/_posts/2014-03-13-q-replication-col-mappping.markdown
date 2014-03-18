---
layout: post
title: "Q Replication Column Mapping"
date: 2014-03-13 09:08:34 +0000
comments: true
categories: 
- Q replication
---

寫起來當個紀錄。

這兩天工作的時候碰到 Q replication 從上游流到下游的時候有 column 對應錯誤。

    Capture  Apply
       A ---> A
       B ---> C
       C ---> B
       D ---> D

大概就是這樣一個情形。
檢查 Apply 端的 `IBMQREP_TRG_COLS` 看到 `SOURCE_COLNAME` 與 `TARGET_COLNAME` 的對應相當正常。

後來才發現 Q replication 是以 `SRC_COL_MAP` 這個欄位來決定上下游 column 的 mapping。

<!-- more -->

## IBMQREP_TRG_COLS 中幾個欄位的定義 ##

看了下 Information Center, `SOURCE_COLNAME` 可能有以下三類情形:

    1. 被訂閱的 source column 名稱
    2. 用來初始化 target column 值的 SQL expression (即 replication 自行加入的值)
    3. An auditing column in a consistent-change-data (CCD) tables

第三點就不是很懂在做什麼了。

由於 replication 不只會抄寫到 table, 也可能會把值丟給 stored procedure，所以 `TARGET_COLNAME` 同時會包含下游的名稱與參數。


`SRC_COL_MAP` 是這麼被描述的:

The column position, data type, length, and code page for all of the columns that are used in an SQL expression.

實際內容長得像這樣:

    448,64,850,0,1;448,64,850,0,1;

## 如何改變 column 的對應? ##


清掉 IBMQREP_TRG_COLS 裡面錯誤 mapping 的 `SRC_COL_MAP` ，然後 REINIT。

這個動作不用重新啟動 replication。

### Clear Wrong Config ###

雖然說應該只要清掉 `SRC_COL_MAP` 即可，不過之前處理的時候為了保險起見就全部清掉了

    UPDATE schema.IBMQREP_TRG_COLS SET
        MSG_COL_CODEPAGE=null,
        MSG_COL_NUMBER=null,
        MSG_COL_TYPE=null,
        MSG_COL_LENGTH=null,
        SRC_COL_MAP=null,
        BEF_TARG_COL_NAME=null
    WHERE ...


### Reinitial Sub ###

REINIT_SUB 這個 signal 會讓 replication 重讀 IBMQREP_SUBS & IBMQREP_SRC_COLS & IBMQREP_SENDQUEUES ，吃最新的設定值。

    insert into schema.IBMQREP_SIGNAL(
         SIGNAL_TIME,
         SIGNAL_TYPE,
         SIGNAL_SUBTYPE,
         SIGNAL_INPUT_IN,
         SIGNAL_STATE
     ) values (
          CURRENT TIMESTAMP,
         'CMD',
         'REINIT_SUB',
         'subname',
         'P' );

`SIGNAL_STATE` 會有幾個狀態

    P: Default 值，pending，表示 Q Capture 尚未收到這個 signal
    R: Receive
    C: Completed process
    F: Fail

REINIT 後過一會子，檢查 `IBMQREP_TRG_COLS` 設定有吃進去之後應該就好了。

##Reference##

1. [Information Center: Control tables at the Q Apply server](https://publib.boulder.ibm.com/infocenter/db2luw/v9/topic/com.ibm.websphere.ii.replication.qrepl.doc/reference/iiyrqctbrapplist.html)
2. [Information Center: Changing the properties of unidirectional Q subscriptions](http://publib.boulder.ibm.com/infocenter/db2luw/v9r5/topic/com.ibm.swg.im.iis.repl.qrepl.doc/topics/iiyrqsubcchgovew.html)


