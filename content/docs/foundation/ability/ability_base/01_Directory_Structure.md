# 目录结构与模块职责

## 目的

本文档详细说明 ability_base 组件的目录结构、文件组织、模块职责划分，帮助开发者快速定位代码和理解代码组织方式。

---

## 1. 顶层目录结构

### 1.1 完整目录树

```
ability_base/
├── BUILD.gn                      # 主构建配置（474 行，定义 10 个生产库）
├── ability_base.gni              # 构建变量和路径定义
├── bundle.json                   # 组件元数据和依赖声明
├── LICENSE                      # Apache 2.0 许可证
├── OAT.xml                      # OSS Attribution Tool 配置
├── README.md                    # 组件说明（中文）
├── interfaces/                   # 所有源代码（~860K，91 个文件）
│   ├── inner_api/               # 系统内部件间接口（~140K）
│   │   ├── base/                # Base 模块：基础数据类型（29 文件）
│   │   │   ├── include/        # 17 个头文件
│   │   │   └── src/           # 12 个实现文件
│   │   └── log/                # 日志工具模块（2 文件）
│   │       └── include/        # 2 个头文件
│   └── kits/                  # 公共 SDK 接口（~720K）
│       ├── c/                   # C/NDK API（4 文件）
│       │   ├── common/         # 公共 C 头文件（1 文件）
│       │   └── cwant/          # C Want API（3 文件）
│       │       ├── include/    # 2 个头文件
│       │       └── src/        # 2 个实现文件
│       └── native/              # Native C++ APIs（83 文件）
│           ├── want/           # Want 模块（23 文件，最大模块）
│           │   ├── include/    # 12 个头文件
│           │   └── src/        # 11 个实现文件
│           ├── configuration/  # Configuration 模块（5 文件）
│           │   ├── include/    # 3 个头文件
│           │   └── src/        # 2 个实现文件
│           ├── uri/            # URI 模块（2 文件）
│           │   ├── include/    # 1 个头文件
│           │   └── src/        # 1 个实现文件
│           ├── view_data/      # ViewData 模块（7 文件）
│           │   ├── include/    # 4 个头文件
│           │   └── src/        # 3 个实现文件
│           ├── session_info/   # SessionInfo 模块（3 文件）
│           │   ├── include/    # 2 个头文件
│           │   └── src/        # 1 个实现文件
│           └── extractortool/  # ExtractorTool 模块（15 文件）
│               ├── include/    # 8 个头文件
│               └── src/        # 7 个实现文件
└── wiki/                         # Wiki 文档工作区
    ├── README.md                  # 文档说明
    ├── SUMMARY.md                 # 文档导航
    ├── index.md                   # 项目概览
    └── _work/                    # 工作区
        ├── NOTES.md               # 事实记录和发现
        └── PLAN.md                # 任务拆解和进度
```

**证据位置**：
- 目录扫描：`wiki/_work/NOTES.md` - Phase 1 扫描结果
- 文件统计：91 个源文件（.h + .cpp），~860K 总大小

### 1.2 目录职责分类

| 目录 | 类型 | 职责 | 文件数 |
|------|------|------|--------|
| `/interfaces/inner_api/` | 内部 API | 系统内部组件间使用的接口（平台 SDK、SA SDK） | 31 |
| `/interfaces/kits/native/` | 原生 SDK | 公开给应用开发者使用的 C++ 接口（平台 SDK） | 83 |
| `/interfaces/kits/c/` | C NDK | 公开给 NDK 开发者的 C 接口（NDK） | 4 |
| `/BUILD.gn` | 构建配置 | GN 构建系统配置文件 | 1 |
| `/ability_base.gni` | 构建变量 | GN 构建变量和路径定义 | 1 |
| `/bundle.json` | 组件元数据 | OpenHarmony 组件描述和依赖声明 | 1 |

---

## 2. Inner API 目录（内部接口）

### 2.1 base 模块 - 基础数据类型

**目录**：`interfaces/inner_api/base/`

**职责**：提供类型安全的基础数据包装器，支持 WantParams 中的多态存储。

**文件组织**：

**include/**（17 个头文件）
```
interfaces/inner_api/base/include/
├── base_def.h                    # 基础定义和常量
├── base_interfaces.h             # IInterface 层次结构定义
├── base_obj.h                    # Object 基类（RefBase 实现）
├── base_types.h                  # 类型定义和枚举
├── bool_wrapper.h                # Boolean 包装器
├── byte_wrapper.h                # Byte 包装器
├── double_wrapper.h              # Double 包装器
├── float_wrapper.h               # Float 包装器
├── int_wrapper.h                 # Integer 包装器
├── long_wrapper.h                # Long 包装器
├── remote_object_wrapper.h       # IRemoteObject 包装器
├── short_wrapper.h               # Short 包装器
├── string_wrapper.h              # String 包装器
├── user_object_base.h            # 用户对象基接口
├── user_object_wrapper.h         # 用户对象包装器
└── zchar_wrapper.h              # ZChar 包装器
```

**src/**（12 个实现文件）
```
interfaces/inner_api/base/src/
├── base.cpp                      # 基础工具函数
├── base_object.cpp               # Object 基类实现（Query、引用计数）
├── bool_wrapper.cpp              # Boolean 实现
├── byte_wrapper.cpp              # Byte 实现
├── double_wrapper.cpp            # Double 实现
├── float_wrapper.cpp             # Float 实现
├── int_wrapper.cpp               # Integer 实现
├── long_wrapper.cpp              # Long 实现
├── remote_object_wrapper.cpp     # IRemoteObjectWrap 实现
├── short_wrapper.cpp             # Short 实现
├── string_wrapper.cpp            # String 实现
├── user_object_wrapper.cpp      # 用户对象包装器实现
└── zchar_wrapper.cpp            # ZChar 实现
```

**关键类**：
- `Object` - 所有包装器的基类，实现 RefBase 引用计数
- `IInterface` - 所有类型包装器的基接口
- `IBoolean/IInteger/IString/IDouble` 等 - 具体类型接口
- `IArray/IPacMap/IUserObject/IRemoteObjectWrap` - 复杂类型接口

**证据位置**：
- 接口定义：`interfaces/inner_api/base/include/base_interfaces.h`
- 基类实现：`interfaces/inner_api/base/include/base_obj.h`

### 2.2 log 模块 - 日志工具

**目录**：`interfaces/inner_api/log/`

**职责**：提供统一的日志包装器和 Parcel 宏。

**文件组织**：
```
interfaces/inner_api/log/include/
├── ability_base_log_wrapper.h    # 日志包装器（基于 HiLog）
└── parcel_macro_base.h          # Parcel 序列化宏
```

**关键功能**：
- `ABILITYBASE_LOG_*` 宏 - 分模块日志输出
- Parcel 序列化辅助宏

---

## 3. Native Kits 目录（Native C++ SDK）

### 3.1 want 模块 - 组件启动参数（最大模块）

**目录**：`interfaces/kits/native/want/`

**职责**：提供 Want、Operation、WantParams 等核心类，用于 Ability 启动和参数传递。

**文件组织**：

**include/**（12 个头文件）
```
interfaces/kits/native/want/include/
├── array_wrapper.h             # IArray 实现
├── element_name.h              # ElementName 组件标识
├── extra_params.h              # 额外参数
├── match_type.h                # 匹配类型枚举
├── operation.h                 # Operation 操作类
├── operation_builder.h         # Operation 构建器
├── pac_map.h                   # PacMap 实现
├── patterns_matcher.h          # 模式匹配器
├── skills.h                    # Skills 匹配规则
├── want.h                      # Want 主类（1040 行）
├── want_params.h               # WantParams 参数存储
└── want_params_wrapper.h       # WantParams 包装器和验证
```

**src/**（11 个实现文件）
```
interfaces/kits/native/want/src/
├── array_wrapper.cpp           # IArray 实现
├── element_name.cpp           # ElementName 实现
├── extra_params.cpp           # 额外参数实现
├── operation.cpp              # Operation 实现
├── operation_builder.cpp       # Operation 构建器实现
├── pac_map.cpp                # PacMap 实现
├── patterns_matcher.cpp       # 模式匹配器实现
├── skills.cpp                 # Skills 实现
├── want.cpp                   # Want 主实现
├── want_params.cpp            # WantParams 实现（含 IPC 序列化）
└── want_params_wrapper.cpp     # WantParams 包装器实现
```

**关键类**：
- `Want` - 启动消息容器（100+ 方法）
- `Operation` - 目标操作（action、entities、flags、URI）
- `WantParams` - 类型安全参数存储
- `ElementName` - 组件标识（deviceId/bundleName/abilityName/moduleName）
- `Skills` - Intent 匹配规则
- `PatternsMatcher` - 模式匹配器

**证据位置**：
- Want 类：`interfaces/kits/native/want/include/want.h:34`
- Want 源文件：`interfaces/kits/native/want/src/want.cpp`

### 3.2 configuration 模块 - 系统环境配置

**目录**：`interfaces/kits/native/configuration/`

**职责**：提供 Configuration 类，用于查询和管理系统环境配置。

**文件组织**：
```
interfaces/kits/native/configuration/
├── include/
│   ├── configuration.h         # Configuration 主类（240 行）
│   ├── configuration_convertor.h # Configuration 转换器
│   └── global_configuration_key.h  # 全局配置键常量
└── src/
    ├── configuration.cpp        # Configuration 实现
    └── configuration_convertor.cpp  # 转换器实现
```

**关键类**：
- `Configuration` - 系统配置容器（线程安全）
- `GlobalConfigurationKey` - 全局配置键常量

**证据位置**：
- Configuration 类：`interfaces/kits/native/configuration/include/configuration.h:68`

### 3.3 uri 模块 - 统一资源标识符

**目录**：`interfaces/kits/native/uri/`

**职责**：提供 Uri 类，用于 URI 解析和访问。

**文件组织**：
```
interfaces/kits/native/uri/
├── include/
│   └── uri.h                 # Uri 类（206 行）
└── src/
    └── uri.cpp                # Uri 实现
```

**关键类**：
- `Uri` - URI 解析和访问类

**证据位置**：
- Uri 类：`interfaces/kits/native/uri/include/uri.h:24`

### 3.4 view_data 模块 - 自动填充视图数据

**目录**：`interfaces/kits/native/view_data/`

**职责**：提供自动填充相关的视图数据结构。

**文件组织**：
```
interfaces/kits/native/view_data/
├── include/
│   ├── auto_fill_type.h       # 自动填充类型
│   ├── page_node_info.h       # 页面节点信息
│   ├── rect.h                # 矩形几何
│   └── view_data.h           # ViewData 主类
└── src/
    ├── page_node_info.cpp     # 页面节点实现
    ├── rect.cpp               # 矩形实现
    └── view_data.cpp          # ViewData 实现
```

**关键类**：
- `ViewData` - 自动填充视图数据
- `PageNodeInfo` - 页面节点信息
- `Rect` - 矩形几何类

### 3.5 session_info 模块 - UI 会话信息

**目录**：`interfaces/kits/native/session_info/`

**职责**：提供 SessionInfo 类，用于 UI Ability 的会话管理。

**文件组织**：
```
interfaces/kits/native/session_info/
├── include/
│   ├── session_info.h         # SessionInfo 主类
│   └── session_info_constants.h  # 会话常量
└── src/
    └── session_info.cpp        # SessionInfo 实现
```

**关键类**：
- `SessionInfo` - UI 会话元数据（含 IRemoteObject tokens）

**证据位置**：
- SessionInfo 类：`interfaces/kits/native/session_info/include/session_info.h`

### 3.6 extractortool 模块 - ZIP 资源提取工具

**目录**：`interfaces/kits/native/extractortool/`

**职责**：提供 HAP 包和 ZIP 文件的解压提取功能。

**文件组织**：
```
interfaces/kits/native/extractortool/
├── include/
│   ├── constants.h            # 常量定义
│   ├── extract_resource_manager.h  # 资源管理器
│   ├── extractor.h             # Extractor 类
│   ├── file_mapper.h           # 文件映射
│   ├── file_path_utils.h       # 文件路径工具
│   ├── zip_file.h             # ZipFile 类
│   ├── zip_file_reader.h      # ZIP 文件读取器
│   └── zip_file_reader_io.h   # ZIP IO 适配
└── src/
    ├── extractor.cpp            # Extractor 实现
    ├── file_mapper.cpp         # 文件映射实现
    ├── file_path_utils.cpp     # 文件路径工具实现
    ├── zip_file.cpp            # ZipFile 实现
    ├── zip_file_reader.cpp     # ZIP 读取器实现
    └── zip_file_reader_io.cpp # ZIP IO 实现
```

**关键类**：
- `Extractor` - HAP/ZIP 提取器
- `ZipFile` - ZIP 文件解析
- `FileMapper` - 文件映射（支持 SAFE_ABC 模式）
- `ExtractorUtil` - 单例工具类

**证据位置**：
- Extractor 类：`interfaces/kits/native/extractortool/include/extractor.h`

---

## 4. C Kits 目录（NDK）

### 4.1 cwant 模块 - Want 的 C API

**目录**：`interfaces/kits/c/cwant/`

**职责**：提供 Want 的 C 语言接口，供 NDK 开发者使用。

**文件组织**：
```
interfaces/kits/c/
├── common/
│   └── ability_base_common.h  # 公共 C 定义和错误码
└── cwant/
    ├── include/
    │   ├── want.h             # Want C API（273 行）
    │   └── want_manager.h    # WantManager C API
    └── src/
        ├── want.cpp            # Want C API 实现
        └── want_manager.cpp   # WantManager C API 实现
```

**关键函数**（`OH_AbilityBase_*` 前缀）：
- `OH_AbilityBase_CreateWant()` - 创建 Want
- `OH_AbilityBase_DestroyWant()` - 销毁 Want
- `OH_AbilityBase_SetWantElement()` - 设置 ElementName
- `OH_AbilityBase_SetWantCharParam()` - 设置字符串参数
- `OH_AbilityBase_SetWantInt32Param()` - 设置整型参数
- `OH_AbilityBase_SetWantBoolParam()` - 设置布尔参数
- `OH_AbilityBase_SetWantDoubleParam()` - 设置双精度参数
- `OH_AbilityBase_AddWantFd()` - 添加文件描述符

**证据位置**：
- C API 定义：`interfaces/kits/c/cwant/include/want.h`
- C API 实现：`interfaces/kits/c/cwant/src/want.cpp`

---

## 5. 构建配置文件

### 5.1 BUILD.gn - 主构建配置

**位置**：`BUILD.gn`（474 行）

**职责**：定义所有 GN targets（10 个生产库 + 测试目标）

**主要 Targets**：
1. `base` - 基础类型库（libbase.so）
2. `configuration` - 配置库（libconfiguration.so）
3. `zuri` - URI 库（libzuri.so）
4. `want` - Want 库（libwant.so）
5. `view_data` - 视图数据库（libview_data.so）
6. `session_info` - 会话信息库（libsession_info.so）
7. `string_utils` - 字符串工具库（libstring_utils.so）
8. `extractortool` - 提取工具库（libextractortool.so）
9. `extractresourcemanager` - 资源管理库（libextractresourcemanager.so）
10. `ability_base_want` - C NDK 库（libability_base_want.so）

**证据位置**：
- BUILD.gn 完整内容：`BUILD.gn:14-473`

### 5.2 ability_base.gni - 构建变量

**位置**：`ability_base.gni`（22 行）

**职责**：定义路径变量供 BUILD.gn 使用

**关键变量**：
```
ability_base_path = "//foundation/ability/ability_base"
ability_base_innerapi_path = "${ability_base_path}/interfaces/inner_api"
ability_base_kits_native_path = "${ability_base_path}/interfaces/kits/native"
ability_base_ndk_path = "${ability_base_path}/interfaces/kits/c"
```

**证据位置**：
- 变量定义：`ability_base.gni:14-17`

### 5.3 bundle.json - 组件元数据

**位置**：`bundle.json`（164 行）

**职责**：声明组件信息、依赖、导出接口

**关键内容**：
- 组件名：`ability_base`
- 子系统：`ability`
- 系统能力：`SystemCapability.Ability.AbilityBase`
- 版本：`3.1`
- 许可证：`Apache License 2.0`
- 依赖组件：13 个（见 index.md）
- 导出接口：10 个库（inner_kits）

**证据位置**：
- 组件元数据：`bundle.json:1-163`

---

## 6. 文件统计与模块大小

### 6.1 按模块统计

| 模块 | 头文件数 | 源文件数 | 总文件数 | 估计大小 | 职责 |
|------|----------|----------|----------|---------|------|
| **Base** | 17 | 12 | 29 | ~100K | 基础数据类型 |
| **Want** | 12 | 11 | 23 | ~280K | 组件启动参数 |
| **ExtractorTool** | 8 | 7 | 15 | ~140K | ZIP 提取 |
| **ViewData** | 4 | 3 | 7 | ~50K | 自动填充数据 |
| **Configuration** | 3 | 2 | 5 | ~60K | 系统配置 |
| **SessionInfo** | 2 | 1 | 3 | ~30K | 会话信息 |
| **URI** | 1 | 1 | 2 | ~40K | URI 处理 |
| **C Want** | 2 | 2 | 4 | ~40K | C NDK API |
| **Log** | 2 | 0 | 2 | ~8K | 日志工具 |
| **Common C** | 1 | 0 | 1 | ~2K | 公共 C 头 |
| **总计** | **52** | **39** | **91** | **~860K** | |

### 6.2 按目录类型统计

| 目录类型 | 文件数 | 大小 |
|----------|--------|------|
| `interfaces/inner_api/` | 31 | ~140K |
| `interfaces/kits/native/` | 83 | ~720K |
| `interfaces/kits/c/` | 4 | ~40K |
| 根目录配置文件 | 4 | ~40K |
| **总计** | **122** | **~940K** |

---

## 7. 模块依赖关系

### 7.1 内部依赖图

```
build target 依赖关系：

base_innerkits_target (group)
├── base (独立）
│
├── zuri (独立）
│
├── configuration (独立）
│
├── string_utils (独立）
│
├── extractresourcemanager (独立）
│
├── view_data (独立）
│
├── extractortool
│   └── deps: string_utils
│
├── want
│   └── deps: base, zuri
│
├── session_info
│   └── deps: want
│
└── ability_base_want (C NDK)
    └── deps: base, want
```

**证据位置**：
- 依赖关系：`BUILD.gn` - 各 target 的 `deps` 字段

### 7.2 外部依赖总结

| 内部模块 | 外部依赖组件 | 依赖内容 |
|----------|--------------|----------|
| base | ipc_core, hilog, c_utils | IPC、日志、工具 |
| configuration | hilog, c_utils, json, resource_management | 日志、工具、JSON、资源管理 |
| zuri | hilog, c_utils | 日志、工具 |
| want | ipc_core, ipc_single, json, jsoncpp, hilog, c_utils, base, zuri | IPC、JSON、基础类型 |
| view_data | hilog, json | 日志、JSON |
| session_info | ipc_core, hilog, c_utils, window_manager, ability_runtime, bundle_framework, want | IPC、日志、窗口、运行时 |
| string_utils | 无 | 仅内部工具 |
| extractortool | hilog, hitrace, json, zlib, c_utils, string_utils | 日志、追踪、JSON、ZIP |
| extractresourcemanager | resource_management | 资源管理 |
| ability_base_want | ipc_single, hilog, c_utils, base, want | IPC、日志、基础类型、Want |

---

## 8. 关键路径映射

### 8.1 API 层级路径

```
应用开发者
    ↓ Native C++ API
interfaces/kits/native/[module]/include/*.h
    ↓ 内部实现
interfaces/kits/native/[module]/src/*.cpp
    ↓ 调用内部类型
interfaces/inner_api/base/include/*.h
    ↓ IPC 传输（如果跨进程）
Marshalling/Unmarshalling (Parcelable)
```

### 8.2 NDK 路径

```
NDK 开发者
    ↓ C API
interfaces/kits/c/cwant/include/want.h (OH_AbilityBase_* 函数)
    ↓ C 实现
interfaces/kits/c/cwant/src/want.cpp
    ↓ 调用 Native C++
interfaces/kits/native/want/include/want.h (Want 类)
```

**证据位置**：
- C API 实现：`interfaces/kits/c/cwant/src/want.cpp` - 调用 `AAFwk::Want`

---

## 9. 文件命名规范

### 9.1 头文件命名

| 模式 | 示例 | 说明 |
|------|------|------|
| `[module].h` | `want.h`, `configuration.h` | 主类定义 |
| `[module]_wrapper.h` | `bool_wrapper.h`, `remote_object_wrapper.h` | 包装器类 |
| `[module]_type.h` | `match_type.h`, `auto_fill_type.h` | 类型定义 |
| `[module]_builder.h` | `operation_builder.h` | 构建器类 |
| `global_[module]_key.h` | `global_configuration_key.h` | 全局常量 |
| `[module]_constants.h` | `session_info_constants.h` | 常量定义 |

### 9.2 源文件命名

源文件与头文件一一对应，使用 `.cpp` 扩展名。

---

## 10. 排除的测试目录

根据文档约束，以下目录**不包含在本文档中**：
- `/test/` - 所有测试代码
  - `/test/unittest/` - 单元测试
  - `/test/fuzztest/` - 模糊测试

**原因**：测试代码不作为业务能力的证据。

---

## 相关跳转

- 🏠 **项目概览**：[index.md](index.md)
- 🏗️ **架构设计**：[02_Architecture.md](02_Architecture.md)
- 🔌 **Native API**：[03_Native_CPP_API.md](03_Native_CPP_API.md)
- 🔧 **C NDK API**：[04_C_NDK_API.md](04_C_NDK_API.md)
- ⚙️ **GN 构建**：[05_GN_Build.md](05_GN_Build.md)

---

**返回导航**：[SUMMARY.md](SUMMARY.md)
