# 00_概览

## 项目定位

公共事件服务（Common Event Service, CES）是 OpenHarmony 通知子系统的核心组件，为应用程序提供**订阅、发布、退订公共事件**的能力。

### 核心能力

| 能力 | 说明 |
|------|------|
| 事件发布 | 应用或系统可以发布公共事件 |
| 事件订阅 | 应用可以订阅感兴趣的公共事件 |
| 事件退订 | 取消订阅，停止接收事件 |
| 有序事件 | 支持按优先级顺序传递事件 |
| 粘性事件 | 支持粘性事件（事件保留） |
| 跨用户 | 支持多用户场景的事件管理 |

### 系统能力

```
Syscap: SystemCapability.Notification.CommonEvent
```

## 运行环境

### 依赖组件

根据 `bundle.json` 配置，CES 依赖以下系统组件：

| 组件 | 用途 |
|------|------|
| bundle_framework | 包管理框架 |
| ipc | 进程间通信 |
| access_token | 权限管理 |
| safwk | System Ability 框架 |
| samgr | 服务管理 |
| ability_base | Ability 基础框架 |
| ability_runtime | Ability 运行时 |
| eventhandler | 事件处理 |
| hilog | 日志 |
| ffrt | 任务调度 |

### 系统服务

CES 以 **System Ability (SA)** 形式运行：

| 属性 | 值 |
|------|-----|
| SA ID | 3299 |
| 进程 | foundation |
| 库路径 | libcesfwk_services.z.so |
| 启动方式 | run-on-create |

## 关键概念

### 公共事件类型

| 类型 | 说明 |
|------|------|
| 系统公共事件 | 系统关键服务发布的事件（如 Hap 安装、关机等） |
| 自定义公共事件 | 应用自定义用于跨应用通信的事件 |

### 事件属性

| 属性 | 类型 | 说明 |
|------|------|------|
| event | string | 事件名称 |
| bundleName | string | 包名 |
| code | int | 结果代码 |
| data | string | 自定义数据 |
| isOrdered | bool | 是否有序事件 |

### 订阅者属性

| 属性 | 类型 | 说明 |
|------|------|------|
| events | Array<string> | 订阅的事件列表 |
| publisherPermission | string | 发布者所需权限 |
| publisherDeviceId | int | 设备ID |
| userId | int | 用户ID |
| priority | int | 优先级 (-100~1000) |

## 项目结构

```
/base/notification/common_event_service/
├── frameworks/          # 框架层
│   ├── core/            # 核心框架 (IPC 接口)
│   ├── native/          # Native 接口
│   ├── common/          # 公共组件
│   └── extension/       # 扩展框架
├── interfaces/          # 对外接口
│   ├── inner_api/       # Native 接口声明
│   └── kits/            # 各类绑定
│       ├── napi/        # JS 绑定
│       ├── ndk/         # C/C++ 绑定
│       ├── ani/         # ANI 绑定
│       └── cj/         # CJ 绑定
├── services/           # 服务实现
├── sa_profile/         # SA 配置
├── tools/              # 工具
├── BUILD.gn            # 构建入口
└── bundle.json         # 包配置
```

## 版本信息

| 信息 | 值 |
|------|-----|
| 包名 | @ohos/common_event_service |
| 子系统 | notification |
| ROM 估算 | 2000KB |
| RAM 估算 | 3000KB |

## 相关文档

- [架构设计](01_Architecture.md)
- [N-API 接口](02_N-API.md)
- [内部 API](03_Inner_API.md)
