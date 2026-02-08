# multimodalinput_input Wiki

## 概述

本文档是 OpenHarmony 多模态输入子系统 (`multimodalinput_input`) 的工程 Wiki，旨在帮助开发者快速理解项目架构、N-API 接口、安全机制和编译构建流程。

## 项目信息

| 属性 | 值 |
|------|-----|
| **仓库路径** | `foundation/multimodalinput/input` |
| **组件名** | `input` |
| **子系统** | `multimodalinput` |
| **版本** | 3.1 |
| **License** | Apache License 2.0 |
| **SA ID** | 3101 |

## 核心定位

基于标准系统提供单点触摸输入能力的模块，向 JS UI 框架或应用框架上报触摸事件，并提供 API 供应用使用。

## 目录结构

```
multimodalinput_input/
├── interfaces/              # 对外接口
│   ├── native/               # Native API
│   │   ├── innerkits/        # 内部子系统API
│   │   │   ├── proxy/        # 代理接口 (input_manager.h)
│   │   │   └── common/       # 公共类型
│   │   └── kits/c/           # C API
│   └── kits/                 # 外部API (预留)
├── frameworks/               # 框架层
│   ├── proxy/                # 客户端代理框架
│   ├── napi/                 # N-API 实现 (14个模块)
│   ├── native/               # Native 框架
│   └── ets/                  # ArkTS API
├── service/                  # 服务层 (40+ 子模块)
├── sa_profile/               # SA 配置文件
├── uinput/                   # 输入事件注入
├── intention/                # 意图识别与跨设备协作
├── util/                     # 工具类
├── common/                   # 公共代码
├── etc/                      # 配置
├── tools/                    # 工具
└── BUILD.gn                 # 根构建文件
```

## 阅读指南

### 新人快速上手顺序

1. **[README](README.md)** - 项目概览与快速开始
2. **[SUMMARY.md](SUMMARY.md)** - 全站导航
3. **[01_Overview.md](01_Overview.md)** - 项目定位与核心能力
4. **[02_NAPI_Reference.md](02_NAPI_Reference.md)** - N-API 接口参考
5. **[03_Architecture.md](03_Architecture.md)** - 架构设计与数据流
6. **[04_GN_Build.md](04_GN_Build.md)** - GN 编译配置
7. **[05_Security_Review.md](05_Security_Review.md)** - 安全风险评审

### 进阶阅读

- **[06_Inner_API.md](06_Inner_API.md)** - 内部模块接口
- **[07_Appendix_Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
- **[08_Appendix_Config.md](appendix/Config_Flags.md)** - 配置开关

## 主要功能模块

### N-API 模块 (14个)

| 模块 | JS API 命名空间 | 功能描述 |
|------|----------------|---------|
| inputEventClient | `@ohos.multimodalInput.inputEventClient` | 事件注入 |
| inputDevice | `@ohos.multimodalInput.inputDevice` | 设备管理 |
| inputMonitor | `@ohos.multimodalInput.inputMonitor` | 事件监控 |
| inputConsumer | `@ohos.multimodalInput.inputConsumer` | 按键消费 |
| pointer | `@ohos.multimodalInput.pointer` | 指针配置 |
| shortKey | `@ohos.multimodalInput.shortKey` | 快捷键 |
| infraredEmitter | `@ohos.multimodalInput.infraredEmitter` | 红外控制 |
| keyEvent | `@ohos.multimodalInput.keyEvent` | 按键事件 |
| mouseEvent | `@ohos.multimodalInput.mouseEvent` | 鼠标事件 |
| touchEvent | `@ohos.multimodalInput.touchEvent` | 触摸事件 |
| gestureEvent | `@ohos.multimodalInput.gestureEvent` | 手势事件 |
| joystickEvent | `@ohos.multimodalInput.joystickEvent` | 摇杆事件 |
| keyCode | `@ohos.multimodalInput.keyCode` | 按键码 |
| intentionCode | `@ohos.multimodalInput.intentionCode` | 意图码 |

### 服务组件 (核心)

- **MMIService** (SA: 3101) - 主服务入口
- **InputEventHandler** - 事件处理责任链
- **DeviceManager** - 设备管理
- **EventDispatcher** - 事件分发
- **IntentionService** - 意图服务

## 设备支持

| 设备 | touch | touchpad | mouse | keyboard |
|:----:|:-----:|:--------:|:-----:|:--------:|
| rk3568 | Y | Y | Y | Y |
| hi3516dv300 | Y | N | N | N |

## SystemCapability 列表

- `SystemCapability.MultimodalInput.Input.Core`
- `SystemCapability.MultimodalInput.Input.InfraredEmitter`
- `SystemCapability.MultimodalInput.Input.Cooperator`
- `SystemCapability.MultimodalInput.Input.Pointer`
- `SystemCapability.MultimodalInput.Input.ShortKey`
- `SystemCapability.MultimodalInput.Input.InputMonitor`
- `SystemCapability.MultimodalInput.Input.InputSimulator`
- `SystemCapability.MultimodalInput.Input.InputDevice`
- `SystemCapability.MultimodalInput.Input.InputConsumer`

## 文档更新

- **生成时间**: 2024-02-06
- **代码版本**: 基于当前仓库代码
- **更新方式**: 手动更新，需随代码变更同步维护

## 相关链接

- [OpenHarmony 多模输入子系统](https://gitee.com/openharmony/multimodalinput_input)
- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [N-API 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/napi-guidelines.md)
