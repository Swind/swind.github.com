.. title: Testing your python REST client
.. slug: testing-your-rest-client
.. date: 05/24/2014 11:18:00 AM UTC+08:00
.. tags: Python, Test, draft
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

.. code:: python

   @get("item")
   def get_item():
       return "get_item" 

   @put("item")
   def put_item():
       return "put_item"

   @post("item")
   def post_item():
       return "post_item"

   @delete("item")
   def delete_item():
       return "delete_item"

除此之外, Python 的 Decorator 是可以直接呼叫的。
所以像上面 get, put, post 與 delete 等於 

.. code:: python
   
   def get_item():
       return "get_item" 

   def put_item():
       return "put_item" 

   def post_item():
       return "post_item" 

   def delete_item():
       return "delete_item" 

   get("item")(get_item)
   put("item")(put_item)
   delete("item")(delete_item)
   post("item")(post_item)



透過 Bottle 我們就可以很簡單的建立我們想要回傳的內容

那麼，如果是想要測試 error 的情況呢？

.. code:: python

   @get("item")
   def get_item():
       abort(401, "Access denied")

這樣就可以任意產生我們想要的資料給 CLI 進行測試了。

結合 unittest
=============

再來就是要在每一個 testcase 執行的時候，啟動我們的 REST API server。
由於 Bottle 本身的啟動方式，並沒有提供結束的方法。
所以這部份我是參考 `Bottle web framework - How to stop`_
回到 Python 本身的 WSGI Server 去做處理。

.. code:: python
   
   from bottle import Bottle, ServerAdapter

   class MockRESTServer(ServerAdapter):
       server = None

       def run(self, handler):
           from wsgiref.simple_server import make_server, WSGIRequestHandler

           if self.quiet:
               class QuietHandler(WSGIRequestHandler):
                   def log_request(*args, **kw): pass
               self.options['handler_class'] = QuietHandler

           self.server = make_server(self.host, self.port, handler, **self.options)
           self.server.serve_forever()

       def stop(self):
           # self.server.server_close() <--- alternative but causes bad fd exception
           self.server.shutdown


最後，讓 testcase setUp 啟動 Server, tearDown 的時候結束 Server 就可以了。

.. code:: python

   class TestServerHandlers(unittest.TestCase):

       server = None
       thread = None

       def setUp(self):
           app = Bottle()
           self.server = MockRESTServer(host="127.0.0.1", port=8357)

           @app.get("/hello_world")
           def hello_world():
               resp ='{"hello":"world"}'
               return resp

           def start_server():
               app.run(server=self.server)

           self.thread = threading.Thread(target=start_server)
           self.thread.start()
           time.sleep(0.5)

       def tearDown(self):
           self.server.stop()

       def test_case_1(self):
           pass


中間會去 sleep(0.5) 是我發現如果我的 testcase 失敗的太快，或者太快結束。會導致 server 還沒有啟動結束，就停止
而導致 testcase 卡住無法繼續執行下去。
因此 sleep(0.5) 讓 server 可以啟動完畢。

.. _Bottle: http://bottlepy.org/docs/dev/index.html
.. _Bottle web framework - How to stop: http://stackoverflow.com/questions/11282218/bottle-web-framework-how-to-stop
