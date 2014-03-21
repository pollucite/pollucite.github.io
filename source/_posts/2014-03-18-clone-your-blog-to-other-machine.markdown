---
layout: post
title: "遍地開花之 octopress 搬家記"
date: 2014-03-18 23:00:07 +0800
comments: true
categories: 
- git
---

小爺我之前把 octopress 架在 koding.com 上，但是他的 terminal 不支援中文輸入，我又很不喜歡他的文件編輯畫面，想設 ssh 卻怎麼也都搞不定 (...)

後來想想樓友的 octopress 都在家裡那台工作站上編寫，我幹嘛這麼鐵齒一定要在 kd.io 上用呢我～★

紀錄 git 初學者 (A.K.A, ME) 搬遷 octopress 到另一個工作站上的過程。

<!-- more -->

## Setup Environment ##

首先把 source 和 _deploy 上去的 Blog 抓下來

    git clone -b source https://github.com/pollucite/pollucite.github.io octopress
    cd octopress
    git clone https://github.com/pollucite/pollucite.github.io _deploy

接下來 setup_github_pages (假設 ruby 相關環境都已經 setup 好了)

setup_github_pages 會切到 _deploy 資料夾並且在 local 建立一個 master branch
，並且 setup remote repositroy 

    rake setup_github_pages

## Deploy ##

寫完 Blog 就可以準備把它 deploy 到 github 上了。

    rake generate
    rake deploy

這時候可能會碰到 conflict ，手動修正 merge 再推回去即可。

    cd _deploy
    git pull origin master
    # fix conflict ...
    git commit -m 'merge deploy'

這個時候 local 的圖會看起來像這樣:

![local git history picture of migrate octopress blog to new host](/images/git_octopress.png)

推到 remote repositroy:

    git push origin master

推上去以後的圖:

![remote git history picture of migrate octopress blog to new host](/images/git_octopress_2.png)



## Reference ##

[Clone Your Octopress to Blog from Two Place](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)
