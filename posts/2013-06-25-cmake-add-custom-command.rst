.. title: CMake - add_custom_command
.. slug: cmake-add_custom_command
.. date: 2013/06/25 19:20:46
.. tags: CMake, C/C++
.. link: 
.. description: 

最近因為必須要示範如何將公司的 autoconf 與 automake 轉成 CMake，所以稍微研究了一下之前完全沒有用過得 `add_custom_command`_ 跟 `add_custom_target`_

.. _add_custom_command: http://www.cmake.org/cmake/help/cmake2.6docs.html#command:add_custom_command
.. _add_custom_target:  http://www.cmake.org/cmake/help/cmake2.6docs.html#command:add_custom_target 

建立 symbolic link
-------------------------------------------

目前有一個需求是要建立 symbolic link 在某些特定的資料夾裡面。
由於 autotools 是可以直接使用 script 的語法，所以建立起來非常簡單。

.. code-block:: bash

   install-exec-hook:        
        ln -fs $(source_prefix)/a $(target_prefix)/path1
        ln -fs $(source_prefix)/a $(target_prefix)/path2
        ln -fs $(source_prefix)/a $(target_prefix)/path3
        ln -fs $(source_prefix)/a $(target_prefix)/path4
        ln -fs $(source_prefix)/a $(target_prefix)/path5

這樣只要直接呼叫 make install-exec-hook 就可以建立 symbolic link 了。
但是如果要在 CMake 裡面作到一樣的事情要怎麼處理呢？

.. code-block:: cmake

    Set(targets
        ${target_prefix}/path1
        ${target_prefix}/path2
        ${target_prefix}/path3
        ${target_prefix}/path4
        ${target_prefix}/path5
    )

    Function(Create_symbolic_link source target)
        Add_custom_command(
            OUTPUT ${target}
            COMMAND ln -fs ${source} ${target}
        )
    Endfunction()

    Foreach(target targets)
       Create_symbolic_link(${source_prefix}/a ${target}) 
    Endforeach()

    Add_custom_target(install-exec-hook DEPENDS ${targets})

因為 CMake 裡面沒有辦法像 shell script 一樣直接下指令，必須透過 Add_custom_command 或者是 Execute_process 等方式執行。
但是若又要跟 Target 建立相依性的話，似乎就一定得靠 Add_custom_command，因為他會設定一個 output file。
這樣就可以讓 Target 依賴這個檔案 ( 以我們的例子來說 Target 就是 install-exec-hook ) 來達到 make Target 就可以建立 symbolic link 的目的。

閃死人不償命的組合 ...

.. figure:: https://dl.dropboxusercontent.com/u/15537823/Blog/banner-of-the-stars_2.jpg 

.. raw:: html

	<blockquote>
	<p>星たちよ</p>
	<p>汝の命短き眷族の望みを聞くがよい。</p>
	<p>我らの望みヽ</p>
	<p>それは</p>
	<p>汝の本降ちゆくを看取ること</p>
	</blockquote>

