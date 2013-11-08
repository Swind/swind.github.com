.. link: 
.. description: 
.. tags: FreeBSD, C
.. date: 2013/09/18 14:00:09
.. title: 加速 FreeBSD Kernel 的編譯
.. slug: use_ccache_to_build_freebsd_kernel

最近大概都脫離不了 FreeBSD 了
====================================================

這幾天要開始 Build FreeBSD Kernel 了，
可是 buildworld 跟 buildkernel 都需要花費非常久的時間。

因此想要研究一下是否能加速 FreeBSD Kernel 的編譯速度，
加快開發測試的流程，以及減少 CI 所需要花費的時間。

.. TEASER_END

ccache
----------------------------------------------------

`ccache`_

第一個想到的工具是之前有使用過的 ccache，
在前公司的時候，因為編譯過程非常複雜 (寫 build script 的人也藏了很多東西)，
很難透過修改 build script 去加速建置流程。

只好先試著透過 ccache 來加速，後來又加上 ramdisk 來放置 cache，
終於將編譯時間從 15 分鐘縮短到 1 分多鐘完成。
(但是有同事跟我說他這樣就沒以煮咖啡的時間了 ...)

所以我對 ccache 印象還蠻良好的，加上影響又低。
所以首先就用他來加速看看啦~~

安裝 ccache
+++++++++++++++++++++++++++++++++++++++++++++++++++++

FreeBSD 可以使用 pkg_add 無腦安裝，也可以到 /usr/ports/devel/ccache 手動編譯安裝。

.. code-block:: bash

    pkg_add -r ccache

修改 /etc/make.conf
+++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: bash

    #CCACHE
    .if !defined(NO_CCACHE)
    CC= /usr/local/libexec/ccache/world/cc
    CCX= /usr/local/libexec/ccache/world/c++
    .endif

    .if ${.CURDIR:M*/ports/devel/ccache}
    NO_CCACHE=yes
    .endif

修改 .zshrc 或者是 .profile
+++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: bash

    export PATH=/usr/local/libexec/ccache:$PATH
    export CCACHE_PATH=/usr/bin:/usr/local/bin
    export CCACHE_DIR=/var/tmp/ccache
    export CCACHE_LOGFILE=/var/log/ccache.log

ccache 指令
-----------------------------------------------------

Display statistics summary

    ccache -s

To zero statistics

    ccache -z

To set ccache temp size ( for example 512MB )

    ccache -M 512m

參考資料
-----------------------------------------------------

`How to enable ccache on FreeBSD 8.2`_

.. _ccache: http://ccache.samba.org/
.. _How to enable ccache on FreeBSD 8.2: http://blog.up-link.ro/freebsd-how-to-enable-ccache-on-freebsd-8-2/

