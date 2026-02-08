# Advertising 子系统概览

## 项目定位

**@ohos/advertising** 是 OpenHarmony 的广告服务组件，使应用无需集成 SDK 即可请求和展示广告。

### 核心能力

| 能力 | 说明 | 入口 |
|-----|------|------|
| 广告请求 | 加载单个/多槽位广告 | `AdLoader.loadAd()` |
| 广告展示 | 全屏/非全屏广告展示 | `advertising.showAd()` |
| 请求体生成 | 获取广告请求参数 | `AdLoader.getAdRequestBody()` |
| 广告组件 | UI 组件方式展示广告 | `AdComponent` |

## 关键概念

### SA (System Ability)

SA 是运行在服务端进程的 IPC 实体，广告 SA (ID: 6104) 接收请求并转发给广告平台处理。

```
广告 SA 接收请求 → 转发给广告平台 → 返回广告内容 → 展示
```

### 广告平台

由第三方实现的 `ServiceExtensionAbility`，负责：
1. 接收 SA 的广告请求
2. 向广告网络请求最佳广告
3. 将广告内容返回给 SA 展示

### 广告类型

- **全屏广告**: 奖励广告等沉浸式广告
- **非全屏广告**: 原生广告等组件化广告

## 运行环境

| 依赖 | 说明 |
|-----|------|
| 系统能力 | `SystemCapability.Advertising.Ads` |
| 运行时 | Standard (标准系统) |
| ROM 占用 | ~300KB |
| RAM 占用 | ~1024KB |

## 目录结构

```
advertising/
├── common/                          # 公共代码
│   ├── error_code/                  # 错误码定义
│   ├── ipc/                         # IPC 通信 (代理/桩)
│   ├── log/                         # 日志封装
│   ├── model/                       # 数据模型
│   └── utils/                       # 工具函数
├── frameworks/
│   └── js/napi/
│       ├── ads/                     # 主广告 N-API
│       │   ├── include/             # 头文件
│       │   └── src/                 # 实现
│       ├── adcomponent/             # 广告组件 N-API
│       ├── autoadcomponent/         # 自动广告组件
│       ├── adsservice_extension_ability/  # SA 扩展能力
│       ├── adsservice_extension_context/  # SA 上下文
│       └── extension/               # 扩展基类
└── frameworks/cj/ffi/ads/          # Cangjie FFI 接口
```

## 模块职责

| 模块 | 职责 |
|-----|------|
| `common/ipc` | IPC 通信代理/桩实现 |
| `common/utils` | JSON 处理、通用工具 |
| `frameworks/js/napi/ads` | N-API 入口、广告加载/展示 |
| `frameworks/js/napi/adcomponent` | 广告组件 UI |
| `frameworks/cj/ffi/ads` | Cangjie FFI 绑定 |

## 相关文档

- [架构说明](Architecture.md) - 详细组件图与数据流
- [N-API 接口](NAPI_Reference.md) - API 使用指南
- [安全评审](Security_Review.md) - 安全考量
