.. title: Setup iSCSI in FreeBSD 
.. slug: setup-iscsi-in-freebsd
.. date: 2013/05/21 18:00:00
.. tags: FreeBSD
.. link: 
.. description: How to setup the iSCSI initiator in FreeBSD 

.. class:: alert alert-info pull-right

.. contents::

PiSCSI Development Environment
=============================================

OS
--------------------------------------------

:Release Note:

`FreeBSD 9.1-RELEASE <http://www.freebsd.org/releases/9.1R/announce.html>`_

:LXR:

`LXR <http://fxr.watson.org/fxr/source/?v=FREEBSD91>`_

:ISO:

`FreeBSD 9.1-RELEASE ISO Download <http://www.freebsd.org/where.html>`_

Tools
--------------------------------------------

`CMake <http://www.cmake.org/>`_

CMake is used to control the software compilation process using simple platform and compiler independent configuration files.
CMake generates native makefiles and workspaces that can be used in the compiler environment of your choice.

**Install**

    pkg_add -r cmake

`Git <http://git-scm.com/>`_

Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

**Install**

    pkg_add -r git

Compiler
--------------------------------------------

Clang and Clang++

The goal of the Clang project is to create a new C, C++, Objective C and
Objective C++ front-end for the LLVM compiler.

::

    clang version 3.2 (tags/RELEASE_32/final)
    Target: x86_64-unknown-freebsd9.0
    Thread model: posix

**Install**

    pkg_add -r clang

Libraries
------------------------------------------

`CppUTest <http://www.cpputest.org/>`_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CppUTest is a C /C++ based unit xUnit test framework for unit testing and for test-driving your code.
It is written in C++ but is used in C and C++ projects and frequently used in embedded systems.

CppUTest has a couple design principles
* Simpl to use and small
* Portable to old and new platforms

**Install**

    git clone https://github.com/cpputest/cpputest.git
    cd cpputest
    cmake .
    make install

`Google Perftools <https://code.google.com/p/gperftools/?redir=1>`_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These tools are for use by developers so that they can create more robust applications. 
Especially of use to those developing multi-threaded applications in C++ with templates. 
Includes TCMalloc, heap-checker, heap-profiler and cpu-profiler.

**Install**

    pkg_add -r google-perftools

`LibConfig <http://www.hyperrealm.com/libconfig/>`_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Libconfig is a simple library for processing structured configuration files
This file format is more compact and more readable than XML. And unlike XML, it is type-aware, so it is not necessary to do string parsing in application code.

**Install**

    pkg_add -r libconfig

Compile the Source Code
======================================================================

Checkout the source code from GitLab
---------------------------------------------------------------------

.. code-block:: bash

    git clone git@gitlab:Pi-Coral/piscsi.git

::

    ╭─root@SwindBSD9  ~/Program/tmp  
    ╰─$ git clone git@gitlab:Pi-Coral/piscsi.git
    Cloning into 'piscsi'...
    The authenticity of host 'gitlab (192.168.1.56)' can't be established.
    ECDSA key fingerprint is f4:ce:20:e4:66:a8:d1:04:da:a6:e5:0c:0b:ea:70:57.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'gitlab' (ECDSA) to the list of known hosts.
    remote: Counting objects: 1202, done.
    remote: Compressing objects: 100% (752/752), done.
    remote: Total 1202 (delta 823), reused 673 (delta 430)
    Receiving objects: 100% (1202/1202), 1.33 MiB | 1.86 MiB/s, done.
    Resolving deltas: 100% (823/823), done.

Use CMake to generate Makefile
---------------------------------------------------------------------

.. code-block:: bash

    cmake .

::

    ╭─root@SwindBSD9  ~/Program/tmp  
    ╰─$ cmake . 
    -- The C compiler identification is Clang 3.2.0
    -- The CXX compiler identification is Clang 3.2.0
    -- Check for working C compiler: /usr/bin/clang
    -- Check for working C compiler: /usr/bin/clang -- works
    -- Detecting C compiler ABI info
    -- Detecting C compiler ABI info - done
    -- Check for working CXX compiler: /usr/bin/clang++
    -- Check for working CXX compiler: /usr/bin/clang++ -- works
    -- Detecting CXX compiler ABI info
    -- Detecting CXX compiler ABI info - done
    -- Found Google perftools: 

    -------------------------------------------------------
    CppUTest Version .

    Current compiler options:
        CC:                                 /usr/bin/clang
        CXX:                                /usr/bin/clang++
        CppUTest CFLAGS:                    
        CppUTest CXXFLAGS:                  
        CppUTest LDFLAGS:                   

    Porject Information:
        Project Name:                       PiSCSI
        Project Source Dir:                 /root/Program/tmp/piscsi
        Project Binary Dir:                 /root/Program/tmp/piscsi
    -------------------------------------------------------

    -- Configuring done
    -- Generating done
    -- Build files have been written to: /root/Program/tmp/piscsi 

Compile the source code
---------------------------------------------------------------------

.. code-block:: bash

    make

::

    ╭─root@SwindBSD9  ~/Program/tmp/piscsi  ‹master*› 
    ╰─$ make
    -- Found Google perftools: 

    -------------------------------------------------------
    CppUTest Version .

    Current compiler options:
        CC:                                 /usr/bin/clang
        CXX:                                /usr/bin/clang++
        CppUTest CFLAGS:                    
        CppUTest CXXFLAGS:                  
        CppUTest LDFLAGS:                   

    Porject Information:
        Project Name:                       PiSCSI
        Project Source Dir:                 /root/Program/tmp/piscsi
        Project Binary Dir:                 /root/Program/tmp/piscsi
    -------------------------------------------------------

    -- Configuring done
    -- Generating done
    -- Build files have been written to: /root/Program/tmp/piscsi
    [  2%] Building C object src/CMakeFiles/iscsi_test.dir/bhs.c.o
    [  5%] Building C object src/CMakeFiles/iscsi_test.dir/pdu.c.o
    [  8%] Building C object src/CMakeFiles/iscsi_test.dir/login_manager.c.o
    [ 11%] Building C object src/CMakeFiles/iscsi_test.dir/socket_acceptor.c.o
    [ 14%] Building C object src/CMakeFiles/iscsi_test.dir/fullfeature.c.o
    [ 17%] Building C object src/CMakeFiles/iscsi_test.dir/logical_unit.c.o
    Linking C static library libiscsi_test.a
    [ 17%] Built target iscsi_test
    [ 20%] Building C object src/event/CMakeFiles/pievent.dir/ae.c.o
    [ 23%] Building C object src/event/CMakeFiles/pievent.dir/eventbus.c.o
    Linking C static library libpievent.a
    [ 23%] Built target pievent
    [ 26%] Building C object src/utils/CMakeFiles/piutils.dir/crc32.c.o
    [ 29%] Building C object src/utils/CMakeFiles/piutils.dir/param_set_list.c.o
    [ 32%] Building C object src/utils/CMakeFiles/piutils.dir/log.c.o
    [ 35%] Building C object src/utils/CMakeFiles/piutils.dir/queue.c.o
    [ 38%] Building C object src/utils/CMakeFiles/piutils.dir/misc.c.o
    Linking C static library libpiutils.a
    [ 38%] Built target piutils
    [ 41%] Building C object src/memory/CMakeFiles/pimemory.dir/pimalloc.c.o
    Linking C static library libpimemory.a
    [ 41%] Built target pimemory
    [ 44%] Building C object src/net/CMakeFiles/pinet.dir/connection.c.o
    [ 47%] Building C object src/net/CMakeFiles/pinet.dir/session.c.o
    Linking C static library libpinet.a
    [ 47%] Built target pinet
    [ 50%] Building C object src/config/CMakeFiles/piconfig.dir/portal.c.o
    [ 52%] Building C object src/config/CMakeFiles/piconfig.dir/target_info.c.o
    [ 55%] Building C object src/config/CMakeFiles/piconfig.dir/auth.c.o
    [ 58%] Building C object src/config/CMakeFiles/piconfig.dir/lun_info.c.o
    [ 61%] Building C object src/config/CMakeFiles/piconfig.dir/config.c.o
    [ 64%] Building C object src/config/CMakeFiles/piconfig.dir/config_loader.c.o
    Linking C static library libpiconfig.a
    [ 64%] Built target piconfig
    [ 67%] Building C object src/CMakeFiles/iscsid.dir/daemon.c.o
    [ 70%] Building C object src/CMakeFiles/iscsid.dir/bhs.c.o
    [ 73%] Building C object src/CMakeFiles/iscsid.dir/pdu.c.o
    [ 76%] Building C object src/CMakeFiles/iscsid.dir/login_manager.c.o
    [ 79%] Building C object src/CMakeFiles/iscsid.dir/socket_acceptor.c.o
    [ 82%] Building C object src/CMakeFiles/iscsid.dir/fullfeature.c.o
    [ 85%] Building C object src/CMakeFiles/iscsid.dir/logical_unit.c.o
    Linking C executable iscsid
    [ 85%] Built target iscsid
    [ 88%] Building CXX object tests/CMakeFiles/Runner.dir/CppUTestExample.cpp.o
    [ 91%] Building CXX object tests/CMakeFiles/Runner.dir/test_config.cpp.o
    [ 94%] Building CXX object tests/CMakeFiles/Runner.dir/test_connection.cpp.o
    [ 97%] Building CXX object tests/CMakeFiles/Runner.dir/test_utils_param_set.cpp.o
    [100%] Building CXX object tests/CMakeFiles/Runner.dir/test_emulator.cpp.o
    Linking CXX executable Runner
    [100%] Built target Runner

PiSCSI Configuration File
=============================================

We use `libconfig <http://www.hyperrealm.com/libconfig/>`_ to read/write our configuration file.

Global Environment Variables
-----------------------------------------------

The global environment variables include debug leven and some constant variables.

.. code-block:: kconfig

    version = "1.0"

    global:
    {
        #Global Debug Level: ERROR, WARN, INFO, DEBUG, NONE
        debug_level = "DEBUG";
        socket_buffer = "1048576";
    }

    logical_unit:
    {
        debug_level = "WARN";

        #Log the scsi command but exclude include READ and WRITE.
        log_scsi_command = true;
    }
    
:version:

The iSCSI target version, it is showed in sysctl.

:global:

debug_level

- ERROR
- WARN
- INFO
- DEBUG
- NONE

**default value is NONE**

:socket_buffer:

The socket buffer for every connection.

**default value is 1048576 (bytes)**

logical_unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:debug_level:

The debug level for logical_unit.c

**default value is NONE**

:log_scsi_command:

Log the SCSI Command opcode except READ and WRITE

**default value is false**

Portal
-------------------------------------------------

.. code-block:: kconfig

    portal_groups:
    (
        {
            Name        = "Group1";
            Portals     =
            (
                "192.168.1.157:3260"
            );
        }
    );

:Name:

    The portal group ID.

:Portals:

    If there are mutiple address or port which are need to be listen, you can use

.. code-block:: kconfig

    Portals =
    (
        "192.168.1.157:3260",
        "192.168.1.158:3260",
        "192.168.1.159:3260"
    )

Logical Units
----------------------------------------------------

.. code-block:: kconfig

    logical_units:
    (
        {
            Description = "Target 1";
            TargetName = "iqn.2007-09.jp.ne.peach.istgt:disk1";
            TargetAlias = "Test Target 1";
            PortalGroup = "Group1";
            Luns = 
            (
                {
                    Id = 1;
                    Path = "/dev/da1";
                    Size = "1024 MB";
                    Type = "Pass";
                }
            );
        }
    )

:Description:

Just a description

:TargetName:

iSCSI Qualified Name (IQN)

Format: The iSCSI Qualified Name is documented in RFC 3720, with further examples of names in RFC 3721. Briefly, the fields are:

- literal iqn
- date (yyyy-mm) that the naming authority took ownership of the domain
- reversed domain name of the authority (org.alpinelinux, com.example, to.yp.cr)
- Optional ":" prefixing a storage target name specified by the naming authority.

From the RFC::

                  Naming     String defined by
     Type  Date    Auth      "example.com" naming authority
    +--++-----+ +---------+ +-----------------------------+
    |  ||     | |         | |                             |     
 
    iqn.1992-01.com.example:storage:diskarrays-sn-a8675309
    iqn.1992-01.com.example
    iqn.1992-01.com.example:storage.tape1.sys1.xyz
    iqn.1992-01.com.example:storage.disk2.sys1.xyz[5]

:TargetAlias:

    Target Alias

:PortalGroup:

    Portal Group ID

:Luns: 

-Id            Lun ID;
-Path          Device path ex."/dev/da1"
-Size          Device size. Now this variable is unuseful, we directly get the disk size information from the device;
-Type          Lun type. "Pass" or "Virtual", "Pass" will redirectly the READ/WRITE to the device.

If there are two targets

.. code-block:: kconfig

    logical_units:
    (
        {
            Description = "Target 1";
            TargetName = "iqn.2007-09.jp.ne.peach.istgt:disk1";
            TargetAlias = "Test Target 1";
            PortalGroup = "Group1";
            Luns = 
            (
                {
                    Id = 1;
                    Path = "/dev/da1";
                    Size = "1024 MB";
                    Type = "Pass";
                }
            );
        }
        ,
        {
            Description = "Target 2";
            TargetName = "iqn.2007-09.jp.ne.peach.istgt:disk2";
            TargetAlias = "Test Target 2";
            PortalGroup = "Group1";
            Luns = 
            (
                {
                    Id = 2;
                    Path = "/dev/da2";
                    Size = "1024 MB";
                    Type = "Pass";
                }
            );
        }
    )

Configuration File Example
====================================================

.. code-block:: kconfig

    #PiSCSI Configuration file

    #####################################################################

    version = "1.0"

    global:
    {
        #Global Debug Level: ERROR, WARN, INFO, DEBUG, NONE
        debug_level = "DEBUG";
        socket_buffer = "1048576";
    }

    logical_unit:
    {
        debug_level = "WARN";

        #Log the scsi command from initiator but doesn't include READ and WRITE.
        log_scsi_command = true;
    }


    #####################################################################
    portal_groups:
    (
        {
            Name        = "Group1";
            Portals     =
            (
                "192.168.1.157:3260"
            );
        }
    );

    logical_units:
    (
        {
            Description = "Target 1";
            TargetName = "iqn.2007-09.jp.ne.peach.istgt:disk1";
            TargetAlias = "Test Target 1";
            PortalGroup = "Group1";
            Luns = 
            (
                {
                    Id = 1;
                    Path = "/dev/da1";
                    Size = "1024 MB";
                    Type = "Pass";
                }
            );
        }
    )

iSCSI Target and Initiator
=====================================================================

Create a Ramdisk
------------------------------------------------------------------------

Before use PiSCSI, you need have a raw disk for PiSCSI.
In order to exclude the I/O effect, we use ramdisk as the downstream disk. 

.. code-block:: bash

    #Create 512 MB ramdisk
    ctladm create -b ramdisk -s 536870912

::

    LUN created successfully
    backend:       ramdisk
    device type:   0
    LUN size:      536870912 bytes
    blocksize      512 bytes
    LUN ID:        0
    Serial Number: MYSERIAL   0
    Device ID;     MYDEVID   0
    Front End Ports enabled


Enable the ctl port

.. code-block:: bash

    ctladm port -o on

::

    Front End Ports enabled
    
You can check the ramdisk information by **ctladm devlist** and **camcontrol devlist**

.. code-block:: bash

    ctladm devlist 

::

    LUN Backend       Size (Blocks)   BS Serial Number    Device ID       
      0 ramdisk             1048576  512 MYSERIAL   0     MYDEVID   0     

.. code-block:: bash

    camcontrol devlist

::

    <NECVMWar VMware IDE CDR10 1.00>   at scbus1 target 0 lun 0 (pass0,cd0)
    <VMware Virtual disk 1.0>          at scbus2 target 0 lun 0 (pass1,da0)
    <VMware Virtual disk 1.0>          at scbus2 target 1 lun 0 (pass2,da1)
    <FREEBSD CTLDISK 0001>             at scbus3 target 1 lun 0 (da2,pass3)

The "FREEBSD CTLDISK 0001" is the ramdisk we created.

Execute PiSCSI
--------------------------------------------------------------------

Before execute the "iscsid", you need finish the configuration file and put the file with the iscsid together.

.. code-block:: bash

    ./iscsid

::

    PiSCSI daemon start...
    Read Configuration...Load config file ./piscsi.config ...
    /root/Program/tmp/piscsi/src/config/config_loader.c:  25: debug_level = DEBUG
    Debug Level = DEBUG(2147483648)
    Log SCSI Command = true
    Add Portal: 
     Name: Group1
     IP: 192.168.1.95:3260
    Add a logical unit:
     Name: iqn.2007-09.jp.ne.peach.istgt:disk1
     Alias: Test Target 1
     Portal Group: Group1
    Load config file finish...
    OK
    Start EventBus...OK
    Start FullFeature Manager...OK
    Start Login Manager...OK
    Create Socket Acceptor and add portal to Socket Acceptor...
    /root/Program/tmp/piscsi/src/daemon.c:  63: Portal Group Group1
    /root/Program/tmp/piscsi/src/daemon.c:  66: Open the portal 192.168.1.95:3260...
    /root/Program/tmp/piscsi/src/utils/misc.c:  91: Listen 192.168.1.95:3260 success, socket 5
    /root/Program/tmp/piscsi/src/event/eventbus.c:  75: Listener 192.168.1.95:3260 has been added
    OK
    /root/Program/tmp/piscsi/src/event/eventbus.c:  62: Eventbus start !

Start FreeBSD iSCSI Initiator 
-----------------------------------------------------------------

Load the iSCSI initiator module.

.. code-block:: bash

    kldload /boot/kernel/iscsi_initiator.ko
    kldstat   

::

    Id Refs Address            Size     Name
     1    6 0xffffffff80200000 1323388  kernel
     2    1 0xffffffff81615000 a268     iscsi_initiator.ko

Create the iSCSI initiator config : iscsi_initiator.config

.. code-block:: kconfig

    officeiscsi {
            authmethod    = NONE
            #chapIName    = YOUR-ISCSI-USERNAME
            #chapSecret   = YOUR-ISCSI-PASSWORD
            initiatorname = nxl
            TargetName    = iqn.XYZZZZZZZZZZZZZ #  whatever "iscontrol -v -d " gives you
            TargetAddress = 127.0.0.1:3260,1 # your iscsi server IP
    }

- officeiscsi { : Start config for iSCSI.
- authmethod : Set authentication method to chap
- chapIName : Your username
- chapSecret : Your password
- initiatorname : if not specified, defaults to iqn.2005-01.il.ac.huji.cs:<hostname>
- TargetName : is the name by which the target is known, not to be confused with target address, either obtained via the target administrator, or from a discovery session.
- TargetAddress : is of the form domainname[:port][,portal-group-tag] to quote the RFC: The domainname can be specified as either a DNS host name, a dotted-decimal IPv4 address, or a bracketed IPv6 address as specified in [RFC2732].
- } : End of config

Start iSCSI initiator

.. code-block:: bash

    iscontrol -c ./iscsi_initiator.config -n testdisk
    camcontrol devlist

Stop FreeBSD iSCSI Initiator
----------------------------------------------------------------

The iscontrol doesn't provide stop command, so we use kill to terminate the iscontrol.

.. code-block:: bash

    ps aux | grep iscontrol | awk '{ print $2 }' | xargs kill -9

And the iscsid doesn't provide too.

.. code-block:: bash

    ps aux | grep iscsid | awk '{ print $2 }' | xargs kill -9

login may create a new session or it may add a connection to an
existing session.  Between a given iSCSI Initiator Node (selected
only by an InitiatorName) and a given iSCSI target defined by an
iSCSI TargetName and a Target Portal Group Tag, the login results are
defined by the following table:

::

     +------------------------------------------------------------------+
     |ISID      | TSIH        | CID    |     Target action              |
     +------------------------------------------------------------------+
     |new       | non-zero    | any    |     fail the login             |
     |          |             |        |     ("session does not exist") |
     +------------------------------------------------------------------+
     |new       | zero        | any    |     instantiate a new session  |
     +------------------------------------------------------------------+
     |existing  | zero        | any    |     do session reinstatement   |
     |          |             |        |     (see section 5.3.5)        |
     +------------------------------------------------------------------+
     |existing  | non-zero    | new    |     add a new connection to    |
     |          | existing    |        |     the session                |
     +------------------------------------------------------------------+
     |existing  | non-zero    |existing|     do connection reinstatement|
     |          | existing    |        |    (see section 5.3.4)         |
     +------------------------------------------------------------------+
     |existing  | non-zero    | any    |         fail the login         |
     |          | new         |        |     ("session does not exist") |
     +------------------------------------------------------------------+

 Determination of "existing" or "new" are made by the target.
