+++
date = "2017-09-26T17:53:21+08:00"
title = "使用 ADB 來監控 APP 的 CPU 使用量"
tags = ["android","adb", "python"]
categories = ["android"]
draft = false 
description = "使用 ADB 來監控 APP 的 CPU 使用量"
+++

其實在 `Linux` 上監控 CPU 的方法在 `Android` 上面也可以用，並且這些方法在很多人的 Blog 與 StackOverflow 也被討論到爛掉了。
這邊主要是紀錄如何使用 `adb` 與 `/proc` 取得 `Process` 與 `Thread` 的 CPU 資訊。

<!--more--> 

# 整體的 CPU 使用量

要取得整個 Device 的 CPU 資訊的話，可以透過 `/proc/stat`。

使用指令 `adb shell cat /proc/stat` 會看到

```
cpu  1286215 71763 779091 4271301 16007 9 16100 0 0 0
cpu0 560338 33626 409552 2845856 9346 0 12458 0 0 0
cpu1 80441 6912 40265 146206 1396 4 379 0 0 0
cpu2 174491 9981 89521 383939 1992 3 838 0 0 0
cpu3 470945 21244 239753 895300 3273 2 2425 0 0 0
intr 56252464 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 11183406 0 0 0 0 0 0 0 0 0 0 0 0 0 165872 1 0 0 0 0 0 2426472 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2184 8 5 0 10 0 0 0 2025880 0 0 0 0 0 0 0 0 0 0 384654 0 0 0 0 5 689 3 2 3 0 0 0 2 2 0 0 0 0 0 0 0 0 0 0 0 0 0 2283211 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 91049 1142156 0 0 4891 0 0 0 0 60 0 6 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 632514 0 12 0 0 0 0 0 5607578 0 0 0 0 0 0 365 0 0 0 36028 5 0 2027914 2134764 0 0 0 0 0 0 0 0 0 2043721 0 5 0 0 0 0 748 263 0 0 0 1584739 0 0 75 0 0 0 0 0 0 0 0 0 0 0 0 11817 0 0 0 0 0 207 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 7067 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 10 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 0 0 0 0 3 0 0 0 0 0 0 0 0 0 0 0 0 0 2 2 16 0 0 0 0 86 0 0 0 0 0 0 0 0 0 0 0 5062 481 1198 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 100 215 12 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
ctxt 198100180
btime 1506325560
processes 344110
procs_running 1
procs_blocked 0
softirq 30624059 14285 6147433 22588 1777078 14285 14285 400466 3969871 2347 18261421
```

像密碼表一樣的資訊，不過我們可以只著重在第一行的 `cpu` 即可。

```
cpu  1286215 71763 779091 4271301 16007 9 16100 0 0 0
```

每個欄位代表的意思是

|     | user    | nice  | system | idle    | iowait | irq | softirq | stealstolen | guest | guest_nice |
| --- | ------- | ----- | ------ | ------- | ------ | --- | ------- | ----------- | ----- | ---------- |
| cpu | 1286215 | 71763 | 779091 | 4271301 | 16007  | 9   | 16100   | 0           | 0     | 0          |

這些數字都是從 Device 啟動之後的累計值, 單位是 `jiffies`，
因此計算在某段時間內的值的話。
就必須在起始跟結束的時候都呼叫一次 `adb shell cat /proc/stat`，
並且把所有得到的數字相減。

例如

開始的時候取得

```
cpu  1430106 74780 838424 4391260 16309 9 16780 0 0 0
```

結束的時候取得

```
cpu  1431109 74806 838854 4392567 16313 9 16783 0 0 0
```

那麼這段時間總共是 `2773 jiffies`
```
# after - before
(1431109 + 74806 + 838854 + 4392567 + 16313 + 9 + 16783 + 0 + 0 + 0) - (1430106 + 74780 + 838424 + 4391260 + 16309 + 9 + 16780 + 0 + 0 + 0) 
= 2773
```


# Process 的 CPU 使用率

有了整體之後，再來就是針對 `Process` , 計算 `Process` 之前需要先取得 `Process` 的 `PID`。
最簡單的方法就是使用 `ps` 來取得。

```
adb shell ps | grep {package name}

u0_a174   11458 460   1781352 365668 sys_epoll_ 00000000 S com.android.chrome
```

第二個 `11458` 就是 `PID`

有了 PID 之後, 就可以透過 `/proc/{PID}/stat` 取得 Process 的 CPU 使用時間

```
adb shell cat /proc/11458/stat

11458 (mosa.puffinFree) S 460 460 0 0 -1 4194624 13380248 0 334 0 120663 47337 0 0 16 -4 133 0 9367583 1535365120 62981 4294967295 1 1 0 0 0 0 4612 0 38648 4294967295 0 0 17 3 0 0 0 0 0 0 0
```

Process 的資料量明顯的多很多，可以參考 [PROC(5)](http://man7.org/linux/man-pages/man5/proc.5.html) 中的 `/proc/[pid]/stat` 來取得每個欄位的意思。
這邊我們只要注意第 14 跟第 15 個。

```
 (14) utime  %lu
        Amount of time that this process has been scheduled
        in user mode, measured in clock ticks (divide by
        sysconf(_SC_CLK_TCK)).  This includes guest time,
        guest_time (time spent running a virtual CPU, see
        below), so that applications that are not aware of
        the guest time field do not lose that time from
        their calculations.

(15) stime  %lu
        Amount of time that this process has been scheduled
        in kernel mode, measured in clock ticks (divide by
        sysconf(_SC_CLK_TCK)).
```

將這兩個值相加就是 `到目前為止` Process 所使用的 CPU 時間。
所以跟前面一樣，需要在開始與結束各取一次，並且**相減**。

# Thread

在 `/proc/{PID}/task` 底下會有一堆名稱是數字的資料夾

```
adb shell ls /proc/11458/task

10143
11458
11463
11465
11466
11468
.
.
.
```

每個數字都是這個 Process 底下 Thread 的 `tid`, 在這些資料夾裡面也有一個 `stat` 的檔案。
所以跟 Process 一樣只要使用

```
adb shell cat /proc/11458/task/10143/stat

10143 (OkHttp Connecti) S 460 460 0 0 -1 4194368 267 0 0 0 0 1 0 0 20 0 129 0 9491883 1599287296 67669 4294967295 1 1 0 0 0 0 4612 0 38648 4294967295 0 0 -1 3 0 0 0 0 0 0 0 0
```

就會取得跟 Process 一樣格式的內容，我們一樣需要注意 `utime` 與 `stime` 但是除此之外還需要注意一個部份。
就是第一個欄位的 tid `10143` 與 thread name `(OkHttp Connecti)`。
由於 thread name 可能會重複，但是 tid 會不相同，所以最好是將 tid 與 thread name 合成一個 key 比較不容易統計錯誤。

將這兩個值相加就是 `到目前為止` Thread 所使用的 CPU 時間。
所以跟前面一樣，需要在開始與結束各取一次，並且**相減**。

# CPU 使用率

就將前面透過 `/proc/stat` 取得的值當分母，去計算 Process 與個 Thread 的百分比就可以了。