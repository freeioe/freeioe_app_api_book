
---

# 从零构造应用

了解了应用本质后，我们着手构造一个不依赖于 FreeIOE 提供的应用基础类的应用。

本章使用了第三方的模块 [middleclass](https://github.com/kikito/middleclass/wiki) 来构造应用模块。如果您对完全的手工构造应用模块感兴趣，请[参考](http://lua-users.org/wiki/ObjectOrientationTutorial)。

## 声明应用模块

在实现模块函数前，我们需要进行模块对象声明(definition)

``` lua
local class = require 'middleclass'
--- 注册对象(请尽量使用唯一的标识字符串)
local app = class('THIS_IS_AN_EXAMPLE_APP_FOR_FREEIOE')
--- 设定应用最小运行接口版本(目前版本为6,为了接口兼容性)
app.static.API_VER = 6
```

注意:
**API_VER** 是标识当前应用运行的最低 API 版本号需求。当 FreeIOE 无法满足这个版本需求时，将不会尝试启动该应用，避免由于接口不兼容性导致的未知问题。

> 参考 API Reference获取接口以及其接口版本号要求等信息

## 模块函数列表

模块需要实现的函数只有极少的数量，列表如下：

1. 模块对象构建函数
   initialize(name, sys, conf)
2. 应用实例启动函数
   start()
3. 应用实例停止/退出函数
   close(reason)
4. 应用逻辑运行函数 (可选)
   run(time_in_ms)

### 函数详细说明

* function app:initialize(name, sys, conf)
  * 参数
    * name: 应用实例名称
    * sys: 系统接口
    * conf: 配置信息
  * 返回值
    * 无
  * 用途
    * FreeIOE 调用此函数，去实例化应用对象，并传入应用初始化所需接口和信息。
  * 注意
    * 应用不应在本函数实现复杂的逻辑。较为耗时和复杂的逻辑需要放到start函数中实现。

* function app:start()
  * 参数
    * 无
  * 返回值
    * boolean 返回值，如果返回非真，框架则认为初始化失败，从而不再继续执行后续动作。
  * 用途
    * 设定应用回调接口，创建设备对象实例，设备通讯接口初始化以及其他应用所需初始化逻辑
  * 注意
    * 如需要进行比较耗时的操作，请使用 sys:fork 或者 sys:timeout 来开辟新的携程来执行


* function app:close(reason)
  * 参数
    * reason: 关闭/退出的原因
  * 返回值
    * 无
  * 用途
    * 应用对象实例生命周期结束（非gc）的回调入口，用以应用清理对象等等。
  * 注意
    * 确保此函数不出现一些异常情况，否则会出现一些数据并未正确清理（如设备模型并未被反注册）

* function app:run(tms)
  * 参数
    * tms：当前周期调用的间隔时间，单位毫秒。第一次调用值为1000毫秒，以后的值为本函数上一次运行的返回值。
  * 返回值
    * 返回下一次调用次函数的时间间隔（单位是毫秒)。
  * 用途
    * 当应用有此接口，框架会周期调用此函数。周期间隔时间取决于本函数的返回值
