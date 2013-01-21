.. title: Scala log 工具 - slf4j + logback
.. slug: scala-log-tool
.. date: 2012-01-22 15:59
.. tags: Scala
.. link: 
.. description:

之前偷懶都直接使用 println 來輸出訊息，但是最近在玩 Akka_ ，我發現如果沒有使用 log tool 的話我根本不知道是哪個 Actor 印訊息的。
只好搜尋一下 Scala 上面有沒有什麼好用的 Log 工具。

slf4s + slf4j + logback
-----------------------------------------
一般依照以往的習慣基本就是 log4j_ + slf4j_ ，這算是最通用的工具組了。但是 Scala 有這麼多特異功能，說不定會有更神奇的 Log 工具，
於是就抱著這樣的心態跟 Google 大神詢問求籤。最後看到 logback 這個工具（雖然跟 Scala 完全沒有關係）但是人都是喜新厭舊 XD，所以
我就從 slf4j + log4j 轉到 slf4j + logback，附帶一提 logback 是 slf4j 預設的 log 工具，因此使用這兩個的組合不需要另外抓 Adapter。（ex.slf4j-log4j12-1.6.4.jar之類的）

.. TEASER_END

設定檔的部分目前我看到最不一樣的就是格式了，log4j 是採用 properities 的方式記錄，也就是 key=value 的格式，而 logback 則是使用 xml。
設定檔一直是讓我懶得使用 Log 工具的地方，好在網路上真的很多好心人，在 `符號記憶-Logback,Log4j設定檔自動產生器`_ 有介紹了 協助產生設定檔的網站_ 
可以讓設定的工作減少很多。下面是我隨便產生的一個範例

.. code-block xml

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration debug="true">
      <appender name="RootConsoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
          <level>debug</level>
        </filter>
        <layout class="ch.qos.logback.classic.PatternLayout">
          <pattern>%d{yyyy-MM-dd HH:mm:ss}, %p, %t, %L, %C{1}, %M %m%n</pattern>
        </layout>
      </appender>

       <root>
          <level value="debug"/>
          <appender-ref ref="RootConsoleAppender"/>
       </root>
    </configuration>

之後我還有多使用 slf4s_ 來讓 slf4j 的使用更有 Scala 的味道，不過其實也就只是多了一個 trait 讓你的 scala class 可以直接繼承，讓你省下一些麻煩
例如：

.. code-block:: scala

    class MyClazz with Logging
    {
        logger.debug("Creator")
    }

最後是額外找到的 `打一句log時間不到一納秒！完爆log4j、logback、slf4j`_ ，但是我沒有用因為還沒有效能上的需求，不過我有稍微看一下程式碼，加上下面留言者給的提示。
在使用logger印出訊息之前，先進行 logger.isTraceEnabled、logger.isDebugEnable 等的判斷，就可以在編譯的時候進行最佳化。

目前使用的 Log 工具大概就這樣了，logback 如果有玩到什麼好玩的會在上來報告。

.. _Akka: http://akka.io/
.. _log4j: http://logging.apache.org/log4j/
.. _slf4j: http://www.slf4j.org/
.. _符號記憶-Logback,Log4j設定檔自動產生器: http://werdna1222coldcodes.blogspot.com/2011/10/logback-log4j.html
.. _協助產生設定檔的網站: http://wizardforge.org/pc?action=displayFlowchartVersionPublic&id=42
.. _slf4s: https://github.com/weiglewilczek/slf4s
.. _打一句log時間不到一納秒！完爆log4j、logback、slf4j: http://www.ac.net.blog.163.com/blog/static/1364905620111023304126/