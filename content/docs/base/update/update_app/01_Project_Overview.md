# 项目概览

> **本文档描述 update_app 模块的项目定位、功能边界、核心能力和运行环境**

## 1 项目定位

### 1.1 整体定位

update_app 是 **OpenHarmony 系统更新客户端应用**，提供用户友好的系统更新界面和升级管理功能。

| 职责 | 描述 | 边界 |
|------|------|------|
| 系统更新 | 检查、下载、安装系统 OTA 更新 | 限于系统级更新 |
| UI 交互 | 提供用户友好的更新界面 | 限于前端展示 |
| 状态管理 | 管理更新状态和进度 | 限于应用层状态 |
| 通知管理 | 显示更新进度和状态通知 | 限于系统通知 |

**核心特征**：
- **应用层应用**：非系统服务，通过 `@ohos.update` 调用系统更新服务
- **纯 ArkTS 实现**：无 C++ 代码，使用 ArkUI 2.0 构建 UI
- **模块化架构**：采用 Entry + HAR 的三层架构

### 1.2 在系统中的位置

```
┌─────────────────────────────────────────────────────────┐
│                应用层 (Applications)                  │
├─────────────────────────────────────────────────────────┤
│  update_app (本文档主题)                             │
│  ┌──────────────────────────────────────────────────┐  │
│  │ UI层：更新界面、对话框、通知                     │  │
│  │ 业务层：状态机、更新管理器                       │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     ↓ IPC/N-API
┌─────────────────────────────────────────────────────────┐
│           系统服务层 (System Services)                 │
│  update_service (系统更新服务，位于其他仓库)          │
│  @ohos.update (系统 N-API 模块)                     │
└─────────────────────────────────────────────────────────┘
```

### 1.3 与其他模块的关系

| 关系类型 | 模块 | 交互方式 | 说明 |
|---------|-------|----------|------|
| **依赖** | `@ohos.update` | 系统接口调用 | 调用系统更新服务的 N-API 接口 |
| **被调用** | Settings/系统应用 | Want 通知 | 系统应用通过 Want 触发更新检查 |
| **IPC 通信** | update_engine | ServiceExtensionAbility | 接收系统更新服务的推送消息 |
| **权限依赖** | 系统权限管理 | 权限声明 | 需要 `ohos.permission.UPDATE_SYSTEM` |

---

## 2 功能边界

### 2.1 核心功能 (Scope In)

| 功能 | 说明 | 代码位置 |
|------|------|----------|
| 版本检查 | 检查系统新版本 | `OtaUpdateManager.checkNewVersion()` |
| 下载更新包 | 下载 OTA 更新包 | `OtaUpdateManager.download()` |
| 应用更新 | 安装系统更新 | `OtaUpdateManager.upgrade()` |
| 进度显示 | 显示下载/安装进度 | `StateManager` + 通知栏 |
| 更新日志 | 展示版本变更内容 | `OtaPage.getNewVersionDescription()` |
| 回滚支持 | 更新失败后状态恢复 | `StateManager.installFailed` |
| 网络控制 | 支持仅 WiFi 下载 | `DialogHelper.displayNetworkDialog()` |

**代码证据**：
```typescript
// feature/ota/src/main/ets/manager/OtaUpdateManager.ets:145
async checkNewVersion(): Promise<UpgradeData<update.CheckResult>> {
    return new Promise((resolve, reject) => {
        this.updateManager.checkNewVersion().then((result) => {
            // ...
        });
    });
}

// common/src/main/ets/manager/UpdateManager.ts:17
import update from '@ohos.update';  // 系统更新接口
```

### 2.2 不包含功能 (Scope Out)

| 功能 | 说明 | 替代方案 |
|------|------|----------|
| 应用包更新 | HAP 应用更新 | PackageManager |
| 差分算法 | 增量计算 | 由系统更新服务提供 |
| 签名验证 | 更新包签名校验 | 由系统更新服务提供 |
| 系统回滚 | 系统级回滚 | 由系统更新服务提供 |

### 2.3 功能边界图

```mermaid
graph TB
    subgraph "update_app 边界内"
        A[版本检查] --> B[下载更新包]
        B --> C[进度显示]
        C --> D[应用更新]
        D --> E[更新完成/失败]
        F[更新日志] --> A
        G[网络控制] --> B
    end

    H[@ohos.update 系统服务] -.-> |N-API| A
    H -.-> |N-API| B
    H -.-> |N-API| D

    I[update_engine] -.-> |IPC 推送| C
```

---

## 3 核心能力详解

### 3.1 系统更新能力

| 能力项 | 说明 |
|--------|------|
| 版本检测 | 调用 `checkNewVersion()` 从更新服务器获取最新版本 |
| 网络选择 | 支持 WiFi/蜂窝网络选择，默认仅 WiFi 下载 |
| 断点续传 | 支持 `resumeDownload()` 断点续传 |
| 进度回调 | 通过 `UpgradeTaskCallback` 实时获取进度 |
| 倒计时安装 | 支持倒计时自动安装对话框 |
| 后台服务 | ServiceExtensionAbility 支持后台处理更新 |

**代码证据**：
```typescript
// feature/ota/src/main/ets/manager/OtaUpdateManager.ets:163
async upgrade(order: update.Order = update.Order.INSTALL_AND_APPLY): Promise<void> {
    let versionDigest: string = await VersionUtils.getNewVersionDigest();
    return new Promise((resolve, reject) => {
        this.updateManager.upgrade(versionDigest, order).then(() => {
            resolve();
        }).catch((err: BusinessError) => {
            reject(err);
        });
    });
}
```

### 3.2 状态管理能力

| 能力项 | 说明 |
|--------|------|
| 状态机模式 | 使用 StateManager 管理更新各状态转换 |
| 状态去重 | OtaStatusHolder 防止重复事件通知 |
| 消息队列 | MessageQueue 串行处理异步状态更新 |
| 超时控制 | `@enableTimeOutCheck` 装饰器添加 30 秒超时 |

**代码证据**：
```typescript
// common/src/main/ets/manager/UpdateManager.ts:27
export function enableTimeOutCheck<T>(timeout?: number): MethodDecorator {
    const TIME = 30000;  // 30 秒超时
    // ...
}
```

### 3.3 UI 交互能力

| 能力项 | 说明 |
|--------|------|
| ArkUI 2.0 | 使用声明式 UI 构建 |
| 对话框管理 | 统一的对话框管理器 `DialogHelper` |
| 通知管理 | `NotificationManager` 处理系统通知栏交互 |
| 页面路由 | 支持多页面路由（首页、新版本、当前版本） |

**代码证据**：
```typescript
// feature/ota/src/main/ets/dialog/DialogHelper.ets:29
export interface DialogOperator {
    displayNetworkDialog(context: Context, confirmCallback: () => void): void;
    displayUpgradeFailDialog(context: Context): void;
    // ...
}
```

---

## 4 运行环境

### 4.1 系统要求

| 要求 | 最小版本 | 推荐版本 |
|------|----------|----------|
| OpenHarmony | API 20 (4.0) | API 20+ |
| 系统架构 | ARM64 | ARM64 |
| 应用模型 | Stage | Stage |

**代码证据**：
```json5
// build-profile.json5:7-8
{
  "compileSdkVersion": 20,
  "compatibleSdkVersion": 20
}
```

### 4.2 运行时依赖

| 依赖模块 | 用途 | 来源 |
|----------|------|------|
| @ohos.update | 系统更新接口 | OpenHarmony 系统 |
| @ohos.app.ability.common | Ability 上下文 | OpenHarmony 系统 |
| @ohos.app.ability.Want | Ability 参数 | OpenHarmony 系统 |

**代码证据**：
```typescript
// common/src/main/ets/manager/UpdateManager.ts:16
import type common from '@ohos.app.ability.common';
import update from '@ohos.update';
```

### 4.3 权限要求

| 权限名 | 用途 | 危险级别 |
|--------|------|----------|
| `ohos.permission.UPDATE_SYSTEM` | 执行系统更新 | 🔴 高危 |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络状态 | 🟡 中危 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 后台启动 Ability | 🟡 中危 |

**代码证据**：
```json5
// product/oh/base/src/main/module.json5:74-82
"requestPermissions": [
  {
    "name": "ohos.permission.UPDATE_SYSTEM"
  },
  {
    "name": "ohos.permission.GET_NETWORK_INFO"
  },
  {
    "name": "ohos.permission.START_ABILITIES_FROM_BACKGROUND"
  }
]
```

---

## 5 关键概念

### 5.1 核心概念定义

| 概念 | 定义 | 相关文件 |
|------|------|----------|
| OTA | Over-The-Air 空中下载，无线更新方式 | `OtaUpdateManager` |
| HAP | Harmony Ability Package，OpenHarmony 应用包格式 | build 产物 |
| HAR | Harmony Archive，OpenHarmony 共享库格式 | build 产物 |
| Updater | 系统更新服务接口实例 | `update.getOnlineUpdater()` |
| StateMachine | 状态机，管理更新各状态 | `StateManager` |

### 5.2 数据结构

```typescript
// OTA 状态
interface OtaStatus {
    status: number;        // 更新状态
    percent: number;       // 进度百分比
    endReason: string;     // 结束原因（错误码）
}

// 更新数据包装
interface UpgradeData<T> {
    callResult: UpgradeCallResult;  // 调用结果
    data?: T;                     // 返回数据
    error?: BusinessError;           // 错误信息
}

// 升级结果
interface CheckResult {
    isExistNewVersion: boolean;       // 是否存在新版本
    newVersionInfo: NewVersionInfo;   // 新版本信息
}
```

---

## 6 约束与限制

### 6.1 功能限制

| 限制项 | 说明 | 影响 |
|--------|------|------|
| 仅系统更新 | 不支持应用级更新 | 仅用于系统 OTA 更新 |
| 依赖系统服务 | 必须依赖系统更新服务 | 系统服务故障时不可用 |
| 单例模式 | OtaUpdateManager 为单例 | 限制多实例并发 |

### 6.2 安全限制

| 限制项 | 说明 | 原因 |
|--------|------|------|
| 高危权限 | 需要 UPDATE_SYSTEM 权限 | 系统更新必需 |
| 签名验证 | 依赖系统服务验证 | 应用层无法绕过 |

---

## 7 构建产物

| 模块 | 类型 | 产物名称 | 说明 |
|------|------|----------|------|
| updateapp | HAP | updateapp.hap | 可安装的应用包 |
| ota | HAR | ota.har | OTA 功能共享库 |
| common | HAR | common.har | 公共工具共享库 |

---

## 8 相关文档

| 文档 | 描述 |
|------|------|
| [02_Module_Architecture.md](./02_Module_Architecture.md) | 模块划分与依赖关系 |
| [03_State_Machine.md](./03_State_Machine.md) | 状态机设计 |
| [04_UI_Interaction.md](./04_UI_Interaction.md) | UI 组件和页面 |
| [05_System_Interface.md](./05_System_Interface.md) | 系统接口调用 |
| [06_Security_Review.md](./06_Security_Review.md) | 权限与安全分析 |
| [appendix/Export_Mapping.md](./appendix/Export_Mapping.md) | 本项目 Export 清单 |

---

**最后更新**: 2026-02-07
**对应代码版本**: 当前 HEAD
