+++
date = "2018-05-09T16:12:58+08:00"
title = "使用 Docker 執行 Android Emulator 進行測試"
categories = ["android"]
draft = true
description = "使用 Docker 執行 Android Emulator 進行測試"
tags = ["android", "docker", "emulator", "testing"]

+++

這邊我使用的是 [thedrhax-dockerfiles/android-avd](https://github.com/thedrhax-dockerfiles/android-avd) 的版本。

他本身的 `README.md` 就已經寫得很詳細了，這邊是順便紀錄如何使用他進行測試

<!--more--> 

# 啟動 adb server

為了我測試的方便，我必須確認 adb server 能在第一時間開啟，讓我測試程式可以進行連線，監控 emulator 的狀況,
所以修改了 `container/entrypoint.sh`。
加入了 `adb -a -P 5037 server nodaemon&`

```
#!/bin/sh -x

echo n | $ANDROID_HOME/tools/bin/avdmanager create avd \
    -k "system-images;${TARGET};${TAG};${ABI}" \
    -n ${NAME} -b ${ABI} -g ${TAG}

(
  # Enable keyboard support in QEMU (for VNC)
  echo 'hw.keyboard = true';
) >> /root/.android/avd/Docker.avd/config.ini

adb -a -P 5037 server nodaemon&

exec $*
```

當然我想更穩定的作法應該是結合使用 `healthcheck` 來確認需要的服務（例如 Adb Server, 或者 Emulator 等)
但是目前先這樣就足夠了。

# 將只有開在 localhost 的 port 導向其他 network interface

這是 [thedrhax-dockerfiles/android-avd](https://github.com/thedrhax-dockerfiles/android-avd) 已經作好的功能。

在 `container/etc/supervisor/conf.d/supervisord.conf 中

```
[supervisord]
nodaemon=true

[program:emulator]
command=/bin/bash -c "$ANDROID_HOME/tools/emulator -avd ${NAME} -no-window -no-audio ${ANDROID_EMULATOR_EXTRA_ARGS}"
stopsignal=KILL
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:socat-5554]
command=/usr/local/bin/socat.sh 5554

[program:socat-5555]
command=/usr/local/bin/socat.sh 5555

[program:novnc]
command=/bin/bash -c '[ "_$noVNC" = "_true" ] && /home/user/noVNC/utils/launch.sh --vnc localhost:5900'
startretries=0
```

使用了 `container/usr/local/bin/socat.sh` 將 emulator 啟動時所開啟的 console port 導出來，
這樣其他 container 或者 host 就可以透過這些 port 來連線到 emulator console。

（其實應該可以這樣將 adb server 的 5037 port 導出，而不用在啟動的時候增加 -a -P 等參數）

# noVNC 

這邊以 docker-compose 的設定檔作說明，我覺的 [thedrhax-dockerfiles/android-avd](https://github.com/thedrhax-dockerfiles/android-avd) 的版本，
比其他的版本方便的地方在於他把 noVNC 也順便放進去了，所以只要在啟動的時候設定環境變數 `noVNC=true` 就可以透過 6080 port 看到 emulator 正在執行的畫面。

```
version: '2'
services:
  emulator:
    image: swind/android-avd:latest
    environment:
     - noVNC=true
    ports:
     - 6080:6080
    devices:
      - "/dev/kvm:/dev/kvm"
```

