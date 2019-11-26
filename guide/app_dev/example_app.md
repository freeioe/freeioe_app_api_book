
---

# 应用示例

本章展示了一个简单的数据采集的应用：

1. 满足 FreeIOE 应用框架的对接口的要求
2. 创建一个虚拟设备模型
   * 包含一个输入项
3. 使用随机数作为数据值发布到虚拟设备模型上
4. 输出报文（虚拟）

## 示例代码

```lua
local class = require 'middleclass'

--- 注册对象(请尽量使用唯一的标识字符串)
local app = class("YOUR_APP_NAME_App")
--- 设定应用最小运行接口版本(目前版本为6,为了以后的接口兼容性)
app.API_VER = 6

---
-- 应用对象初始化函数
-- @param name: 应用本地安装名称。 如modbus_com_1
-- @param sys: 系统sys接口对象。参考API文档中的sys接口说明
-- @param conf: 应用配置参数。由安装配置中的json数据转换出来的数据对象
function app:initialize(name, sys, conf)
	self._name = name
	self._sys = sys
	self._conf = conf
	--- 获取数据接口
	self._api = self._sys:data_api()
	--- 获取日志接口
	self._log = sys:logger()
	--- 设备实例
	self._devs = {}

	self._log:debug("XXXX Application initlized")
end

--- 应用启动函数
function app:start()
	self._api:set_handler({
		--[[
		--- 处理设备输入项数值变更消息，当需要监控其他设备时才需要此接口，并在set_handler函数传入监控标识
		on_input = function(app, sn, input, prop, value, timestamp, quality)
		end,
		]]
		on_output = function(app, sn, output, prop, value, timestamp)
		end,
		on_command = function(app, sn, command, param)
		end,
		on_ctrl = function(app, command, param, ...)
		end,
	})

	--- 生成设备唯一序列号
	local sys_id = self._sys:id()
	local sn = sys_id.."."..self._sys:gen_sn('example_device_serial_number')

	--- 增加设备实例
	local inputs = {
		{name="tag1", desc="tag1 desc", unit="KV"}
	}
	local meta = self._api:default_meta()
	meta.name = "Example Device"
	meta.description = "Example Device Meta"
	local dev = self._api:add_device(sn, meta, inputs)
	self._devs[#self._devs + 1] = dev

	return true
end

--- 应用退出函数
function app:close(reason)
    -- 处理通讯链路关闭等
end

--- 应用运行入口
function app:run(tms)
	for _, dev in ipairs(self._devs) do
		dev:dump_comm("IN", "XXXXXXXXXXXX")
		dev:set_input_prop('tag1', "value", math.random())
	end

	return 10000 --单位是ms, 10000代表下一采集间隔为10秒。 等同于sleep(10000)
end

--- 返回应用对象(标准模块做法)
return app
```