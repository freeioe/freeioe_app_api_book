
----

# OpcUA 数据采集应用示例

* 使用了opcua的Lua模块: [open62541-lua](https://github.com/freeioe/open62541-lua)
* 读取Simulation节点下所有数据项


***代码:***

```lua
--- 导入需求的模块
local class = require 'middleclass'
local opcua = require 'opcua'

--- 注册对象(请尽量使用唯一的标识字符串)
local app = class("IOT_OPCUA_CLIENT_APP")
--- 设定应用最小运行接口版本(目前版本为1,为了以后的接口兼容性)
app.API_VER = 1

---
-- 应用对象初始化函数
-- @param name 应用本地安装名称。 如modbus_com_1
-- @param sys 系统sys接口对象。参考API文档中的sys接口说明
-- @param conf 应用配置参数。由安装配置中的json数据转换出来的数据对象
function app:initialize(name, sys, conf)
    self._name = name
    self._sys = sys
    self._conf = conf
    --- 获取数据接口
    self._api = sys:data_api()
    --- 获取日志接口
    self._log = sys:logger()
    self._connect_retry = 1000
end

---
-- 检测连接可用性
function app:is_connected()
    if self._client then
        return true
    end
end

---
-- 获取设备的OpcUa节点
function app:get_device_node(namespace, obj_name)
    if not self:is_connected() then
        self._log:warning("Client is not connected!")
        return
    end

    local client = self._client
    local nodes = self._nodes

    --- 获取Objects节点
    local objects = client:getObjectsNode()
    --- 获取名字空间的id号
    local idx, err = client:getNamespaceIndex(namespace)
    if not idx then
        self._log:warning("Cannot find namespace", err)
        return
    end
    --- 获取设备节点
    local devobj, err = objects:getChild(idx..":"..obj_name)
    if not devobj then
        self._log:error('Device object not found', err)
        return
    else
        self._log:debug("Device object found", devobj)
    end

    --- 返回节点对象
    return {
        idx = idx,
        name = obj_name,
        device = device,
        devobj = devobj,
        vars = {}
    }
end

---
-- 定义需要获取数据的输入项
local inputs = {
    { name = "Counter1", desc = "Counter1"},
    { name = "s1", desc = "Simulation 1"},
}

---
-- 连接成功后的处理函数
function app:on_connected(client)
    -- Cleanup nodes buffer
    self._nodes = {}
    -- Set client object
    self._client = client

    --- Get opcua object instance by namespace and browse name
    -- 根据名字空间和节点名称获取OpcUa对象实体
    local namespace = self._conf.namespace or "http://www.prosysopc.com/OPCUA/SimulationNodes"
    local obj_name = "Simulation"
    local node, err = self:get_device_node(namespace, obj_name)
    ---
    -- 获取设备对象节点下的变量节点
    if node then
        for _,v in ipairs(inputs) do
            local var, err = node.devobj:getChild(v.name)
            --print(_,v.name,var)
            if not var then
                self._log:error('Variable not found', err)
            else
                node.vars[v.name] = var
            end
        end
        local sn = namespace..'/'..obj_name
        self._nodes[sn] = node
    end
end

---
-- 连接断开后的处理函数
function app:on_disconnect()
    self._nodes = {}
    self._client = nil
    self._sys:timeout(self._connect_retry, function() self:connect_proc() end)
    self._connect_retry = self._connect_retry * 2
    if self._connect_retry > 2000 * 64 then
        self._connect_retry = 2000
    end
end

---
-- 连接处理函数
function app:connect_proc()
    self._log:notice("OPC Client start connection!")
    local client = self._client_obj

    local ep = self._conf.endpoint or "opc.tcp://172.30.1.162:53530/OPCUA/SimulationServer"
    local username = self._conf.username or "user1"
    local password = self._conf.password or "password"
    --local r, err = client:connect_username(ep, username, password)
    local r, err = client:connect(ep)
    if r then
        self._log:notice("OPC Client connect successfully!", self._sys:time())
        self._connect_retry = 2000
        self:on_connected(client)
    else
        self._log:error("OPC Client connect failure!", err, self._sys:time())
        self:on_disconnect()
    end
end

--- 应用启动函数
function app:start()
    self._nodes = {}
    self._devs = {}

    --- 设定OpcUa连接配置
    local config = opcua.ConnectionConfig.new()
    config.protocolVersion = 0  -- 协议版本
    config.sendBufferSize = 65535  -- 发送缓存大小
    config.recvBufferSize = 65535  -- 接受缓存大小
    config.maxMessageSize = 0    -- 消息大小限制
    config.maxChunkCount = 0    --

    --- 生成OpcUa客户端对象
    local client = opcua.Client.new(5000, 10 * 60 * 1000, config)
    self._client_obj = client

    --- 发起OpcUa连接
    self._sys:fork(function() self:connect_proc() end)

    --- 设定接口处理函数
    self._api:set_handler({
        on_output = function(...)
            print(...)
        end,
        on_ctrl = function(...)
            print(...)
        end
    })

    --- 创建设备对象实例
    local sys_id = self._sys:id()
    local meta = self._api:default_meta()
    meta.name = "Unknow OPCUA"
    meta.description = "Unknow OPCUA Device"
    meta.series = "X1"
    local dev = self._api:add_device(sys_id..'.OPCUA_TEST', meta, inputs)
    self._devs['Simulation'] = dev

    return true
end

--- 应用退出函数
function app:close(reason)
    print('close', self._name, reason)
    --- 清理OpcUa客户端连接
    self._client = nil
    if self._client_obj then
        self._nodes = {}
        self._client_obj:disconnect()
        self._client_obj = nil
    end
end

--- 应用运行入口
function app:run(tms)
    if not self._client then
        return 1000
    end

    --- 获取节点当前值数据
    for sn, node in pairs(self._nodes) do
        local dev = self._devs[node.name]
        assert(dev)
        for k, v in pairs(node.vars) do
            local dv = v:getValue()
            --[[
            print(dv, dv:isEmpty(), dv:isScalar())
            print(dv:asLong(), dv:asDouble(), dv:asString())
            ]]--
            local now = self._sys:time()
            --- 设定当前值
            dev:set_input_prop(k, "value", dv:asDouble(), now, 0)
        end
    end

    --- 返回下一次调用run函数的间隔
    return 2000
end

--- 返回应用对象
return app
```



