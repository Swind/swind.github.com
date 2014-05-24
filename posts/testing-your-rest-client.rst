.. title: Testing your python REST client
.. slug: testing-your-rest-client
.. date: 05/24/2014 11:18:00 AM UTC+08:00
.. tags: Nikola, draft
.. link: 
.. description: 
.. type: text

我又換公司了
============

最近因為在修改新公司的 CLI 程式, 所以順便想了一下要怎麼測試比較好。
這個 CLI 是透過 server 所提供的 REST API 來操作遠端的 Server,
因此我剛接手的時候都是真的去連到一個 Server 進行開發。

但是 Server 上有很多的操作需要硬體的配合, 或者要產生某種 error code 並不是這麼容易。
所以想要在 unittest 的測試過程中建立假的 REST API Server 讓 testcase 針對各種不同的回傳值進行測試。

當然, 在程式架構設計的好的情況下, 其實不需要建立這種假的 Server。
但我目前正在重構中, 要將 CLI 的架構重新設計, 並且在這過程之中要維持程式可以正常運作。
所以我必須確保還未重構的功能還能正常運作, 因此我想建立假的 REST API Server 是個可以同時兼顧新舊架構的好方法。

Web framework -- Bottle
=======================

為了要讓 Testcase 可以快速且簡單的更換 REST API server 的內容, 我選擇了 Bottle_ 作為 server 的框架。
雖然在 Python 的 web framework 中, 我比較偏好 flask。
但是 Bottle_ 整個 framework 只有一個檔案且無依賴其他第三方的 library的特性。
我覺的非常適合放在測試的之中進行操作。

Bottle_ 的語法與使用方式也非常的直覺,  REST API 最基本要測試的情況就是 GET, POST, PUT 與 DELETE。
這幾種功能要在 Bottle 裡面表達的話就只要

.. code:python

   @get("item")
   def get_item():
       pass

   @put("item")
   def put_item():
       pass

   @post("item")
   def post_item():
       pass

   @delete("item")
   def delete_item():
       pass




.. _Bottle: http://bottlepy.org/docs/dev/index.html
