.. title: Python logging 筆記 
.. slug: python_logging_note
.. date: 07/19/2014 08:45:37 PM UTC+08:00
.. tags: Python,draft
.. link: 
.. description: 
.. type: text

使用 Python logging 紀錄 message
===================================

因為之前都偷懶使用 print 直接輸出一些 debug 訊息，沒有好好的進行分類。
所以最後程式在執行的時候，看到了很多垃圾訊息，阻礙了開發。

尤其是之前都偷懶直接用 basicConfig 去設定，所以連第三方 package 的 log 都印出來了 Orz，
因此想順便藉這個機會好好弄清楚 Python 裡面 logging 的使用方式。

.. TEASER_END
