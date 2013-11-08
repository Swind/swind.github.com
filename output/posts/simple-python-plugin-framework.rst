.. link: 
.. description: 
.. tags: 
.. date: 2013/08/25 15:26:12
.. title: 簡單易懂的 Python Plugin 架構 (大概 ...)
.. slug: simple-python-plugin-framework

學習用 Python 寫測試工具快兩個星期了，主要使用 clime + ucltip 跟 pyinstaller。
可是目前遇到一個問題，當我使用 pyinstaller 將測試工具打包之後，
他會很貼心的幫你將所有 .py 檔包裝成一個可執行檔。
所以當我要任何新增 command 都必須要重新部屬那個執行檔，這樣實在非常的麻煩。

因此才會開始嘗試是否有可能執行檔只包裝 clime, ucltip 以及我自己所提供的 API，
然後 command 的部份就利用 plugin 的方式來增加。

.. TEASER_END

資料夾結構
------------------------
分成兩部分，一部分是放共用 package 的 "site-packages"，另外一個就是放 plugin 的 "plugins"。

Content::

    |
    ----site-packages
        |----ucltip
            |----__init__.py
    ----plugins
        |----cli
            |----hello.py
    ----plugin_loader.py

我放了一個 ucltip packages 在裡面，這是為了要順便測試是否能順利 import packages 所放的。

載入 Plugin
------------------------

首先要讓 Python 可以正確的載入 site-packages 所以需要

.. code:: python
    
    sys.path.append("./site-packages")

這樣 import 的時候才會去我們的資料夾底下找 packages。

接著設定全域的環境變數

.. code:: python

    plugins_root       = "plugins"
    cmd_obj = sys.modules['__main__']

因為 clime 預設是去 main 這個 modules 搜尋需要轉換成 command 的 function，
所以我需要將從 plugins 載入的 module 的 command function 加入 main object，
當然 clime 有一個 obj 的參數，所以更好的作法應該是建立一個 command obj。

接著我們就可以載入 hello.py 裡面的 command function。

.. code:: python

    def __import_plugin_cmd(module_name, plugin_name):

        module_path = join(plugins_root, module_name)
        plugin_path = join(plugins_root, module_name, plugin_name + '.py')

        #Load plugin module
        cli_print("Loading plugin %s", plugin_path)
        plugin_module = load_module(module_name, *find_module(plugin_name, [module_path]))

        #Get the command method and set it to command object
        methods = dir(plugin_module)
        for method in methods:
            if method.endswith("cmd"):
                cli_print("Set command %s", method)
                method_obj = getattr(plugin_module, method)
                setattr(cmd_obj, method, method_obj)

        return plugin_module

首先 module_name 就是 plugin 底下的資料夾名稱，在這邊我會輸入 "cli"。
plugin_name 是要載入的 python 檔案名稱，在這裡為 "hello"。

之後就透過 find_module 找到 module 的資訊，可以參考 Python 的 imp_ 文件的範例，
再透過 load_module 載入。

.. code:: python

    plugin_module = load_module(module_name, *find_module(plugin_name, [module_path]))

最後找尋所有 cmd 結尾的 function，透過 setattr 加入 cmd_obj

.. code:: python

    #Get the command method and set it to command object
    methods = dir(plugin_module)
    for method in methods:
        if method.endswith("cmd"):
            cli_print("Set command %s", method)
            method_obj = getattr(plugin_module, method)
            setattr(cmd_obj, method, method_obj)

之後使用 clime 的

.. code:: python

    import clime
    clime.start(white_pattern=clime.CMD_SUFFIX, debug=True)

就可以將 cmd_obj 以 cmd 結尾的 function 轉成 command 了。

下面附上完整的程式碼

.. gist:: 6332885

.. _imp: http://docs.python.org/2/library/imp.html#examples 


