+++
title = "python-zeroconf 範例"
tags = ["python","zeroconf"]
categories = ["python"]
date = "2017-09-11T13:55:48+08:00"
draft=false
description = "python-zeroconf client/server 範例"
+++

# zeroconf

zeroconf ( Zero configuration networking )，主要透過 mDNS ( Multicast DNS ) 與 DNS-SD ( DNS Service Discovery ) 來實現。
而 [python-zeroconf](https://github.com/jstasiak/python-zeroconf) 就是 `zeroconf` 的 **純 Python** 實現。

<!--more-->

在 Apple 的機器上的實做就是 `Bonjour`。

而 Linux 則是 `Avachi`

關於 `zeroconf` 詳細的介紹可以參考 [设备发现之Bonjour协议原理分享](http://bbs.mico.io/card/1236)

# Server

`python-zeroconf` 的 `README.md` 裡面雖然沒有提到 `mDNS` 的部份，但是其實在 [examples](https://github.com/jstasiak/python-zeroconf/blob/master/examples/registration.py)
有個簡單的範例。

下面是稍微整理一下 `examples` 裡面的範例

```python
from zeroconf import ServiceInfo, Zeroconf
import socket

import logging 

logger = logging.getLogger(__name__)

class Server:
    def __init__(self, name, address, port, desc):
        self.zeroconf = Zeroconf()
        self.name = name

        desc = {
            'description': desc 
        }

        self.info = ServiceInfo(type_="_http._tcp.local.",
                           name="{}._http._tcp.local.".format(name),
                           address=socket.inet_aton(address), 
                           port=port, 
                           weight=0, 
                           priority=0, 
                           properties=desc)


    def register(self):
        self.zeroconf.register_service(self.info)

    def unregister(self):
        self.zeroconf.unregister_service(self.info)

    def close(self):
        self.zeroconf.close()


if __name__ == "__main__":
    server = Server(
        name="_testwebsite", 
        address="127.0.0.1",
        port=9999, 
        desc="0.0.1")

    server.register()

    import time
    try:
        time.sleep(600)
    except KeyboardInterrupt:
        pass
    finally:
        server.unregister()
        server.close()
```

主要要看的就只有 `ServiceInfo` 跟呼叫 `zeroconf.register_service` 的部份而已

# Client

Client 的部份一樣有 [examples](https://github.com/jstasiak/python-zeroconf/blob/master/examples/browser.py) 可以參考。

```python
from zeroconf import ServiceBrowser, ServiceStateChange, Zeroconf
from threading import Event
import socket
import time

import logging
logger = logging.getLogger("chatroom")

class Service:
    ADDED = "added"
    REMOVED = "removed"

    def __init__(self, zeroconf, service_type, name, state_change):
        self.state = None
        self.name = name
        self.zeroconf = zeroconf

        self._update_info(service_type, name)
        self._state_change(state_change)

    def _update_info(self, service_type, name):
        info = self.zeroconf.get_service_info(service_type, name)

        self.address = socket.inet_ntoa(info.address)
        self.port = info.port

        self.weight = info.weight
        self.priority = info.priority

        self.server_name = info.server

        if info.properties:
            self.version = info.properties.get("version")
        else:
            self.version = None

    def __str__(self):
        return "[{}] {}:{} {}".format(self.state, self.address, self.port, self.name)


    def _state_change(self, state_change):
        if state_change is ServiceStateChange.Added:
            self.state = self.ADDED
        elif state_change is ServiceStateChange.Removed:
            self.state = self.REMOVED


class Browser:
    def __init__(self):
        self.zeroconf = Zeroconf()
        self.handlers = []

        self.services = {}

    def list(self):
        return self.services.values()

    def get(self, name):
        return self.services.get(name)

    def wait(self, name):
        event = Event()

        def wait_handler(service):
            if name == service.name:
                event.set()
                self.unregister_handler(wait_handler)

        self.register_handler(wait_handler)

        if name in self.services:
            event.set()
            self.unregister_handler(wait_handler)

        return event

    def handler(self, zeroconf, service_type, name, state_change):
        service = Service(zeroconf, service_type, name, state_change)

        if service.state == service.ADDED:
            self.services[name] = service
        else:
            del self.services[name]

        for handler in self.handlers:
            handler(service)

    def register_handler(self, handler):
        self.handlers.append(handler)

    def unregister_handler(self, handler):
        self.handlers.remove(handler)

    def start(self):
        self.browser = ServiceBrowser(self.zeroconf, "_tcp.local.", handlers=[self.handler])

    def close(self):
        self.zeroconf.close()

if __name__ == "__main__":
    browser = Browser()

    def handler(state):
        print(state)

    browser.register_handlers(handler)
    browser.start()

    time.sleep(600)
```

比較不一樣的是，`ServiceBrowser` 會建立一個 `Thread` 去取得資訊，並且當有 state 變化的時候（ADDED 或 REMOVED) 時，呼叫註冊的 handlers。
