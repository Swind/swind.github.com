.. title: Octopress安裝筆記 - 我跳槽到 Pelican 了 - 又跳到 Nikola 了
.. slug: octopress-note
.. date: 2011-10-19 11:17
.. tags: Octopress
.. link: 
.. description: 

2013.01.20 我又跳槽到 Nikola 了
--------------------------------------------------------------------

2012.11.28 我已經跳槽到 Pelican 了，不過還是把這篇留著當紀念好了。
--------------------------------------------------------------------

2012.05.21 更新這次好像真的解決在 Win7 上面的問題了
----------------------------------------------------

其實在 Windows 上面使用 Octopress 沒有什麼特別大的問題，注意文章的編碼就好。
但是有一點一直非常困擾著我的就是， Codeblock 的問題。由於 rake generator 所提供的錯誤訊息非常的少，所以很難找出錯誤。
最常在網頁上面看到的就是這個

::

      Liquid error: undefined method `Py_IsInitialized’ for RubyPython::Python:Module

一直到我今天無意間搜尋到這篇文章 (話說 Octopress 的安裝教學文真的越來越多了)
`Windows 8 安裝 Octopress 紀錄`_ 他真的找出問題了，重點就在於！ *Python 請安裝 32-bit 的版本*。
真的這樣就讓 Codeblock 可以正常運作了 T\_T 
真的是太讓人心酸了。

.. _Windows 8 安裝 Octopress 紀錄: http://hivan.me/octopress-install-to-windows8/

.. TEASER_END

在Ubuntu上面實在是有夠難安裝的
------------------------------------

目前使用的環境是在Ubuntu上面，並且使用rvm安裝Ruby 1.9.2與Gem。但是就是那個但是這樣安裝的Ruby其實是不能用的，還需要用rvm安裝zlib與openssl才能使用。
安裝方法可以參考 `How to Install Octopress on Ubuntu 11.04 and Deploy on GitHub`_ （本連結已經失效了，所以請另外 Google 安裝方式吧！），若前面這個網站有不清楚的部分或者想要找指令來源的可以到 RVM_ 的網站查詢。

.. _How to Install Octopress on Ubuntu 11.04 and Deploy on GitHub: http://www.distancetohere.com/how-to-deploy-jekyll-slash-octopress-to-heroku/
.. _RVM: http://beginrescueend.com/

在Windows上面也沒有好裝多少，但是我好像成功了
----------------------------------------------------

Windows上面沒有好用的RVM可以用，而且Ruby安裝包比較不一樣，叫做 RubyInstaller_ 。RubyInstaller 的安裝方式蠻簡單的，就一直下一步就可以了。
安裝完畢之後你會發現 RubyInstaller 連 gem 都幫你安裝好了( Linux 我都用 RVM 裝，所以我不知道 gem 是不是安裝 Ruby 一定會有的)，但是這個版本的 gem 有點舊，所以需要下指令升級一下。

    gem update --system

再來就是需要另外安裝一個 DevKit_ ，這是Windows特有的老實說我也不知道這個要做啥。但是如果沒裝的話，你在使用gem安裝某些東西的時候他會跳出訊息要你安裝。

P.S 附帶一提，這是未確認事項不過在Windows上面安裝Git的時候，請選擇讓他將Git的執行檔路徑加入Path裡面的選項，也就是說讓你可以在命令提示字元上使用Git指令。
因為我想Octopress應該沒有聰明到會自己去抓Git的安裝路徑。

.. _RubyInstaller: http://rubyinstaller.org/
.. _DevKit: http://rubyinstaller.org/add-ons/devkit/

安裝與設定octopress
--------------------------------

終於可以進入octopress的安裝了，在這裡安裝的順暢與否就取決於你的平台了(誤)。詳細安裝方式可以參考Octopress官網的 `Octopress Setup`_ 。

首先先使用git將octopress cloe下來

    git clone git://github.com/imathis/octopress.git octopress

進入octopress資料夾，使用gem安裝所需要的Ruby套件與設定octopress

    gem install bundler
    bundle install
    rake install

到這邊我們終於踏進了八奇思考領域的第一步 Orz ... (我好久沒有看火鳳了)

.. _Octopress Setup: http://octopress.org/docs/setup/

Deploy到GitHub上面
---------------------------------

Octopress的官網有提供 `Github Pages`_、Heroku_ 與 Rsync_ 的上傳教學。Heroku還蠻多人使用的，在網路上也可以找到不少教學。
但是我還是對Github比較熟悉，所以這邊只列出Github的設定方式

Github Pages的功能老實說我也是因為看到Octopress才知道的，所以不清楚他能做啥，反正他可以放網頁就是了(大誤)。

首先請先在Github上面開啟一個Repository，Repository的名稱格式為：

    帳號名稱.github.com

例如我的id是swind，所以名稱就如你現在網址列所看到的為

    swind.github.com

接著呢要跟Octopress說你的Repository的路徑，進入Octopress的資料夾，輸入下面的指令

    rake setup_github_pages

Octopress會要你輸入Repository的路徑，這時請將Github上面SSH的路徑複製下來貼上，例如我的就是

    git@github.com:Swind/swind.github.com.git

完成之後你就可以輸入

    rake generate
    rake deploy

來測試是否可以正常deploy了。

P.S如果是使用Windows的朋友，然後你的文章檔案又是使用UTF-8存檔的話，應該會在rake generate的指令上遇到編碼問題。這是Jekyll的問題，可以透過設定環境變數來解決。
在執行rake generate的指令之前先輸入:

    set LC_ALL=zh_TW.UTF-8
    set LANG=zh_TW.UTF-8
    
.. _Github Pages: http://octopress.org/docs/deploying/github
.. _Heroku: http://octopress.org/docs/deploying/heroku
.. _Rsync: http://octopress.org/docs/deploying/rsync

同場加映-同步與版本控管的方式
---------------------------------
`Github Pages`_ 上教學的最後有教你使用Git的Branch來將Octopress的原始碼以及Blog的頁面同時儲存在同一個Repository上，我有這樣用，而且還蠻有趣的。
Octopress會將\_deploy資料夾中的檔案Push到主分支，也就是Master。而你可以透過在Octopress的資料夾中輸入

    git add .
    git commit -m 'your message'
    git push origin source

這樣就會將Octopress的原始碼Commit到source的分支上面，而不會影響到Blog的內容。但是這樣我在其他電腦要使用的時候都還要clone兩次(Octopress一次，\_deploy一次)實在很麻煩。
所以我最後還是使用Dropbox來進行同步，我乾脆就將整個Octopress放在Dropbox裡面，這樣不管我Linux或Windows都可以隨時編輯我的文章。

以上心得分享完畢，那個某人不要再來找我要心得了阿 XD