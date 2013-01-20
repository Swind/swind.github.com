.. title: Akka 2.0 筆記(6) - Dispatcher 的設定
.. slug: akka-2_0-note-6
.. date: 2012-06-17 20:19
.. tags: Scala,Akka
.. link: 
.. description: 

今天早上先簡單了依照昨天畫得架構圖做了一個雛型，整個跑起來功能是以了，但是流程整個悲劇 Orz ... 
先是 Exception Handling 做的不夠好，結果就真的是 "Let it Crash" 一直死一直死一直死，好像在玩 D3 的煉獄模式一樣。

再來是我沒有去設定 Dispatcher 所以 Thread 用了十幾個。如果我沒有看錯的話，依照 Akka `reference.conf`_ 的設定，
Default Dispatcher 使用 fork-join-executor，他的預設設定如下

.. code-block:: scala

    fork-join-executor {
      # Min number of threads to cap factor-based parallelism number to
      parallelism-min = 8
    
      # Parallelism (threads) ... ceil(available processors * factor)
      parallelism-factor = 3.0
    
      # Max number of threads to cap factor-based parallelism number to
      parallelism-max = 64
    }

很簡單的 parallelism-min 與 parallelism-max 的值代表著 Thread 的上下限，那麼 parallelism_factor 呢？

他同樣也是限制 Thread 的上限，上限的計算方式為 

      parallelism-factor * 你的處理器核心數

所以我的四核心桌機使用這個 Dispatcher 最多會開到 12 個 Thread Orz ...

難怪我的 Thread 會滿天飛了

這邊我要先搞定我之前一直逃避的 Akka 設定檔，之前會逃避是因為我用 Eclipse 的 ScalaIDE 加上 sbt 來開發。
不知道為什麼之前設定檔在 build 的時候一直不會自動放到編譯完成的目錄下面，所以就整個怒完全不想搞設定檔這種東西。
那時候是偷懶直接寫在程式碼裡面。

.. code-block:: scala

     val config = """
        my-dispatcher {
    	  # Dispatcher is the name of the event-based dispatcher
    	  type = Dispatcher
    	  # What kind of ExecutionService to use
    	  executor = "fork-join-executor"
    	  # Configuration for the fork join pool
    	  fork-join-executor {
    	    # Min number of threads to cap factor-based parallelism number to
    	    parallelism-min = 2
    	    # Parallelism (threads) ... ceil(available processors * factor)
    	    parallelism-factor = 2.0
    	    # Max number of threads to cap factor-based parallelism number to
    	    parallelism-max = 10
    	  }
    	  # Throughput defines the maximum number of messages to be
    	  # processed per actor before the thread jumps to the next actor.
    	  # Set to 1 for as fair as possible.
    	  throughput = 1
    	}
        """
     val system = ActorSystem("MySystem",ConfigFactory.load(customConf))

上面把設定檔內容直接儲存成字串，然後用 ConfigFactory.load 就可以產生設定檔了，不過一直放在程式碼裡面也不是一件健康的事情。
所以還是花了一點時間處理之前在 Eclipse 上面遇到的問題。

不過俗話說得好，越難找的 Bug 越蠢 ... 我又再次驗證了這個道理 Orz 

如果你是使用 sbt 上面的 sbt-eclipse 來建立 Eclipse Project，由於 plugin 不貼心或者很貼心的設計。
他在 Eclipse 裡面並不會幫你建立 resources 的 source folder，
所以請手動在 src/main 下面建立一個 resources 的 source folder。

接著是笨點 Orz，請進入 Project 的設定畫面，然後選擇 Java Build Path 的內 Source 的 Tab。
請將 **src/main/resources** 的 **Output folder** 的路徑設定的跟 src/main/scala 一樣。

.. image:: https://dl.dropbox.com/u/15537823/Blog/Akka%202.0%20%E7%AD%86%E8%A8%98%286%29%20-%20Dispatcher%20%E7%9A%84%E8%A8%AD%E5%AE%9A/Eclipse_Setting.png

之前都沒有部屬到我想要得地方就是因為他是放到 default output folder 也就是 bin 底下 Orz。

之後就可以參考 Configure_ 與 Dispatcher_ 的內容去設定了。

這樣就解決了我之前一直懶得找得問題，再來應該就會詳細設定一下 Scatter-Gather 所要用的 Dispatcher 了。

.. _reference.conf: https://github.com/akka/akka/blob/master/akka-actor/src/main/resources/reference.conf
.. _Configure: http://doc.akka.io/docs/akka/2.0.1/general/configuration.html
.. _Dispatcher: http://doc.akka.io/docs/akka/2.0.2/scala/dispatchers.html