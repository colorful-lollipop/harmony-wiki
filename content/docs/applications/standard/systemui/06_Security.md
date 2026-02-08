# SystemUI 安全评审

## 1. 评审概述

### 1.1 评审范围

| 范围 | 说明 |
|------|------|
| **代码范围** | 除测试外的所有源码 |
| **模块** | common, entry, features, product |
| **语言** | ArkTS / ArkUI |

### 1.2 评审方法

- **静态分析**: 代码审查 ArkTS 语法安全
- **权限分析**: 检查敏感权限使用
- **架构分析**: 评估信任边界
- **API 分析**: 检查系统 API 调用安全

### 1.3 重要声明

⚠️ **本项目为纯 ArkTS/ArkUI 项目**，不存在以下风险：

- ❌ 内存安全问题 (托管内存，无 C/C++)
- ❌ 缓冲区溢出 (ArkTS 类型安全)
- ❌ 整数溢出 (JavaScript 数字类型)
- ❌ Use-after-free (垃圾回收)

## 2. 权限评审

### 2.1 权限声明

**证据**: `entry/phone/src/main/module.json5:32-116`

SystemUI 声明了 **19 个系统权限**：

| 权限 | 敏感级别 | 用途 |
|------|----------|------|
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 高 | 获取应用包信息 |
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 高 | 管理本地账户 |
| `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS_EXTENSION` | 高 | 跨账户交互 |
| `ohos.permission.NOTIFICATION_CONTROLLER` | 高 | 通知控制 |
| `ohos.permission.CAPTURE_SCREEN` | 高 | 截屏 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 高 | 修改安全设置 |
| `ohos.permission.GET_TELEPHONY_STATE` | 中 | 获取电话状态 |
| `ohos.permission.GET_WIFI_INFO` | 低 | 获取 WiFi 信息 |
| `ohos.permission.SET_WIFI_INFO` | 低 | 设置 WiFi |
| `ohos.permission.MANAGE_WIFI_CONNECTION` | 低 | 管理 WiFi 连接 |
| `ohos.permission.GET_NETWORK_INFO` | 低 | 获取网络信息 |
| `ohos.permission.USE_BLUETOOTH` | 低 | 使用蓝牙 |
| `ohos.permission.DISCOVER_BLUETOOTH` | 低 | 发现蓝牙设备 |
| `ohos.permission.MANAGE_BLUETOOTH` | 低 | 管理蓝牙 |
| `ohos.permission.ACCESS_NOTIFICATION_POLICY` | 中 | 访问通知策略 |
| `ohos.permission.MODIFY_AUDIO_SETTINGS` | 低 | 修改音频设置 |
| `ohos.permission.START_INVISIBLE_ABILITY` | 中 | 启动不可见 Ability |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 中 | 后台启动 Ability |
| `ohos.permission.PERMISSION_USED_STATS` | 低 | 权限使用统计 |

### 2.2 权限使用场景

| 权限 | 使用组件 | 调用 API |
|------|----------|----------|
| `NOTIFICATION_CONTROLLER` | noticeitem | `notificationManager.setNotificationEnable()` |
| `GET_BUNDLE_INFO_PRIVILEGED` | - | `bundleManager.getBundleInfo()` |
| `MANAGE_SECURE_SETTINGS` | - | 系统设置 API |
| `GET_TELEPHONY_STATE` | signalcomponent | 电话状态 API |
| `GET_WIFI_INFO` | wificomponent | `wifiManager.getWifiInfo()` |

### 2.3 权限风险评估

| 风险 | 权限 | 缓解措施 |
|------|------|----------|
| 信息泄露 | `GET_BUNDLE_INFO_PRIVILEGED` | 仅读取必要信息 |
| 隐私访问 | `GET_TELEPHONY_STATE` | 遵守隐私政策 |
| 安全设置篡改 | `MANAGE_SECURE_SETTINGS` | 仅修改系统级设置 |
| 通知滥用 | `NOTIFICATION_CONTROLLER` | 仅管理通知 |

## 3. 输入校验

### 3.1 ArkTS 类型安全

ArkTS 提供了强类型检查，降低了类型错误风险：

```typescript
// ✅ 正确: 类型声明
let batteryLevel: number = 50;

// ❌ 错误: 类型不安全 (ArkTS 会报错)
let level = "50"; // 编译错误
```

### 3.2 空值检查

**证据**: `common/src/main/ets/default/CheckEmptyUtils.ts`

```typescript
function checkEmpty(target: any): boolean {
  return target === null || target === undefined;
}

// 使用
if (checkEmpty(userInput)) {
  Log.showWarn(TAG, 'Empty input detected');
  return;
}
```

### 3.3 敏感信息过滤

**证据**: `common/src/main/ets/default/Log.ts:24-35`

```typescript
const FILTER_KEYS = [
  new RegExp('hide', "gi")  // 过滤敏感关键词
];

Log.showInfo(TAG, `Password: hide123`); // 输出: Password: **
```

## 4. 攻击面分析

### 4.1 外部输入

| 输入源 | 处理方式 | 风险等级 |
|--------|----------|----------|
| 用户点击事件 | ArkUI onClick | 低 |
| 系统广播 | EventManager.subscribe | 低 |
| 通知数据 | NotificationManager | 中 |
| 系统参数 | Systemparameter.getSync | 低 |

### 4.2 敏感操作

| 操作 | 权限要求 | 风险 |
|------|----------|------|
| 修改安全设置 | `MANAGE_SECURE_SETTINGS` | 高 |
| 截屏 | `CAPTURE_SCREEN` | 高 |
| 跨账户访问 | `INTERACT_ACROSS_LOCAL_ACCOUNTS` | 高 |
| 管理通知 | `NOTIFICATION_CONTROLLER` | 中 |

### 4.3 数据流

```
外部输入 (用户/系统)
    ↓
类型校验
    ↓
业务逻辑处理
    ↓
敏感操作 (权限检查)
    ↓
数据输出 (UI/API)
```

## 5. 信任边界

### 5.1 边界定义

```
┌─────────────────────────────────────────────────────────┐
│                    信任边界                              │
│  ┌───────────────────────────────────────────────────┐  │
│  │  SystemUI 进程                                     │  │
│  │  - ArkTS 代码 (受信任)                             │  │
│  │  - EventBus (受信任)                               │  │
│  │  - common 模块 (受信任)                            │  │
│  └───────────────────────────────────────────────────┘  │
│                          ↑                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │  边界                                              │  │
│  │  - @ohos.* API 调用 (框架验证)                     │  │
│  │  - 外部通知数据 (需校验)                            │  │
│  │  - 配置文件 (需解析)                                │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 5.2 边界防护

| 边界 | 防护措施 |
|------|----------|
| API 调用 | OpenHarmony 框架验证 |
| 权限检查 | 系统强制权限检查 |
| 数据解析 | ArkTS 类型安全 |
| 事件传递 | EventBus 类型校验 |

## 6. 安全风险与建议

### 6.1 已识别风险

| 风险 | 证据 | 影响 | 建议 |
|------|------|------|------|
| **敏感信息日志** | `Log.ts:21-22` 定义过滤规则 | 低风险 | 持续完善过滤规则 |
| **权限过度** | 19 个权限声明 | 中风险 | 按需申请最小权限 |
| **外部数据解析** | `NotificationManager.ts:108-139` | 低风险 | 增加数据校验 |

### 6.2 风险缓解措施

#### 6.2.1 日志过滤

**当前状态**: ✅ 已实现

**证据**: `Log.ts:21-22`

```typescript
const FILTER_KEYS = [
  new RegExp('hide', "gi")
];
```

**建议**: 扩展过滤关键词列表

```typescript
const FILTER_KEYS = [
  new RegExp('hide', "gi"),
  new RegExp('password', "gi"),
  new RegExp('token', "gi"),
  new RegExp('secret', "gi"),
  new RegExp('key', "gi")
];
```

#### 6.2.2 权限最小化

**当前状态**: ⚠️ 需审计

**建议**: 
- 审计每个权限的实际使用
- 移除未使用的权限
- 考虑运行时权限请求

#### 6.2.3 数据校验

**当前状态**: ⚠️ 部分实现

**建议**: 增加外部数据校验

```typescript
function validateNotification(data: any): boolean {
  // 校验通知数据格式
  if (!data || typeof data !== 'object') {
    return false;
  }
  if (!data.title || typeof data.title !== 'string') {
    return false;
  }
  return true;
}
```

### 6.3 安全最佳实践

| 实践 | 状态 | 说明 |
|------|------|------|
| ArkTS 类型安全 | ✅ | 强类型检查 |
| 敏感日志过滤 | ✅ | FILTER_KEYS |
| 权限声明 | ⚠️ | 需最小化 |
| 空值检查 | ✅ | CheckEmptyUtils |
| 错误处理 | ✅ | try/catch + Log |

## 7. 不涉及的安全领域

由于项目特性，以下安全领域**不涉及**：

| 领域 | 原因 |
|------|------|
| 内存安全 | ArkTS 托管内存，无 C/C++ |
| 缓冲区溢出 | ArkTS 类型安全 |
| 整数溢出 | JavaScript 数字类型 |
| Use-after-free | 垃圾回收机制 |
| N-API 安全 | 无 N-API 层 |
| Native 漏洞 | 无 native 代码 |

## 8. 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [目录结构](02_Directory_Structure.md) - 详细目录说明
- [架构设计](03_Architecture.md) - 系统架构图
- [内部 API](04_API_Inner.md) - 模块接口
- [构建指南](05_Build.md) - 编译配置
