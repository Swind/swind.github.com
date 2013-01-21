.. title: EPUBConverter 0.0.2 - 其實已經重寫了 Orz
.. slug: epubconverter-0_0_2
.. date: 2012-12-08 18:00
.. tags: Go
.. link: 
.. description: 

一開始的 Scala 版本本來是自己寫來自己用的小工具，可是沒想到竟然有人有用還有回報問題。
大受感動之餘（誤），而出現了這個全新感受的 EPUBConverter 0.0.2，絕對不是因為手癢想要試試看 Go。

P.S 2012.12.09 更新 - 修正了鳥 Bug 所以到 0.0.3 了。

`EPUBConverter 0.0.3 下載`_

`EPUBConverter Source Code`_

這個版本比之前好得地方在於，這次使用了 新同文堂_ 的簡繁轉換對照表。
除了單字轉換之外，還多了一個詞彙轉換。

所以轉換品質應該會比之前 Scala 的版本好很多，感謝新同文堂分享對照表。
他的 Bookmarklet 版本也非常好用，基本上我 Go 的轉換邏輯就是從 Bookmarklet 版本移植過來的。

但是由於我直接將對照表轉換成 Golang 裡面的 map 了，所以這次就沒有辦法讓使用者自己改對照表了。

不過有興趣的人，在原始碼的 tongwen_table_ 資料夾底下，我有放了一個簡單的轉換工具。
可以利用那個重新產生 Go 的 map，但是最後還是得重新編譯整個 EPUBConverter 就是了。

.. TEASER_END

延續之前的我的偷懶寫法，這次也是寫死只會轉換 source 資料夾底下的 epub 檔案，輸出到 target 資料夾。
可是這次全部都在記憶體裡面處理，所以不會先解壓縮到磁碟上然後壓縮再刪除了。

::

	/EPUBConverter
		   source
  		   target
  		   epubconverter.exe

以上，如果有什麼問題的話也歡迎回覆，如果我有錢有閒而且沒事做的話 Orz。
會找個時間繼續修正的，雖然目前我也想不到要加啥功能。

Go 的心得
-----------------------------------------------------------------

其實這段才是本文 Orz。
這次寫 Go 我是使用 `Sublime Text 2`_ 來作為我的編輯器。
雖然萬能的 Elcipse 也有支援，但是我現在是能不用就不用，他真的太肥了。

Go 的設計讓我覺得他非常的務實，考慮到現在 Coding 的需求。
例如自己就有包涵 gofmt 這個 format 工具，並且可以使用 go install 來安裝 remote respository 的 package。

至少免去了每次要學一個新語言，就必須東找西找各式各樣工具來進行整合。

Go 的語法紀錄
----------------------------------------------------------------

這一次寫得時候還沒有把 Go 的語法看完，所以有很多都是亂用的，並且把 Go 當 C 在寫 Orz。

for 迴圈
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

寫習慣了 Scala，看到 for 迴圈就很自然的會去找是否有 for-each 的語法。
在 Go 裡面如果要尋訪 slice 或 map 等容器裡面的每一個元素，使用的是 for + range

.. code-block:: go

	for index, element := range slice {
		...
	}

由於 Go 支援多個回傳值，所以 for 後面的 **index** 與 **element** 是會隨著 range 後面所接的對象而改變的。
例如是 map 的話，那麼回傳值就會是 key, value。

函式與變數的名稱
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

大寫開頭的就是可以在其他 package 被呼叫，小寫得就只能在自己的 package 裡面被呼叫。

讀檔與寫檔
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

先來段程式碼

.. code-block:: go

	for {
		if length, err = reader.Read(part); err != nil {
			break
		}

		if length != 0 {
			f.Write(part[:length])
		} else {
			break
		}
	}

在 C、C++、Java、Scala 等語言基本上在 Write 的 function 中，都會有一個參數是用來帶要寫的長度。
可是 Go 沒有 T_T，所以這段程式碼在剛寫好得時候寫了一堆垃圾資料進去。因為每次都把整個 buffer 寫入。

一直到去 stackoverflow 查詢才發現，原來要善用 

	part[:length]

這樣的語法來取出 slice 中特定的某段區域。
更詳細的 slice 內容可以參考 `Go Slices - usage and internals`_ 

剩下的之後有想到再補上，不過 Go 寫起來也蠻愉快的，我想以後用 Scala 跟 Go 的時間應該會一半一半了。

.. _EPUBCOnverter 0.0.3 下載: https://dl.dropbox.com/u/15537823/EPUBConverter_0.0.3.7z
.. _EPUBConverter Source Code: https://github.com/Swind/EPUBConverter-Go
.. _新同文堂: http://tongwen.openfoundry.org/
.. _tongwen_table: https://github.com/Swind/EPUBConverter-Go/tree/master/tongwen_table
.. _Sublime Text 2: http://www.sublimetext.com/2
.. _Go Slices - usage and internals: http://blog.golang.org/2011/01/go-slices-usage-and-internals.html
