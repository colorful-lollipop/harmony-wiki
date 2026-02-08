# 项目概述

## 模块定位

`notification_cangjie_wrapper` 是 OpenHarmony 系统中 **Cangjie 语言版本的公共事件管理模块**，为使用 Cangjie 语言开发的应用提供公共事件（Common Event）能力。

### 核心能力

| 能力 | 描述 | 代码位置 |
|------|------|----------|
| 发布公共事件 | 应用可以发布自定义或系统公共事件 | `common_event_manager.cj:57` |
| 创建订阅者 | 根据订阅信息创建事件订阅者 | `common_event_manager.cj:82` |
| 订阅公共事件 | 注册回调接收指定公共事件 | `common_event_manager.cj:109` |
| 取消订阅 | 移除已注册的订阅关系 | `common_event_manager.cj:134` |

### 系统定位

```
用户应用 (Cangjie)
       ↓
notification_cangjie_wrapper (FFI 包装层)
       ↓
common_event_service (C++ 服务层)
       ↓
CES (Common Event Service 系统服务)
```

## 运行环境

| 属性 | 值 | 证据 |
|------|-----|------|
| 设备类型 | 标准设备 (standard) | `bundle.json:18` |
| API Level | 22+ | `common_event_manager.cj:37` |
| 系统能力 | SystemCapability.Notification.CommonEvent | `common_event_manager.cj:38` |
| ROM 占用 | ~400KB | `bundle.json:20` |
| RAM 占用 | ~432KB | `bundle.json:21` |

## 关键概念

### 公共事件类型

| 类型 | 描述 | 示例 |
|------|------|------|
| 系统公共事件 | 系统预定义的事件，由系统服务发布 | `BOOT_COMPLETED`, `BATTERY_CHANGED` |
| 自定义公共事件 | 应用自定义用于跨应用通信 | `com.example.MY_EVENT` |

### 订阅信息

`CommonEventSubscribeInfo` 定义订阅条件：

- `events`: 订阅的事件列表
- `priority`: 订阅者优先级 (-100 ~ 1000)
- `userId`: 用户 ID
- `publisherPermission`: 发布者权限要求
- `publisherBundleName`: 发布者包名限制

### 发布数据

`CommonEventPublishData` 定义事件属性：

- `bundleName`: 目标订阅者包名
- `code`/`data`: 事件数据
- `isOrdered`: 是否有序事件
- `isSticky`: 是否粘性事件 (需要权限)
- `parameters`: 扩展参数 (HashMap)

## 模块边界

### 职责范围

**负责**：
- Cangjie API 暴露与参数校验
- FFI 数据结构转换 (C ⇄ Cangjie)
- 错误码映射与异常抛出
- 系统事件常量定义

**不负责**：
- 底层 IPC 通信 (由 common_event_service 实现)
- 事件持久化与分发
- 权限实际校验 (由系统服务完成)

### 对外依赖

| 依赖组件 | 用途 | 证据 |
|----------|------|------|
| `common_event_service` | 底层 C++ 事件服务 | `BUILD.gn:33` |
| `cangjie_ark_interop` | FFI 类型与异常处理 | `BUILD.gn:35-37` |
| `hiviewdfx_cangjie_wrapper` | 日志输出 | `BUILD.gn:39` |
