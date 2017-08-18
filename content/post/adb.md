+++
draft = false 
description = "ADB 的傳輸格式"
tags = ["android","adb"]
categories = ["program"]
date = "2017-07-11T10:46:29+08:00"
title = "ADB 傳輸格式的筆記"
+++

緩慢補充中...

<!--more-->

## ADB 傳輸格式

在傳送指令給 `ADB Server` 的時候，
需要在開頭使用 4 個 16進位的數字表示後面要傳遞的資料長度。

例如要列出所有 Devices 的時候，
所使用的指令是 `host:devices`。
所以加上長度資訊就會變成 `000Chost:devices`。

`host:devices` 是 12 個字所以用 16 進位表示長度就是 `000C`。

下面是 Python 負責轉換指令的範例程式碼

```python
def encode(cmd):
  length = "{0:04X}{}".format(len(cmd))
  return "{}{}".format(length, cmd)

if __name__ == "__main__":
  print(encode("host:devices"))
```

而 ADB Server 在收到指令之後，會立刻回傳一個長度為 4 bytes 的結果。
這個結果指的是**傳送指令的結果**，而不是**執行指令的結果**。
如果接收指令沒有任何問題的話，會回傳 `OKAY`。

如果前 4 個 bytes 不是 `OKAY` 的話，
那麼他後面就會帶著 Error Message，要記得把他一起讀完。

## ADB 的指令

ADB 的傳輸內容大致上可以分為幾類

1. host
2. host-serial
3. host-transport


當你想透過 ADB 對某個 Device 下指令的時候，需要先使用 `host:transport:{serialno}` 來切換目標。
而 `host` 底下的指令則是針對 ADB Server 的指令，所以不需要進行 `host:transport`。

### host

#### connect

讓 ADB Server 連線到位於 {host}:{port} 的 Device ( 需要設定 Device 讓他的 Adb daemon 可以透過網路連接 )

`host:connect:{host}:{port}`

e.g.

`host:connect:192.168.1.63:5555`

#### devices

列出所有 Devices，也就是 `adb devices`

`host:devices`

#### disconnect 

中斷與某台 Device 的連線

`host:disconnect:{hostla}:{port}`

#### kill

叫 Adb Service 自殺

`host:kill`

#### track-devices

`host:track-devices`

將這個連線轉為專門 track devices 的連線，當 ADB Server 發現有 Device 被新增/移除的時候。
就會將最新的 Device List 傳送過來。

#### transport

`host:track-devices:<serial-number>`

transport 中最重要的指令，跟 ADB Service 說之後將這條連線的指令全都送到 `serial-number` 這台機器或者模擬器上面。

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
2. [adb protocol](https://android.googlesource.com/platform/system/core/+/master/adb/protocol.txt)
3. [adb services](https://android.googlesource.com/platform/system/core/+/master/adb/SERVICES.TXT)
