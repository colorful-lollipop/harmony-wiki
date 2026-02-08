# 目录结构与模块职责

> 各目录职责说明与模块边界划分

## 顶层目录结构

```
napi_generator/                              # 项目根目录
├── src/                                    # 源代码目录
│   └── cli/                                # 代码生成工具集 (6 个)
├── examples/                               # 示例项目
├── docs/                                   # 工具文档
├── figures/                                # 架构图等资源
├── release-notes/                          # 版本说明
└── wiki/                                   # 本 Wiki 文档
```

---

## src/cli/ - 代码生成工具集

### 目录概览

```
src/cli/
├── dts2cpp/                                 # ⭐ 核心: TS → N-API
├── h2sa/                                    # Service Ability 生成
├── h2dtscpp/                                # C++ → TS + NAPI + 测试
├── h2dts/                                   # C++ → TypeScript
├── cmake2gn/                                # CMake → GN
└── h2hdf/                                   # HDF 驱动生成
```

### 1. dts2cpp - TypeScript 到 N-API

**路径**: `src/cli/dts2cpp/`

**职责**: 将 TypeScript 声明文件转换为完整的 C++ N-API 框架代码

**证据**: `src/cli/dts2cpp/src/gen/cmd_gen.js`

```
dts2cpp/
├── src/
│   ├── gen/                                 # 核心生成代码
│   │   ├── cmd_gen.js                      # 命令行入口
│   │   ├── main.js                         # 主流程控制
│   │   ├── analyze.js                      # TS 解析器
│   │   ├── generate.js                     # 代码生成器
│   │   ├── analyze/                        # 解析子模块
│   │   │   ├── namespace.js
│   │   │   ├── interface.js
│   │   │   ├── function.js
│   │   │   └── ...
│   │   ├── generate/                       # 生成子模块
│   │   │   ├── function_sync.js
│   │   │   ├── function_async.js
│   │   │   ├── function_direct.js
│   │   │   └── ...
│   │   ├── extend/                         # 扩展功能
│   │   │   ├── tool_utility.js            # XNapiTool 生成
│   │   │   ├── binding_gyp.js
│   │   │   └── build_gn.js
│   │   └── tools/                          # 工具函数
│   │       ├── common.js
│   │       ├── re.js
│   │       └── NapiLog.js
│   ├── package.json                        # 依赖配置
│   └── jsconfig.json
├── docs/                                   # 文档
└── dts2cpp_README_ZH.md                    # 工具说明
```

**模块职责**:
| 子目录/文件 | 职责 |
|------------|------|
| `analyze/` | TypeScript 语法解析，提取接口、函数、枚举等 |
| `generate/` | 根据解析结果生成 N-API C++ 代码 |
| `extend/` | 生成 BUILD.gn、binding.gyp 等构建脚本 |
| `tools/` | 公共工具函数 |

### 2. h2sa - Service Ability 框架生成

**路径**: `src/cli/h2sa/`

**职责**: 根据 .h 头文件生成完整的 System Ability 框架代码

**证据**: `src/cli/h2sa/src/gen/main.js`

```
h2sa/
├── src/
│   ├── gen/                                 # 代码生成器
│   │   ├── main.js                         # 主入口
│   │   ├── analyze.js                      # .h 解析器
│   │   ├── generate.js                     # 框架代码生成
│   │   ├── file_template.js               # 代码模板
│   │   └── test.h                         # 测试输入
│   ├── tools/                              # 工具模块
│   │   ├── common.js                       # MessageParcel 类型映射
│   │   ├── tool.js
│   │   ├── file_rw.js
│   │   └── re.js
│   └── package.json
├── docs/                                   # 使用文档
└── README_ZH.md                            # 工具说明
```

**模块职责**:
| 子目录/文件 | 职责 |
|------------|------|
| `gen/` | SA 框架代码生成 (proxy/stub/service) |
| `tools/` | MessageParcel 类型映射、文件操作 |

### 3. h2dtscpp - 完整开发链

**路径**: `src/cli/h2dtscpp/`

**职责**: 从 C++ 头文件生成 TypeScript 声明 + N-API 实现 + 测试用例

**证据**: `src/cli/h2dtscpp/src/src/main.js`

```
h2dtscpp/
└── src/
    ├── src/
    │   ├── main.js                         # CLI 入口
    │   ├── tsGen/                          # TypeScript 生成
    │   │   ├── tsMain.js                  # 主生成逻辑
    │   │   └── ...
    │   ├── napiGen/                        # N-API 生成
    │   │   ├── functionDirect.js         # 直接调用函数
    │   │   ├── functionDirectTest.js     # 测试用例
    │   │   └── ...
    │   ├── json/                           # 模板文件
    │   │   ├── function.json
    │   │   └── directFunction/           # 模板目录
    │   │       ├── cppTempleteDetails/
    │   │       ├── testTempleteDetails/
    │   │       └── dtsTempleteDetails/
    │   └── tools/                          # 工具函数
    └── package.json
```

**模块职责**:
| 子目录/文件 | 职责 |
|------------|------|
| `tsGen/` | 解析 C++ 头文件，生成 TS 声明 |
| `napiGen/` | 生成 N-API 实现代码和测试用例 |
| `json/` | 代码模板定义 |

### 4. h2dts - C++ 到 TypeScript

**路径**: `src/cli/h2dts/`

**职责**: 将 C++ 头文件转换为 TypeScript 声明文件

```
h2dts/
├── src/                                    # 工具源码
├── docs/                                   # 文档
├── examples/                               # 示例
└── README_ZH.md                            # 说明文档
```

### 5. cmake2gn - CMake 到 GN

**路径**: `src/cli/cmake2gn/`

**职责**: 将 CMakeLists.txt 转换为 BUILD.gn

```
cmake2gn/
├── src/                                    # 工具源码
├── docs/                                   # 文档
└── README_ZH.md                            # 说明文档
```

### 6. h2hdf - HDF 驱动生成

**路径**: `src/cli/h2hdf/`

**职责**: 生成 Hardware Driver Framework 相关代码

---

## examples/ - 示例项目

### 目录概览

```
examples/
├── napitutorials/                          # ⭐ N-API 教程 (75+ 示例)
├── akitutorials/                           # AKI 框架示例
├── serviceCode/                            # SA 服务示例
├── pluginCase/                             # 插件案例
├── p7zipTest/                              # 完整 N-API 应用
├── ts/                                     # TypeScript 定义示例
└── appCodeGen/                             # 应用代码生成示例
```

### 1. napitutorials - N-API 教程

**路径**: `examples/napitutorials/`

**职责**: 提供完整的 N-API 开发教程，涵盖 75+ 个 C++ 示例

**证据**: `examples/napitutorials/entry/src/main/cpp/javascriptapi/`

```
napitutorials/
└── entry/src/main/
    ├── cpp/
    │   ├── javascriptapi/                 # JS API 示例
    │   │   ├── jsvalues/                 # 值操作 (napi_create_*)
    │   │   ├── jsobjectwrap/            # 对象包装 (napi_wrap)
    │   │   ├── jsabstractops/            # 抽象操作 (napi_typeof)
    │   │   └── jsproperty/              # 属性操作 (napi_set_property)
    │   ├── nodeapi/                      # Node-API 底层
    │   └── ncpp/                         # Native C++
    ├── ets/                              # ArkTS UI
    └── js/                               # JS 测试
```

**模块职责**:
| 子目录 | 职责 |
|--------|------|
| `jsvalues/` | napi_create_* / napi_get_value_* 示例 |
| `jsobjectwrap/` | napi_wrap / napi_unwrap 示例 |
| `jsabstractops/` | napi_typeof / napi_coerce_* 示例 |
| `jsproperty/` | napi_set/get_property 示例 |

### 2. akitutorials - AKI 框架示例

**路径**: `examples/akitutorials/`

**职责**: 展示 AKI (简化 N-API 开发框架) 的使用

```
akitutorials/
└── entry/src/main/
    ├── cpp/
    │   ├── napi_init.cpp                 # AKI 绑定代码
    │   └── types/libentry/
    │       └── Index.d.ts                # 类型定义
    └── ets/
        └── pages/                        # UI 页面
```

### 3. p7zipTest - 完整应用示例

**路径**: `examples/p7zipTest/`

**职责**: 展示生产级 N-API 应用架构 (集成第三方库)

```
p7zipTest/
└── entry/src/main/
    ├── cpp/
    │   ├── napi/                         # N-API 入口
    │   │   ├── napi_init.cpp
    │   │   ├── napi_compress_async.cpp
    │   │   └── napi_decompress_async.cpp
    │   └── 7z/                           # 7z C++ 库
    └── ets/
        └── pages/                        # UI 页面
```

---

## docs/ - 工具文档

```
docs/
├── log/                                   # 开发日志
│   └── meeting-minutes/                   # 会议纪要
└── (各工具子目录)                          # 工具特定文档
```

---

## 模块依赖关系

```
                    ┌─────────────────────────────────────┐
                    │            examples/                 │
                    │   (依赖 cli 生成的框架代码)          │
                    └─────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           src/cli/                                          │
│                                                                             │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐          │
│  │ dts2cpp  │────▶│  h2sa    │     │ h2dtscpp │     │ h2dts    │          │
│  │ (核心)   │     │          │     │          │     │          │          │
│  └──────────┘     └──────────┘     └──────────┘     └──────────┘          │
│       │               │               │               │                     │
│       └───────────────┴───────────────┴───────────────┘                     │
│                               │                                             │
└───────────────────────────────┼─────────────────────────────────────────────┘
                                │
                                ▼
                    ┌─────────────────────────────────────┐
                    │         docs/                        │
                    │     (工具使用文档)                   │
                    └─────────────────────────────────────┘
```

---

## 相关章节

- 项目概览: [01_Overview.md](01_Overview.md)
- 架构设计: [03_Architecture.md](03_Architecture.md)
- API 参考: [04_NAPI_Reference.md](04_NAPI_Reference.md)

---

[返回 SUMMARY.md](SUMMARY.md)
