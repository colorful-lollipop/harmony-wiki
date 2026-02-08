# 目录结构与模块职责

## 1. 顶层目录结构

```
resource_management_lite/
├── frameworks/                    # 核心代码目录
│   └── resmgr_lite/              # 资源管理核心实现
│       ├── include/               # 头文件目录
│       ├── src/                   # 源文件目录
│       └── test/                  # 测试代码（不计入业务代码）
├── interfaces/                    # 接口目录
│   └── inner_api/                # 内部 API（供子系统内使用）
│       └── include/               # 内部 API 头文件
├── wiki/                         # 本 Wiki 文档
│   ├── README.md                  # 文档说明
│   ├── SUMMARY.md                 # 导航索引
│   ├── 00_Overview.md            # 项目概览
│   ├── 01_Directory_Structure.md  # 本文档
│   ├── 02_Architecture.md         # 架构设计
│   ├── 03_C_API.md               # C API 文档
│   ├── 04_Cpp_API.md             # C++ API 文档
│   ├── 05_Build_System.md         # 构建系统
│   ├── 06_Security_Analysis.md    # 安全分析
│   └── 07_Troubleshooting.md      # 常见问题
├── bundle.json                   # 组件配置
├── BUILD.gn                      # GN 构建入口
├── CMakeLists.txt                # CMake 构建入口（模拟器）
├── README.md                     # 英文 README
├── README_zh.md                  # 中文 README
└── LICENSE                       # Apache 2.0 许可证
```

---

## 2. 框架目录详解

### 2.1 include/ 头文件目录

| 文件路径 | 模块 | 职责 |
|----------|------|------|
| `res_config.h` | 资源配置 | 定义 ResConfig 抽象基类 |
| `res_config_impl.h` | 资源配置 | ResConfigImpl 具体实现 |
| `res_locale.h` | 区域配置 | 定义 ResLocale 区域信息类 |
| `res_common.h` | 公共定义 | 定义枚举类型（ResType, KeyType, DeviceType 等） |
| `res_desc.h` | 资源描述 | 定义 ResDesc/ResKey/ResId 等资源描述结构 |
| `resource_manager.h` | 资源管理 | 定义 ResourceManager 虚基类 |
| `resource_manager_impl.h` | 资源管理 | ResourceManagerImpl 具体实现 |
| `hap_manager.h` | HAP 管理 | 管理多个 HAP 包的资源 |
| `hap_resource.h` | HAP 资源 | 表示单个 HAP 包的资源 |
| `hap_parser.h` | HAP 解析 | 解析 HAP 包中的资源索引 |
| `global_utils.h` | 工具函数 | 定义 GlobalUtilsImpl 工具接口 |
| `global.h` | C API | C 接口声明 |
| `locale_matcher.h` | 区域匹配 | LocaleMatcher 类定义 |
| `lock.h` | 同步 | 互斥锁定义 |
| `auto_mutex.h` | 同步 | RAII 互斥锁封装 |
| `hilog_wrapper.h` | 日志 | 日志包装器 |

#### utils/ 子目录

| 文件路径 | 职责 |
|----------|------|
| `locale_data.h` | 语言代码映射数据 |
| `errors.h` | 错误码定义 |
| `utils.h` | 工具函数接口 |
| `common.h` | 公共常量定义 |
| `string_utils.h` | 字符串工具 |
| `date_utils.h` | 日期工具 |

### 2.2 src/ 源文件目录

| 文件路径 | 模块 | 职责 |
|----------|------|------|
| `global.c` | C API | GLOBAL_* 函数实现（liteos_m 编译单元） |
| `global.cpp` | C API | GLOBAL_* 函数实现（liteos_a 编译单元） |
| `global_utils.c` | 工具实现 | GlobalUtilsImpl 函数实现 |
| `resource_manager_impl.cpp` | 资源管理 | ResourceManagerImpl 实现 |
| `res_config_impl.cpp` | 资源配置 | ResConfigImpl 实现 |
| `res_locale.cpp` | 区域配置 | ResLocale 实现 |
| `hap_manager.cpp` | HAP 管理 | HapManager 实现 |
| `hap_resource.cpp` | HAP 资源 | HapResource 实现 |
| `res_desc.cpp` | 资源描述 | ResDesc 实现 |
| `locale_matcher.cpp` | 区域匹配 | LocaleMatcher 实现 |
| `lock.cpp` | 同步 | Lock 实现 |

#### utils/ 子目录

| 文件路径 | 职责 |
|----------|------|
| `hap_parser.cpp` | HAP 资源索引解析 |
| `utils.cpp` | 工具函数实现 |
| `string_utils.cpp` | 字符串工具实现 |

#### 数据文件

| 文件路径 | 职责 |
|----------|------|
| `likely_subtags_key_data.cpp` | Likely Subtags 键数据 |
| `likely_subtags_value_data.cpp` | Likely Subtags 值数据 |

---

## 3. 接口目录详解

### 3.1 interfaces/inner_api/include/

| 文件路径 | 职责 |
|----------|------|
| `global.h` | 内部 C API 声明，供子系统内部使用 |

**证据来源**：
- `interfaces/inner_api/include/global.h` (lines 1-50)

---

## 4. 模块职责划分

### 4.1 模块依赖图

```
┌─────────────────────────────────────────────────────────────────────┐
│                           C API 层                                  │
│   interfaces/inner_api/include/global.h                              │
│   frameworks/resmgr_lite/src/global.c / global.cpp                   │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       资源管理层                                     │
│   frameworks/resmgr_lite/include/resource_manager.h                  │
│   frameworks/resmgr_lite/src/resource_manager_impl.cpp               │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       HAP 管理层                                     │
│   frameworks/resmgr_lite/include/hap_manager.h                        │
│   frameworks/resmgr_lite/src/hap_manager.cpp                          │
│   frameworks/resmgr_lite/include/hap_resource.h                       │
│   frameworks/resmgr_lite/src/hap_resource.cpp                        │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       解析层                                         │
│   frameworks/resmgr_lite/include/hap_parser.h                         │
│   frameworks/resmgr_lite/src/utils/hap_parser.cpp                     │
│   frameworks/resmgr_lite/include/res_desc.h                           │
│   frameworks/resmgr_lite/src/res_desc.cpp                             │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       配置层                                         │
│   frameworks/resmgr_lite/include/res_config.h                         │
│   frameworks/resmgr_lite/src/res_config_impl.cpp                      │
│   frameworks/resmgr_lite/include/res_locale.h                         │
│   frameworks/resmgr_lite/src/res_locale.cpp                           │
│   frameworks/resmgr_lite/include/locale_matcher.h                      │
│   frameworks/resmgr_lite/src/locale_matcher.cpp                       │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
┌─────────────────────────────────▼───────────────────────────────────┐
│                       工具层                                         │
│   frameworks/resmgr_lite/include/global_utils.h                       │
│   frameworks/resmgr_lite/src/global_utils.c                           │
│   frameworks/resmgr_lite/include/utils/*.h                            │
│   frameworks/resmgr_lite/src/utils/*.cpp                             │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 各模块职责

| 模块 | 主要类/函数 | 职责描述 |
|------|-------------|----------|
| **C API** | `GLOBAL_GetValueById`, `GLOBAL_GetValueByName` | 提供 C 接口供 native 应用调用 |
| **ResourceManager** | `ResourceManager`, `ResourceManagerImpl` | 资源管理的 C++ 接口，封装 HAP 管理 |
| **HapManager** | `AddResource`, `FindResourceById` | 管理多个 HAP 包，资源匹配和选择 |
| **HapResource** | `LoadFromIndex`, `GetIdValues` | 单个 HAP 包的资源索引管理 |
| **HapParser** | `ParseResHex`, `ReadFileFromZip` | 解析 HAP 中的 resources.index 二进制文件 |
| **ResConfig** | `SetLocaleInfo`, `Match` | 资源配置抽象接口 |
| **ResConfigImpl** | `IsMoreSuitable`, `Copy` | 资源配置具体实现 |
| **ResLocale** | `BuildFromString`, `BuildFromParts` | BCP 47 Locale 解析和构建 |
| **LocaleMatcher** | `Match`, `IsMoreSuitable` | Locale 匹配算法 |
| **GlobalUtils** | `GetOffsetByLocale`, `GetIdItem` | 底层工具函数（ID 解析、文件读取） |

---

## 5. 稳定性标注

### 5.1 公开接口稳定性

| 接口 | 位置 | 稳定性 | 说明 |
|------|------|--------|------|
| `GLOBAL_*` | `global.h` | **稳定** | 对外 C API，长期维护 |
| `ResourceManager` | `resource_manager.h` | **稳定** | 对外 C++ 抽象接口 |
| `ResConfig` | `res_config.h` | **稳定** | 对外抽象基类 |
| `CreateResourceManager` | `resource_manager.h` | **稳定** | 工厂函数 |
| `CreateResConfig` | `res_config_impl.h` | **稳定** | 工厂函数 |
| `ResConfigImpl` | `res_config_impl.h` | **内部** | 实现类，非公开 |
| `HapManager` | `hap_manager.h` | **内部** | 内部实现类 |
| `HapResource` | `hap_resource.h` | **内部** | 内部实现类 |
| `GlobalUtilsImpl` | `global_utils.h` | **内部** | 内部工具接口 |

### 5.2 头文件层级

```
对外公开 (interfaces/inner_api/include/)
    └── global.h

框架公开 (frameworks/resmgr_lite/include/)
    ├── resource_manager.h    (C++ 公共接口)
    ├── res_config.h          (配置抽象接口)
    ├── res_common.h          (公共枚举定义)
    ├── rstate.h              (错误码定义)
    └── res_desc.h            (数据结构定义)

内部使用 (frameworks/resmgr_lite/include/)
    ├── resource_manager_impl.h
    ├── hap_manager.h
    ├── hap_resource.h
    ├── hap_parser.h
    ├── res_config_impl.h
    ├── res_locale.h
    ├── locale_matcher.h
    ├── global_utils.h
    ├── lock.h
    ├── auto_mutex.h
    └── hilog_wrapper.h
```

---

## 6. 测试目录（仅供参考）

```
test/
└── unittest/
    └── lite/
        └── common/
            ├── global_test.cpp/h
            ├── hap_manager_test.cpp/h
            ├── hap_parser_test.cpp/h
            ├── hap_resource_test.cpp/h
            ├── locale_info_test.cpp/h
            ├── res_config_impl_test.cpp/h
            ├── res_config_test.cpp/h
            ├── res_desc_test.cpp/h
            ├── resource_manager_test.cpp/h
            ├── string_utils_test.cpp/h
            └── ... 其他测试文件
```

**注意**：测试代码不计入业务代码统计，仅供参考。

---

## 7. 文档链接

| 主题 | 文档 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| C API | [03_C_API.md](./03_C_API.md) |
| C++ API | [04_Cpp_API.md](./04_Cpp_API.md) |
| 构建系统 | [05_Build_System.md](./05_Build_System.md) |
| 安全分析 | [06_Security_Analysis.md](./06_Security_Analysis.md) |
