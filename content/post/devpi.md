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

將`ip`, `port` 與 `path` 替換成自家 server 的位置。

```config
[global]
index-url = http://{ip}:{port}/{path}/+simple/
trusted-host = {ip} 

[search]
index = http://{ip}:{port}/cloudmosa/dev/
```
