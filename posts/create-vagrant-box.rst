.. title: 自製 Vagrant Box 
.. slug: create-vagrant-box
.. date: 09/25/2014 04:16:25 PM UTC+08:00
.. tags: draft
.. link: 
.. description: 
.. type: text

在 VirtualBox 上安裝系統
=====================================

建立 vagrant 帳號
------------------------------------

.. code:: bash
   
   useradd -m vagrant
   passwd vagrant

修改預設 editor 與 sudoers 
------------------------------------

.. code:: bash

   sudo update-alternatives --config editor
   sudo su -
   visudo

增加

.. code::

   vagrant ALL=(ALL) NOPASSWD:ALL


移除已經存在的網路卡 ( 除了第一個 NAT 的 eth0 之外）
-----------------------------------------------------

錯誤訊息

.. code::

    The following SSH command responded with a non-zero exit status.
    Vagrant assumes that this means the command failed!

    /sbin/ip addr flush dev eth1 2> /dev/null

    Stdout from the command:



    Stderr from the command:

    stdin: is not a tty

原因是有可能之前有增加過網路卡，所以 eth1 代表那個已經不存在的網路卡。
導致 vagrant 沒有辦法初始化網路設定

.. code::

    sudo vim /etc/udev/rules.d/70-persistent-net.rules

.. _Creating a Custom Vagrant Box: http://williamwalker.me/blog/creating-a-custom-vagrant-box.html
.. _How to install VirtualBox Guest Additions for Linux: http://xmodulo.com/how-to-install-virtualbox-guest-additions-for-linux.html
