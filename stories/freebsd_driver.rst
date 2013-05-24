.. title: FreeBSD Driver 開發筆記 
.. slug: freebsd-driver-dev
.. date: 2013/03/05 10:16:20
.. tags: FreeBSD
.. link: 
.. description: FreeBSD Driver 開發筆記

莫名其妙進入了 FreeBSD Driver 領域 Orz 說好的 Java 呢 ? T_T

參考資料
--------------------------------------

FreeBSD Device Drivers

先別管這個了，你有聽過 Hello World 嗎 ?
------------------------------------------------

.. code-block:: c

	#include <sys/param.h>
	#include <sys/module.h>
	#include <sys/kernel.h>
	#include <sys/systm.h>                                                                                                                                                                                                                                               

	static int 
	hello_modevent(module_t mod __unused, int event, void *arg __unused)
	{
	    int error = 0;
	    
	    switch(event){
	        case MOD_LOAD:
	            uprintf("Hello, world!\n");
	            break;
	        case MOD_UNLOAD:
	            uprintf("Good-bye, cruel world!\n");
	            break;
	        default:
	            error = EOPNOTSUPP;
	            break;
	    }   

	    return (error);
	}

	static moduledata_t hello_mod = { 
	    "hello",
	    hello_modevent,
	    NULL
	};

	DECLARE_MODULE(hello, hello_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

.. code-block:: makefile

	KMOD= hello
	SRCS= hello.c

	.include <bsd.kmod.mk>

這邊不得不說 FreeBSD 在開發 Kernel Module 的時候比 Linux 方便很多。
連 Makefile 的設都定幫你準備好。

在 include "bsd.kmod.mk" 之後，可以使用

	make load

與

	make unload

來代替 kldload ./hello.ko 與 kldunload hello.ko。

Note::

	.. <sys/systm.h>:: 是 systm.h 不是 system.h

Hello Character Drivers
-------------------------------------

struct cdevsw 定義在 <sys/conf.h> 之中。


Common Access Method
-------------------------------------

FreeBSD CAM implementation contains SIMs for

1. SCSI Parallel Interface (SPI)
2. Fibre Channel (FC)
3. USB Mass Storage (UMASS)
4. FireWire (IEEE 1394)
5. Advanced Technology Attachment Packet Interface (ATAPI)

It has peripheral modules

1. disks (ds)
2. CD-ROMs (cd)
3. tapes (sa)
4. tape changers (ch)
5. processor type devices (pt)
6. enclosure services (ses)

Also, it provides a "pass-through" interface that allows user applications to send I/O requests directly to any CAM-controlled device.

縮寫翻譯
----------------------------------

CTL (CAM target layer)

http://www.youtube.com/watch?v=eJv7GQ9rfDo

