.. title: Golang - Goroutine 筆記 (1)
.. slug: goroutine-note-1
.. date: 2012-12-09 18:00
.. tags: Go
.. link: 
.. description: 

會想到用這個是由於 EPUBConverter Go 的版本雖然運作上已經比 Scala 版本快上不少了。
但我發現在執行的時候 CPU 使用率一直都只有 25 %，所以才想試試看使用 goroutine 看能不能讓他轉換更快一點。

依照文件所說，在執行 IO 相關動作的時候，會將 CPU 讓給 goroutine 來使用，那麼對於 EPUBConverter 的效能應該會有顯著的提昇。

雖然有人說 goroutine 跟 Scala 的 actor 有點像，但是 actor 的模型就很具體跟明確。
而 goroutine 則是提供一個方便快速的 thread 操作方式。

.. TEASER_END

goroutine 的使用方式
-----------------------------------------

goroutine 的使用非常簡單，只要在呼叫 function 的時候在前面加上 go 就可以了。

	go func()

至於要跟 goroutine 的溝通則是使用 channel。

channel 目前我使用的感覺很像是 java 中的 blocking queue。
他的宣告方式只要在一般宣告的時候加個 **chan** 的關鍵字就可以了。

.. code-block:: go

	var count chan int

這樣就可以透過 <- 賦值與取值

.. code-block:: go
	
	//將 i assign 給 count
	count <- i
	//從 count 取出值
	value := <- count

而要決定 channel 可以放的數量則是可以透過 

.. code-block:: go

	count := make(chan int, 4)

的方式來決定。

以上面的程式碼為例子，當 count 裡面沒有東西時，使用 <- count 會 block 住直到有值寫入。
而使用 count <- 的時候當裡面的值超過 4 個的時候，則會 block 住無法寫入，直到有值被取出。

goroutine 的設定
------------------------------------------

雖然後來使用了 goroutine 讓每一個 EPUB 電子書的轉換都可以平行處理，
但是我發現 CPU 使用率卻依然只有 25 %。

後來才發現是還要再設定 GOMAXPROCS 來讓 Go 知道要用幾個核心。

.. code-block:: go

	//透過 runtime.NumCPU() 取得 CPU 核心數
	runtime.GOMAXPROCS(runtime.NumCPU())

這樣就終於看得到 CPU 使用率到 100% 了 ...

Golang 的 benchmark 測試
-------------------------------------------

`testing package`_

Go 另外一點讓我還蠻喜歡的就是，他的設計非常的實用。
雖然還沒有很完整，但是一般開發常用的東西他都會幫你想到。
例如寫在 test case 內的 benchmark。

.. code-block:: go

	package main

	import (
		"runtime"
		"testing"
	)

	func BenchmarkConvertCore4(b *testing.B) {
		// 4 代表使用的核心數
		executeConvert("./testdata", 4)
	}

測試結果

使用四核心::

	BenchmarkConvertCore4	       1	9775559100 ns/op
	ok  	epubconverter	10.282s

使用單核::

	BenchmarkConvertCore1	       1	33146895900 ns/op
	ok  	epubconverter	33.673s

.. _testing package: http://golang.org/pkg/testing/
.. _EPUBConverter 0.0.3: https://dl.dropbox.com/u/15537823/EPUBConverter_0.0.3.7z
	