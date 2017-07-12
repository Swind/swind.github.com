+++
tags = ["jenkins","html", "docker"]
categories = ["jenkins"]
date = "2017-04-18T11:51:34+08:00"
title = "使用 docker 與 docker-compose 啟動 Jenkins"
draft = false 
description = "使用官方的 Jenkins Docker Image 結合 docker-compose 快速啟動 Jenkins"
+++

## Docker Hub - Jenkins

[Docker Hub -Jenkins](https://hub.docker.com/_/jenkins/) 上其實文章已經很完整了。
這邊只是把我整理成 docker-compose 的過程紀錄一下。

<!--more-->

## 在 Docker Jenkins 內使用 Docker

有點饒舌，但是其實就只是要讓 Jenkins 的 Master 可以使用 Docker 指令而已。
當然更好的作法是使用 `Slave` 外部也就是 Host 的機器上負責需要使用 Docker 的 Job。

所以這邊紀錄的不是很正統的執行方式。

### 在 Jenkins docker image 裡面安裝 docker-compose 指令

建立一個新的 `Dockerfile` 來建立新的包含 Docker 與 docker-compose 指令的 Jenkins Images。

```
FROM jenkins:latest

USER root
RUN apt-get update
RUN apt-get install -y sudo
RUN rm -rf /var/lib/apt/lists/*
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
RUN curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN ls /usr/local/bin
RUN chmod +x /usr/local/bin/docker-compose
```

### 啟動 Jenkins 的 docker-compose.yml

```yaml
version: '2'
services:
  jenkins:
    build: . 
    ports:
      - "80:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker 
    environment:
      - JAVA_OPTS='-Dhudson.model.DirectoryBrowserSupport.CSP=""'
```

這個設定檔的重點只有在 `/var/run/docker.sock:/var/run/docker.sock` 與 `/usr/bin/docker:/usr/bin/docker`。
由於 docker daemon 並不是執行在 container 裡面，並且在 container 裡面在啟動一個 container 怎麼想都不合理。

所以這邊的作法其實是把 Host 的 Docker 環境塞進去 Container 裡面讓他可以操作而已。

之後這個 Container 裡面的 Master 應該就可以使用 Docker 與 docker-compose 指令了
