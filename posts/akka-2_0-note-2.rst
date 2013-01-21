.. title: Akka 2.0 筆記 (2) - 開始使用 Future
.. slug: akka-2_0-note-2
.. date: 2012-06-08 12:18
.. tags: Scala,Akka
.. link: 
.. description:

以前再使用 Java Thread 的時候最困擾我的就是 Thread 之間的溝通。
要怎樣讓一個 Thread 去等待另外一個 Thread？要怎樣才能讓工作分配的平均？
最後是靠 BlockingQueue 才讓實作變得簡單一點，
不過想當然問題當然一大堆 ... Orz 

所以想來實驗看看 Akka 的 Future 與 Router 到底是如何使用，與可以做到什麼事情。

.. TEASER_END

Future
------------------------

請搭配 Akka 官網 Future_ 服用。

看 Future 一定要看一下文件第一句話

	In Akka, a Future is a data structure used to retrieve the result of some concurrent operation.

是的，他只是一個 Data Structure 而已，**Future 最主要的功能就是可以透過他取得 Actor 回傳的 message**
也可以將他想成他代表我們未來會取得的回傳值，我想這也就是它叫做 Future 的原因了。

.. code-block:: scala

	implicit val timeout = Timeout(5 seconds)
	val future = actor ? msg // enabled by the "ask" import
	val result = Await.result(future, timeout.duration).asInstanceOf[String]

偷一下官網的範例，我們一行一行看上面的程式碼。

一開始的 

.. code-block:: scala

	implicit val timeout = Timeout(5 seconds)

就先忽略 implicit 吧，因為這邊還沒有用到 implicit 的特性。

現在只要知道他建立了一個 Timeout 物件，內容是 5 seconds，這個在之後設定 wait timeout 的時候會用到‧
為了避免無止盡的等待，在 Actor 裡面只要有關等待的操作都要給一個 Timeout 時間。

.. code-block:: scala

	val future = actor ? msg // enabled by the "ask" import

這裡發一個 msg 給 actor ，特別需要注意的地方是這裡不是使用* ! *而是* ? *。
要使用這個 ? 需要 *import akka.pattern.ask*，跟 ! 不一樣的地方在於他會回傳一個 Future[Any] 物件

.. code-block:: scala

	val result = Await.result(future,timeout.duration).asInstanceOf[String]

最後使用 Await 來透過 Future 取得 Actor 的回傳值，並且設定一個 timeout 時間，如果時間到 Actor 沒有回傳任何東西的話，就會丟出一個 Timeout Exception。

如果要等待很多個 Actor 回傳值呢？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果要將工作分派給多個 Actor 進行運作，那麼最直覺的想法就是

.. code-block:: scala

	val f1 = actor1 ? msg1
	val f2 = actor2 ? msg2
	 
	val a = Await.result(f1, 1 second).asInstanceOf[Int]
	val b = Await.result(f2, 1 second).asInstanceOf[Int]
	 
	val f3 = actor3 ? (a+b)
	 
	val result = Await.result(f3, 1 second).asInstanceOf[Int]

我先將工作發給 actor1 等他回傳，然後再發給 actor2 等待回傳，最後在將回傳結果發給 actor3。

為了要取得 a、b 而使用了 Await 兩次，這樣的作法非常沒有效率，因此有了 sequence 與 traverse 這兩個 function。

舉例來說，如果我有一個 

.. code-block:: scala

	List[String]("Tony","Lion","Teddy","Brain","Jess","Kay","Michael")

並且想要這個 List 內所有字串的長度總和。

.. code-block:: scala

	val list = List[String]("Tony","Lion","Teddy","Brain","Jess","Kay","Michael")
	val listOfFutures = list map {
	   name =>
	      (countActor ? name).mapTo[Int]
	}

在這邊我們得到了一個型態為 List[Future[Int]] 的 listOfFutures，再來就可以使用 sequence 將其轉成 Future[List[Int]]
最後就可以使用 Await 來取得所有字串長度了，並且統計字數了。

.. code-block:: scala

	val futureList = Future.sequence(listOfFutures)
	val lengthList = Await.result(futureList,1 second)

而 sequence 與 traverse 兩個不一樣的地方在於說

sequence 接受一個 List[Future[A]] 轉成 Future[List[A]]。

.. code-block:: scala

	val futureList:Future[List[Int]] = Future.sequence((1 to 100).toList.map(x=>Future(x+1)))

而 traverse 則是接受一個 List[B] 與一個 function (B=>Future[A]) 最後也是產生 Future[List[A]]

.. code-block:: scala

	val futureList:Future[List[Int]] = Future.traverse((1 to 100).toList)(x => Future(x+1))

用 traverse 的好處在於少產生了中間過程的 List[Future[Int]] (至少我們的程式碼看不到，背後實際上有沒有我就不確定了，不過文件是這樣說的)，而直接產生了 Future[List[Int]]。

寫了這麼久，才寫這麼一點點 Orz ... 而且還完全沒有進入重點。

.. _Future: http://doc.akka.io/docs/akka/2.0/scala/futures.html