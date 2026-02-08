# 项目概览

## 1.1 组件定位与核心能力

HiAppEvent（Application Event Service）是 OpenHarmony DFX（Developer Experience）子系统下的核心组件之一，为 OpenHarmony 应用提供统一的事件打点（Event Logging）能力。该组件的核心设计目标是帮助应用开发者记录应用在运行过程中产生的各类关键事件信息，包括但不限于故障信息、统计信息、安全信息和用户行为信息，从而支撑开发者对应用运行状态的全面分析和问题诊断。

从组件定位角度来看，HiAppEvent 扮演着应用与 DFX 基础设施之间的桥梁角色。应用层通过调用 HiAppEvent 提供的标准化 API 接口，将各类业务事件上报至系统；系统层则负责事件的接收、存储、分发和持久化处理。这一设计使得应用开发者无需关心底层的事件处理细节，只需聚焦于业务逻辑中的关键节点标记，即可实现完整的应用行为追踪能力。

HiAppEvent 的核心能力可以从以下几个维度进行理解。首先是**事件打点能力**，支持应用以同步或异步方式记录各类事件，事件类型涵盖 FAULT（故障类型）、STATISTIC（统计类型）、SECURITY（安全类型）和 BEHAVIOR（行为类型）四种预定义分类。其次是**事件配置能力**，允许开发者对打点功能进行开关控制、存储配额配置等自定义设置。再次是**事件观察能力**，支持注册事件观察者（Watcher），当匹配条件的事件发生时自动触发回调通知。最后是**事件转发能力**，支持配置事件处理器（Processor）将事件上报至远程服务器，实现应用数据的云端分析。

从技术实现角度来看，HiAppEvent 采用分层架构设计，底层基于 C/C++ 实现核心事件处理逻辑，上层通过 N-API（Native API）封装为 JavaScript 接口供应用调用，同时提供 Native API 供 C/C++ 原生应用直接使用。这种多层次的接口设计满足了不同开发场景的需求，既保证了高频调用的性能要求，又提供了良好的开发便利性。

## 1.2 运行环境与系统依赖

HiAppEvent 组件的运行环境具有明确的系统依赖要求，开发者和部署人员需要确保目标系统满足这些依赖条件才能正常使用组件功能。

**系统版本要求**：根据 bundle.json 文件中的配置信息，HiAppEvent 适配的 OpenHarmony 系统类型为 standard（标准系统），版本从 API 8 开始支持，当前最新支持的 API 版本达到 20。这表明 HiAppEvent 是一个面向标准设备（如手机、平板、智能电视等）的事件服务组件。

**系统能力依赖**：HiAppEvent 依赖的系统能力（SystemCapability）定义为 `SystemCapability.HiviewDFX.HiAppEvent`，所有使用 HiAppEvent API 的应用必须在应用的配置文件中声明对该系统能力的依赖。具体而言，需要在 module.json5 或 equivalent 配置文件中添加如下声明：

```json
{
  "requestPermissions": [],
  "abilities": [],
  "processes": [],
  "module": {
    "requestPermissions": [],
    "metadata": []
  }
}
```

关于权限声明，由于 HiAppEvent 的核心 API 主要面向应用自身的事件记录场景，因此在标准使用情况下无需申请额外的敏感权限。但如果应用需要访问其他应用产生的事件数据，则可能需要申请相应的跨应用访问权限，这部分内容将在安全评审章节详细说明。

**编译器要求**：HiAppEvent 的原生代码部分（Native 代码）依赖 Clang 8.0.0 及以上版本的编译器进行编译，C++ 标准要求为 C++11 及以上。这一要求在 hiappevent.gni 构建配置文件中明确声明，任何尝试使用其他编译器或更低 C++ 标准版本的构建都将失败。

**运行时依赖库**：HiAppEvent 在运行时依赖多个 OpenHarmony 系统库，包括但不限于：hilog（日志库）、hisysevent（系统事件库）、ffrt（任务调度库）、ipc_core（IPC 通信库）、relational_store（关系型数据库存储库）等。这些依赖关系在 BUILD.gn 文件的 external_deps 配置段中声明，构建系统会自动处理这些依赖的链接。

## 1.3 目录结构与模块职责

HiAppEvent 组件的代码目录结构清晰地反映了其模块化设计思想，各目录承担明确的职责边界。以下是经过归类整理的目录结构说明（测试目录 test 相关代码不计入本次文档覆盖范围）：

```
/base/hiviewdfx/hiappevent/
├── interfaces/                    # 对外接口存放目录
│   └── native/                    # Native 接口实现
│       ├── inner_api/             # Inner API（系统内部使用）
│       │   ├── include/           # 头文件目录
│       │   │   ├── app_event.h
│       │   │   ├── app_event_processor.h
│       │   │   ├── app_event_processor_mgr.h
│       │   │   └── base_type.h
│       │   └── src/              # 实现代码
│       └── kits/                 # Kits API（对外 NDK）
│           └── include/hiappevent/
│               ├── hiappevent.h
│               ├── hiappevent_cfg.h
│               ├── hiappevent_event.h
│               └── hiappevent_param.h
├── frameworks/                    # 框架代码目录
│   ├── native/                   # Native 框架实现
│   │   ├── libhiappevent/        # 核心库（libhiappevent_base）
│   │   │   ├── include/          # 核心头文件
│   │   │   ├── cache/            # 事件缓存模块
│   │   │   ├── cleaner/          # 事件清理模块
│   │   │   ├── dfr/              # DFR（Design for Reliability）配置
│   │   │   ├── load/             # 模块加载模块
│   │   │   ├── observer/         # 事件观察者模块
│   │   │   ├── utility/          # 工具模块
│   │   │   ├── BUILD.gn          # 构建配置
│   │   │   └── *.cpp             # 核心实现文件
│   │   └── ndk/                  # NDK 接口实现
│   │       ├── include/          # NDK 头文件
│   │       ├── src/              # NDK 实现代码
│   │       └── BUILD.gn
│   └── js/                       # JS 接口框架
│       └── napi/                 # N-API 实现
│           ├── include/          # N-API 头文件
│           ├── src/              # N-API 实现代码
│           └── BUILD.gn
├── sa_profile/                    # SA（System Ability）配置
├── hiappevent.gni                # GN 配置导入
├── hiappevent_aafwk.gni          # AAFWK 配置导入
├── bundle.json                   # 组件清单文件
├── README_zh.md                  # 中文 README
└── README.md                     # 英文 README
```

各模块的核心职责可以归纳如下：

**interfaces/native/inner_api/** 目录包含 Inner API 的实现代码，这部分 API 供 OpenHarmony 系统内部的其他组件使用，不直接对应用开发者开放。这些接口通常具有更强大的功能或更灵活的配置能力，但使用时有更严格的约束条件。

**interfaces/native/kits/include/hiappevent/** 目录包含 NDK（Native Development Kit）头文件，定义了供 C/C++ 原生应用调用的 API 接口。这些接口以 C 语言风格提供，具有良好的 ABI 兼容性，是高性能场景或系统级应用的首选接入方式。

**frameworks/native/libhiappevent/** 目录是 HiAppEvent 的核心实现所在，包含以下关键子模块：
- **cache/**：事件数据的本地缓存管理，负责将事件数据写入本地存储并在存储空间不足时进行清理
- **cleaner/**：事件数据的清理逻辑，包括数据库清理和日志文件清理两种机制
- **dfr/**：可靠性设计相关的配置管理，如事件存储配额、开关配置等
- **load/**：模块加载机制，支持动态加载不同的事件处理器配置
- **observer/**：事件观察者机制，支持异步事件分发和回调通知
- **utility/**：通用工具函数，包括时间处理、JSON 解析、文件操作等

**frameworks/js/napi/** 目录包含 N-API 封装层实现，负责将 Native 接口桥接到 JavaScript 层。该层处理 JavaScript 参数的解析校验、Promise/Callback 异步模式的封装、以及与 JS 引擎的交互细节。

## 1.4 关键概念与术语

为便于后续文档的理解，本节对 HiAppEvent 中的关键概念和术语进行统一定义说明。

**Event（事件）**：指应用在运行过程中产生的需要记录的信息单元。每个事件由三个核心要素唯一标识：domain（领域）、name（名称）和 type（类型）。其中 domain 用于标识事件的来源领域，name 用于标识事件的具体名称，type 用于标识事件的类型分类。事件还可以携带一组可选的键值对参数（Param），用于记录事件的详细上下文信息。

**EventType（事件类型）**：HiAppEvent 预定义的四种事件分类枚举值，包括 FAULT（故障类型，值为 1）、STATISTIC（统计类型，值为 2）、SECURITY（安全类型，值为 3）和 BEHAVIOR（行为类型，值为 4）。不同类型的事件在存储、分发等环节可能具有不同的处理策略。

**ParamList（参数列表）**：用于组织事件参数的链表结构。在 Native API 中，ParamList 是一个指向 ParamListNode 链表头节点的指针，每个节点包含参数名称和参数值。在 C++ 接口中，ParamList 通常对应 STL 容器（如 std::map 或 std::vector）。

**Watcher（事件观察者）**：一种事件订阅机制，允许应用注册对特定条件事件的监听。当满足条件的事件发生时，系统会自动调用 Watcher 注册的回调函数。Watcher 支持配置过滤条件（事件领域、类型、名称等）和触发条件（事件数量、时间间隔等）。

**Processor（事件处理器）**：一种事件转发机制，允许应用配置将事件数据上报至远程服务器。Processor 支持配置路由信息（服务器地址、认证信息等）、上报策略（实时上报、批量上报、定时上报等）和上报过滤条件。

**AppEventPack（事件包）**：HiAppEvent 内部用于封装完整事件信息的数据结构，包含事件的 domain、name、type、params 等全部信息，以及时间戳、应用标识等元数据。AppEventPack 是事件在 HiAppEvent 内部各模块间流转的主要载体。

**EventStore（事件存储）**：HiAppEvent 的持久化存储模块，负责将事件数据写入本地文件或数据库。存储策略支持按事件类型分类存储，支持存储空间配额控制，支持自动清理过期数据。

## 1.5 版本与兼容性

HiAppEvent 组件遵循 OpenHarmony 的版本管理规范，其 API 版本演进和兼容性策略如下所述。

**API 版本演进**：
- **API 8**：HiAppEvent 首次对外发布，支持基础的 JS API 和 Native API，包括 write() 接口和 configure() 接口
- **API 12**：新增 Watcher 功能，支持事件观察者机制；新增 Processor 功能，支持事件处理器机制
- **API 15**：新增 Config 功能，支持事件配置；新增 HiAppEvent_Config 结构体和相关操作接口
- **API 18**：新增 Processor 高级功能，包括路由配置、用户 ID 配置、用户属性配置等
- **API 20**：Processor 新增 SetConfigName 接口，支持配置名称设置

**向后兼容性**：HiAppEvent 严格遵循 OpenHarmony 的 API 兼容性要求，已发布的正式 API 不会发生不兼容变更。这意味着使用低版本 API 开发的应用，在升级到更高版本的 OpenHarmony 系统后，无需修改代码即可继续正常运行。

**废弃策略**：对于计划废弃的 API，OpenHarmony 采用标准的废弃通知流程。首先在 API 文档中标记废弃注解（deprecated），并提供替代方案的说明。在至少两个主要版本周期后，才会真正移除该 API。这种策略为应用开发者提供了充足的迁移时间窗口。

**SDK 适配**：不同版本的 HiAppEvent N-API 对应不同的 JS SDK 包：
- API 7 及更早版本：`hiappevent` 模块
- API 9 及以后版本：`hiappevent_v9` 模块（作为默认推荐）

应用开发者应根据目标设备的 OpenHarmony 版本来选择合适的 API 集进行开发。
