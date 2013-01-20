.. title: 用 Spray 建立一個簡單的 RESTful API Server
.. slug: build-restful-api-server-by-spray
.. date: 2012-11-06 20:11
.. tags: Scala,Akka
.. link: 
.. description: 

Spray
=========================

Spray_

最初是想要建立一個提供查詢服務的 RESTful API Server，可以讓 Client 的應用程式透過 RESTful API 查詢一些資料。
但是看來看去，用 Play2、Jersey、RESTEasy 等都蠻麻煩的，而且還需要使用 Web container（像 Tomcat、Jetty 等）。
所以才找到這個 Spray，而且又是用 Scala 開發的，實在沒有理由不試試看阿。 XD

若你連到官網看會發現他分成很多模組，老實說我也沒有詳細研究每個模組的功能。
因為 `spray-can`_ 的範例看起來最簡單並且也符合我的需求，所以就直接用他了。

想瞭解更詳細內容的可以參考 `spray-can 範例文件`_ 與 `Running Standalone Web Applications on Cloud Foundry`_ 

環境準備
=====================

`spray-can 的範例`_ 已經有建立好一個簡單的 HttpServer 了，
所以我們先將 Spray 從 Github 上 Clone 下來。

.. code-block:: bash

    git clone git://github.com/spray/spray.git

然後複製出 spray/examples/spray-can/simple-http-server。
接著在 simple-http-server 底下，建立 build.sbt 讓 SBT 去 include 所需要的 Library。

順便設定 `package-dist`_  這個 sbt plugin 所需要的資料。
package-dist 可以協助我們將 Project 所需要的 Library 與程式本身全部打包成一個 zip 檔。
但是這個 plugin 只能使用在 sbt 0.11.x，目前我還沒有看到可以在 0.12.x 的版本。
如果你不想用的話可以拿掉 Start 到 End 中間的內容。

而最下面的 

  EclipseKeys.createSrc := EclipseCreateSrc.Default + EclipsCreateSrc.Resource

則是針對 sbteclipse 做的設定，讓 plugin 在產生 eclipse project 的時候可以將 src/main/resources 也加入 source folder。

.. code-block:: scala

    //Start - For package-dist
    import com.twitter.sbt._
     
    seq(PackageDist.newSettings: _*)

    seq(GitProject.gitSettings: _*)

    packageDistZipName := "DictionaryServer.zip"
    //End - For package-dist

    organization := "cc.spray"
     
    name := "DictionaryServer"

    mainClass in (Compile, packageBin) := Some("cc.spray.example.Main")
     
    version := "0.1.0-SNAPSHOT"
     
    scalaVersion := "2.9.2"

    resolvers ++= Seq(
    "Typesafe repo" at "http://repo.typesafe.com/typesafe/releases/",
    "spray repo" at "http://repo.spray.cc/"
    )

    libraryDependencies ++= Seq(
        "com.typesafe.akka" % "akka-actor" % "2.0.3",
        "cc.spray" % "spray-server" % "1.0-M2",
        "cc.spray" % "spray-can" % "1.0-M2",
        "org.slf4j" % "slf4j-api" % "1.6.6",
        "ch.qos.logback" % "logback-classic" % "1.0.7"
    )

    //sbteclipse setting
    EclipseKeys.createSrc := EclipseCreateSrc.Default + EclipseCreateSrc.Resource

這邊注意一下 Spray 的版本問題，現在最新的 Example 程式碼是針對最新版本的 spray，但他們還沒有正式 release，
並且他們還換了 package name 等，所以我都是使用 1.0-M2 的版本，當然 Library 也是使用 1.0-M2。

到這邊，我們的環境就準備的差不多了。接下來，只要執行範例程式碼就可以有一個可以提供 Web Service 的 Http Server 了。

Start Http Server 
====================================

如果你對於 Akka 的用法有一點概念的話，那麼 Spray 對你來說應該是非常好上手的。

Main.scala 啟動一個 IoWorker Actor 負責 low-level 的 network I/O，
接下來將 IoWorker Actor 以及 Service Actor 傳入 HttpServer Actor並啟動 Http Server。
之後透過發送一個 HttpServer.Bind 的 Message 給 Server 設定 IP 與 Port。
這樣就完成了 Server 的啟動與設定了。

.. code-block:: scala

    // we need an ActorSystem to host our application in
    val system = ActorSystem("SimpleHttpServer")

    // the handler actor replies to incoming HttpRequests
    val handler = system.actorOf(Props[TestService])

    // every spray-can HttpServer (and HttpClient) needs an IoWorker for low-level network IO
    // (but several servers and/or clients can share one)
    val ioWorker = new IoWorker(system).start()

    // create and start the spray-can HttpServer, telling it that we want requests to be
    // handled by our singleton handler
    val server = system.actorOf(
    props = Props(new HttpServer(ioWorker, MessageHandlerDispatch.SingletonHandler(handler))),
    name = "http-server"
    )

    // a running HttpServer can be bound, unbound and rebound
    // initially to need to tell it where to bind to
    server ! HttpServer.Bind("localhost", 8080)

    // finally we drop the main thread but hook the shutdown of
    // our IoWorker into the shutdown of the applications ActorSystem
    system.registerOnTermination {
    ioWorker.stop()
    }

提供 Service 
=============================

處理 Http Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

提供 Service 的 Actor 會接收到 HttpRequest 的 Message，HttpRequest 的內容如下。

.. code-block:: scala

    case class HttpRequest(
                method   : HttpMethod = HttpMethods.GET,
                uri      : String = "/",
                headers  : List[HttpHeader] = Nil,
                content  : Option[HttpContent] = None,
                protocol : HttpProtocol = `HTTP/1.1`)

然後就可以利用 match 來處理各種不同的 HttpRequest，
例如有連線到 http://localhost:8080/ 的 GET 請求，
就會對應到第一個 HttpRequest "/"。

.. code-block:: scala

    protected def receive = {

        case HttpRequest(GET, "/", _, _, _) =>
            sender ! index

        case HttpRequest(GET, "/ping", _, _, _) =>
            sender ! response("PONG!")

        case HttpRequest(GET, "/stats", _, _, _) =>
            val client = sender
            context.actorFor("../http-server").ask(HttpServer.GetStats)(1.second).onSuccess {

        case x: HttpServer.Stats => client ! statsPresentation(x)
        }
    .
    .
    .
    }

建立 Http Response
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Response 的內容也非常好設定，以剛剛送回 sender 的 index 為例子，
Spray 使用一個 HttpResonse 的 case class 來代表 Response 的內容。
只要設定好 headers、body 與 status 就可以傳送回去 Sender 了。

.. code-block:: scala

    lazy val index = HttpResponse(
    headers = List(HttpHeader("Content-Type", "text/html")),
    body =
      <html>
        <body>
          <h1>Say hello to <i>spray-can</i>!</h1>
          <p>Defined resources:</p>
          <ul>
            <li><a href="/ping">/ping</a></li>
            <li><a href="/search">/search</a></li>
            <li><a href="/stats">/stats</a></li>
            <li><a href="/crash">/crash</a></li>
            <li><a href="/timeout">/timeout</a></li>
            <li><a href="/timeout/timeout">/timeout/timeout</a></li>
            <li><a href="/stop">/stop</a></li>
          </ul>
        </body>
      </html>.toString.getBytes("ISO-8859-1"),
    status = 200)

.. _Spray: http://spray.cc/
.. _spray-can: http://spray.cc/documentation/spray-can/
.. _spray-can 的範例: https://github.com/spray/spray/tree/release-1.0-M2/examples/spray-can/
.. _spray-can 範例文件: https://github.com/spray/spray/tree/release-1.0-M2/examples/spray-can/simple-http-server/
.. _spray github: https://github.com/spray/spray
.. _Running Standalone Web Applications on Cloud Foundry: http://blog.cloudfoundry.com/2012/05/11/running-standalone-web-applications-on-cloud-foundry/
.. _package-dist: https://github.com/twitter/sbt-package-dist