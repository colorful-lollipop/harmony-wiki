# 项目概览

## 1.1 项目定位

应用文件服务（AppFileService）是 OpenHarmony 系统中负责**应用数据管理**和**文件分享**的核心子系统。该模块为上层应用提供统一的文件操作接口，同时为系统备份恢复功能提供底层支撑。

### 核心职责

应用文件服务主要承担以下三大核心职责：

**第一，文件分享能力**。该模块实现应用间 URI 授权机制，允许应用将特定文件或目录的访问权限临时或永久地授予其他应用。通过统一资源标识符（URI）的方式，既保护了原始文件的路径信息，又实现了灵活细粒度的权限控制。此能力主要面向第三方应用开发者，用于实现应用间数据共享场景。

**第二，备份恢复能力**。该模块提供完整的数据备份和恢复解决方案，支持将应用数据从源设备迁移至目标设备。该能力涵盖完整备份、增量备份、选择性恢复等多种场景，支持系统应用、三方应用以及公共数据的统一备份恢复。备份服务以独立系统能力（SA）的方式运行，具有独立的进程生命周期管理。

**第三，URI 管理能力**。该模块提供文件路径与 URI 之间的相互转换功能，支持路径标准化、URI 合法性验证等操作。作为文件分享和备份恢复的基础能力，URI 管理确保了跨应用、跨设备数据操作的正确性和一致性。

### 所属子系统与版本

该模块隶属于 OpenHarmony 的 **filemanagement** 子系统，当前版本为 **3.1**，适配标准系统和轻量系统两种系统类型。模块编译后的 ROM 和 RAM 占用均控制在 1024KB 以内，具有良好的资源占用表现。

## 1.2 核心能力详解

### 1.2.1 文件分享能力

文件分享能力通过 URI 授权机制实现，其工作原理如下：分享方应用调用授权接口，生成具有特定权限的 URI；被授权方应用通过该 URI 访问分享方文件，无需知道文件的实际存储路径。权限模式支持读模式（READ_MODE）和写模式（WRITE_MODE）两种基本类型，以及创建模式（CREATE_MODE）、删除模式（DELETE_MODE）、重命名模式（RENAME_MODE）等扩展模式。

权限授予后可选择持久化保存或临时使用。持久化权限在系统重启后仍然有效，适用于长期的数据访问关系；临时权限在权限授予应用退出后自动失效，适用于单次数据访问场景。权限支持激活和停用操作，便于应用动态管理已分享的权限。系统同时提供权限查询接口，应用可检查特定 URI 或路径的持久化权限状态。

### 1.2.2 备份恢复能力

备份恢复能力是应用文件服务的核心功能之一，采用分层架构设计。最上层为 JS N-API 接口层，面向应用开发者提供易用的编程接口；中间层为备份套件内部接口（backup_kit_inner），封装核心业务逻辑；底层为备份系统能力（backup_sa），以独立进程方式运行，负责实际的备份调度和数据传输。

完整备份流程包括以下阶段：首先调用初始化接口创建备份会话；然后通过扩展接口获取待备份数据；数据经过 tar 归档和可选的加密压缩处理后，通过 IPC 传输至目标位置。增量备份在完整备份基础上，仅备份自上次备份以来发生变化的数据，显著减少备份时间和存储空间占用。

恢复流程支持全量恢复和选择性恢复两种模式。全量恢复将备份数据完整还原至设备；选择性恢复允许用户指定恢复特定应用或特定类型的数据。恢复过程中支持安装应用、恢复数据等多种组合操作，满足不同迁移场景的需求。

### 1.2.3 URI 管理能力

URI 管理能力提供文件路径与 URI 之间的双向转换功能。从路径生成 URI 时，系统自动添加标准的 URI 前缀和文件类型标识；从 URI 解析路径时，系统验证 URI 格式的正确性并提取路径信息。URI 对象还提供丰富的方法，包括获取文件名、获取目录 URI、判断是否为远程 URI、路径标准化等操作。

URI 标准化功能用于处理路径中的特殊字符和相对路径引用，确保生成的 URI 具有唯一性和可移植性。远程 URI 判断功能用于区分本地文件和分布式文件系统中的远程文件，为跨设备文件访问提供判断依据。

## 1.3 系统能力依赖

### 1.3.1 System Capabilities

该模块依赖以下系统能力声明：

```
SystemCapability.FileManagement.AppFileService
SystemCapability.FileManagement.StorageService.Backup
SystemCapability.FileManagement.AppFileService.FolderAuthorization
```

第一项能力声明为基础文件服务能力，任何使用该模块的应用都必须声明此能力。第二项能力声明用于启用备份恢复功能，适用于需要数据迁移或备份功能的场景。第三项能力声明用于启用文件夹授权功能，适用于需要向其他应用分享文件权限的场景。

### 1.3.2 外部组件依赖

该模块的正常运行依赖以下外部组件：

**Ability 框架相关**：`ability_base` 和 `ability_runtime` 提供 Want 机制、Ability 生命周期管理、Extension 能力框架等基础能力。文件分享功能依赖 `ability_runtime` 中的 URI 权限管理器；备份恢复功能依赖 `ability_runtime` 中的 Extension 能力框架来加载备份扩展。

**包管理相关**：`bundle_framework` 提供应用包信息查询、应用安装状态检查等能力。备份恢复过程中需要查询目标应用是否已安装、获取应用版本信息等，这些能力由 `bundle_framework` 提供。

**系统能力框架**：`safwk`（System Ability Framework）和 `samgr`（System Ability Manager）提供系统能力的注册、发现、生命周期管理等基础能力。备份服务作为系统能力运行，必须依赖这两个组件。

**进程间通信**：`ipc` 提供Binder通信机制，支撑服务框架的IPC调用。备份恢复过程中涉及大量IPC数据传输，是性能关键路径。

**存储管理**：`storage_service` 提供存储设备信息、磁盘空间查询等能力。备份服务在开始备份前需要检查目标存储空间是否充足。

**权限管理**：`access_token` 提供权限校验、访问令牌管理等能力。文件分享和备份恢复都涉及敏感数据操作，必须通过权限管理模块进行访问控制。

## 1.4 目录结构

### 1.4.1 整体目录布局

```
app_file_service/
├── interfaces/                  # 接口层，对外提供编程接口
│   ├── api/                    # API实现（N-API内部代码）
│   ├── inner_api/              # 对内内部接口（InnerAPI）
│   ├── innerkits/              # 对内Native接口
│   └── kits/                   # 对外接口（JS/NDK/CJ/泰河）
├── frameworks/                 # 框架层，实现业务逻辑
│   ├── js/                     # JS框架代码
│   └── native/                 # Native框架代码
├── services/                   # 服务层，系统能力实现
│   └── backup_sa/              # 备份系统能力
├── utils/                      # 工具层，通用功能模块
├── tools/                      # 工具层，命令行工具
├── BUILD.gn                    # 根构建配置
├── bundle.json                 # 组件配置文件
└── app_file_service.gni        # GN构建变量定义
```

### 1.4.2 接口层详解

接口层是对外暴露编程接口的层次，按照接口类型和调用方进行细分：

**interfaces/kits/** 目录包含对外接口定义，面向应用开发者使用。其中 **js/** 子目录包含 JS N-API 实现，每个功能模块（fileshare、fileuri、backup）都有独立的子目录和构建配置。**ndk/** 子目录包含 C 语言的原生接口，支持Native开发场景。**ani/** 子目录包含 ANI（ArkNative Interface）接口定义。**taihe/** 子目录包含泰河（ArkUI配套语言）绑定接口。**cj/** 子目录包含仓颉语言绑定接口。

**interfaces/innerkits/** 目录包含对内 Native 接口，这些接口仅限 OpenHarmony 系统内部模块调用，不对第三方应用开放。主要包括 fileshare_native、fileuri_native、remote_file_share_native、sandbox_helper_native 四个模块。

**interfaces/inner_api/** 目录包含 InnerAPI 级别的内部接口，当前仅包含 backup_kit_inner 模块，提供备份恢复的内部编程接口。

**interfaces/api/** 目录包含 N-API 的内部实现代码，包括 backup_ext（备份扩展）和 backup_ext_context（备份扩展上下文）的 N-API 实现。

### 1.4.3 框架层详解

框架层实现具体的业务逻辑，承上启下连接接口层和服务层：

**frameworks/js/** 目录包含 JS 框架代码。**backup_ext/** 实现备份扩展 Ability 的 JS 绑定，负责将 C++ 实现的备份扩展能力暴露给 JS 调用者。**backup_ext_context/** 实现备份扩展上下文的 JS 绑定。

**frameworks/native/** 目录包含 Native 框架代码。**backup_ext/** 实现备份扩展 Ability 的 Native 核心逻辑，包括数据打包、解压、扩展生命周期管理等。**backup_ext/ani/** 子目录包含 ANI 字节码生成配置。

### 1.4.4 服务层详解

服务层以系统能力（SA）方式运行，是备份恢复功能的核心：

**services/backup_sa/** 目录包含备份系统能力的完整实现。**src/** 子目录按功能模块组织代码：**module_ipc/** 包含 IPC 通信相关实现，包括服务主循环、扩展连接管理、会话管理等；**module_sched/** 包含调度器实现，负责备份任务的队列管理和执行调度；**module_external/** 包含外部服务适配器，包括 Bundle Manager Service、Storage Manager Service 等；**module_app_gallery/** 包含应用画廊（应用迁移中心）的交互实现；**module_notify/** 包含通知相关实现。**include/** 子目录包含对应的头文件。**IService.idl**、**IServiceReverse.idl**、**IExtension.idl** 等 IDL 文件定义 IPC 接口。

### 1.4.5 工具层详解

工具层提供通用的功能模块，被其他层次复用：

**utils/include/** 目录包含工具模块的头文件。**b_anony/** 提供匿名化处理功能，用于日志脱敏。**b_encryption/** 提供加密和校验和计算功能。**b_error/** 提供错误码和异常处理机制。**b_filesystem/** 提供文件系统操作功能。**b_hiaudit/** 提供审计日志功能。**b_hilog/** 提供日志宏定义。**b_json/** 提供 JSON 实体类和解析功能。**b_jsonutil/** 提供 JSON 工具函数。**b_ohos/** 提供 OHOS 系统参数访问功能。**b_process/** 提供进程管理功能。**b_radar/** 提供性能统计功能。**b_resources/** 提供常量定义。**b_sa/** 提供系统能力工具函数。**b_tarball/** 提供 tar 归档功能。**b_utils/** 提供通用工具函数。

**utils/src/** 目录包含对应的实现代码，组织结构与 include 目录一一对应。

**tools/backup_tool/** 目录包含备份命令行工具的源码，提供独立的备份恢复操作入口。

## 1.5 关键概念

### 1.5.1 URI 与路径

URI（Uniform Resource Identifier）是系统中标识文件资源的标准方式，格式为 `file://<path>`。路径是文件系统中的实际位置，如 `/data/storage/el2/base/files/document.txt`。URI 和路径之间可以相互转换，但 URI 相比路径具有更强的抽象性，可用于跨应用、跨设备的文件引用。

### 1.5.2 权限策略

权限策略（Policy）定义了对资源的访问规则，包括操作类型（读、写、创建、删除、重命名等）、目标 URI、授权模式（临时或持久）等属性。权限策略在授权成功后生成对应的授权令牌，持有令牌的应用可在有效期内访问对应资源。

### 1.5.3 备份扩展

备份扩展（Backup Extension）是应用实现数据备份恢复逻辑的载体。应用通过实现备份扩展接口，在备份时提供待备份数据、在恢复时接收恢复数据。备份扩展以 Extension Ability 方式运行，由备份服务统一调度管理。

### 1.5.4 会话

会话（Session）是备份或恢复操作的上下文环境，包含操作类型、目标应用列表、传输配置等状态信息。一次备份或恢复操作对应一个会话，会话内的数据流在独立的线程中处理，确保操作的原子性和一致性。

### 1.5.5 单会话模型

备份服务采用单会话模型，即同一时刻只处理一个备份或恢复会话。新的会话请求在有进行中的会话时会被拒绝，必须等待当前会话结束。这种设计简化了状态管理，避免了并发冲突，但限制了并行备份多个应用的能力。

## 1.6 相关文档

| 文档 | 说明 |
|------|------|
| [系统架构](01_Architecture.md) | 组件图、数据流、线程模型 |
| [JS N-API 接口](10_NAPI_JS.md) | FileShare/FileURI/Backup JS API |
| [NDK 接口](11_NAPI_NDK.md) | Native 开发接口 |
| [内部 API](20_Inner_API.md) | 模块间接口、依赖方向 |
| [备份服务 SA](02_Service_SA.md) | Service Ability 实现、IPC 模式 |
| [工具库](03_Utils.md) | Utils 模块职责与依赖 |
| [GN 构建配置](04_GN_Build.md) | Targets、产物映射、编译开关 |
| [安全评审](05_Security_Review.md) | 攻击面、风险点、修复建议 |
