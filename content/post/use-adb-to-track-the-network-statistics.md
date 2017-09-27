+++
draft = false 
description = "使用 adb 來追蹤查看 APP 的網路流量"
tags = ["android","adb"]
categories = ["android"]
date = "2017-07-12T10:46:29+08:00"
title = "使用 adb 來追蹤查看 APP 的網路流量"
+++

## 使用 /proc 底下的資訊

雖然 Android 7 之後似乎一般應用程式已經沒有辦法存取 `/proc` 底下的東西了。 但是使用 ADB 的話還是沒有問題的。

這次我們使用的目標是

1. `/proc/uid_stat/{uid}/tcp_rcv` 
1. `/proc/uid_stat/{uid}/tcp_snd`

`tcp_rcv` 與 `tcp_snd` 分別所紀錄的是該 uid 所使用的 `TCP` 傳輸量（注意 `UDP` 並不包含在內)

<!--more-->

## 取得 APP 的 uid

取得 APP 的 uid 比較方便的方式有兩種

### 啟動 APP 透過 ps 取得 pid 再從 /proc 來查詢 uid

1. 啟動 APP
2. 執行 `adb shell ps | grep {app 的 package 關鍵字}`，例如 `adb shell ps | grep chrome` 就可以查詢到 Chrome 的 `pid`

```bash
> adb shell ps | grep chrome

u0_a96    25910 460   1089096 104992 sys_epoll_ 00000000 S com.android.chrome
u0_a96    25960 460   1011240 65940 sys_epoll_ 00000000 S com.android.chrome:privileged_process0
u0_i1     26047 460   1045356 65916 sys_epoll_ 00000000 S com.android.chrome:sandboxed_process0
```

我們只要使用只有 `package name` 的 `pid` 就可以了。以此為例，就是 `25910`

3. 透過 `adb shell cat /proc/{pid}/status` 查詢 `uid`

```bash
> adb shell cat /proc/25910/status

 Name:   .android.chrome
State:  S (sleeping)
Tgid:   25910
Pid:    25910
PPid:   460
TracerPid:      0
Uid:    10096   10096   10096   10096
Gid:    10096   10096   10096   10096
FDSize: 256
Groups: 3003 9997 50096
VmPeak:  1590940 kB
VmSize:  1068072 kB
VmLck:         0 kB
VmPin:         0 kB
VmHWM:    130672 kB
VmRSS:    114316 kB
VmData:   180240 kB
VmStk:      8192 kB
VmExe:        20 kB
VmLib:    107956 kB
VmPTE:       424 kB
VmSwap:        0 kB
Threads:        49
SigQ:   0/11726
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000001204
SigIgn: 0000000000000000
SigCgt: 00000002000094f8
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: fffffff000000000
Cpus_allowed:   f
Cpus_allowed_list:      0-3
voluntary_ctxt_switches:        171855
nonvoluntary_ctxt_switches:     22283
```

從上面 `Uid` 欄位可以得知 Chrome 的 `uid` 是 `10096`

### 透過 dumpsys 與 package name 查詢

`adb shell "dumpsys package {package 名稱} | grep userId"`

```bash
> adb shell "dumpsys package com.android.chrome | grep userId"

    userId=10096
```

## 透過 `/proc/uid_stat/{uid}` 查詢 TCP 的傳輸量

**透過 TCP 送出的流量 (bytes)**

```bash
> adb shell cat /proc/uid_stat/10096/tcp_snd

11120
```

**透過 TCP 接收的流量 (bytes)**

```bash
> adb shell cat /proc/uid_stat/10096/tcp_rcv

597160
```

## 透過 `/proc/net/xt_qtaguid/stats` 查詢傳輸量

有些 Device 沒有 `uid_stat` (例如 Galaxy S8+) 所以另外一個方法就是使用 `xt_qtaguid` 來查詢。
這個方法一樣要使用到 `uid`, 指令為

`adb shell cat /proc/net/xt_qtaguid/stats | grep {uid}`

e.g.

```bash
> adb shell cat /proc/net/xt_qtaguid/stats | grep 10029

72 tun0 0x0 10229 0 1627251 1728 123455 1525 1627251 1728 0 0 0 0 123455 1525 0 0 0 0
73 tun0 0x0 10229 1 341358394 381688 21708316 286335 341358394 381688 0 0 0 0 21381218 281971 327098 4364 0 0
288 wlan0 0x0 10229 0 387926 4640 506630 4639 387926 4640 0 0 0 0 506630 4639 0 0 0 0
289 wlan0 0x0 10229 1 63445949 170110 13704545 156389 63445949 170110 0 0 0 0 13572037 154582 132508 1807 0 0
488 wlan0 0xffffff0400000000 10229 0 18079 17 1444 14 18079 17 0 0 0 0 1444 14 0 0 0 0
489 wlan0 0xffffff0400000000 10229 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
504 lo 0x0 10229 0 17712 372 12376 238 17712 372 0 0 0 0 12376 238 0 0 0 0
505 lo 0x0 10229 1 297993712 91716 298233535 92271 297993712 91716 0 0 0 0 298233535 92271 0 0 0 0
```

以空白作為分隔，每一欄的數值所代表的意思為

`idx iface acct_tag_hex uid_tag_int cnt_set rx_bytes rx_packets tx_bytes tx_packets rx_tcp_bytes rx_tcp_packets rx_udp_bytes rx_udp_packets rx_other_bytes rx_other_packets tx_t cp_bytes tx_tcp_packets tx_udp_bytes tx_udp_packets tx_other_bytes tx_other_packets`

我們所需要的只有 `tx_bytes` 與 `rx_bytes` 

如果只有需要 TCP/UDP 的話也可以順便查詢 `tx_udp_bytes` 或 `tx_tcp_bytes` 與 `rx_udp_bytes` 或 `rx_tcp_bytes`。

所以修改一下剛剛的指令，增加 `cut` 來取得我所想要的欄位

`adb shell cat /proc/net/xt_qtaguid/stats | grep {uid} | cut -d " " -f 2,3,4,5,6,8`

```
> adb shell cat /proc/net/xt_qtaguid/stats | grep 10229 | cut -d " " -f 2,3,4,5,6,8

wlan0 0x0 10096 0 0 0
wlan0 0x0 10096 1 15659531 414473
```

1. `iface` 代表所使用的網路界面，例如 `tun0` 應該就是 VPN
1. `acct_tag_hex` 開發人員可以使用 `TrafficStats.setThreadStatsTag` 標記某個 Thread，這樣之後就可以在 xt_qtaguid 裡面依照 `acct_tag_hex` 來區分是哪個 Thread 所使用的流量。
1. `uid`
1. `cnt_set` 0 代表 APP 的背景程式， 1 代表 APP 的前景程式
1. `rx_bytes` 代表所接收的 bytes 數
1. `tx_bytes` 代表送出去的 bytes 數

## 參考資料

1. [Tracking an application's network statistics (netstats) using ADB](https://stackoverflow.com/questions/12904809/tracking-an-applications-network-statistics-netstats-using-adb)
1. [Android 獲得 UID 的三個辦法](http://www.cnblogs.com/FallenHWer/p/3359359.html)
1. [Android 流量優化(一): 模塊化流量統計](http://www.voidcn.com/blog/focusjava/article/p-6152550.html)
