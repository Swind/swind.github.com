.. title: Akka 2.0 筆記(4) - Dataflow 請不要期待這篇我會提到什麼
.. slug: akka-2_0-note-4
.. date: 2012-06-12 21:55
.. tags: Scala,Akka
.. link: 
.. description:

因為我也看不懂 Orz ...
================================

這整篇的原因起於，洗澡的時候想到的，
Akka 的 Future 到底有沒有存在的意義呢？

因為當一個 Actor 用 reply 或者 sender 回傳結果的時候，其實可以再 receive 不同的型態的 Message 就好了。
而且這樣還不會因為 Await 而需要 block thread，減少 dead lock 的發生。
我之前就幹過很蠢的事情，因為 Actor 把 Thread Pool 裡面的 Thread 用光了，所以他底下幫他工作的 Actor 就取不到 Thread 可以執行，
因此全部 Timeout Orz ...

目前唯一能想到的用法就是在發出訊息的地方不是一個 Actor 的時候，這時候就沒有 receive 可以接收結果了。
所以需要使用 Future 來等待結果，也就是 Future 應該只被用在需要等待的時候，如果是在一個 Actor 被執行的時候使用 Future。
很容易浪費掉一個 Thread，應該是要另外開一個新的 case class 來專門處理結果才是。

但是這邊又出現了一個更複雜的問題了，因為我將工作分給不同的 Actor 去執行，必須要等到所有 Actor 都執行完畢，整合所有的結果。
那麼如果是用 receive 我要怎樣才能知道我所有的工作都已經完成了呢？

簡單來說，我就是需要實做一個  `Scatter-Gather`_ ，才會莫名其妙的看到 Akka 的 Dataflow。

Akka Dataflow
-------------------------------

起因在於我用 Scatter-Gather 與 Akka 搜尋文章的時候找到這篇 `Scatter-Gather with Akka Dataflow`_ 。
一看裡面的 Code 真是驚為天人，我一行都看不懂 Orz ...

然後 Akka  `Dataflow Concurrency`_ 的內容又非常的精簡 Orz，而且他又用到了 `Scala Continuations`_ ，
這東西的文件也非常的 Orz，連 Programming in Scala 2nd 都沒有提到這個東西，讓我完全放棄這個東西 Orz ..

但是找到都找到了，我想實做個標本出來也不錯，所以才會有這篇東西。

下面這個版本改自上面所提到的 `Scatter-Gather with Akka Dataflow`_  

因為那個 Blog 的範例我估計來自於 Akka 1.3.1 之前的版本，所以我做了一些更動才能在 Akka 2.0 上面跑。

很幸運或者是很不幸的是 Dataflow 的部份則是從 Akka 1.x 就沒有變動過了，所以基本邏輯都沒有改變。

.. code-block:: scala

    import scala.util.continuations._
    
    import akka.actor.Actor
    import akka.actor.ActorRef
    import akka.actor.ActorSystem
    import akka.actor.Props
    import akka.dispatch.Future.flow
    import akka.dispatch._
    import akka.pattern.ask
    import akka.util.duration._
    import akka.util.Timeout
    
    object ScatterGatherDataFlow {
      def main(args: Array[String]) {
        implicit val timeout = Timeout(10 seconds)
        val system = ActorSystem()
        val recipients = (1 to 5).map(i => system.actorOf(Props(new Recipient(i))))
        val aggregator = system.actorOf(Props(new Aggregator(recipients)))
        val results1 = Await.result(aggregator ? Message("Hello"), timeout.duration)
        val results2 = Await.result(aggregator ? Message("World"), timeout.duration)
    
        results1.asInstanceOf[Future[String]] map { res =>
          println("Result: %s" format (res))
        }
        results2.asInstanceOf[Future[String]] map { res =>
          println("Result: %s" format (res))
        }
    
        system.awaitTermination
      }
    }
    
    class Aggregator(recipients: Iterable[ActorRef]) extends Actor {
    
      implicit val defaultDispatcher = context.dispatcher
      implicit val timeout = Timeout(10 seconds)
    
      def receive = {
        case msg @ Message(text) =>
          println("Started processing message `%s`" format (text))
    
          val result = Promise[String]()
          val promises = List.fill(recipients.size)(Promise[String]())
    
          recipients.zip(promises).map {
            case (recipient, promise) =>
              (recipient ? msg).mapTo[String] map { result: String =>
                println("Binding recipient's response: %s" format (result))
                flow {
                  promise << result
                }
              }
          }
    
          flow {
              def gather(promises: List[Future[String]], result: String = ""): String @cps[Future[Any]] =
                promises match {
                  case head :: tail => gather(tail, head() + result)
                  case Nil          => result
                }
    
            println("Binding result...")
            result << gather(promises)
          }
    
          sender ! result
      }
    
    }
    
    class Recipient(id: Int) extends Actor {
    
      def receive = {
        case Message(msg) =>
          Thread.sleep(1000)
          sender ! ("%s, [%s]! ".format(msg, id))
      }
    
    }
    
    case class Message(text: String)

附上執行結果

.. code-block

    Started processing message `Hello`
    Started processing message `World`
    Binding result...
    Binding result...
    Binding recipient's response: Hello, [1]! 
    Binding recipient's response: Hello, [2]! 
    Binding recipient's response: Hello, [3]! 
    Binding recipient's response: Hello, [4]! 
    Binding recipient's response: Hello, [5]! 
    Result: Hello, [5]! Hello, [4]! Hello, [3]! Hello, [2]! Hello, [1]! 
    Binding recipient's response: World, [2]! 
    Binding recipient's response: World, [1]! 
    Binding recipient's response: World, [3]! 
    Binding recipient's response: World, [4]! 
    Binding recipient's response: World, [5]! 
    Result: World, [5]! World, [4]! World, [3]! World, [2]! World, [1]! 

我目前只有做到將這段程式碼修改到可以動而已，實際整個運作流程與大概還不是完全了解。

一方面是因為有關於 Dataflow 的資料太少，二來是 Scala Continuations 的資料也很少 Orz 。
並且這個功能我覺得不太好 Debug ，閱讀上也有點不習慣，因此不會採用這個方式。
或許有一天我了解了 Dataflow 的好的時候我會在回頭把這程式碼的說明補齊。

Eclipse 與 Sbt 的設定
-----------------------------------

上面這段程式碼由於有用到 `Scala Continuations`_  所以 Eclipse 或者 sbt 需要作一些設定，讓他可以使用 scala 的 continuations plugin。

Eclipse 的部份需要在 Compiler Standard 的設定中 p 的部份增加 **continuations:enable**

.. image:: https://dl.dropbox.com/u/15537823/Blog/2012-06-12-akka-dataflow/ScalaEclipse_continuations.png

在 Compiler Advanced 的設定中 Xplugin 增加 **lib\continuations.jar**

.. image:: https://dl.dropbox.com/u/15537823/Blog/2012-06-12-akka-dataflow/ScalaEclipse_continuations_2.png

如果是 sbt 的話只要增加下面的內容到 **build.sbt** 裡面即可。

.. code-block

    autoCompilerPlugins := true
    
    libraryDependencies <+= scalaVersion { v => compilerPlugin("org.scala-lang.plugins" % "continuations" % "2.9.1") }
    
    scalacOptions += "-P:continuations:enable"

.. _Scatter-Gather: http://www.eaipatterns.com/BroadcastAggregate.html
.. _Scatter-Gather with Akka Dataflow: http://blog.vasilrem.com/scatter-gather-with-akka-dataflow
.. _Dataflow Concurrency: http://doc.akka.io/docs/akka/2.0/scala/dataflow.html
.. _Scala Continuations: http://www.scala-lang.org/node/2096