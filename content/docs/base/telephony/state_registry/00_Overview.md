# 项目概览

## 模块定位

State Registry（状态注册表）是 OpenHarmony Telephony 子系统的核心模块之一，负责提供电信子系统状态变化的观察者注册与注销能力。该模块采用发布-订阅模式，使上层应用能够实时感知电信相关事件的发生。

**证据来源**：`interfaces/kits/js/@ohos.telephony.observer.d.ts` 定义了 `observer` 模块的 JS API 入口。

### 核心能力

1. **事件订阅**：支持订阅多种电信状态变化事件
2. **多槽位支持**：通过 `slotId` 参数区分不同 SIM 卡槽位
3. **异步回调**：基于 AsyncCallback 的事件通知机制
4. **资源管理**：自动化的观察者注册与注销生命周期管理

### 能力边界

| 能力 | 说明 |
|------|------|
| ✅ 支持的事件类型 | 网络状态、信号强度、小区信息、蜂窝数据、通话状态、SIM 卡状态 |
| ❌ 不支持的能力 | 直接发起通话、发送短信、修改网络设置等操作类 API |
| 📌 依赖组件 | 依赖 core_service 提供底层电信能力，依赖 safwk 加载系统服务 |

## 运行环境

### 硬件要求

- 必须配备调制解调器（Modem）
- 必须具备可独立进行蜂窝通信的 SIM 卡槽
- 设备需支持蜂窝网络连接

**证据来源**：`README.md` 中 "Constraints" 章节声明的硬件约束。

### 软件依赖

| 依赖组件 | 用途 |
|----------|------|
| core_service | 提供核心电信能力接口 |
| napi | 提供 JS Native 接口绑定能力 |
| ipc/binder | 进程间通信机制 |
| samgr | 系统服务管理框架 |
| safwk | 系统能力框架 |

### 系统能力声明

```
SystemCapability.Telephony.StateRegistry
```

**证据来源**：`bundle.json` 中 `component.syscap` 声明。

## 关键概念

### slotId（卡槽标识符）

- `0`：主卡槽（通常为 SIM 卡槽 1）
- `1`：副卡槽（通常为 SIM 卡槽 2，部分设备可能不支持）
- `-1` 或未指定：默认行为（部分 API 支持）

### AsyncCallback 模式

所有 N-API 接口采用异步回调模式，函数签名为：

```typescript
function on(type: string, options: { slotId?: number }, callback: AsyncCallback<T>): void
```

### 事件类型

| 事件名 | 描述 | 所需权限 |
|--------|------|----------|
| networkStateChange | 网络状态变化 | ohos.permission.GET_NETWORK_INFO |
| signalInfoChange | 信号强度变化 | 无 |
| cellInfoChange | 小区信息变化 | ohos.permission.LOCATION + ohos.permission.APPROXIMATELY_LOCATION |
| cellularDataConnectionStateChange | 蜂窝数据连接状态变化 | 无 |
| cellularDataFlowChange | 蜂窝数据流变化 | 无 |
| callStateChange | 通话状态变化 | ohos.permission.READ_CALL_LOG |
| simStateChange | SIM 卡状态变化 | 无 |

## 版本信息

| 属性 | 值 |
|------|-----|
| 模块版本 | 4.0 |
| 发布类型 | code-segment |
| ROM 占用 | 550KB |
| RAM 占用 | 1MB |

## 相关文档

- [目录结构](01_Directory_Structure.md)
- [架构设计](02_Architecture.md)
- [JS API](03_JS_API.md)
