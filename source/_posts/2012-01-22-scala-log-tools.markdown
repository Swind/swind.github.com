---
layout: post
title: "Scala log 工具筆記"
date: 2012-01-22 15:59
comments: true
categories: Scala 
---
之前偷懶都直接使用println來輸出訊息，但是最近在玩[Akka][]，我發現如果沒有使用log tool的話我根本不知道是哪個Actor印訊息的。
只好搜尋一下Scala上面有沒有什麼好用的Log工具。

##slf4s + slf4j + logback##
一般依照以往的習慣基本就是[log4j][] + [slf4j][]，這算是最通用的工具組了。但是Scala有這麼多特異功能，說不定會有更神奇的Log工具，
於是就抱著這樣的心態跟Google大神詢問求籤。最後看到logback這個工具（雖然跟Scala完全沒有關係）但是人都是喜新厭舊 XD，所以
我就從slf4j+log4j轉到slf4j+logback，附帶一提logback是slf4j預設的log工具，因此使用這兩個的組合不需要另外抓Adapter。（ex.slf4j-log4j12-1.6.4.jar之類的）

設定檔的部分目前我看到最不一樣的就是格式了，log4j是採用properities的方式記錄，也就是key=value的格式，而logback則是使用xml。
設定檔一直是讓我懶得使用Log工具的地方，好在網路上真的很多好心人，在[符號記憶-Logback,Log4j設定檔自動產生器][LogConfigGenerator]有介紹了[協助產生設定檔的網站][LogConfigGeneratorTool]
可以讓設定的工作減少很多。下面是我隨便產生的一個範例

{%codeblock lang:xml%}

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

<root><level value="debug"/>
     <appender-ref ref="RootConsoleAppender"/>
   </root>

</configuration>

{%endcodeblock%}

之後我還有多使用[slf4s][]來讓slf4j的使用更有Scala的味道，不過其實也就只是多了一個trait讓你的scala class可以直接繼承，讓你省下一些麻煩
例如：

{%codeblock lang:scala%}
class MyClazz with Logging
{
    logger.debug("Creator")
}
{%endcodeblock%}

最後是額外找到的[打一句log時間不到一納秒！完爆log4j、logback、slf4j][optimize]，但是我沒有用因為還沒有效能上的需求，不過我有稍微看一下程式碼，加上下面留言者給的提示。
在使用logger印出訊息之前，先進行logger.isTraceEnabled、logger.isDebugEnable等的判斷，就可以在編譯的時候進行最佳化。

目前使用的Log工具大概就這樣了，logback如果有玩到什麼好玩的會在上來報告。

[Akka]:http://akka.io/
[log4j]:http://logging.apache.org/log4j/
[slf4j]:http://www.slf4j.org/
[LogConfigGenerator]:http://werdna1222coldcodes.blogspot.com/2011/10/logback-log4j.html
[LogConfigGeneratorTool]:http://wizardforge.org/pc?action=displayFlowchartVersionPublic&id=42
[slf4s]:https://github.com/weiglewilczek/slf4s
[optimize]:http://www.ac.net.blog.163.com/blog/static/1364905620111023304126/
