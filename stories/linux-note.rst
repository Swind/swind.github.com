.. title: Linux 指令筆記 
.. slug: linux-note
.. date: 2013/01/20 10:16:20
.. tags: Linux
.. link: 
.. description: 我常用的 Linux& FreeBSD 指令筆記

這篇是永久置底 
=============================================

.. class:: alert alert-info pull-right

.. contents::

Git
-----------------------------

漂亮一點的 log 顯示方式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --"

出處 `酷殼 - Coolbash.cn - Git 顯示漂亮日誌的小技巧`_

.. _酷殼 - Coolbash.cn - Git 顯示漂亮日誌的小技巧: http://coolshell.cn/articles/7755.html

讓所有 submodule 執行 git pull
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    git submodule foreach git pull

所有 submodule checkout master
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    git submodule foreach git checkout master


sg3 utils
-----------------------------

**列出所有 scsi 硬體**

.. code-block:: bash

	sg_scan

**讀檔案**
     
.. code-block:: bash

     sg_raw.exe -v -r 512 –o read512byte.txt PD3 28 00 00 00 03 00 00 00 01 00

**寫檔案**

其中 wdata512byte 是事先寫好的一個檔案，後面的 01 代表的是要寫的 sector 數量。

.. code-block:: bash

     sg_raw -v -s 512 PD3 2a 00 00 00 01 00 00 00 01 00 < wdata512byte


Linux 系統設定
-------------------------------

**SSH login without password**

.. code-block:: bash

    #!/bin/bash

    if [ -f ~/.ssh/id_rsa.pub 0 ]
    then
            account=$1
            ip=$2
            ssh-keygen -t rsa
            ssh $account@$ip mkdir -p .ssh
            cat ~/.ssh/id_rsa.pub | ssh $account@$ip 'cat >> ~/.ssh/authorized_keys'
    fi

**使用 rz 與 sz**

.. code-block:: bash

    yum install lrzsz

**判斷 Linux 發行版本**

.. code-block:: bash

    cat /etc/issue


Eclipse 相關
-------------------------------

**搜尋**

    Shift + Ctrl + T 可以搜尋 Class

    Shift + Ctrl + R 可以搜尋檔案

**改變選擇之後會把相同名稱的選擇起來的功能的顏色**

    Mark occurences

修改的地方在

    Preferences > General > Editors > Text Editors > Annotations


GDB
-------------------------------

**使用 gdb 觀看 core file 資訊**

    gdb -c <corefile> <execfile>

**顯示所有 thread**

    info threads

**進入某個 thread**

    thread <thread number>

VIM
----------------------------------

**避免在 paste 的時候觸發 tab 功能**

    set paste

vgod 有作個快速鍵 **, + p**


tmux - The Terminal Multiplexer
---------------------------------------

- 官網 http://tmux.sourceforge.net/

- 設定檔 https://github.com/Swind/linux-config/blob/master/.tmux.conf

**指令**

.. code-block:: bash

    tmux        #啟動 tmux 並且建立新的 Session
    tmux attach #連線到已經存在的 Session

**熱鍵**

+------------+------------+-----------+ 
| Ctrl + a   | 啟動熱鍵               | 
+============+============+===========+ 
|     n      | 下一個視窗             | 
+------------+------------+-----------+ 
|     p      | 上一個視窗             | 
+------------+------------+-----------+ 
|     \|     | 在前後兩個視窗移動     | 
+------------+------------+-----------+ 
|     c      | 建立新視窗             | 
+------------+------------+-----------+ 
|     &      | 關閉視窗               | 
+------------+------------+-----------+ 

CMake
--------------------------------------------------

建立在最上層的 CMakeLists.txt，底下有 src 與 tests 分別放置原始碼與測試程式碼
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cmake

    project(ProjectName)

    set(ProjectName_version_major 0)
    set(ProjectName_version_minor 1)

    set(CMAKE_VERBOSE_MAKEFILE OFF) #Show all compile message

    # 2.6.3 is needed for ctest support
    cmake_minimum_required(VERSION 2.6.3)

    link_directories("/usr/local/lib")

    #CMake Config
    set (CMAKE_C_COMPILER "/usr/bin/clang")      #Use Clang as C default compiler
    set (CMAKE_CXX_COMPILER "/usr/bin/clang++")  #Use Clang++ as C++ default compiler

    set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMakeModules/")
    enable_testing()

    add_subdirectory(src)
    add_subdirectory(tests)

    # Show the environment variables
    message("
    -------------------------------------------------------
    ProjectName Version ${ProjectName_version_major}.${ProjectName_version_minor}

    Current compiler options:
        CC:                                 ${CMAKE_C_COMPILER}
        CXX:                                ${CMAKE_CXX_COMPILER}
        CFLAGS:                             ${CPPUTEST_C_FLAGS}
        CXXFLAGS:                           ${CPPUTEST_CXX_FLAGS}
        LDFLAGS:                            ${CPPUTEST_LD_FLAGS}

    Porject Information:
        Project Name:                       ${PROJECT_NAME}
        Project Source Dir:                 ${PROJECT_SOURCE_DIR}
        Project Binary Dir:                 ${PROJECT_BINARY_DIR}
    -------------------------------------------------------
    ")

在 src 底下的 CMakeLists.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cmake
    
    set(CMAKE_VERBOSE_MAKEFILE ON) #Show all compile message

    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Werror")

    SUBDIRS(subdirs)

    INCLUDE_DIRECTORIES(./subdirs)

    #Use Google Performance Tools
    FIND_PACKAGE(GooglePerfTools REQUIRED)
    include_directories(${GOOGLE_PERFTOOLS_INCLUDE_DIR})

    SET(sub_modules 
            sub_modules.c
    )

    SET(LinkLibraries
            sub_modules
        )

    ADD_EXECUTABLE(iscsid ${sub_modules})
    ADD_LIBRARY(iscsi_test ${LinkLibraries})

    TARGET_LINK_LIBRARIES(iscsid ${LinkLibraries})

在 tests 底下的 CMakeLists.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cmake

    #Tell cmake there are c and cpp files in this project
    project(UnitTest C CXX)

    set(CMAKE_VERBOSE_MAKEFILE OFF)
    cmake_minimum_required(VERSION 2.6.3)

    #========CMake parameters
    enable_testing()

    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMakeModules/")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g")

    include_directories("/usr/local/include")
    include_directories("${CMAKE_SOURCE_DIR}/src/config")
    link_directories("/usr/local/lib")

    #========Add libraries

    SET(LIBS 
            ${LIBS} 
            ${CHECK_LIBRARIES}
            pthread

        SOURCE
            source_modules

        TEST
            test_modules
    )

    set(SOURCE
        CppUTestExample.cpp    
    )

    include_directories(. ../src/sub_modules)
    INCLUDE_DIRECTORIES(.)

    add_executable(Runner ${SOURCE})
    target_link_libraries(Runner ${LIBS})

    add_test(Runner ${CMAKE_CURRENT_BINARY_DIR}/Runner)


