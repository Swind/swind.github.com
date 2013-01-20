.. title: Akka 2.0 筆記(5) - Scatter Gather
.. slug: akka-note-5-aggregator
.. date: 2012-06-15 08:58
.. tags: Scala,Akka
.. link: 
.. description: 

繼上次完全看不懂的 Dataflow 之後，我還是想要一個 Akka 的 Scatter Gather 實作方式，因此想試著自己實做看看。

下圖是 `Enterprise Integration Patterns`_  書中的 Scatter-Gather 概念圖。

.. image:: http://www.eaipatterns.com/img/BroadcastAggregate.gif

從圖中可以發現，中間對於 Vendor A、B 與 C 進行 Broadcast 的行為跟 Akka 內的 Router_ 很像，
所以當我正在思考 Router 是否合適作為這種用途的時候，我想到了 `Let it Crash`_ 裡面 `Watch the Routees`_ 這篇文章。

這篇文章裡面他紀錄了在 Akka mailinglist 中回答別人問題時所寫的範例程式。

整個 use case 是要執行一個 Job，這個 Job 由許多 Tasks 組成，在執行的過程中

1. 會將這些 Tasks 透過 router 分派給 worker actors 執行。
2. 最後收集所有 worker 的執行結果，並且將他們合併之後回傳。
3. 除此之外， worker 在執行過程中發生錯誤時，應該要進行 retry，在進行幾次 retry 後依然有錯誤時，Worker 就會停止，並且中止整個 Job。

Master Actor 
--------------------------------

.. code-block:: scala

    /*
     *  Master Actor，負責整個工作運作流程的 Actor。
     *  接收 Job、分派 Task 以及當發現 Worker Terminal 或者是全部 Task 都執行完畢之後。
     *  將結果整理好之後回傳 
     */
    
    class Master extends Actor {
      var results = Map[Int, Int]()
      var replyTo: ActorRef = _
    
    /*
     *  Router 這邊的 Router 種類是使用 RoundRobinRouter，
     *  他會輪流將 Message 傳給底下的 Worker。
     *  例如：若他底下有 5 個 Worker，那麼就會
     *
     *     Worker 1
     *     Worker 2
     *     ...
     *     Worker 5
     *     Worker 1
     *
     *  這樣的順序傳遞。
     */
    
      val router = context.actorOf(Props(new Worker).
        withRouter(RoundRobinRouter(5,
          supervisorStrategy = OneForOneStrategy(
            maxNrOfRetries = 2) {
              case _: IOException => Restart
            })), name = "router")
    
      router ! CurrentRoutees
    
      def receive = {
        case RouterRoutees(routees) =>
          routees foreach context.watch
    
        case "start" =>
          replyTo = sender
          for (id ← 1 to 10) router ! (id, "22")
    
        case (id: Int, result: Int) =>
          results += (id -> result)
          if (results.size == 10)
            replyTo ! results.values.sum
    
        case Terminated(actor) =>
          replyTo ! -1
          context.stop(self)
      }
    }

Master Actor 的初始化 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

上面的程式碼的註解有稍微介紹一下 Master 與 Router 的功能，這邊就針對程式碼的運作內容作些講解。

首先我們先看到這段程式碼

.. code-block:: scala

    router ! CurrentRoutees

**CurrentRoutess** 這邊困擾了我一段時間，因為我在 Akka 的文件裡面並沒有看到這個物件的說明，且在 API Document 裡面我也沒有看到相關的介紹 Orz ... 

一直到仔細看 receive 裡面的內容之後才發現，這程式碼的功能是發一個 Message 給 Router，Router 在接收到這個訊息之後，
就會將目前 Router 裡面的所有的 Worker Actor（在 Let it Crash 裡面稱他為 Routee） 用 RouterRoutees 這個 case class 包裝回傳。

.. code-block

    case RouterRoutees(routees) =>
         routees foreach context.watch

之後就可以利用 context.watch 將這個 Worker Actor 註冊起來。

這樣當 Worker Actor 因為錯誤太多次而結束的時候，就會接收到一個 Terminated 的 Message，
之後我們就回傳 -1 並且呼叫 context.stop(self) 結束 Master Actor，這也會連帶讓 Worker Actor 結束。

.. code-block

   case Terminated(actor) =>
      replyTo ! -1
      context.stop(self)

Master Actor 的 Scatter 與 Gather 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

說到 Scatter 的部份的話我想應該就要介紹一下我們這邊使用的 Router 了。

Router 其實本身跟我們這邊所實做的 Master Actor 很像，透過 Router 就可以將訊息分派給他底下的 Actor ，他就有點像是仲介商一樣。
你把要做的工作交給他，他會負責找到合適的人將工作分派出去。

Router 分派工作的方式有好幾種，可以到  Router_  的文件看一下，這邊使用的是經典的 RoundRobinRouter。

他會將工作輪流 **forward** 給底下的 Actor，這邊會用到 forward 是因為在 Akka 裡面，當你傳送一個 Message 給某個 Actor 的時候，
其實還會附帶上了 Sender 的資訊，所以如果 Message 的傳送路徑是用一般的傳送方式的話

     Main -> Router -> Worker Actor

那麼 Worker Actor 的 Sender 就會變成 Router，所以 Router 所使用的傳遞方式是 forward，這樣 Worker Actor 接收到 Message 之後就還是會認為 Main 是 Sender。
這樣才能將訊息正確的回傳回去。

Scatter 的部份就這麼簡單解決，但是下一個 Gather 的部份就比較麻煩了。
理由很簡單。因為 Router 只有負責分派的部份，Worker Actor 回傳的對象是一開始發 Message 的 Actor，
所以要另外針對回傳的 Message 作處理。下面的程式碼會在回傳的訊息累積到 10 個之後才將全部一起加總回傳。

.. code-block:: scala

    case (id: Int, result: Int) =>
       results += (id -> result)
       if (results.size == 10)
         replyTo ! results.values.sum

但是很明顯的，這樣還不足夠，如果同時間 Master Actor 接收到多個 Job，那麼要如何分辨 Worker Actor 所回傳的 Result 是哪個 Job 的？

目前想到的解決方式有

1. 簡單來說就是限制讓一個 Master Actor 同時只會有一個 Job 進行，如果有多個 Job 要進行就使用多個 Master Actor，附帶一題Actor 的數量並不代表 Thread 的數量，這部份可以透過 Dispatcher 的設定來控制。
2. 增加 Job ID，讓每個 Job 都有自己的 ID，並且這個 ID 也會加在 Result 裡面，這樣就可以辨識 Result 是哪個 Job的了。

本來我覺得第一個方法是不可行的，因為這樣太浪費資源且每次都要建立 Master Actor 很麻煩。
但是這樣的作法很單純也很好擴充，反正只要增加 Master Actor 就能多處理幾個 Job，要管理資源就等於管理 Master Actor 的數量就好了。

第二種作法會增加整個程式的複雜度，因為需要多一個 Map 來暫存 Result（光這個就有點麻煩了），需要多傳遞 Job ID 的資訊等。但好處應該就是會比較節省記憶體與 CPU 等。

但是在這個 CPU、Memory 不值錢的年代且這個又只是我的玩具的，所以我會選擇第一種方式來實做。

Worker Actor 
--------------------------------

這是在 Let it Crash 裡面 Worker Actor 的程式碼，他接收的 Message 型態很簡單只有一種 (id,s:String)，這是在 Scala 稱之為 Tuple 的物件。
會有兩種是因為這個範例會模擬當 Worker Actor 執行發生 Exception 的情況。

.. code-block:: scala

    class Worker extends Actor {
      def receive = {
        case (id, s: String) if Random.nextInt(4) == 0 =>
          throw new IOException("failed")
        case (id, s: String) =>
          sender ! (id, s.toInt)
      }
    
      override def preRestart(
        reason: Throwable, message: Option[Any]) {
        // retry
        message foreach { self forward _ }
      }
    }

比較有趣的是 preRestart ，這可以跟前面 Master Actor 內的 Router 一起看一下。

.. code-block:: scala

    val router = context.actorOf(Props(new Worker).
        withRouter(
        RoundRobinRouter(5,
          supervisorStrategy = OneForOneStrategy(
            maxNrOfRetries = 2) {
              case _: IOException => Restart
            })), name = "router")

上面的 OneForOneStrategy 是 Akka 的特產，他可以設定 Supervisor（就是該 Actor 的管理者，例如 Worker Actor 的 supervisor 是 Router）

對於他底下 Actor 的應對策略。這邊是設定當 Worker Actor 有丟出 IOException 的時候，所採取的應對是將該 Worker Actor Restart。
但是這個 restart 次數有限制，在 maxNrOfRetries = 2 ，因此 Worker Actor 最多只會 Restart 兩次，兩次之後就會被停止並且傳一個 Terminated 的訊息給 Master Actor。

而 preRestart 裡面做的事情就是就是在 Restart 前將目前的 Message 全部用 forward 的方式傳給自己一次，否則 Restart 後這些 message 就會被清理掉。
這樣之前的工作就會遺失了。

Let it Crash 的 `Watch the Routees`_ 幫助真的很大，加上最近再看 `Scala in Depth`_  對於要做的 Scatter-Gather 比較有一些想法了，
下一篇應該就會說明如何時做的。

下集預告 ?
-----------------------------

.. image:: https://dl.dropbox.com/u/15537823/Blog/Akka%202.0%20%E7%AD%86%E8%A8%98%285%29%20-%20Scatter%20Gather/Photo%2012-6-16%20%E4%B8%8B%E5%8D%8810%2042%2021.png

.. _Enterprise Integration Patterns: http://www.eaipatterns.com/
.. _Router: http://doc.akka.io/docs/akka/2.0/scala/routing.html
.. _Watch the Routees: http://letitcrash.com/post/23532935686/watch-the-routees
.. _Scala In Depth: http://www.amazon.com/Scala-Depth-Joshua-Suereth-D/dp/1935182706
.. _Let it Crash: http://letitcrash.com/