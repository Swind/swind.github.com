+++
draft = false 
description = "ADB 的傳輸格式"
tags = ["android","adb"]
categories = ["program"]
date = "2017-07-11T10:46:29+08:00"
title = "ADB 傳輸格式的筆記"
+++

緩慢補充中...

## ADB 傳輸格式

ADB 的傳輸內容大致上可以分為幾類

1. host
2. host-serial
3. host-transport

<!--more-->

當你想透過 ADB 對某個 Device 下指令的時候，需要先使用 `host:transport:{serialno}` 來切換目標。
而 `host` 底下的指令則是針對 ADB Server 的指令，所以不需要進行 `host:transport`。

### host

#### connect

`host:connect:{host}:{port}`

e.g.

`host:connect:192.168.1.63:5555`

#### devices

`host:devices`

#### disconnect 

`host:disconnect:{hostla}:{port}`

#### kill

`host:kill``

#### track-devices



### transport

### version


整個 Protocol 的指令一共有以下幾種

```python
class Protocol:
    OKAY = 'OKAY'
    FAIL = 'FAIL'
    STAT = 'STAT'
    LIST = 'LIST'
    DENT = 'DENT'
    RECV = 'RECV'
    DATA = 'DATA'
    DONE = 'DONE'
    SEND = 'SEND'
    QUIT = 'QUIT'

    @staticmethod
    def decode_length(length):
        return int(length, 16)

    @staticmethod
    def encode_length(length):
        return "{0:04X}".format(length)

    @staticmethod
    def encode_data(data):
        b_data = data.encode('utf-8')
        b_length = Protocol.encode_length(len(b_data)).encode('utf-8')
        return b"".join([b_length, b_data])
```

# 參考資料

1. [adbkit](https://github.com/openstf/adbkit)