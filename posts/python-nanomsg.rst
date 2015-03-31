.. title: 在 Python 使用 nanomsg
.. slug: python-nanomsg
.. date: 03/31/2015 10:36:25 PM UTC+08:00
.. tags: python, draft 
.. link: 
.. description: 
.. type: text

這幾個月，難得有東西可以完全照自己的意思亂寫（因為開發者只有我一個）

所以就開始測試一些只有聽過得工具。

因為用膩了 Python 裡面的 Queue，並且幻想以後程式可以在不同的地方上執行。

所以選擇了 `nanomsg`_ 雖然現在只是在 Process 間溝通，但是真要改成透過 http 的時候，

只需要改變 nanomsg 的 protocol 就可以了。

.. TEASER_END

安裝 nanomsg wrapper for python
==================================================

在 nanomsg 的官網上，有列出 language bindings

其中 Python 推薦的是 `nanomsg-python`_

但是 nanomsg-python 沒有辦法透過 pip 或者是 easy-install 安裝，

所以必須手動 clone repository 執行 setup.py 來安裝。

.. code:: bash

    clone https://github.com/tonysimpson/nanomsg-python.git
    cd nanomsg-python
    python setup.py build_ext
    python setup.py install
    
不過 nanomsg-python 沒有什麼文件，基本上 nanomsg 的界面他都有包裝出來。

但是除此之外，他自己還有做更高一階的 API，可以參考 nanomsg-python 的 `tests`_

( 測試程式果然是一個很棒的文件阿！）

nanomsg-python 的 protocols
===============================================

nanomsg 有支援 

1. INPROC
2. IPC
3. TCP

這三種 protocols 的 address 長的樣子分別是

.. code:: bash

    # INPROC
    inproc://address
    
    # IPC
    ipc:///tmp/address
    
    # TCP
    tcp://*:5555

nanomsg-python 的 PAIR 與 PUBSUB 模式
================================================

nanomsg 本身有實做好幾種模式，其中我最常用的就是 PAIR 與 PUBSUB。

詳細的介紹可以參考 `Getting Started with 'nanomsg'`_  裡面有程式碼跟架構圖，

不過程式碼是 C code 就是了，所以要用 nanomsg-python 來實做還是需要參考一下 tests 裡面的範例，

或者是直接看 nanomsg-python 的原始碼。

PAIR
------------------------------------------------

簡單來說就是一個雙向的一對一溝通

.. figure:: http://tim.dysinger.net/img/2013-09-16-getting-started-with-nanomsg/pipeline.png

.. code:: python

    from nanomsg import (
        PAIR,
        Socket
    )
    
    address = "inproc://address"
    # Server
    socket_server = Socket(PAIR)
    socket_server.bind(address)
    socket_server.send(b'ABC')
    
    # Client
    socket_client = Socket(PAIR)
    socket_client.connect(address)
    recieved = socket_client.recv()



    
.. _nanomsg: http://nanomsg.org/
.. _nanomsg-python: https://github.com/tonysimpson/nanomsg-python
.. _tests: https://github.com/tonysimpson/nanomsg-python/tree/master/tests
.. _Getting Started with 'nanomsg': http://tim.dysinger.net/posts/2013-09-16-getting-started-with-nanomsg.html
