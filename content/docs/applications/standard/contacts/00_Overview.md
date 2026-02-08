# 项目概览

## 项目定位

Contacts（联系人）是 OpenHarmony 标准系统中预置的系统应用，提供完整的联系人管理解决方案。

### 核心功能

| 功能模块 | 功能描述 |
|----------|----------|
| 拨号盘 | 数字键盘输入、联系人搜索、快速拨号 |
| 通话记录 | 查看、删除通话记录、通话记录批量管理 |
| 联系人列表 | 分组显示联系人、提供搜索和索引 |
| 联系人详情 | 查看联系人详细信息、编辑、删除 |
| 联系人新建 | 创建新联系人、导入联系人 |
| 账号管理 | 关联 SIM 卡账号联系人 |

### 系统集成

- **数据提供方**: [applications_contactsdata](https://gitee.com/openharmony/applications_contactsdata)
- **短信集成**: [applications_mms](https://gitee.com/openharmony/applications_mms)
- **通话集成**: [applications_call](https://gitee.com/openharmony/applications_call)

## 技术栈

### 核心框架

| 组件 | 技术 | 说明 |
|------|------|------|
| 开发语言 | ArkTS | TypeScript 的超集，静态类型 |
| UI 框架 | ArkUI | 声明式 UI 开发框架 |
| 构建工具 | hvigor | 基于 Gradle 的构建系统 |
| 数据存储 | RDB | 关系型数据库 |

### 系统能力

| API 模块 | 用途 |
|----------|------|
| `@ohos.telephony.call` | 拨打电话、呼叫管理 |
| `@ohos.telephony.sim` | SIM 卡管理 |
| `@ohos.app.ability.*` | Ability 生命周期管理 |
| `@ohos.window` | 窗口管理 |
| `@ohos.notificationManager` | 通知管理 |
| `@ohos.worker` | 多线程 Worker |

## 运行环境

### 系统要求

| 要求 | 规格 |
|------|------|
| 系统类型 | OpenHarmony 标准系统 |
| 编译 SDK | 23 |
| 兼容 SDK | 23 |

### 资源占用

| 资源 | 大小 |
|------|------|
| ROM | 6 MB |
| RAM | 86,650 KB |

### 设备适配

- **minWindowWidth**: 320 vp
- **minWindowHeight**: 700 vp
- **deviceTypes**: default

## 项目结构

### 顶层目录

```
contacts/
├── AppScope/           # 应用作用域配置
├── common/             # 公共模块 (工具类、常量、权限)
├── doc/                # 文档和资源图片
├── entry/              # 主入口模块
│   └── src/main/ets/
│       ├── Application/     # 应用生命周期
│       ├── MainAbility/    # 主入口 Ability
│       ├── StaticSubscriber/  # 静态订阅者
│       ├── workers/        # Worker 多线程
│       ├── model/          # 数据层
│       ├── pages/          # UI 页面
│       ├── presenter/      # MVP Presenter
│       ├── component/      # UI 组件
│       └── util/           # 工具类
├── feature/             # 功能模块
│   ├── account/         # 账号管理
│   ├── call/           # 通话记录
│   ├── contact/        # 联系人核心
│   ├── dialpad/       # 拨号盘
│   └── phonenumber/   # 电话号码处理
├── hvigor/             # 构建配置
├── sign/               # 签名文件
├── bundle.json         # 模块配置
├── build-profile.json5  # 构建配置
└── wiki/              # 本文档
```

### 关键入口

| 文件路径 | 职责 |
|----------|------|
| `entry/src/main/ets/MainAbility/MainAbility.ts` | 主 Ability，处理生命周期 |
| `entry/src/main/ets/Application/MyAbilityStage.ts` | 应用 Stage 入口 |
| `entry/src/main/ets/MainAbility/index.ts` | 页面路由入口 |

## 权限声明

本应用声明了以下系统权限：

### 联系人相关

| 权限名 | 用途 |
|--------|------|
| `ohos.permission.READ_CONTACTS` | 读取联系人数据 |
| `ohos.permission.WRITE_CONTACTS` | 新建、编辑、删除联系人 |

### 通话相关

| 权限名 | 用途 |
|--------|------|
| `ohos.permission.READ_CALL_LOG` | 读取通话记录 |
| `ohos.permission.WRITE_CALL_LOG` | 写入通话记录 |
| `ohos.permission.PLACE_CALL` | 发起电话呼叫 |

### 电话状态

| 权限名 | 用途 |
|--------|------|
| `ohos.permission.GET_TELEPHONY_STATE` | 获取电话状态 |
| `ohos.permission.SET_TELEPHONY_STATE` | 设置电话状态 |
| `ohos.permission.MANAGE_VOICEMAIL` | 管理语音信箱 |

### 其他

| 权限名 | 用途 |
|--------|------|
| `ohos.permission.VIBRATE` | 振动反馈 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 获取包信息 |
| `ohos.permission.NOTIFICATION_CONTROLLER` | 通知控制 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 后台启动 |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 |

> **证据来源**: `entry/src/main/module.json5` lines 61-114

## 术语表

| 术语 | 说明 |
|------|------|
| ArkTS | OpenHarmony 的应用开发语言，TypeScript 超集 |
| ArkUI | OpenHarmony 的声明式 UI 框架 |
| Ability | OpenHarmony 的应用组件 |
| RDB | Relational Database，关系型数据库 |
| Worker | OpenHarmony 的多线程机制 |
| Stage | 应用模型的核心概念 |
