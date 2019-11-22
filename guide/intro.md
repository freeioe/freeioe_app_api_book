
---

# FreeIOE 入门

本教程包含若干模块，每个模块旨在向您介绍 FreeIOE 基础知识，并帮助您快速入门。本教程将讲述以下内容：

* FreeIOE 编程模型
* 基本概念，例如设备、模型和分发等
* 在网关上部署应用的过程

## 要求

在开始阅读本章前，请先了解：

* **[Lua](http://www.lua.org/manual/5.3/)** 语言
* 如何构建简单的 Lua 模块
* Lua 中的面向对象编程

要完成本教程、您需要：

* Windows、Mac PC 或 Linux发行版系统
* 冬笋云开发者账户，请参阅[创建冬笋云开发者账户](#创建冬笋云开发者账户)
* 使用集成有 FreeIOE 的网关产品
  可以使用网关硬件产品，也可以使用虚拟机运行 [FreeIOE 虚拟网关](../dev_setup/vbox.md)
* 安装 VSCode
  成功后需要安装我们提供的 IOT Editor，并学习如何使用它进行应用的代码编辑，以及设备在线开发的操作

## 创建冬笋云开发者账户

如果您没有冬笋云账户，请执行以下步骤：
1. 打开[冬笋云](https://cloud.thingsroot.com)
2. 选择账户注册。您需要一个合法的邮箱来进行注册，且在注册过程中您会接收到账户注册链接
3. 登录冬笋云，按照在线说明进行开发者申请
   您的申请会在24H内进行审批
