.. title: Pelican 轉移到 Nikola
.. slug: pelican-to-nikola
.. date: 2013/01/22 09:11
.. tags: Other
.. link: 
.. description: 20 min

之前雖然從 Octopress 跳到 Pelican 脫離了 Ruby 的環境，也花了一些時間在修改 Theme。
但是還是很難喜歡上 Pelican，最主要的就是他的手冊很難閱讀，而且資訊有點少 Orz。

Nikola 的手冊就比較簡潔清楚，也比較好查詢。
且查詢了一下 Nikola 的事蹟之後發現他太帥了！

`Wiki 尼古拉 特斯拉`_

所以後來就決定跳過來了～

在 Windows 上安裝 Nikola
-------------------------------------------

在 Windows 上面安裝 Nikola 實在是有點悲劇，因為他所使用的 Library 有一些會使用到 C 的程式碼。
因此需要編譯，可是他又會連結到其他 C Library（例如 pillow or pil），所以就卡在安裝相依性套件這一關。 

雖然用 pip 安裝 pil 或者是 pillow 也可以，但是安裝完之後會因為缺少某些 library 而導致像 jpg 等圖片格式不被支援。

最後是找到了 `Unofficial Windows Binaries for Python Extension Packages`_ 下載了 pil 才解決了相依性安裝問題。
這些相依性的 Library 安裝完畢之後，就可以使用 pip 安裝 Nikola 了。

**But ! 就是這個 But !。**

當你開心安裝完畢 Nikola 之後會發現無法使用 nikola 的指令。

因為他放在 Python Scripts 資料夾底下的 Script 檔名是 **nikola**，Windows 不會覺得他是執行檔，所以沒有辦法執行指令。
( 如果有人有知道其他執行方式的話請跟我說 m(_ _)m )

一開始我很天真的覺得改成 nikola.py 就沒有問題了，但是在執行中 import 會出問題，因為 nikola 這個名字重複了 Orz。
所以最後是多寫一個 nikola.bat 

.. code-block:: shell

	C:\Python27\python.exe C:\Python27\Scripts\nikola %1 %2 %3 %4

放在 Scripts 資料夾底下，來解決這個問題。

上面的 Script 因為我找不到 Windows 裡面代表所有傳入參數的符號，
只好就土法煉鋼一個一個輸入，所以最多只能輸入四個參數。

（如果有人有好得解決方式也請教我一下 m(_ _)m )

.. TEASER_END

Nikola 的一些指令
-------------------------------------------------

初始化

	nikola init <<資料夾名稱>>

編譯

	nikola build

啟動測試 Server

	nikola serve 

或
	
	nikola serve --address 0.0.0.0 --port 8080

Nikola 的語法筆記
-------------------------------------------------

這邊紀錄一下 Nikola 裡面才有的語法

Note::

	.. Note:: 這就是 Note 的效果

------------------

.. Note:: 這就是 Note 的效果

------------------

Contents::

	.. contents::

-------------------

Contents 加上一些其他的 Class 屬性::

	.. class:: alert alert-info pull-left

	.. contents::

---------------------

Sildes::

	.. slides:: 
	   
	   https://dl.dropbox.com/u/15537823/Blog/SildeDemo/wallpaper-1427130.jpg
	   https://dl.dropbox.com/u/15537823/Blog/SildeDemo/wallpaper-1427132.jpg
	   https://dl.dropbox.com/u/15537823/Blog/SildeDemo/wallpaper-1427197.jpg

.. slides:: 

	   https://dl.dropbox.com/u/15537823/Blog/SildeDemo/wallpaper-1427130.jpg
	   https://dl.dropbox.com/u/15537823/Blog/SildeDemo/wallpaper-1427132.jpg
	   https://dl.dropbox.com/u/15537823/Blog/SildeDemo/wallpaper-1427197.jpg

----------------------

Youtube::
	
	.. youtube:: gtTyWCjvSfI

----------------------

.. youtube:: gtTyWCjvSfI

.. raw:: html

	<blockquote>
	<p>這並非單純只是流逝的季節，亦是那「永遠的夏季」逝去的年代。夢幻般的 80's </p>
	<small>Orage Road</small><cite title="Source Title">きまぐれオレンジ☆ロード</cite></small>
	</blockquote>

這部動畫真的影響了我整個人生阿，夢幻般的 80 年代 XDDDD 

.. _Unofficial Windows Binaries for Python Extension Packages: http://www.lfd.uci.edu/~gohlke/pythonlibs/
.. _Octopress: http://octopress.org/
.. _Pelican: https://pelican.readthedocs.org/en/3.1.1/
.. _Nikola: http://nikola.ralsina.com.ar/
.. _Wiki 尼古拉 特斯拉: http://zh.wikipedia.org/wiki/%E5%B0%BC%E5%8F%A4%E6%8B%89%C2%B7%E7%89%B9%E6%96%AF%E6%8B%89