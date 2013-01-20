.. title: 好用的 HTML Parser - jsoup
.. slug: jsoup
.. date: 2011-10-27 08:47
.. tags: Scala
.. link: 
.. description: 

What is jsoupsoup
=====================

HTML Parser出門在外,居家旅遊必備良品。無論是要自動下載漫畫、種子或者是做個自動天氣噗浪機全都需要他。
為了要從網頁中可以取出內容,我曾經用過不少方式,包括直接硬幹用字串搜尋、修改Scala本身內部的XML Parser等。
但是一直都找不到一個滿意又順手的解決方式,一直到後來在 CSDN_ 沒錯就是那個大陸網站 CSDN_
(雖然說CSDN訊息太多太雜亂,但是偶爾還是可以看到一些不錯的東西),才找到jsoup這個library。

jsoup的官方網站_

jsoup的教學文件_

.. _jsoup的官方網站: http://jsoup.org/
.. _jsoup的教學文件: http://jsoup.org/cookbook/
.. _CSDN: http://www.oschina.net/

但是其實最重要的是這一 教學文件_ ,介紹了他與別人最不一樣的的地方。
一般的html parser還是比較傾向於跟xml parser的作法一樣,把html分析完之後建成樹
然後操作node將資料取出,但是除了這種操作方式之外josue還有一種是類似jquery的selector的操作方式。

.. _教學文件: http://jsoup.org/cookbook/extracting-data/selector-syntax

.. code-block:: scala

    file input = new file("/tmp/input.html");
    document doc = jsoup.parse(input, "utf-8", "http://example.com/");

    //document doc = jsoup.connect("http://example.com/");

    //取出擁有herf屬性的<a>
    elements links = doc.select("a[href]");
    //取出擁有src屬性且值是以png結尾的<img>
    elements pngs = doc.select("img[src$=.png]");

上面是官網的範例,從這邊就可以看到 jsoup 不同於一般html parser的地方。
jsoup除了有支援讀取檔案之外,也可以直接輸入網址讓他自己去連線與parser,
最重要的是可以透過selector-syntax來取出想要的內容。接下來我會整理一些我常用的用法。

p.s我所使用的語言是scala，但是我會盡量不要用scala的<del>奇淫技巧</del>特殊語法讓他看起來跟java差不多。

出發總要有個方向,這邊就以一個我想幹的壞事情來當作例子好了，順便強迫我自己把東西寫完。

說出你的願望，否則不會讓你如願
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我想要一個可以自動檢查 某大論壇_ 的動畫討論區新番是否有更新的機器人。
（因為它新番更新在同一個文章裡面，每周找新番真的好麻煩）

.. _某大論壇: http://www.cwb.gov.tw/v7/forecast/taiwan/taipei_city.htm

哈利路亞 ! chance !

Prepare
===========

首先呢，我習慣先將網頁內容儲存成html檔案,放到測試資料夾中。
然後，寫個測試先決定整個程式的主要介面。
因為我不喜歡一直連線到網站做測試。
因為:

1. 浪費流量
2. 我要驗證程式執行結果的時候必須要開網頁才能驗證。
    (因為文章會一直新增第一頁的內容會被擠到後面去)
3. 你不一定隨時都有網路可以用

因此先將今天的文章列表儲存起來，然後寫個測試來驗證我最後的輸出結果是否如我所預期。
有人會想說這是不是什麼軟體工程技巧阿，是不是什麼測試先行（tdd）的開發方式之類的？
其實也沒有特別想說要使用什麼樣的開發方式或開發技巧，單純就只是因為我覺得這樣比較方便而已。
我覺得沒有必要被偉大或者是只要這樣做就對了等口號給迷惑，選擇自己需要且足夠的方式就好了。

要分析擷取的網頁內容
~~~~~~~~~~~~~~~~~~~~~

這邊是網頁的部份內容，我在做測試資料的時候是將整份的網頁原始碼都儲存起來，下面的內容是為了後面的說明需要。

.. code-block:: html

    <tbody id="normalthread_6961970">
     <tr onmouseover="l_pic_6961970.style.display='block'" onmouseout="l_pic_6961970.style.display='none'">
      <td class="folder">
        <a href="viewthread.php?tid=6961970&amp;extra=page%3d1%26amp%3borderby%3ddateline" title="新窗口打開" target="_blank">
            <img src="images/default/folder_new.gif" />
        </a>
     </td>
      <td class="icon"><img src="http://a18.file-static.com/attachments/lib_picture/19/70/6961970.jpg" width="50" height="40" class="l_bpic" />
       <div class="l_spic" id="l_pic_6961970">
        <img src="http://a18.file-static.com/attachments/lib_picture/19/70/6961970.jpg" width="100" height="80" class="l_bpic" />
       </div></td>
      <th class="subject new">
        <label>&nbsp;</label> 
        <em>[<a href="forumdisplay.php?fid=22&amp;filter=type&amp;typeid=2">分享</a>]</em>
        <span id="thread_6961970">
            <a href="viewthread.php?tid=6961970&amp;extra=page%3d1%26amp%3borderby%3ddateline">
            (mu@繁體@rmvb)onepiece海賊王 第521話 (1p)</a>
        </span>
      </th>
      <td class="author"><cite><a href="space.php?uid=933315">likesea</a></cite><em>2011-10-30</em></td>
      <td class="nums"><strong>0</strong>/<em>2</em></td>
      <td class="lastpost">
      <cite>
        <a href="space.php?username=likesea">likesea</a>
      </cite>
      <em><a href="redirect.php?tid=6961970&amp;goto=lastpost#lastpost"><span title="2011-10-30 03:13 pm">1&nbsp;分鐘前</span></a></em></td>
     </tr>
    </tbody>

想要的輸出結果
~~~~~~~~~~~~~~~~

    uid       : 933315
    title     : onepiece海賊王 第521話
    author    : likesea
    link      : http://.............

程式的介面
=============

雖然jsoup可以直接接收網址去取得網頁內容，但測試程式有讀取檔案的需求所以這邊會提供兩種介面。
一種允許使用者直接傳入string型態的參數，也就是網址。另外一種則是讓使用者傳入file型態的參數。
而回傳值的形態則是存放data object的list，這邊data object的名稱就先定義為envypost好了。

.. code-block:: scala

    class envypost{
        val uid:integer
        val title:string
        val lastposttime:date
    }

    def parse(url:string):list[envypost]={}
    def parse(file:file):list[envypost]={}

開始用jsoup分析網頁資訊吧
===========================

找出所有文章
~~~~~~~~~~~~~~~

雖然jsoup有提供很多種分析的方式，但是我最喜歡用的還是 selector_ 的語法。因為簡單明瞭又好閱讀！！
執行效率我則完全不在意，反正我的需求也不是一秒幾十萬上下的東西。能夠讓我愉快又快速的寫好才是重要的。

.. _selector: http://jsoup.org/cookbook/extracting-data/selector-syntax

.. code-block:: scala

    object envyexample {
      def parse(file:file,encode:string="big5"):list[envypost]={
        val doc = jsoup.parse(file,encode,envyurl)
        val posts = parsepage(doc)
      }
      def parsepage(page:document)={
        doc.select("tbody[id^=normalthread]")
      }
    }

由於該論壇的每一篇文章都被一個tbody的tag包圍，且此tbody的id開頭為normalthread。因此我的第一步就是先找出此頁中每一篇文章的element。

雖然說jsoup跟jquery一樣對於tag的id與class屬性都有特殊的語法例如 tag#id 或 tag.class 但是由於我需要使用正規表示是來找出id為normalthread開頭的tbody。
所以這邊使用 

    tbody[id^=normalthread]

因為若使用 # 的語法就沒有辦法使用正規表示式（至少我目前在官網的說明文件還沒有找到 orz）
有沒有覺得這個語法真的超級方便的，如果是用其他工具的話，我現在應該還在處理把id屬性取出來，然後用string的startwith來判斷是不是normalthread開頭。

另外，我喜歡把每一個步驟分解成很多小函式，因為這樣方便我進行測試。例如上面這一段程式碼，我相對應的測試程式碼會長這樣。

.. code-block:: scala

    class testenvyexample extends funsuite with shouldmatchers{
      test("there should be 10 post in the test file"){
        val doc = jsoup.parse(new file("./testdata/envy.html"),"big5",envyexample.envyurl)
        val posts = envyexample.parsepage(doc)
        posts.size should be (18)
      }
    }

順便說明一下語法，我所使用的測試framework是 scalatest_ ，這邊使用java的junit也是可以。
testenvyexample繼承funsuite跟shouldmatchers兩個class，這兩個class主要讓測試程式可以使用**test**跟**should be**兩種語法。

.. _scalatest: http://www.scalatest.org/

到這邊程式執行完畢之後我就有目前此頁面每一篇文章的所有內容了。下一步就是要分析這些文章內容了。

分析文章的內容-取得id、title、author與link
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

取得post的功能再另外獨立一個parsepost的function，傳入的參數則為包含所有文章內容的element，並且希望這個function可以回傳一個envypost物件。

.. code-block:: scala

    def parsepost(post:element):envypost={
    }

文章id的取得是透過tbody本身的id屬性，他的格式是normalthread_xxxxxx，後面的xxxxxx就是此篇文章的id，

.. code-block:: scala

    def parsepost(post:element)={
        val id = post.attr("id").replace("normalthread_","")
        val (title,link) = parserpost_titleandlink(id,post)
        val author = parserpost_author(post)

        new envypost(id,title,author,link)
    }

link跟title其實是一起取得的，它們可以從id為thread_xxxxxx的span tag中取得。

.. code-block:: scala

    def parserpost_titleandlink(id:string,post:element)={
        val element = post.select("span#thread_"+id).first
        (element.text,element.select("a").first.attr("href"))
    }

author的名字則在class為author的td tag內cite中因此取的時候使用

    tag1 tag2

的語法，這代表搜尋tag1底下所有的tag2

.. code-block:: scala

    def parserpost_author(post:element)={
        post.select("td.author cite").first.text
    }

打完收工，這樣的短短的程式碼就把一個網站都分析完了，真的太棒了 qq 
以前要分析一個網站超血淚的，真的感謝open source的眾多好心人，願意分享他們的成果與程式碼。
當然後續還有換頁讀取或者是比對之前的內容看是否有更新之類的工作，但是這都是後話了。

p.s一篇文章我竟然要打快兩個月，真佩服那些有辦法每天都有產出的人

