# 项目概览

## 目的

本文档提供 ability_base 组件的整体概览，包括项目定位、核心能力、运行环境、关键概念和依赖关系，帮助读者快速理解该组件在 OpenHarmony 系统中的位置和作用。

**适用范围**：
- OpenHarmony 标准系统（standard system type）
- API 版本：15+
- 组件版本：3.1

---

## 1. 项目定位

### 1.1 组件定义

**ability_base** 是 OpenHarmony Ability 子系统的**基础定义部件**，提供组件启动参数（Want）、系统环境参数（Configuration）、URI 参数（Uniform Resource Identifier）的定义，用于启动应用、获取环境参数等功能。

**证据位置**：
- 说明：`README.md:3-5`
- 组件名：`bundle.json:13` - `"name": "ability_base"`

### 1.2 在子系统中的位置

ability_base 是 Ability 子系统的最底层基础库，为上层组件提供数据结构和接口：

```
Ability 子系统层次结构：
┌───────────────────────────────────────┐
│      ability_runtime (运行时)         │  ← Ability 启动、生命周期管理
│         ├─ N-API 绑定              │  ← JS API 层
│         └─ AbilityManager           │  ← Ability 管理服务
├───────────────────────────────────────┤
│      ability_base (基础层) [本文档] │  ← 数据结构、基础接口
│         ├─ Want                     │
│         ├─ Configuration            │
│         ├─ URI                      │
│         └─ Base 类型                │
├───────────────────────────────────────┤
│      dmsfwk (分布式管理)            │  ← 分布式能力调度
├───────────────────────────────────────┤
│      form_fwk (卡片框架)            │  ← 卡片/Widget 管理
└───────────────────────────────────────┘
```

**证据位置**：
- 相关仓库：`README.md:34-44` - 列出 ability_runtime、dmsfwk、form_fwk

### 1.3 项目边界

**ability_base 负责**：
- ✅ 数据结构定义（Want、Configuration、URI、Base 类型）
- ✅ 基础接口实现（类型包装器、参数存储）
- ✅ IPC 序列化支持（Parcelable 接口）
- ✅ 跨设备通信支持（URI、Want）

**ability_base 不负责**：
- ❌ Ability 生命周期管理（由 ability_runtime 负责）
- ❌ 权限验证实际执行（在 ability_runtime 或系统服务层）
- ❌ N-API JavaScript 绑定（此仓库无 N-API 代码）
- ❌ Ability 启动调度（由 AbilityManager 负责）
- ❌ 系统服务注册（由 system_ability 负责）

**证据位置**：
- README.md:22 - 提到 `frameworks/js/napi` 但目录不存在
- 构建输出：`BUILD.gn` - 仅生成 .so 共享库，无服务可执行文件

---

## 2. 核心能力

### 2.1 Want 模块 - 组件启动参数

**职责**：组件启动参数模块，开发者可以使用 Want 携带自定义参数，显示/隐式启动应用，同时支持 Pending 机制，可本地及跨设备延迟启动目标组件。

**证据位置**：
- 职责说明：`README.md:10-11`
- 类定义：`interfaces/kits/native/want/include/want.h:34` - `class Want final`

**核心能力**：
- 📋 **显式启动**：通过 bundleName + abilityName 直接指定目标
- 📋 **隐式启动**：通过 action + URI + type 匹配目标
- 📋 **Pending 启动**：延迟启动能力，支持跨设备
- 📋 **参数传递**：类型安全的键值存储（Bool/Int/Long/String/Array 等）
- 📋 **权限传递**：URI 读写权限、ability 前转结果等

**关键类**：
- `Want` - 启动消息容器
- `Operation` - 目标操作（action、entities、flags、URI）
- `WantParams` - 类型安全参数存储
- `ElementName` - 组件标识（deviceId/bundleName/abilityName/moduleName）
- `Skills` - 匹配规则（action/uri/type）
- `PatternsMatcher` - 模式匹配器

### 2.2 Configuration 模块 - 系统环境参数

**职责**：系统环境参数模块，支持开发者查询当前环境配置信息，感知系统环境变化。

**证据位置**：
- 职责说明：`README.md:12-13`
- 类定义：`interfaces/kits/native/configuration/include/configuration.h:68` - `class Configuration final`

**核心能力**：
- ⚙️ **配置查询**：按 displayId + key 获取配置值
- ⚙️ **配置更新**：支持修改并合并配置
- ⚙️ **差异比较**：比较两个 Configuration 对象的差异键
- ⚙️ **线程安全**：使用 recursive_mutex 保护配置数据
- ⚙️ **多屏支持**：支持多 displayId 的配置管理

**关键类**：
- `Configuration` - 系统配置容器
- `GlobalConfigurationKey` - 全局配置键常量

### 2.3 URI 模块 - 统一资源标识符

**职责**：URI 参数定义模块，提供本地及跨设备资源访问能力，开发者可以使用 URI 访问文件等资源。

**证据位置**：
- 职责说明：`README.md:13-14`
- 类定义：`interfaces/kits/native/uri/include/uri.h:24` - `class Uri`

**核心能力**：
- 🌐 **URI 解析**：解析 URI 的 scheme/authority/path/query/fragment
- 🌐 **URI 构建**：构建符合规范的 URI 字符串
- 🌐 **路径访问**：获取 path segments
- 🌐 **跨设备支持**：支持 deviceId 前缀的跨设备 URI

**关键类**：
- `Uri` - URI 解析和访问类

### 2.4 Base 模块 - 基础数据类型

**职责**：基础数据类型模块，提供 Boolean、Integer、String 等支持 Want 携带的基础数据类型定义，方便开发者启动过程中传递自定义参数。

**证据位置**：
- 职责说明：`README.md:14`
- 接口定义：`interfaces/inner_api/base/include/base_interfaces.h`

**核心能力**：
- 🔢 **类型包装**：将基本类型包装为 IInterface 接口
- 🔢 **多态存储**：在 WantParams 中统一存储不同类型
- 🔢 **IPC 支持**：支持序列化和跨进程传输
- 🔢 **引用计数**：基于 RefBase 的生命周期管理

**关键类**：
- `IInterface` - 所有类型包装器的基接口
- `IBoolean/IInteger/ILong/IString/IDouble` - 数值类型包装器
- `IArray` - 数组包装器
- `IPacMap` - Map 包装器
- `IRemoteObjectWrap` - 远程对象包装器

---

## 3. 运行环境

### 3.1 适配系统类型

**适用系统**：OpenHarmony 标准系统（standard）

**证据位置**：
- `bundle.json:19` - `"adapted_system_type": ["standard"]`

### 3.2 系统能力

**系统能力标签**：`SystemCapability.Ability.AbilityBase`

**证据位置**：
- `bundle.json:15-17` - `"syscap": ["SystemCapability.Ability.AbilityBase"]`

### 3.3 API 版本要求

**最低 API 版本**：15

**证据位置**：
- NDK 接口：`interfaces/kits/c/cwant/include/want.h:23` - `@since 15`
- 原生接口：多个头文件包含 `@since 15` 注释

### 3.4 编译与运行要求

**编译环境**：
- **编译器**：支持 C++14 及以上
- **构建系统**：GN (Generate Ninja)
- **目标架构**：支持 arm/arm64/x86_64

**运行时要求**：
- **系统库依赖**：IPC、HiLog、c_utils、zlib、json
- **系统服务依赖**：resource_management、window_manager（部分模块）
- **权限要求**：无特殊权限要求（纯库）

---

## 4. 关键概念

### 4.1 Want (意图）

**定义**：Want 是 Ability 之间通信的消息容器，类似于 Android 的 Intent，携带启动目标、操作、参数等信息。

**组成结构**：
```
Want
├── Operation (操作目标)
│   ├── action (操作类型，如 "ohos.action.home")
│   ├── entities (实体列表，如 "entity.video")
│   ├── flags (标志位，如 FLAG_ABILITY_FORWARD_RESULT)
│   ├── deviceId (设备 ID)
│   ├── bundleName (应用包名)
│   ├── abilityName (Ability 名称)
│   ├── moduleName (模块名称)
│   └── uri (URI 目标)
└── WantParams (参数)
    ├── key-value pairs (键值对，类型安全)
    └── IRemoteObject (远程对象)
```

**证据位置**：
- Want 类定义：`interfaces/kits/native/want/include/want.h:34`
- WantParams 定义：`interfaces/kits/native/want/include/want_params.h`

### 4.2 Operation (操作)

**定义**：Operation 定义了 Want 的目标操作，包括 action、entities、flags 和 URI。

**核心字段**：
- `action`：操作类型（如 `"ohos.action.home"`）
- `entities`：实体类别（如 `"entity.video"`）
- `flags`：标志位（如 `FLAG_ABILITY_FORWARD_RESULT`）
- `deviceId`：目标设备 ID（用于跨设备）
- `bundleName` + `abilityName`：显式指定目标组件
- `moduleName`：HAP 模块名

**证据位置**：
- Operation 定义：`interfaces/kits/native/want/include/operation.h`

### 4.3 WantParams (参数)

**定义**：WantParams 是类型安全的键值存储，支持多种数据类型和嵌套结构。

**支持类型**：
- 基本类型：Boolean, Byte, Char, Short, Integer, Long, Float, Double, String
- 复杂类型：Array, WantParams (嵌套), IRemoteObject
- 文件描述符：int32_t fd

**证据位置**：
- 类型枚举：`interfaces/kits/native/want/src/want_params.cpp:204-233`

### 4.4 Configuration (配置)

**定义**：Configuration 表示系统环境配置，如语言、主题、方向等。

**键格式**：`"displayId#key"`
- `displayId`：显示器 ID（0 表示默认）
- `key`：配置键（如 `"ohos.system.language"`）

**示例**：
- `"0#ohos.system.language"` → `"zh-CN"`
- `"1#ohos.application.direction"` → `"vertical"`

**证据位置**：
- 键格式：`interfaces/kits/native/configuration/include/configuration.h:215` - `MakeTheKey()` 方法

### 4.5 Parcelable (可序列化)

**定义**：Parcelable 是 OpenHarmony IPC 序列化接口，所有需要跨进程传输的对象必须实现此接口。

**核心方法**：
- `Marshalling(Parcel &parcel)` - 序列化写入 Parcel
- `Unmarshalling(Parcel &parcel)` - 反序列化从 Parcel 读取
- `ReadFromParcel(Parcel &parcel)` - 从 Parcel 读取

**证据位置**：
- Want 实现：`interfaces/kits/native/want/include/want.h:34` - `class Want final : public Parcelable`
- Configuration 实现：`interfaces/kits/native/configuration/include/configuration.h:68` - `class Configuration final: public Parcelable`

### 4.6 IRemoteObject (远程对象)

**定义**：IRemoteObject 是 OpenHarmony IPC 的远程对象接口，用于跨进程传递对象引用。

**在 ability_base 中的使用**：
- WantParams 支持存储 IRemoteObject 类型的参数
- SessionInfo 包含多个 IRemoteObject token（sessionToken, callerToken 等）
- RemoteObjectWrap 类封装 IRemoteObject 用于类型安全存储

**证据位置**：
- WantParams 支持：`interfaces/kits/native/want/include/want.h:443` - `SetParam(key, remoteObject)`
- SessionInfo token：`interfaces/kits/native/session_info/include/session_info.h:45-48`
- RemoteObjectWrap：`interfaces/inner_api/base/include/remote_object_wrapper.h`

---

## 5. 依赖关系

### 5.1 外部组件依赖

ability_base 依赖以下 OpenHarmony 系统组件：

**证据位置**：`bundle.json:24-37`

| 依赖组件 | 用途 | 依赖模块 |
|----------|------|----------|
| **ability_runtime** | Ability 运行时类型 | session_info |
| **bundle_framework** | Bundle 框架基础 | session_info |
| **c_utils** | 通用工具库 | 所有模块 |
| **hilog** | 日志系统 | 所有模块 |
| **hitrace** | 性能追踪 | extractortool |
| **ipc** (ipc_core, ipc_single) | IPC 通信 | base, want, session_info, ability_base_want |
| **resource_management** | 资源管理 | configuration, extractresourcemanager |
| **json** (nlohmann_json_static) | JSON 解析 | want, configuration, view_data, extractortool |
| **jsoncpp** | JSON 解析（旧） | want |
| **zlib** (libz, shared_libz) | ZIP 压缩 | extractortool |
| **window_manager** | 窗口管理 | session_info |

### 5.2 内部模块依赖关系

```
ability_base 内部依赖图：

base (基础类型)
  ↑
  │
  ├─→ want (依赖 base + zuri)
  │     ↑
  │     │
  │     └─→ session_info (依赖 want)
  │
zuri (URI 处理)

configuration (独立，无内部依赖）
string_utils (独立）
  ↑
  │
  └─→ extractortool (依赖 string_utils)

extractresourcemanager (独立）
view_data (独立）
```

**证据位置**：`BUILD.gn` - 各 target 的 `deps` 字段

### 5.3 被依赖关系

哪些组件依赖 ability_base：

**内部子系统**：
- ✅ **ability_runtime** - 使用 Want、Configuration、SessionInfo 管理 Ability 生命周期
- ✅ **dmsfwk** - 使用 Want 进行分布式调度
- ✅ **form_fwk** - 使用 Want 启动卡片 Ability

**第三方应用**：
- ✅ 所有需要启动 Ability 的应用通过 Want 与系统交互
- ✅ 需要获取系统配置的应用通过 Configuration 查询

**证据位置**：
- bundle.json `innerapi_tags` - 标记为 platformsdk/sasdk，表示被 SDK 导出

---

## 6. 许可与版权

**许可证**：Apache License 2.0

**证据位置**：
- 许可证文件：`LICENSE`
- bundle.json 声明：`bundle.json:5` - `"license": "Apache License 2.0"`

**开源义务**：
- ✅ 保留版权声明
- ✅ 保留许可证副本
- ✅ 修改文件需注明变更
- ✅ 贡献者需贡献者协议

---

## 7. 相关跳转

### 进一步阅读

- 📁 **目录结构**：[01_Directory_Structure.md](01_Directory_Structure.md)
- 🏗️ **架构设计**：[02_Architecture.md](02_Architecture.md)
- 🔌 **Native API**：[03_Native_CPP_API.md](03_Native_CPP_API.md)
- 🔧 **C NDK API**：[04_C_NDK_API.md](04_C_NDK_API.md)
- ⚙️ **GN 构建**：[05_GN_Build.md](05_GN_Build.md)
- 🔒 **安全评审**：[06_Security_Review.md](06_Security_Review.md)

### 外部资源

- **仓库主页**：https://gitee.com/openharmony/ability_ability_base
- **开发指南**：https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/application-models/Readme-CN.md
- **相关仓库**：
  - ability_runtime: https://gitee.com/openharmony/ability_ability_runtime
  - dmsfwk: https://gitee.com/openharmony/ability_dmsfwk
  - form_fwk: https://gitee.com/openharmony/ability_form_fwk

---

**返回导航**：[SUMMARY.md](SUMMARY.md)
