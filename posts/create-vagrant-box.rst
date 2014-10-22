.. title: 自製 Vagrant Box 
.. slug: create-vagrant-box
.. date: 09/25/2014 04:16:25 PM UTC+08:00
.. tags: vagrant 
.. link: 
.. description: 
.. type: text

最近都在使用 Vagrant 以及看在 `C.C Agile`_ 中 beelit94_ 所介紹的 Packer。

目前是使用 Vagrant 來統一公司同事的測試環境，減少因為環境不同而導致的 bug。

這邊主要是筆記自己建立 box 時所要設定的內容，主要都是參考 `Creating a Custom Vagrant Box`_ 的文章。

在 VirtualBox 上安裝系統
=====================================

VirtualBox 安裝 Extension Pack

.. code:: bash

   wget http://download.virtualbox.org/virtualbox/4.3.16/Oracle_VM_VirtualBox_Extension_Pack-4.3.16-95972.vbox-extpack
   VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-4.3.16-95972.vbox-extpack

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

增加 Vagrant 的 ssh key 到 authorized_keys
-----------------------------------------------------

.. code:: bash

   mkdir -p /home/vagrant/.ssh
   wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys
   chmod 0700 /home/vagrant/.ssh
   chmod 0600 /home/vagrant/.ssh/authorized_keys
   chown -R vagrant /home/vagrant/.ssh

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

.. code:: bash

    sudo vim /etc/udev/rules.d/70-persistent-net.rules

安裝 Virtualbox Guest Additions
-----------------------------------------------------

直接到 http://download.virtualbox.org/virtualbox/ 下載對應版本的 Guest Addition，例如 `VBoxGuestAdditions_4.3.18.iso`

.. code:: bash

   wget http://download.virtualbox.org/virtualbox/4.3.18/VBoxGuestAdditions_4.3.18.iso
   apt-get install dkms gcc
   mount -o loop VBoxGuestAdditions_4.3.18.iso /mnt 
   cd /mnt
   ./VBoxLinuxAdditions.run

打包成 box
-----------------------------------------------------

<vm-name> 的部份需要自己換成 VM 的名稱

.. code:: bash

    vagrant package --base <vm-name>

.. figure:: https://dl.dropboxusercontent.com/u/15537823/Blog/Lord_Marksman_and_Vanadis.jpg

.. raw:: html
   
   <blockquote>
   <p>これは英雄へと至る物語</p>
   </blockquote>

作畫品質蠻好的，而且鈴木このみ - 銀閃の風也非常的和我胃口～～

.. youtube:: https://www.youtube.com/watch?v=dP9rUj_ENS0

.. _Creating a Custom Vagrant Box: http://williamwalker.me/blog/creating-a-custom-vagrant-box.html
.. _How to install VirtualBox Guest Additions for Linux: http://xmodulo.com/how-to-install-virtualbox-guest-additions-for-linux.html
.. _C.C Agile: http://teddysoft.tw/category/ccagile/ 
.. _beelit94: http://ithelp.ithome.com.tw/profile?id=20014061
