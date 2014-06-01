.. title: 透過 Travis 自動佈署 Nikola 產生的網頁到 GitHub 上
.. slug: auto-deploying-my-nikola-blog-to-github-by-travis-ci
.. date: 2014/03/04 17:49:24
.. tags: Nikola, Python
.. link: 
.. description: 透過 Travis 自動佈署 Nikola 產生的網頁到 GitHub 上
.. type: rst

之前一直想要找 Server 可以結合 Jenkins 或者 Script 讓我不用每次都要重複安裝 Nikola 這件事情。
可是一直都沒有找到合用的，後來發現 Travis_ + GitHub_ 這個很多人使用的組合。
因此開始嘗試在將文章 commit 到 GitHub 上面之後， Travis 可以自動使用 nikola build 產生 HTML 檔案，
並且 Commit 回去 GitHub。

由於我的 GitHub 上面的網頁換了很多次系統 Octopress -> Pelican -> Nikola，在加上那時候對於 GitHub Pages 的設定不是很了解。
所以 git 上的 branch 有點奇怪。

我目前的 branch 有兩個

.. code::
    
    - master
    - source

master 就是 GitHub Pages 的部份，而 source 則是放 Nikola 的設定檔以及文章的原始檔。
現在 GitHub 預設放網頁的 branch 好像可以改成  gh-pages。不過改設定又是浩大工程，我以我就繼續沿用之前的 branch 了。
另外 Nikola 現在應該也有部屬到 GitHub 的功能，所以下面的流程說不定可以弄個更簡潔。

.. TEASER_END

讓 Travis 可以對 GitHub 上的 Repository 進行持續整合
-------------------------------------------------------

Travis_ 對 GitHub 的支援非常親切，只要使用 GitHub 帳號登入 Travis 之後，
到 Account -> Repositories 將 repository 開啟就可以了

.. figure:: https://dl.dropboxusercontent.com/u/15537823/blog/2014-03-04-auto-deploying-my-nikola-blog-to-github-by-travis-ci/enable_repository_on_travis.png

Travis 設定檔
------------------

再來就要在 Repository 中增加 *.travis.yml* 讓 Travis 知道該如何編譯 Repository。
設定的詳細方式可以參考 `Travis - Building a Python Project`_ 我只針對如何讓 Nikola 可以運作做紀錄。

安裝 Nikola
=================

毫無難度的使用 pip 安裝就結束了 Orz
這邊你會看到我想設定 cache 讓 Nikola 不用每次都重新下載安裝。
但是似乎是失敗的，我從 log 上都還是看得到 Nikola 的安裝過程。

.. code:: yaml

    language: python

    cache:
      directories:
        - /home/travis/virtualenv/python2.7/lib/python2.7/site-packages

    python:
    - '2.7'

    install:
    - pip install nikola

設定 nikola build 的流程
============================

nikola build 會將檔案產生在 output 的資料夾裡面，
因此我先 clone master 到 output，這樣在 nikola build 之後就可以直接在 output 裡面 commit 跟 push 了。

.. code:: yaml

    before_script:
    - git config --global user.name <Name>
    - git config --global user.email <Email>
    - git clone --quiet https://${GH_TOKEN}@github.com/<Repository Path> -b master output > /dev/null

另外為了避免在透過 SSH 連到 github.com 的時候需要確認
所以還需要在 before_script 增加

.. code:: yaml

    - echo -e "Host github.com\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config

當環境都設定完畢之後，就可以直接呼叫 nikola build 了

.. code:: yaml

    script:
    - nikola build

最後就是將 output 的內容上傳到 GitHub, *[ci skip]* 的部份是要跟 Travis 說這個 Commit 不要觸發 CI 流程，否則就會無窮迴圈了。

.. code::

    after_success:
    - cd output
    - git add .
    - git commit -m "[ci skip] Update output from Travis CI"
    - git push origin master

或者是加上 branch blacklist 略過所有 master 上的 commit 應該也可以。

.. code:: yaml

    # blacklist
    branches:
      except:
        - master

GitHub Token
=======================

再來就是要處理沒有解釋到的 *${GH_TOKEN}* 的部份。

由於 push 回 GitHub 上面是需要權限的，理論上這可以用兩種方式解決。
一種是放一個 SSH Key 在 Travis 上面，這可以參考 `Auto-deploying to My Octopress Blog With Travis-CI`_ 但是這個方法我沒有成功 Orz。
因為那個 Key 我一直處理不好。

所以我使用的是另外一個網頁 `Octopress+Prose+Github+Travis CI`_ 的方式，使用 GitHub Token。
有了 Token 就跟有了密碼一樣，可以直接去存取 Repository。

產生 Token 的步驟可以參考 GitHub 上的 `Creating an access token for command-line use`_ 。

在取得 Token 之後我們就可以使用

.. code:: bash

    git clone https://<Token>@github.com/<Repository Path>

的方式來 clone 跟 push。

但是 GitHub 上面的 Repository 是所有人都看得到的，所以如果你直接把 Token 放在 .travis.yml 上面，那麼就所有人都可以存取你這個 Repository 了 Orz。
所以我們需要透過 Travis 的工具將這段 Token 加密。詳細流程可以參考 `Travis - Encryption keys`_ 

.. code:: bash

    gem install travis
    travis encrypt GH_TOKEN=<GitHub Token>

這樣我們就可以將產生出來的

.. code::
    
    secure: .... encrypted data ....
    
部份放到 .travis.yml 內

.. code:: yaml

    env:
      global:
        secure: .... encrypted data .... 

之後就可以在裡面使用 ${GH_TOKEN} 存取了。

P.S 文章中使用 <...> 包起來的部份記得要自己替換掉

最後想在哪裡寫就在哪裡寫阿！
--------------------------------

之後只要你能 Commit 到 GitHub 上，你想用什麼寫就用什麼寫。
像 GitHub 上直接編輯的功能，StackEdit 等線上編輯軟體都可以。

但是 Commit 上去之後需要等 Travis 排程，所以不會立刻更新。
最後附上我目前的完整 .travis.yml

.. code:: yaml

    language: python

    # blacklist
    branches:
      except:
        - master

    cache:
      directories:
        - /home/travis/virtualenv/python2.7/

    python:
    - '2.7'

    install:
    - pip install nikola

    before_script:
    - echo -e "Host github.com\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
    - git config --global user.name "Swind (via Travis-CI)"
    - git config --global user.email "swind@code-life.info"
    - git clone --quiet https://${GH_TOKEN}@github.com/Swind/swind.github.com -b master output > /dev/null

    script:
    - nikola build

    after_success:
    - cd output
    - git add .
    - git commit -m "[ci skip] Update output from Travis CI"
    - git push origin master

    env:
      global:
        secure: bgD+Q7LIpym+pEmrL2+Ro3VfCLhSOGddRDcDXQoA6wAoL7ZIlXeVSgh2Q/iLy1SlEwIVhA+M9wPCDXbhRgWT54/qgDofwutLTmGZCU2ACN3XH4ZVfrqP13FdQ/+qiSRyOBnlmqPlxnDt6gHI7aRpC0fsBLIFiwWYpLDuqhWK2LU=

.. figure:: https://dl.dropboxusercontent.com/u/15537823/Blog/Valkyrie_Profile.jpg

.. raw:: html

	<blockquote>
	<p>Valkyrie Profile</p>
	<cite title="Source Title">Should Deny The Divine Destiny of The Destinies</cite></small>
	</blockquote>

可惜上面的圖少了這句話，當初 VP 剛出的時候我才高中，每天都跑去 Nova 的地下一樓看店家路人玩。
雖然沒有 PS 我還是買了片子跟攻略，一直到了高中畢業，才借到主機這片玩完。
回憶總是比較美好的，所以一直到現在，我還是沒有覺的有比這片好玩的 RPG 。

總之，長馬尾讚 XD

.. _Nikola: http://getnikola.com/
.. _Travis: https://travis-ci.org
.. _GitHub: https://github.com
.. _Travis - Building a Python Project: http://docs.travis-ci.com/user/languages/python/ 
.. _Octopress+Prose+Github+Travis CI: http://rogerz.github.io/blog/2013/02/21/prose-io-github-travis-ci/
.. _Creating an access token for command-line use: https://help.github.com/articles/creating-an-access-token-for-command-line-use
.. _Travis - Encryption keys: http://docs.travis-ci.com/user/encryption-keys/ 
.. _Auto-deploying to My Octopress Blog With Travis-CI: http://www.harimenon.com/blog/2013/01/27/auto-deploying-to-my-octopress-blog/

