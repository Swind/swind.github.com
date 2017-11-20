+++
title = "用 devpi 建立私有的 pypi server"
tags = ["python", "pypi"]
categories = ["program"]
draft = false 
description = "用 devpi 建立私有的 pypi server"
date = "2017-09-20T11:16:26+08:00"

+++

簡短作個筆記

# Dockerfile

[docker-devpi](https://github.com/Swind/docker-devpi) fork 自 [saily/openshift-devpi](https://github.com/saily/openshift-devpi)
不過如果可以的話我希望有機會可以改用 `alpine`。

<!--more--> 

```yaml
FROM python:3.6

ENV DEVPI_SERVERDIR=/mnt/server \
    DEVPI_CLIENTDIR=/mnt/client \
    DEVPI_MIRROR_CACHE_EXPIRY=86400

COPY ["requirements.txt", "logger_cfg.json", "run.sh", "/"]

RUN pip install --no-cache-dir -r /requirements.txt && \
    rm /requirements.txt

VOLUME /mnt
EXPOSE 3141

CMD "/run.sh"
```
CMD 所執行的 `run.sh` 在 repository 裡面有。

# pip.conf

`pip.conf` 的位置在 `~/.pip/pip.conf`

將`ip`, `port` 與 `path` 替換成自家 server 的位置。

```config
[global]
index-url = http://{ip}:{port}/{path}/+simple/
trusted-host = {ip} 

[search]
index = http://{ip}:{port}/cloudmosa/dev/
```

# devpi client 指令說明

參考 devpi 官網的 [Quickstart: uploading, testing, pushing releases](https://devpi.net/docs/devpi/devpi/stable/%2Bd/index.html)

## 安裝 devpi client

```sh
pip install -U devpi-client
```

## 指定 devpi server

```sh
devpi use http://localhost:3141
```

```
using server: http://localhost:3141/ (not logged in)
no current index: type 'devpi use -l' to discover indices
~/.pydistutils.cfg     : http://localhost:4040/alice/dev/+simple/
~/.pip/pip.conf        : http://localhost:4040/alice/dev/+simple/
~/.buildout/default.cfg: http://localhost:4040/alice/dev/+simple/
always-set-cfg: no
```

## 建立 User 與 Login

```sh
devpi user -c testuser password=123
devpi login testuser --password=123
```

## 建立與指定要使用的 index

我記得我在這邊有遇到權限的問題，看起來使用者權限的部份還需要詳細讀一下文件。

```sh
devpi index -c dev bases=root/pypi
devpi use testuser/dev
```

