.. title: Akka 2.0 筆記(3) - Future 之實在不是我想要拖稿
.. slug: akka-2_0-note-3
.. date: 2012-06-10 17:30
.. tags: Scala,Akka
.. link: 
.. description:

而是我實在是不擅長寫作，過去作文只有 20 分真的不是拿假的。
前一篇的 **開始使用 Future** 只有介紹一些基本的使用方式而已，然後依照前面的程式碼是絕對跑不起來的 Orz ...

.. TEASER_END

原因在於 Future 需要一個 Execution Contexts 類似 Java 裡面的 Executor，也就是 Thread Pool。

Execution Contexts
------------------------------------

因為 Future 除了跟 Actor 配合使用之外，也可以直接使用。

例如前面與 Actor 配合使用的例子scala

.. code-block:: scala

    val list = List[String]("Tony","Lion","Teddy","Brain","Jess","Kay","Michael")
    val listOfFutures = list map {
       name =>
          (countActor ? name).mapTo[Int]
    }
    val futureList = Future.sequence(listOfFutures)


其實可以改成這樣:

.. code-block:: scala

    val futureList = Future.sequence(list.map(name => Future(name.length)))

因此在使用 Future 之前必須要設定 ExecutionContext 讓 Future 可用來執行工作。

P.S
    這樣的結果也是會回傳一個 Future[List[Int]]。
    很明顯的這樣的作法簡單很多，因為少建立 Actor 也不需要再將 Actor 所回傳的值轉型（就是前面看到的 mapTo[Int]）。
    我是還蠻喜歡這樣做的，不過由於後面要使用到 Router 所以就沒有直接使用 Future 了。

設定 Execution Context
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

雖然文件中有提到如果 scope 內存在 ActorSystem ，他會自己使用 ActorSystem 的 default dispatcher，
不過很可惜的這個功能我沒有測試出來，不然就是我誤解了這段話的意思。
如果有朝一日我突然開悟想通了再來補上這一段。

而在 Akka 裡面所有的 Dispatcher 都是繼承自 ExecutionContext 
所以要設定這個也不太花時間。

如果要使用 Future 的是一個 Actor，那麼只要直接設定 Actor 的 dispatcher 就可以了。

.. code-block:: scala

    class MyActor extends Actor{
    implicit val defaultDispatch = context.dispatcher
    .
    .
    .
    }

至於為什麼要使用 implicit 呢？因為 Future 的 Function 中 dispatcher 都是用 implicit 的型態，
所以如果沒有傳參數給他的時候，他會自動在這個 scope 裡面找尋是否有宣告成 implicit 的 ExecutionContext 變數。

如果使用 Future 的地方不是在 Actor 內，或不想使用 Actor 的 dispatch。那麼可以自己建立 Java 的 Executor ，並且將他轉成 ExecutionContext。

.. code-block:: scala

    val es = Executors.newFixedThreadPool(threadSize)             //Java Executor
    implicit val ec = ExecutionContext.fromExecutorService(es)

這樣在使用 Future 的時候他就會自動去找 ExecutionContext 來執行。

如果 Future 執行的過程中有發生 Exception
-----------------------------------------------

Future 用 **recover** 與 **recoverWith** 來處理 Future 執行過程中的所丟出的 Exception。

使用方式非常的簡單，寫起來也蠻漂亮的，只要在建立 Future 時使用 recover 或 recoverWith 並且加入要處理的例外就好了。

例如

.. code-block:: scala

    Future(parsePage(pageNumber)) recover { case e:Exception => List[Post]() }
    
上面表示當我執行 parsePage 時如果發生任何 Excetpion 那麼就回傳一個空的 list。

而 recover 與 recoverWith 的差別就在於，recoverWith 內的 Function 要回傳的型態是 Future[Int] 而 recover 則是 Int。

晚點整理一個使用 Future 的完整例子好了，現在寫的東西真的太不入流了 XDDDDDD 
我很難把程式碼直接貼上來當範例。