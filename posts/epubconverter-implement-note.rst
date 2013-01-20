.. title: 實作 EPUBConverter 的筆記
.. slug: epubconverter-implement-note
.. date: 2012-05-11 18:19
.. tags: Scala
.. link: 
.. description:

好吧，我要開始筆記一下這次的實作心得了
==========================================

其實在學 Scala 的這一段時間中，我最困擾的就是到底怎樣才是好的 Scala 寫作風格。
因為 Scala 最吸引我的部份就是在於他的語法可以不斷的精鍊再精鍊，最後變成*魔法文字* Orz。
所以要怎樣善用 Scala 的語言特性，卻又不失可讀性，我覺得真的很難。
因此在這次的小工具實作中，我試著摸索 Curry 的用法。

Curry
-----------------------

說到教學文件，就一定要推一下 良葛格的學習筆記_ 裡面他有提到 Curry_  的語法與何謂 Curry 。
但是我更喜歡在 jserv's blog `以 C 語言實做 Functional Language 的 Currying`_  看到的說明。

Curry 可以看成數學的多項式，舉例來說現在有一個多項式

      F(X,Y) = X + Y

那麼 Curry 的感覺就是我現在已知 X 為 10 之類的，然後將其代入，所以上面的多項式又變成另外一個多項式

      F(Y) = 10 + Y

如果將這段用 Scala 來表示就是

.. code-block:: scala

    // 這是 F(X,Y)
    def F(X:Int)(Y:Int)={
       X + Y
    }
    
    // 帶入 X = 10
    
    val F1 = F(10)

Curry 的使用案例
-----------------------------

在 jserv 的文章有提到一個 Ruby 目錄樹尋訪的範例，剛好我在這次有實作類似的東西，但是功力不足沒有辦法寫的像 Ruby 那麼優雅。

.. code-block:: scala

    def walkDir(fileList: List[File] {})(expr: (File) => Unit): Unit = {
        fileList match {
          case file :: files =>
            if (file.isDirectory)
              walkDir(file.listFiles.toList ::: files)(expr)
            else
            {
              expr(file)
              walkDir(files)(expr)
            }
          case _ =>
        }
      }

我想這樣寫最大的問題應該是會在 

.. code-block:: scala

       walkDir(file.listFiles.toList ::: files)(expr)

這邊每次都會產生一個新的 List ，這是優點也是缺點，效率我想可能會差了一點，但是我不在意，反正這個程式不是一秒幾十萬上下的東西。
那麼這個東西可以怎麼用呢？

例如要印出所有資料夾底下的檔案

.. code-block:: scala

    walkDir(folder){println}

刪除底下所有名稱包含 test 的檔案

.. code-block:: scala

    walkDir(folder){
       file=>
          if(file.getName.contains("test"))
             file.delete
    }

我另外一個有使用 Curry 的地方

.. code-block:: scala

    def InputToOutput(buffer: Array[Byte])(fis: InputStream, fos: OutputStream) = 
    {
       def bufferReader(fis: InputStream)(buffer: Array[Byte]) = (fis.read(buffer), buffer)
    
       def writeToOutputStream(reader: (Array[Byte]) => Tuple2[Int, Array[Byte]], fos: OutputStream): Boolean = {
         val (length, data) = reader(buffer)
         if (length >= 0) {
           fos.write(data, 0, length)
           writeToOutputStream(reader, fos)
         } else
           true
       }
    
       writeToOutputStream(bufferReader(fis)_, fos)
    
     (fis, fos)
    }

這個 function 主要負責將 InputStream 的資料寫到 OutputStream。

第一個 Curry 是 InputToOutput 他讓我可以用 InputToOutput(buffer)_ 建立一個已經宣告好 buffer 的 IO 操作 function。
這樣我就不用每次都還要找一個 buffer 才可以開始我的 IO 操作，反正 buffer 的內容讀完就可以丟了，不過這個沒有考慮 multiple thread 的情況就是。

第二個 Curry 是讓 Read InputStream 的 function 跟傳進來的 InputStream 綁定，這單純只是想簡化之後的操作，讓我可以不用再考慮 InputStream 這個參數。
反正我只要給一個 buffer 他就會自動讀進來，並且回傳讀取的大小與 buffer 本身。

解壓縮的部份也用了類似的技巧

.. code-block:: scala

    def unzipAllFile(entryList: List[ZipEntry], getInputStream: (ZipEntry) => InputStream, targetFolder: File): Boolean = {
        entryList match {
          case entry :: entries =>
    
            if (entry.isDirectory)
              new File(targetFolder, entry.getName).mkdirs
            else
            {
              val (input,output) = InputToOutput(getInputStream(entry), new FileOutputStream(new File(targetFolder, entry.getName)))
              
              input.close
              output.close
            }
    
            unzipAllFile(entries, getInputStream, targetFolder)
    
          case _ =>
            true
        }
    }

大概就這些了，總覺得程式還是寫的不夠多，這些 Code 應該可以寫的再更優雅一點才是。

.. _良葛格的學習筆記: http://caterpillar.onlyfun.net/Gossip/Scala/index.html
.. _Curry: http://caterpillar.onlyfun.net/Gossip/Scala/Curry.html
.. _以 C 語言實做 Functional Language 的 Currying: http://blog.linux.org.tw/~jserv/archives/002029.html