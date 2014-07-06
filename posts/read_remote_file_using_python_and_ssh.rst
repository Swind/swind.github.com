.. title: 使用 Python 透過 SSH 讀取遠端的檔案 
.. slug: read_remote_file_using_python_and_ssh
.. date: 07/06/2014 08:45:37 PM UTC+08:00
.. tags: Python
.. link: 
.. description: 
.. type: text

tail 遠端的檔案
===================================

最近的自動化測試需要去做類似 tail 的動作來確認在執行 TestCase 的過程中，
server log 沒有吐出不該出現的錯誤。

可是又不能透過 fabric 直接下 tail 指令，因此這次透過了 SFTP 來實現這個功能。

.. TEASER_END

使用 paramiko
==================================

paramiko_ 是 Python 上處理 SSH2 通訊協定的 module，fabric 底層也是使用他。
在這邊我則是使用他的 SFTP 的功能去開啟遠端的檔案。

.. code-block:: python
    
    import paramiko

    def sftp_open(ip, port, user, password, file_path, open_type):

        transport = paramiko.Transport((ip, port))
        transport.connect(username=user, password=password, hostkey=None)

        sftp = paramiko.SFTPClient.from_transport(transport)

        return sftp.open(file_path, open_type)

在這邊我省略了 hostkey 的驗證部份

.. code-block:: python

    paramiko.util.load_host_keys(os.path.expanduser('~/.ssh/known_hosts')) 

因為 paramiko 讀取 know_hosts 會花一小段時間。但是我這個連線會開開關關的非常頻繁，所以為了測試的順暢，我略過了這個驗證。


實現類似 tail 的功能
=================================

其實原理非常簡單，當透過 SFTP 開啟檔案之後，我們就可以任意對他做讀寫操作，當然也包括了 seek。
所以我另外實做了一個 Tail 物件。

.. code-block:: python

    class Tail(object):

        def __init__(self, ip, port, user, password, file_path):

            self.ip = ip
            self.port = port
            self.user = user
            self.password = password
            self.file_path = file_path
            self.__lastposition = 0

            self.seek_to_end()
            self.with_statement_log = ""

        def __open(self):
            #Open the file and record the last position
            return sftp_open(self.ip, self.port, self.user, self.password, self.file_path, "r")

        def __enter__(self):
            self.seek_to_end()

        def __exit__(self, type, value, traceback):
            self.with_statement_log = self.read()

        def seek_to_end(self, read_file = False):
            server_file = self.__open()
            server_file.seek(self.__lastposition)

            result = ""
            if read_file:
                result = server_file.read()

            self.__lastposition = server_file.tell()
            server_file.close()

            return result

        def read(self):
            return self.seek_to_end(read_file = True)

其實整個原理很簡單，在建立 Tail 的時候，seek 到檔案的尾端，並且紀錄位置。
在經過一些動作之後，想要取得檔案的增加部份，就是再從上次紀錄的尾端往後讀到底就好了。

並且在這邊實做了 with statement ( __enter__ 與 __exit__ ) 來方便紀錄。

使用方式會類似這樣

.. code-block:: python

    test_file_path = "/tmp/test.log"
    open(test_file_path, 'w').close()

    tail_log = Tail("127.0.0.1", 22, "root", "password", test_file_path)

    with tail_log:
        with open(test_file_path, "a") as test_file:
                test_file.write("appended text")

    print tail_log.with_statement_log

在 with statement 開始的時候會去 seek 到檔案最尾端，等離開 with statement 的時候，
再去讀取到最尾端，並且除存在 with_statement_log。

參考資料：

`Read a file from server with ssh using python`_
`paramiko sftp example`_

.. _paramiko: https://github.com/paramiko/paramiko 
.. _Read a file from server with ssh using python: http://stackoverflow.com/questions/1596963/read-a-file-from-server-with-ssh-using-python 
.. _paramiko sftp example: https://github.com/paramiko/paramiko/blob/master/demos/demo_sftp.py
