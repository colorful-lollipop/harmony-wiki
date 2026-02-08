# 项目概述

## 1. 项目定位

**applications_call** 是 OpenHarmony 系统的原生通话管理应用，提供完整的移动通信功能。

### 1.1 核心定位

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              applications_call                       │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐    │   │
│  │  │语音通话 │  │视频通话 │  │  移动网络设置   │    │   │
│  │  └─────────┘  └─────────┘  └─────────────────┘    │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐    │   │
│  │  │SIM管理  │  │紧急拨号 │  │  通话参数设置   │    │   │
│  │  └─────────┘  └─────────┘  └─────────────────┘    │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                 │
│                  @ohos.telephony.*                         │
│                          │                                 │
│              ┌───────────┴───────────┐                      │
│              │   Telephony Framework  │                     │
│              │   (通话子系统)          │                     │
│              └───────────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 功能特性

| 功能 | 描述 | 状态 |
|-----|------|-----|
| 语音通话 | 拨打/接听/挂断语音电话 | ✅ 完整 |
| 视频通话 | 拨打/接听视频电话 | ✅ 完整 |
| 来电显示 | 号码归属地、联系人信息 | ✅ 完整 |
| 通话记录 | 历史通话记录管理 | ✅ 完整 |
| 通讯录集成 | 联系人搜索和显示 | ✅ 完整 |
| 通话保持 | 保持/恢复通话 | ✅ 完整 |
| 会议通话 | 多方会议支持 | ✅ 完整 |
| DTMF 支持 | 双音多频信号 | ✅ 完整 |
| 移动网络设置 | APN/网络模式配置 | ✅ 完整 |
| SIM 卡管理 | SIM 状态和信息 | ✅ 完整 |
| 紧急拨号 | 紧急呼叫功能 | ✅ 完整 |

## 2. 模块结构

### 2.1 代码模块

```
callui (entry 模块 - 主入口)
├── src/main/ets/
│   ├── Application/
│   │   └── MyAbilityStage.ts    # 应用生命周期
│   ├── MainAbility/
│   │   └── MainAbility.ts       # UI Ability
│   ├── ServiceAbility/
│   │   ├── ServiceAbility.ts    # Service Ability (IPC)
│   │   └── TelephonyApi.ets     # Telephony API 封装
│   ├── model/                   # 数据模型
│   │   ├── CallManager.ts       # 通话管理器
│   │   ├── CallServiceProxy.ts  # 通话服务代理
│   │   ├── CallDataManager.ts   # 通话数据管理
│   │   └── NotificationManager.ts # 通知管理
│   ├── common/                  # 公共组件
│   │   ├── components/          # UI 组件
│   │   ├── constant/             # 常量定义
│   │   ├── utils/               # 工具类
│   │   └── struct/               # 数据结构
│   └── pages/                    # 页面组件
│       └── index.ets             # 主页面
├── src/main/resources/          # 资源文件
└── module.json                  # 模块配置

mobiledatasettings (feature 模块 - 移动数据设置)
├── src/main/ets/
│   ├── MainAbility/
│   │   └── MainAbility.ts
│   ├── pages/                   # 页面
│   │   ├── index.ets
│   │   ├── apnList.ets
│   │   ├── apnDetail.ets
│   │   └── networkStand.ets
│   ├── model/                   # 数据模型
│   └── common/                  # 公共组件
└── src/main/resources/

common (har 模块 - 公共组件库)
├── src/main/ets/
│   └── components/
│       └── MainPage.ets
└── src/main/module.json5
```

### 2.2 构建产物

| 模块 | 类型 | 产物 | 安装路径 |
|-----|------|-----|---------|
| callui | entry | CallUI.hap | app/com.ohos.callui/ |
| mobiledatasettings | feature | MobileDataSettings.hap | app/com.ohos.callui/ |
| common | har | common.har | (静态库) |

## 3. 技术栈

### 3.1 开发框架

| 技术 | 版本 | 用途 |
|-----|------|-----|
| ArkUI | - | 声明式 UI 框架 |
| ArkTS | - | 类型化 JavaScript |
| OpenHarmony SDK | 9+ | 系统 API |
| hvigor | - | 构建工具 |

### 3.2 系统 API 依赖

| API 模块 | 用途 |
|---------|------|
| `@ohos.telephony.call` | 通话控制 |
| `@ohos.telephony.sim` | SIM 卡管理 |
| `@ohos.telephony.radio` | 无线通信 |
| `@ohos.telephony.sms` | 短信服务 |
| `@ohos.app.ability.*` | Ability 框架 |
| `@ohos.rpc` | IPC 通信 |
| `@ohos.commonEvent` | 公共事件 |
| `@ohos.notification` | 通知管理 |

## 4. 运行环境

### 4.1 设备支持

| 设备类型 | 支持状态 |
|---------|---------|
| 手机 | ✅ 支持 |
| 平板 | ✅ 支持 |
| 智慧屏 | ❌ 不支持 |
| 智能手表 | ❌ 不支持 |
| 车机 | ❌ 不支持 |

### 4.2 系统要求

- **最低 API Level**: 9
- **目标 API Level**: 9
- **系统权限**: 需用户授权 (详见权限清单)

## 5. 关键约束

### 5.1 权限要求

应用启动时需要用户授权的关键权限：

| 权限 | 用途 | 敏感度 |
|-----|------|-------|
| `ohos.permission.PLACE_CALL` | 发起通话 | 高 |
| `ohos.permission.ANSWER_CALL` | 接听电话 | 高 |
| `ohos.permission.GET_TELEPHONY_STATE` | 获取通话状态 | 中 |
| `ohos.permission.SET_TELEPHONY_STATE` | 设置通话状态 | 高 |
| `ohos.permission.READ_CONTACTS` | 读取联系人 | 高 |
| `ohos.permission.SEND_MESSAGES` | 发送短信 | 高 |

### 5.2 安全约束

- 应用签名: `signature/callui.p7b`
- 签名算法: RSA2048-SHA256 (OpenHarmony 标准)
- 网络访问: 仅移动网络相关配置

## 6. 版本历史

| 版本 | 日期 | 变更 |
|-----|------|-----|
| 1.0.4.032 | - | 当前版本 |
| 1.0.3.xxx | - | 早期版本 |

## 7. 相关文档

- [架构设计](02_Architecture.md)
- [API 参考](03_API_Reference.md)
- [构建指南](04_Build_Guide.md)
- [安全评审](05_Security_Review.md)
- [模块详解](06_Module_Details.md)
