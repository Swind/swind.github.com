+++
categories = ["program"]
date = "2017-04-12T14:40:30+08:00"
description = "這是我用來執行 Android 測試用的..."
draft = false 
tags = ["python", "docker", "adb"]
title = "建立可以執行 python3 與 adb 指令的 docker images"
toc = true
+++

## Python3 + adb

這個 Image 使用 Alpine Linux Image 安裝了 Python3 與 ADB。
整體大小約 9x MB，用於執行 Python3 撰寫的 Android 測試。

整個 `Dockerfile` 是各家 Dockerfile 的大雜燴。感謝 

- [frol/docker-alpine-python3](https://github.com/frol/docker-alpine-python3), 
- [sorccu/docker-adb](https://github.com/sorccu/docker-adb) 
- [cdrx/docker-pyinstaller](https://github.com/cdrx/docker-pyinstaller)

<!--more-->

## 原始碼與 Docker Image

- [Github: swind/docker-python3-adb](https://github.com/Swind/docker-python3-adb)
- [Docker-Hub: swind/docker-python3-adb](https://hub.docker.com/r/swind/docker-python3-adb)

### 使用方式

#### 執行 ADB

```sh
docker run --rm -ti --net host swind/docker-python3-adb adb devices
```

`sorccu/docker-adb` 的 container 內容本來是預設會啟動 `adb server`，但這部份被我拿掉了。
因為我想讓 container 內的 adb 可以直接跟 Host 的 adb server 溝通。
所以在執行的時候要增加 `--net host` 讓 container 內的 adb 與 Host 的 adb server 可以直接連線。

#### 執行 Python3 程式

使用這個 image 進行 build 的時候，會自動載入 `requirements.txt` 並且安裝（也就是 Dockerfile 中 `ONBUILD` 的部份）。
並且在執行的時候預設 `WORKDIR` 的路徑是 `/code`，所以需要使用 `-v` 將要執行的 Python project 掛載到 `/code`。

```sh
docker run -v "$(pwd):/code" --rm -ti --net host swind/docker-python3-adb python3 helloworld.py 
```

``` python
import os
print("Hello World")
```

#### License

See [LICENSE](LICENSE).



