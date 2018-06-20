# 应用开发接口\(SYS部分\)

* sys:log\(level, ...\)

记录应用日志。 level是字符串类型，可用值:trace, debug, info, notice, warning, error, fatal.

```
sys:log("debug", "this is a log content")
```

* sys:logger\(\)

获取logger实例。 实例包含:trace, debug, info, notice, warning, error, fatal. 例如:

```
local log = sys:logger()
log:debug("this is a log content")
```

* sys:dump\_comm\(sn, dir, ...\)

记录应用报文。 sn是应用创建的设备序列号/为空时代表设备无关报文。参考api:create\_device\(\)函数。 dir是报文方向： IN, OUT。

* sys:fire\_event\(sn, level, type, info, data, timestamp\)

<<<<<<< HEAD
记录应用报文。 sn是应用创建的设备序列号/为空时代表设备无关报文。参考api:create_device()函数。 dir是报文方向： IN, OUT。

* sys:fire_event(sn, level, type, info, data, timestamp)

记录应用事件。 sn是应用创建的设备序列号。level是事件等级的整数,type是事件类型(如果是字符串类型则是自定义类型),info是事件描述字符串,data是时间附带数据,timestamp是时间戳。
=======
记录应用事件。 sn是应用创建的设备序列号。level是事件等级的整数,type是事件类型\(如果是字符串类型则是自定义类型\),info是事件描述字符串,data是时间附带数据,timestamp是时间戳。
>>>>>>> 1d33ae5b816d808131b46ae72402713244eb6144

* sys:fork\(func, ...\)

创建独立携程,入口执行函数func. 后面是变参参数。

```
sys:fork(function(a)
    print(a)
end, 1)
```

其中a是传入的参数1

* sys:timeout\(ms, func\)

创建延迟执行携程, ms为延迟时间（单位是毫秒）, func为携程入口函数。

* sys:cancelable\_timeout\(ms, func\)

创建可以取消的延迟携程，返回对象是取消函数

```
local timer_cancel = sys:cancelable_timeout(...)
timer_cancel()
```

* sys:exit\(\)

应用退出接口（特殊应用使用）

* sys:abort\(\)

<<<<<<< HEAD
系统退出接口，调用此接口会导致FreeIOE系统退出。 请谨慎调用。 
=======
系统退出接口，调用此接口会导致IOT系统退出。 请谨慎调用。
>>>>>>> 1d33ae5b816d808131b46ae72402713244eb6144

* sys:now\(\)

返回操作系统启动后的时间计数。 单位是微妙，最小有效精度是10毫秒。

* sys:time\(\)

获取系统时间，单位是秒，并包含两位小数的毫秒。

* sys:start\_time\(\)

系统启动的UTC时间，单位是秒。

* sys:yield\(\)

交出当前应用对CPU的控制权。相当与sys:sleep\(0\)。

* sys:sleep\(ms\)

挂起当前应用， ms是挂起时常，单位是毫秒

* sys:data\_api\(\)

获取数据接口，参考app:api

* sys:self\_co\(\)

获取当前运行的携程对象

* sys:wait\(\)

挂起当前携程，结合sys:wakeup使用

* sys:wakeup\(co\)

唤醒一个被sys:sleep或sys:wait挂起的携程。

* sys:app\_dir\(\)

获取当前应用所在的目录。

* sys:app\_sn\(\)

获取当前应用的序列号。

* sys:get\_conf\(default\_config\)

获取应用配置，default\_config默认配置

* sys:set\_conf\(config\)

设定应用配置

* sys:version\(\)

获取应用版本号

* sys:gen\_sn\(dev\_name\)

生成独立的设备序列号，dev\_name为设备名称，必须指定。

* sys:hw_id()

获取FreeIOE设备序列号

* sys:id\(\)

获取FreeIOE连接云平台所用的序列号(此ID可不同与设备序列号)

* sys:cleanup\(\)

应用清理接口

