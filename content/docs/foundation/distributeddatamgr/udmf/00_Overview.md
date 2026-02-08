# 项目概述

## 项目定位

统一数据管理框架（Unified Data Management Framework，简称 UDMF）是 OpenHarmony 分布式数据管理子系统的核心组件，专注于定义跨应用、跨设备以及跨平台过程中的数据标准。UDMF 的核心目标是提供统一的 OpenHarmony 数据语言和标准化的数据接入与读取通路，从而解决数据协同通道繁杂、数据协同标准不一致、数据协同安全策略不一致等痛点问题。

从系统架构角度看，UDMF 位于业务层与存储层之间，向上为各类应用提供统一的数据操作接口，向下对接分布式 KVDB、分布式数据对象和分布式 RDB 等多种存储后端。这种中间层定位使得 UDMF 能够统一管理不同应用、不同设备之间的数据交换，同时保证数据的一致性和安全性。

根据源码证据（`README_zh.md`），UDMF 实现了以下核心价值：第一，构建 OpenHarmony 数据跨应用、跨设备交互的标准定义，降低应用和业务数据交互的成本，促进数据生态建设；第二，提供安全、标准化的数据通路，使业务能够以统一的方式进行跨应用、跨设备数据交互。

## 核心能力

UDMF 框架提供四大核心能力，分别对应不同的应用场景和技术实现。

**第一种能力是标准化数据定义。** UDMF 定义了三种类型的数据模型：基础数据类型（如 File、Text 等）支持跨应用、跨设备和跨平台流转；系统定义类型（System Defined Type，SDT）与具体平台绑定，如 Form（UI 卡片信息）、AppItem（应用描述信息）、PixelMap（缩略图格式）等；应用自定义类型（App Defined Type，ADT）允许单个应用定义自己的数据结构，仅在应用内部跨设备和跨平台流转。这三种类型构成了 UDMF 完整的数据类型体系。

**第二种能力是标准化数据通路。** UDMF 为各种业务场景（如跨应用跨设备数据拖拽）提供了统一的数据接入与读取通路。数据 URI 定义为 `udmf://intension/bundleName/groupName/guid`，其中协议名固定为 `udmf`，`intension` 表示通道分类（如数据拖拽），`bundleName` 是数据来源应用的包名，`groupName` 支持批量数据分组管理，`guid` 是系统生成的全唯一数据标识。

**第三种能力是数据生命周期管理。** UDMF 框架负责管理数据的创建、存储、检索、更新和删除等全生命周期操作。通过统一的客户端接口（`UdmfClient` 和 `UtdClient`），应用可以方便地进行数据操作，而无需关心底层存储细节。

**第四种能力是数据安全与权限控制。** UDMF 实现了基于访问令牌（AccessToken）的权限检查机制，支持应用级别的数据隔离和跨应用数据共享控制。共享选项（ShareOptions）定义了两种模式：`SHARE_OPTIONS_IN_APP` 表示仅同一应用内可用，`SHARE_OPTIONS_CROSS_APP` 表示允许跨应用使用。

## 架构特点

UDMF 架构设计遵循分层和模块化原则，具有以下技术特点。

**分层架构设计。** UDMF 采用三层架构：业务接口层提供 N-API（JS 接口）、NDK（C 接口）和 InnerKit（C++ 接口）三种接入方式；框架核心层实现数据管理、类型管理和存储适配等核心逻辑；存储后端层对接分布式 KVDB、分布式数据对象和分布式 RDB 等存储系统。这种分层设计使得各层职责清晰，便于维护和扩展。

**多语言接口支持。** 为满足不同开发场景的需求，UDMF 提供了三种接口层：N-API 接口面向 JS/ArkTS 应用开发者，位于 `interfaces/jskits/` 目录；NDK 接口面向 Native 应用开发者，位于 `interfaces/ndk/` 目录；InnerKit 接口面向系统组件开发者，位于 `interfaces/innerkits/` 目录。三种接口层通过统一的框架核心层实现功能复用。

**分布式数据同步。** UDMF 原生支持跨设备数据同步，通过 `Sync` 接口可以触发数据在不同设备间的同步操作。同步机制基于分布式 KVDB 实现，支持增量同步和冲突处理。

**统一类型系统。** UTD（Uniform Type Descriptor）类型系统是 UDMF 的核心抽象，定义了约 200 种预定义数据类型，涵盖文本、图像、音频、视频、文件、应用程序等多种数据格式。类型系统支持层次关系查询（如 `BelongsTo`、`IsLower`、`IsHigher`），并支持应用注册自定义类型。

## 技术约束

使用 UDMF 进行开发需要了解以下技术约束。

**数据大小限制。** 单条数据记录大小不超过 2MB（`UnifiedData::MAX_DATA_SIZE = 200 * 1024 * 1024`），每个分组整体大小不超过 4MB。这些限制确保了数据操作的性能和内存使用可控。

**系统能力要求。** 使用 UDMF 需要声明系统能力 `SystemCapability.DistributedDataManager.UDMF.Core`，该能力在 `bundle.json` 中配置。

**运行时依赖。** UDMF 依赖以下系统服务：Ability Runtime（用于应用上下文）、KV Store（用于数据持久化）、IPC/Samgr（用于进程间通信）、Access Token（用于权限管理）。这些依赖在 `bundle.json` 的 `deps` 字段中声明。

## 源码位置

UDMF 源码位于 OpenHarmony 仓库的 `foundation/distributeddatamgr/udmf` 目录，目录结构如下：

```
foundation/distributeddatamgr/udmf/
├── interfaces/           # 对外接口声明
│   ├── jskits/         # N-API 接口
│   ├── ndk/            # NDK 接口
│   ├── innerkits/      # InnerKit 接口
│   ├── taihe/          # Taihe ANI/ETS 绑定
│   ├── cj/             # Cangjie FFI 绑定
│   └── components/     # UI 组件
├── framework/           # 核心实现
│   ├── common/         # 公共工具
│   ├── innerkitsimpl/  # Native 接口实现
│   └── jskitsimpl/     # JS 接口实现
├── adapter/            # 适配层
├── conf/               # 配置
└── BUILD.gn            # 根构建文件
```

## 相关文档

建议结合以下文档进一步了解 UDMF：

- [N-API 接口规范](./10_NAPI_Reference.md)：JS/ArkTS 应用开发接口
- [NDK 接口规范](./11_NDK_Reference.md)：Native 应用开发接口
- [目录结构](./01_Directory_Structure.md)：模块职责划分
- [数据类型体系](./02_Data_Types.md)：UTD 类型系统详解
- [服务层架构](./20_Service_Layer.md)：IPC 通信和服务实现
