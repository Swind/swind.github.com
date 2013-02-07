.. title: Scala 使用 Play 筆記 (1)
.. slug: play-note-1
.. date: 2013/02/02 09:11
.. tags: Other
.. link: 

會想要重拾 Scala 跟 Play 主要是因為看到了這篇文章

`怎麼設計22K?`_

因為覺得這個設計理念真的是太正面了 XDDD 

    所以我要以正面的意念來設計這個網站！

    -- Art Pai 

剛好之前也一直對 Play Framework 很有興趣，所以就大概花了兩個小時把 API 簡單的接上去，順便玩一下 Play Framework 2.0。

`原始碼`_ 

自從開始工作之後，我就放棄了我會把東西記起來這件事情。
所以我決定把使用 Play 實作的過程記錄一下，以方便我日後回憶。

整個實作過程是參考 `Your first Play application`_ 這篇文章的。

P.S 這篇文章寫到一半的時候 Play 2.1 出了 Orz ...

.. TEASER_END

安裝 Play Framework
--------------------------------------------

Play 聽說是學習 Ruby On Rails 風格的 Java/Scala web framework，我從 1.x 的時候就有在關注他了。
也曾經想要推坑給學弟，讓實驗室使用 Play + Scala 來開發，可惜最後因為這兩者都太新還不穩定而作廢。

但是到了現在，我發現他的安裝與使用變得非常容易上手。
安裝 Play 唯一要作得事情就是下載 Play。

Play官網_

`play-2.0.4.zip下載點`_

解壓縮之後，將你解壓縮的目錄位置加入 PATH 就算安裝完畢了。

建立 Play Project
--------------------------------------------

	play new 22k

.. figure:: https://dl.dropbox.com/u/15537823/Blog/Play%E7%AD%86%E8%A8%98/CreateProject.png

會在你所執行的位置下建立一個 22k 的資料夾，其資料夾結構如下 ::

	├── app
	│   ├── controllers
	│   │   └── Application.scala
	│   └── views
	│       ├── index.scala.html
	│       └── main.scala.html
	├── conf
	│   ├── application.conf
	│   └── routes
	├── project
	│   ├── build.properties
	│   ├── Build.scala
	│   └── plugins.sbt
	├── public
	│   ├── images
	│   │   └── favicon.png
	│   ├── javascripts
	│   │   └── jquery-1.7.1.min.js
	│   └── stylesheets
	│       └── main.css
	└── README

使用 Play
-------------------------------------------

在 22k 的資料夾底下輸入

	play

.. figure:: https://dl.dropbox.com/u/15537823/Blog/Play%E7%AD%86%E8%A8%98/StartPlay.png

之後就進入 Play 的命令列模式::

	Welcome to Play 2.0!

	These commands are available:
	-----------------------------
	classpath                  Display the project classpath.
	clean                      Clean all generated files.
	compile                    Compile the current application.
	console                    Launch the interactive Scala console (use :quit to exit).
	dependencies               Display the dependencies summary.
	dist                       Construct standalone application package.
	exit                       Exit the console.
	h2-browser                 Launch the H2 Web browser.
	license                    Display licensing informations.
	package                    Package your application as a JAR.
	play-version               Display the Play version.
	publish                    Publish your application in a remote repository.
	publish-local              Publish your application in the local repository.
	reload                     Reload the current application build file.
	run <port>                 Run the current application in DEV mode.
	test                       Run Junit tests and/or Specs from the command line
	eclipsify                  generate eclipse project file
	idea                       generate Intellij IDEA project file
	sh <command to run>        execute a shell command 
	start <port>               Start the current application in another JVM in PROD mode.
	update                     Update application dependencies.

	Type `help` to get the standard sbt help.

Play 本身也將開發常用的工具都整合在一起了，所以不需要去煩惱 Scala 或 Library 要哪裡下載等問題。
並且也提供了跟 IDE 整合的方式。::

	eclipsify
	idea

會自動產生 eclipse 或 idea 的 Project 設定，讓使用者以直接匯入 IDE 中開發，
但是我在使用的時候發現 eclipse 並無法正確編譯與檢查語法錯誤，因此還是只能靠 play 來進行編譯與偵錯。

	run

可以啟動 Server ( http://localhost:9000 ) 讓你直接觀看執行結果，並且也有提供良好的 error 畫面讓你可以直接在瀏覽器上面看到執行錯誤的地方。

.. figure:: https://dl.dropbox.com/u/15537823/Blog/Play%E7%AD%86%E8%A8%98/RunPlay.png

連接 `揭露22k`_ 的 API
---------------------------------------------

揭露22k 所提供的 API 是 XML 的格式，正好是 Scala 的強項之一，因此分析內容變得非常容易。

.. code-block:: scala

	package models

	import scala.xml.XML
	import java.net.URL
	import scala.xml.Node

	object 22KOpenData {
	  val listDataURL = "http://www.22kopendata.org/api/list_data"

	  def listData(size:Int, pageNum:Int) = {
		val url = new URL(listDataURL+"/"+size+"/"+pageNum)
	    val connection = url.openConnection()
	    val feedXML = XML.load(connection.getInputStream)
	    parseFeedXML(feedXML)
	  }

	  def parseFeedXML(feedXML: Node) = (feedXML \ "job").map(parseJob)

	  def findFirstElement(node: Node)(tagName: String) = (node \ tagName).first.text

	  def parseJob(jobNode: Node) = {
	    val nodeParser = findFirstElement(jobNode)_
	    Job(
	      nodeParser("count"),
	      nodeParser("company_name"),
	      nodeParser("company_location"),
	      nodeParser("job_name"),
	      nodeParser("salary"),
	      nodeParser("note1"),
	      nodeParser("note2"),
	      nodeParser("job_url"),
	      nodeParser("job_url_screenshot"),
	      nodeParser("job_salary_pic"))
	  }
	}

	case class Job(
		count: String,
		company_name: String,
		company_location: String,
		job_name: String,
		salary: String,
		note1: String,
		note2: String,
		job_url: String,
		job_url_screenshort:String,
		job_salary_pic: String
	)

主要就只是將 XML 的內容對應到一個 case class 內而已。

這個我將他歸類在 models 底下，所以目前 app 的資料夾結構長成這樣::

	├── app
	│   ├── controllers
	│   │   └── Application.scala
	│   └── views
	│       ├── index.scala.html
	│       └── main.scala.html
	│   ├── models
	│   │   └── 22KOpenData.scala

套用 `Art Pai`_ 所設計的 CSS Template
---------------------------------------------

至 `Art Pai`_ 的網站下載 `CSS Template`_

將壓縮檔的檔案除了 index.html 之外，全部複製到 22k 資料夾中 public 資料夾底下::

	public
	├── images
	│   ├── background.png
	│   ├── glyphicons-halflings.png
	│   ├── glyphicons-halflings-white.png
	│   └── middleman.png
	├── javascripts
	│   ├── all.js
	│   ├── bootstrap.min.js
	│   └── jquery.min.js
	└── stylesheets
	│   ├── all.css
	│   ├── bootstrap.min.css
	│   ├── bootstrap-responsive.min.css
	│   └── normalize.css

建立 Controller
----------------------------------------------

Play 設定 routes 的方式非常的直覺（遠望 struts 1.x Orz）::

	# Routes
	# This file defines all application routes (Higher priority routes first)
	# ~~~~

	# Home page
	GET     /                           controllers.Application.index

	# Tasks          
	GET		/jobs/:pageNum				controllers.Application.jobs(pageNum: Int)

	# Map static resources from the /public folder to the /assets URL path
	GET     /assets/*file               controllers.Assets.at(path="/public", file)

可以直接將 path 與 function 做對應。

配合 API 的設計方式，所以我直接增加了一個 

	/jobs/:pageNum

的路徑，對應 `揭露22k`_ 的 

	http://www.22kopendata.org/api/list_data/(每頁筆數,最大20)/(目前頁數)

筆數則對應 Art Pai 的設計每頁 9 筆。

.. code-block:: scala

	package controllers

	import play.api._
	import play.api.mvc._
	import play.api.data._
	import play.api.data.Forms._
	import models._

	object Application extends Controller {
	  
	  val rowSize = 3
	  
	  def index = Action {
	    Redirect(routes.Application.jobs(1))
	  }
	  
	  def jobs(pageNum:Int = 1) = Action {
	    val dataList = OpenDataAPI.listData(9,pageNum).toList
		Ok(views.html.index(dataList.grouped(rowSize).toList, dataList.size < 9 ? 1:pageNum + 1 ))
	  }
	  
	}

之後 index 這個 function 再 redirect 至 jobs 這個 function 並且預設值為 page 1。

建立 View
-------------------------------------------------------------

參考 Art Pai 的 index.html 來建立 view。

**view 的 參數**

在 view 的檔案裡面，雖然是 html 的格式，但是我們還是可以在這裡面使用 scala 的語法。
所以要表示這個 view 可以接收兩個參數的話，使用 @ 作為跳脫符號

	@(jobrows: List[List[Job]], nextPageNum: Int)
	@import helper._

對應到 controller 的 

.. code-block:: scala

	Ok(views.html.index(dataList.grouped(rowSize).toList, dataList.size < 9 ? 1:pageNum + 1 ))

可是個人覺得不應該在 view 裡面放太多邏輯，否則會讓 view 的權責太複雜。

**修改 css 的 include 路徑**

.. code-block:: html

	<link href="@routes.Assets.at("stylesheets/bootstrap.min.css")" media="screen" rel="stylesheet" type="text/css" />
	<link href="@routes.Assets.at("stylesheets/all.css")" media="screen" rel="stylesheet" type="text/css" />
	<script src="@routes.Assets.at("javascripts/jquery.min.js")" type="text/javascript"></script>
	<script src="@routes.Assets.at("javascripts/bootstrap.min.js")" type="text/javascript"></script>

其中的

	@routes.Assets.at("stylesheets/bootstrap.min.css")

對應到 route 中預設的::

	# Map static resources from the /public folder to the /assets URL path
	GET     /assets/*file               controllers.Assets.at(path="/public", file)

也就是連結到 public 資料夾中的 stylesheets/bootstrap.min.css。

**使用 for 迴圈**

在接收了 List 型態的 Job 資料之後，就可以使用 for 迴圈來處理，不要手工複製 1, 2 ,3 ... 的貼上 Orz。
我看過類似的程式碼，非常的 ...

.. code-block:: scala

        @for(jobrow <- jobrows){
        <div class="row">
           @for(job <- jobrow){
           <div class="span4">
              <article class="job">
                 <header class="clearfix">
                    <div class="meta">
                       <span class="count">@job.count</span>
                    </div>
                    <div class="more">
                       <a href="#job-detail@job.count" data-toggle="modal">詳細資訊</a>
                    </div>
                 </header>
                 <div class="card">
                    <div class="card-body">
                       <h1>@job.company_name</h1>
                       <p>@job.job_name</p>
                    </div>
                    <div class="card-footer">	                          
                       <span class="label label-important">@job.note1</span>
                       <span class="label label-info">@job.note2</span>	                           
                    </div>
                 </div>
              </article>
           </div>
           }
        </div>
        }

這邊我用了兩個 for 迴圈來處理九宮格的排版方式（第一個 for 處理 row，第二個處理 job 的資料）

在這裡面可以看到 

	@job.company_name

之類的語法，這代表直接存取這個變數。

下面是 index.scala.html 的完整內容。

.. code-block:: html

	@(jobrows: List[List[Job]], nextPageNum: Int)
	@import helper._
	@main("22K Job Board") {
	<!doctype html>
	<html>
	   <head>
	      <meta charset="utf-8" />
	      <!-- Always force latest IE rendering engine or request Chrome Frame -->
	      <meta content="IE=edge,chrome=1" http-equiv="X-UA-Compatible" />
	      <!-- Use title if it's in the page YAML frontmatter -->
	      <title>22K Job Board</title>
	      <link href="@routes.Assets.at("stylesheets/bootstrap.min.css")" media="screen" rel="stylesheet" type="text/css" />
	      <link href="@routes.Assets.at("stylesheets/all.css")" media="screen" rel="stylesheet" type="text/css" />
	      <script src="@routes.Assets.at("javascripts/jquery.min.js")" type="text/javascript"></script>
	      <script src="@routes.Assets.at("javascripts/bootstrap.min.js")" type="text/javascript"></script>
	   </head>
	   <body class="index">
	      <div class="navbar navbar-fixed-top">
	         <div class="navbar-inner">
	            <div class="container">
	               <a class="brand" href="1">22K Job Board <span class="slogan">22K也有超級好工作</span></a>
	            </div>
	         </div>
	      </div>
	      <div id="main">
	         <div class="container">
	            @for(jobrow <- jobrows){
	            <div class="row">
	               @for(job <- jobrow){
	               <div class="span4">
	                  <article class="job">
	                     <header class="clearfix">
	                        <div class="meta">
	                           <span class="count">@job.count</span>
	                        </div>
	                        <div class="more">
	                           <a href="#job-detail@job.count" data-toggle="modal">詳細資訊</a>
	                        </div>
	                     </header>
	                     <div class="card">
	                        <div class="card-body">
	                           <h1>@job.company_name</h1>
	                           <p>@job.job_name</p>
	                        </div>
	                        <div class="card-footer">	                          
	                           <span class="label label-important">@job.note1</span>
	                           <span class="label label-info">@job.note2</span>	                           
	                        </div>
	                     </div>
	                  </article>
	               </div>
	               }
	            </div>
	            }
	         </div>
	         <div class="more-job">
	            <a href="@nextPageNum" class="btn btn-large btn-block">更多學習機會</a>
	         </div>
	      </div>
	      </div>
	      <footer id="footer">
	         <div class="container">
	            <blockquote>
	               <p>同樣在「22K 薪水」這條垂直虛線上，有超級爛的工作，也有超級好的工作，端看你如何選擇。事實上，同樣在任何薪資水準的垂直線上，都有超級爛的工作，也有超級好的工作。
	               </p>
	               <small>Mr Jamie <cite title="Source Title">林之晨</cite></small>
	            </blockquote>
	            <blockquote>
	               <p>所以重點根本不是 22K，重點是年輕人，既然你的政府、你的學校已經誤了你，你該用什麼方法，找到願意教你的師父，花 1 萬個小時跟他學習，最後變成炙手可熱，有超強戰力的人才。那時什麼 22K，花 100K 也不一定請得動你。
	               </p>
	               <small>Mr Jamie <cite title="Source Title">林之晨</cite></small>
	            </blockquote>
	         </div>
	      </footer>
	      @for(jobrow <- jobrows){
	      @for(job <- jobrow){
	      <div id="job-detail@job.count" class="modal job-detail hide fade">
	         <div class="modal-header">
	            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
	            <h3><span class="salary">月薪 @job.salary 元</span>的學習機會！</h3>
	         </div>
	         <div class="modal-body">
	            <table class="table table-bordered">
	               <tr>
	                  <th>學習環境</th>
	                  <td>@job.company_name</td>
	                  <th>學習職位</th>
	                  <td>@job.job_name</td>
	               </tr>
	               <tr>
	                  <th>學習地點</th>
	                  <td>@job.company_location</td>
	                  <th>學習內容</th>
	                  <td><a href="@job.job_url">你可以學到...</a></td>
	               </tr>
	            </table>
	         </div>
	         <div class="modal-footer">
	            <a href="@job.job_url" class="btn btn-primary btn-block btn-large">應徵職缺</a>
	         </div>
	      </div>
	      }
	      }
	   </body>
	   }

測試結果
---------------------------------------------------------------------

執行

	run

就可以在 http://localhost:9000 看到執行結果了。

部屬到 Heroku
---------------------------------------------------------------------

Heroku 現在有支援 Play framework 了，所以整個部屬的過程也非常簡單。
可以參考 `Getting Started with Play! on Heroku`_。




.. _怎麼設計22K?: http://minipai.tumblr.com/post/41882685541/22k
.. _原始碼: https://github.com/Swind/22k
.. _Play官網: http://www.playframework.org/
.. _play-2.0.4.zip下載點: http://download.playframework.org/releases/play-2.0.4.zip
.. _Your first Play application: http://www.playframework.org/documentation/2.1-RC4/ScalaTodoList
.. _Art Pai: http://minipai.tumblr.com/
.. _CSS Template: http://cl.ly/MYmn
.. _揭露22k: http://www.22kopendata.org/
.. _Getting Started with Play! on Heroku: https://devcenter.heroku.com/articles/play