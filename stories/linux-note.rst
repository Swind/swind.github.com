.. title: Linux 指令筆記 
.. slug: linux-note
.. date: 2013/01/20 10:16:20
.. tags: Linux
.. link: 
.. description: 我常用的 Linux 指令筆記

因為公司偉大的 IT 把 Evernote 給我擋掉了 T_T。

雖然開了 SSH Tunnel，可是 Evernote 竟然不支援 Proxy。
才會有這篇出來，基本上這篇沒有什麼營養價值，單純只是我自己常用的指令而已。

這篇是永久置底 Orz 

.. class:: alert alert-info pull-right

.. contents::

Git 相關
-----------------------------

漂亮一點的 log 顯示方式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

出處 `酷殼 - CoolShell.cn - Git 顯示漂亮日誌的小技巧`_

.. code-block:: shell

	git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --"

.. _酷殼 - CoolShell.cn - Git 顯示漂亮日誌的小技巧: http://coolshell.cn/articles/7755.html

讓所有 submodule 執行 git pull
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    git submodule foreach git pull

所有 submodule checkout master
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    git submodule foreach git checkout master


Storage 相關
-----------------------------

sg3 utils
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

列出所有 scsi 硬體

	sg_scan


讀檔案
     
     sg_raw.exe -v -r 512 –o read512byte.txt PD3 28 00 00 00 03 00 00 00 01 00

寫檔案
其中 wdata512byte 是事先寫好的一個檔案

     sg_raw -v -s 512 PD3 2a 00 00 00 01 00 00 00 01 00 < wdata512byte


Linux 相關
-------------------------------

SSH login without password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: shell

	#!/bin/bash
	account=$1
	ip=$2
	ssh-keygen -t rsa
	ssh $account@$ip mkdir -p .ssh
	cat ~/.ssh/id_rsa.pub | ssh $account@$ip 'cat >> ~/.ssh/authorized_keys'

使用 rz 與 sz
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	yum install lrzsz

判斷 Linux 發行版本
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	cat /etc/issue


Eclipse 相關
-------------------------------

搜尋
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Shift + Ctrl + T 可以搜尋 Class
	Shift + Ctrl + R 可以搜尋檔案

顏色
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

選擇之後會把相同名稱的選擇起來的功能叫做

    Mark occurences

修改的地方在 

	Preferences > General > Editors > Text Editors > Annotations


GDB
-------------------------------

使用 gdb 觀看 core file 資訊
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	gdb -c <corefile> <execfile>

Thread 相關操作
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

顯示所有 thread
++++++++++++++++++++++++++++++++

	info threads

進入某個 thread
++++++++++++++++++++++++++++++++

	thread <thread number>

