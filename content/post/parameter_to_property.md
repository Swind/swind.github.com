+++
categories = ["Python parameter to property"]
draft = true
description = "Python parameter to property"
date = "2017-09-27T15:25:07+08:00"
title = "parameter_to_property"
tags = ["python"]
+++

在 Python package [uiautomator](https://github.com/xiaocong/uiautomator) 看到的有趣寫法

<!--more-->

# d(text="Settings").long_click() and d(text="Settings").long_click.bottomright()

在 Python package `uiautomator` 的說明文件裡面可以看到這個有趣的語法被大量使用。

例如

```
# long click on the center of the specific ui object
d(text="Settings").long_click()
# long click on the bottomright corner of the specific ui object
d(text="Settings").long_click.bottomright()
# long click on the topleft corner of the specific ui object
d(text="Settings").long_click.topleft()
```

或

```
# wait until the ui object appears
d(text="Settings").wait.exists(timeout=3000)
# wait until the ui object gone
d(text="Settings").wait.gone(timeout=1000)
```

本來覺的應該就只是 `long_click` 或者 `wait` 本身就是一個實做了 `__call__` 與其他 function 的物件。
只是有點費工而已。

但是之前有機會看了一下 uiautomator 的原始碼才發現，他是使用 python decorator 來實做的。

```
def param_to_property(*props, **kwprops):
    if props and kwprops:
        raise SyntaxError("Can not set both props and kwprops at the same time.")

    class Wrapper(object):

        def __init__(self, func):
            self.func = func
            self.kwargs, self.args = {}, []

        def __getattr__(self, attr):
            if kwprops:
                for prop_name, prop_values in kwprops.items():
                    if attr in prop_values and prop_name not in self.kwargs:
                        self.kwargs[prop_name] = attr
                        return self
            elif attr in props:
                self.args.append(attr)
                return self
            raise AttributeError("%s parameter is duplicated or not allowed!" % attr)

        def __call__(self, *args, **kwargs):
            if kwprops:
                kwargs.update(self.kwargs)
                self.kwargs = {}
                return self.func(*args, **kwargs)
            else:
                new_args, self.args = self.args + list(args), []
                return self.func(*new_args, **kwargs)
    return Wrapper

``` 

[原始碼連結](https://github.com/xiaocong/uiautomator/blob/master/uiautomator/__init__.py)

與這個 decorator 的使用範例

```
 @property
    def wait(self):
        '''
        Waits for the current application to idle or window update event occurs.
        Usage:
        d.wait.idle(timeout=1000)
        d.wait.update(timeout=1000, package_name="com.android.settings")
        '''
        @param_to_property(action=["idle", "update"])
        def _wait(action, timeout=1000, package_name=None):
            if timeout / 1000 + 5 > int(os.environ.get("JSONRPC_TIMEOUT", 90)):
                http_timeout = timeout / 1000 + 5
            else:
                http_timeout = int(os.environ.get("JSONRPC_TIMEOUT", 90))
            if action == "idle":
                return self.server.jsonrpc_wrap(timeout=http_timeout).waitForIdle(timeout)
            elif action == "update":
                return self.server.jsonrpc_wrap(timeout=http_timeout).waitForWindowUpdate(package_name, timeout)
        return _wait
```

`param_to_property` 內的 `Wrapper` 基本上 `__call__` 的部份就如預期，沒有什麼意外, 但是可以注意一下 `self.kwargs` 與 `self.args` 的部份。

另外比較重要的就是在 `__getattr__` 與 `param_to_property` 初始化的部份。

