.. title: 一鍵安裝 Linux 設定檔
.. slug: install-linux-config
.. date: 2013/01/21 22:36:41
.. tags: 
.. link: 
.. description: 

我常用的 Linux 環境一定會裝下面幾個工具

1. zsh 與 oh-my-zsh
2. vim 很多很多 vim plugin
3. tmux 與他的快樂設定檔

一開始開發環境只有一兩台，一個個手動安裝還沒什麼問題。
再加上那時後有從 vgod 的 `分享我的 vim 設定檔`_ 這文章中學到使用 pathogen + github 來管理 vim。
所以一直就懶得去想辦法整理這些設定檔。

可是最近要管理的機器變多了，而且加上最近再進行 porting 的工作，所以又有各種不同版本的 Linux 需要設定。

就在今天，終於 **爆～～～～發～～～～啦！！！！**

感謝 JosephJ 的 `一個指令安裝所有 Linux 設定檔`_ 以及該篇文章的回應。
我最後也從 pathogen.vim 叛逃，使用 github + shell script + vundle 來完成我 zsh、tmux 與 vim 的設定。

關於 vundle 的使用方式與設定我是參考 `使用Vundle更好的管理你的Vim插件`_ 雖然是簡體的。不過設定與使用方式都寫得非常詳細。

經過一個早上的努力，我也終於可以使用

.. code-block:: bash

	wget -O - https://raw.github.com/Swind/linux-config/master/install.sh | sh

把 Linux 給設定完畢了～

**2013.02.08 update**

我發現在其他機器上使用的時候會有::

	ERROR: certificate common name “*.a.ssl.fastly.net” doesn’t match requested host name “raw.github.com”.
	To connect to raw.github.com insecurely, use ‘--no-check-certificate’.

的錯誤訊息，目前我還沒有很仔細的去找原因。

不過如他所敘述的，加入 --no-check-certificate 就沒有問題了。

.. code-block:: bash

	wget --no-check-certificate -O - https://raw.github.com/Swind/linux-config/master/install.sh | sh
	

除此之外 JosephJ 所給的建議也非常的實用，
將常用的工具與指令整理成一份文件，來協助管理與記憶。
目前我是用 Nikola (對沒錯，所從 Octopress 跳到 Pelican 又跳到 Nikola 了 XD) stories 來作筆記。

.. TEASER_END

目前安裝的 script 是參考 JosephJ 的內容修改成這樣

.. code-block:: bash

	#!/bin/bash
	CONFIG_HOME=.myconfig

	warn(){
		echo "$1" >&2
	}

	die(){
		warn "$1"
		exit 1
	}

	[ -e "~/.config" ] && die "~/config already exists."

	cd ~

	#Checkout my config
	git clone git://github.com/Swind/linux-config.git "$CONFIG_HOME"

	ln -s $CONFIG_HOME/.vim .vim
	ln -s .vim/.vimrc .vimrc
	ln -s $CONFIG_HOME/.zshrc .zshrc
	ln -s $CONFIG_HOME/.tmux.conf .tmux.conf

	#install vundle
	git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle

	#install oh-my-zsh
	curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh

	echo "Configuration files has been installed."
	cd "$CONFIG_HOME"

----------------------------

.. figure:: https://dl.dropbox.com/u/15537823/Blog/ARIA.jpg
	:class: thumbnail
	
話說，水星領航員真的不愧是文青動畫之首阿！！今天的情況讓我想起了動畫中的這句話。

.. raw:: html

	<blockquote>
	<p>如果如果對自己的能力感到滿足的話，就不會再進步了。所以認清自己還遠遠不行是非常重要的事情。</p>
	<small>晃</small><cite title="Source Title">ARIA The ORIGINATION</cite></small>
	</blockquote>

.. _分享我的 vim 設定檔: http://blog.vgod.tw/2011/03/19/vimrc/
.. _一個指令安裝所有 Linux 設定檔: http://josephj.com/entry.php?id=374
.. _使用Vundle更好的管理你的Vim插件: http://yishanhe.net/using-vim-vundle-for-better-plugin-management/
