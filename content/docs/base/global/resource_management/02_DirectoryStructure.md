# 目录结构与模块职责

## 目的

本文档详细说明 OpenHarmony 资源管理组件的代码组织方式、目录结构和各模块职责。

## 适用范围

本文档覆盖所有源代码目录，**不包含测试目录**（test/, tests/, unittest/, fuzztest/ 等）。

## 关键结论

| 分类 | 目录 | 职责 | 关键文件 |
|------|------|------|----------|
| 核心框架层 | `frameworks/resmgr/` | 资源解析、加载、管理核心实现 | hap_manager.cpp, resource_manager_impl.cpp |
| 接口层 - Inner API | `interfaces/inner_api/` | C++ 内部子系统接口 | resource_manager.h |
| 接口层 - Native API | `interfaces/native/` | C 语言原生接口 | raw_file.h |
| 接口层 - JS | `interfaces/js/` | JavaScript N-API 接口 | resource_manager_napi.cpp |
| 接口层 - ETS | `interfaces/ets/` | ArkTS ANI 接口 | resmgr_ani.cpp |
| 接口层 - CJ | `interfaces/cj/` | Cangjie FFI 接口 | resource_manager_ffi.cpp |
| 诊断层 | `dfx/` | 系统事件适配器 | hisysevent_adapter.cpp |

## 完整目录结构

```
/base/global/resource_management/
├── bundle.json                    # 组件构建配置
├── hisysevent.yaml               # 系统事件配置
├── resmgr.gni                    # GN 构建配置
├── README.md / README_zh.md      # 项目文档
├── LICENSE                       # 许可证
├── OAT.xml                       # OAT 合规配置
│
├── dfx/                          # 【诊断与监控】
│   └── hisysevent_adapter/       # HiSysEvent 事件适配器
│       ├── hisysevent_adapter.h
│       └── hisysevent_adapter.cpp
│
├── frameworks/                   # 【核心框架层】
│   └── resmgr/                   # 资源管理核心实现
│       ├── include/              # 头文件
│       │   ├── utils/            # 工具类头文件
│       │   │   ├── common.h      # 通用定义
│       │   │   ├── date_utils.h  # 日期工具
│       │   │   ├── errors.h      # 错误码定义
│       │   │   ├── psue_manager.h    # 原始字符串更新管理
│       │   │   ├── string_utils.h     # 字符串工具
│       │   │   └── utils.h       # 工具函数
│       │   ├── hap_manager.h           # HAP 管理器
│       │   ├── hap_parser.h            # HAP 解析器基类
│       │   ├── hap_parser_v1.h        # HAP V1 解析器
│       │   ├── hap_parser_v2.h        # HAP V2 解析器
│       │   ├── hap_resource.h          # HAP 资源基类
│       │   ├── hap_resource_manager.h  # HAP 资源管理器
│       │   ├── hap_resource_v1.h        # HAP V1 资源
│       │   ├── hap_resource_v2.h        # HAP V2 资源
│       │   ├── hilog_wrapper.h         # 日志包装器
│       │   ├── locale_matcher.h        # 区域设置匹配器
│       │   ├── mmap_file.h             # 内存映射文件
│       │   ├── res_config_impl.h       # 资源配置实现
│       │   ├── res_desc.h              # 资源描述符
│       │   ├── res_locale.h            # 资源区域设置
│       │   ├── resource_manager_ext_mgr.h   # 资源管理器扩展管理器
│       │   ├── resource_manager_impl.h     # 资源管理器实现
│       │   ├── system_resource_manager.h    # 系统资源管理器
│       │   ├── theme_pack_config.h          # 主题包配置
│       │   ├── theme_pack_manager.h         # 主题包管理器
│       │   └── theme_pack_resource.h        # 主题包资源
│       │
│       └── src/                  # 实现代码
│           ├── utils/            # 工具类实现
│           │   ├── locale_data.cpp           # 区域设置数据
│           │   ├── psue_manager.cpp         # 原始字符串更新管理
│           │   ├── string_utils.cpp         # 字符串工具
│           │   └── utils.cpp               # 工具函数
│           ├── hap_manager.cpp                    # HAP 管理器实现
│           ├── hap_parser.cpp                     # HAP 解析器基类
│           ├── hap_parser_v1.cpp                 # HAP V1 解析器实现
│           ├── hap_parser_v2.cpp                 # HAP V2 解析器实现
│           ├── hap_resource.cpp                   # HAP 资源基类实现
│           ├── hap_resource_manager.cpp           # HAP 资源管理器实现
│           ├── hap_resource_v1.cpp               # HAP V1 资源实现
│           ├── hap_resource_v2.cpp               # HAP V2 资源实现
│           ├── likely_subtags_key_data.cpp       # 语言匹配数据 (key)
│           ├── likely_subtags_value_data.cpp     # 语言匹配数据 (value)
│           ├── locale_matcher.cpp                # 区域设置匹配器实现
│           ├── mmap_file.cpp                     # 内存映射文件实现
│           ├── native_resource_manager.cpp       # Native 资源管理器
│           ├── raw_file_manager.cpp              # 原始文件管理器
│           ├── res_config_impl.cpp               # 资源配置实现
│           ├── res_desc.cpp                      # 资源描述符实现
│           ├── res_locale.cpp                    # 资源区域设置实现
│           ├── resmgr.js                         # JS 资源管理器 (用于 ABC 编译)
│           ├── resource_manager.cpp               # 资源管理器基类
│           ├── resource_manager_ext_mgr.cpp       # 资源管理器扩展管理器
│           ├── resource_manager_impl.cpp           # 资源管理器实现 (核心)
│           ├── system_resource_manager.cpp        # 系统资源管理器实现
│           ├── theme_pack_config.cpp              # 主题包配置实现
│           ├── theme_pack_manager.cpp             # 主题包管理器实现
│           └── theme_pack_resource.cpp            # 主题包资源实现
│
├── interfaces/                   # 【接口层】
│   ├── inner_api/                # 内部子系统 API
│   │   └── include/
│   │       ├── res_common.h           # 通用定义
│   │       ├── res_config.h            # 资源配置接口
│   │       ├── resource_manager.h      # 资源管理器接口
│   │       └── rstate.h                # 资源状态定义
│   │
│   ├── native/                   # Native C API
│   │   └── resource/
│   │       └── include/
│   │           ├── ohresmgr.h                 # Native 资源管理器接口
│   │           ├── raw_dir.h                  # 原始目录接口
│   │           ├── raw_file.h                 # 原始文件接口
│   │           ├── raw_file_manager.h         # 原始文件管理器接口
│   │           └── resmgr_common.h            # 通用定义
│   │
│   ├── js/                       # JavaScript API
│   │   ├── innerkits/            # JS 内部实现
│   │   │   └── core/
│   │   │       ├── include/
│   │   │       │   ├── resource_manager_addon.h                # JS 对象包装类
│   │   │       │   ├── resource_manager_data_context.h          # 数据上下文
│   │   │       │   ├── resource_manager_napi_async_impl.h       # 异步 NAPI 实现
│   │   │       │   ├── resource_manager_napi_base.h            # NAPI 基类
│   │   │       │   ├── resource_manager_napi_context.h         # NAPI 上下文
│   │   │       │   ├── resource_manager_napi_sync_impl.h       # 同步 NAPI 实现
│   │   │       │   ├── resource_manager_napi_utils.h           # NAPI 工具函数
│   │   │       │   └── resource_table_loader.h                 # 资源表加载器
│   │   │       └── src/
│   │   │           ├── resource_manager_addon.cpp              # JS 对象包装实现
│   │   │           ├── resource_manager_napi_async_impl.cpp     # 异步方法实现
│   │   │           ├── resource_manager_napi_context.cpp        # NAPI 上下文实现
│   │   │           ├── resource_manager_napi_sync_impl.cpp       # 同步方法实现
│   │   │           ├── resource_manager_napi_utils.cpp           # NAPI 工具函数实现
│   │   │           └── resource_table_loader.cpp                # 资源表加载器实现
│   │   │
│   │   └── kits/                 # JS 对外接口
│   │       └── src/
│   │           ├── resource_manager_napi.cpp          # NAPI 模块入口
│   │           └── sendable_resource_manager_napi.cpp  # Sendable JS API
│   │
│   ├── ets/                      # ArkTS/ETS API
│   │   └── ani/
│   │       └── resourceManager/
│   │           ├── ets/          # ETS 类型定义
│   │           │   ├── @ohos.resourceManager.ets   # 主模块
│   │           │   └── global/
│   │           │       ├── rawFileDescriptor.ets     # 原始文件描述符
│   │           │       ├── rawFileDescriptorInner.ets # 内部描述符
│   │           │       ├── resource.ets              # 资源
│   │           │       └── resourceInner.ets         # 内部资源
│   │           ├── include/      # ANI 桥接头文件
│   │           │   ├── ani_signature.h   # ANI 签名
│   │           │   ├── ani_utils.h       # ANI 工具
│   │           │   ├── resmgr_ani.h      # 资源管理器 ANI
│   │           │   └── resourceManager.h # 资源管理器 ANI
│   │           └── src/          # ANI 桥接实现
│   │               ├── ani_utils.cpp       # ANI 工具实现
│   │               ├── resmgr_ani.cpp      # 资源管理器 ANI 实现
│   │               └── resourceManager.cpp # 资源管理器 ANI 实现
│   │
│   └── cj/                       # Cangjie API
│       └── src/
│           ├── resource_manager_ffi.cpp       # FFI 接口
│           ├── resource_manager_ffi.h
│           ├── resource_manager_impl.cpp       # FFI 实现
│           ├── resource_manager_impl.h
│           ├── resource_manager_interface.h    # FFI 接口定义
│           ├── resource_manager_log.h         # 日志
│           ├── resource_manager_mock.cpp       # Mock 实现 (SDK)
│           ├── utils.cpp
│           └── utils.h
│
└── wiki/                         # 【文档目录】
    ├── _work/                    # 工作区
    ├── README.md
    ├── SUMMARY.md
    └── ...
```

## 模块职责详解

### 1. 核心框架层 (frameworks/resmgr/)

#### 1.1 HAP 管理模块

**职责**: HAP 包解析、加载和管理

| 模块 | 文件 | 职责 |
|------|------|------|
| HapManager | hap_manager.cpp | 管理 HAP 资源包的生命周期 |
| HapParser | hap_parser.cpp (基类) | 解析 HAP 包格式 |
| HapParserV1 | hap_parser_v1.cpp | 解析 HAP V1 格式 |
| HapParserV2 | hap_parser_v2.cpp | 解析 HAP V2 格式 |
| HapResourceManager | hap_resource_manager.cpp | 管理 HAP 资源 |

**关键类**: `HapManager`, `HapParser`, `HapResource`, `HapResourceManager`

**证据**: `frameworks/resmgr/include/hap_manager.h`, `frameworks/resmgr/include/hap_parser.h`

#### 1.2 资源管理器模块

**职责**: 资源查询、匹配和管理

| 模块 | 文件 | 职责 |
|------|------|------|
| ResourceManager | resource_manager.cpp | 资源管理器基类 |
| ResourceManagerImpl | resource_manager_impl.cpp | 资源管理器实现 (核心) |
| SystemResourceManager | system_resource_manager.cpp | 系统资源管理器 |
| ResourceManagerExtMgr | resource_manager_ext_mgr.cpp | 资源管理器扩展管理 |

**关键类**: `ResourceManager`, `ResourceManagerImpl`, `SystemResourceManager`

**证据**: `frameworks/resmgr/include/resource_manager.h`, `frameworks/resmgr/include/resource_manager_impl.h`

#### 1.3 配置与匹配模块

**职责**: 资源配置管理和最佳资源匹配

| 模块 | 文件 | 职责 |
|------|------|------|
| ResConfigImpl | res_config_impl.cpp | 资源配置实现 |
| LocaleMatcher | locale_matcher.cpp | 语言/区域匹配 |
| ResLocale | res_locale.cpp | 资源区域设置 |

**关键类**: `ResConfigImpl`, `LocaleMatcher`, `ResLocale`

**证据**: `frameworks/resmgr/include/res_config_impl.h`, `frameworks/resmgr/include/locale_matcher.h`

#### 1.4 工具模块

**职责**: 通用工具函数

| 模块 | 文件 | 职责 |
|------|------|------|
| Utils | utils.cpp | 通用工具函数 |
| StringUtils | string_utils.cpp | 字符串工具 |
| DateUtils | date_utils.h | 日期工具 |
| PsueManager | psue_manager.cpp | 原始字符串更新管理 |
| MmapFile | mmap_file.cpp | 内存映射文件 |

**证据**: `frameworks/resmgr/include/utils/utils.h`, `frameworks/resmgr/include/utils/string_utils.h`

#### 1.5 原始文件模块

**职责**: 原始文件访问

| 模块 | 文件 | 职责 |
|------|------|------|
| RawFileManager | raw_file_manager.cpp | 原始文件管理器 |
| NativeResourceManager | native_resource_manager.cpp | Native 资源管理器 |

**证据**: `frameworks/resmgr/src/raw_file_manager.cpp`, `frameworks/resmgr/src/native_resource_manager.cpp`

#### 1.6 主题包模块

**职责**: 主题资源包管理

| 模块 | 文件 | 职责 |
|------|------|------|
| ThemePackManager | theme_pack_manager.cpp | 主题包管理器 |
| ThemePackConfig | theme_pack_config.cpp | 主题包配置 |
| ThemePackResource | theme_pack_resource.cpp | 主题包资源 |

**证据**: `frameworks/resmgr/include/theme_pack_manager.h`

### 2. 接口层

#### 2.1 Inner API (interfaces/inner_api/)

**职责**: C++ 内部子系统接口

**导出符号** (从 bundle.json inner_kits):
- `res_common.h` - 通用定义
- `res_config.h` - 资源配置接口
- `resource_manager.h` - 资源管理器接口
- `rstate.h` - 资源状态定义

**使用场景**: 其他子系统调用资源管理组件

**证据**: `interfaces/inner_api/include/resource_manager.h`

#### 2.2 Native API (interfaces/native/)

**职责**: C 语言原生接口

**导出符号** (从 bundle.json inner_kits):
- `ohresmgr.h` - Native 资源管理器接口
- `raw_file.h` - 原始文件接口
- `raw_dir.h` - 原始目录接口
- `raw_file_manager.h` - 原始文件管理器接口
- `resmgr_common.h` - 通用定义

**使用场景**: C/C++ 原生应用开发 (NDK)

**证据**: `interfaces/native/resource/include/raw_file.h`

#### 2.3 JS/N-API (interfaces/js/)

**职责**: JavaScript Native API 接口

**子模块**:
- `innerkits/core/` - NAPI 核心实现 (C++)
- `kits/` - JS 对外接口 (NAPI 模块)

**导出方法**: 65 个 JS 方法 (详见 [N-API 接口](04_NAPI.md))

**使用场景**: JavaScript/ArkUI 应用开发

**证据**: `interfaces/js/kits/src/resource_manager_napi.cpp`

#### 2.4 ETS/ANI (interfaces/ets/)

**职责**: ArkTS Native Interface 桥接

**子模块**:
- `ets/` - ETS 类型定义
- `include/` - ANI 桥接头文件
- `src/` - ANI 桥接实现

**输出产物**:
- `resourceManager.abc`
- `resource.abc`
- `rawFileDescriptor.abc`

**使用场景**: ArkTS 应用开发

**证据**: `interfaces/ets/ani/resourceManager/src/resmgr_ani.cpp`

#### 2.5 Cangjie/FFI (interfaces/cj/)

**职责**: Cangjie FFI 接口

**子模块**:
- `resource_manager_ffi.cpp/h` - FFI 接口
- `resource_manager_impl.cpp/h` - FFI 实现
- `utils.cpp/h` - 工具函数

**使用场景**: Cangjie 语言应用开发

**证据**: `interfaces/cj/src/resource_manager_ffi.cpp`

### 3. 诊断层 (dfx/)

**职责**: 系统事件上报

**模块**:
- `hisysevent_adapter/` - HiSysEvent 适配器

**使用场景**: 系统监控和诊断

**证据**: `dfx/hisysevent_adapter/hisysevent_adapter.cpp`

## 源码统计（非测试文件）

| 分类 | 文件类型 | 数量 |
|------|----------|------|
| 头文件 (.h) | C/C++ 头文件 | ~45 个 |
| 实现文件 (.cpp/.c) | C/C++ 源文件 | ~55 个 |
| 接口定义 (.ets) | ArkTS 类型定义 | 5 个 |
| JavaScript (.js) | JS 模块 | 1 个 |
| 构建配置 (.gni/.json/.yaml) | 构建脚本 | 4 个 |

## 依赖关系

### 内部依赖

```
interfaces/js/kits (JS API)
    ↓
interfaces/js/innerkits/core (NAPI 实现)
    ↓
frameworks/resmgr (核心框架)
    ↓
interfaces/inner_api (Inner API)
interfaces/native (Native API)
```

### 外部依赖

| 组件 | 用途 |
|------|------|
| napi | N-API 框架 |
| hilog | 日志系统 |
| hisysevent | 系统事件 |
| icu | 国际化支持 |
| zlib | 压缩解压 |
| cJSON | JSON 解析 |
| bundle_framework | Bundle 框架 |

## 相关文档

- [概述](01_Overview.md) - 组件定位和核心能力
- [架构设计](03_Architecture.md) - 组件架构和数据流
- [N-API 接口](04_NAPI.md) - JavaScript API 详细文档
- [GN Targets](06_GNTargets.md) - 构建系统和依赖关系

---

**生成时间**: 2026-02-06
**证据来源**: 完整目录结构扫描, bundle.json, BUILD.gn
