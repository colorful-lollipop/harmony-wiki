# 项目概览

## 目的与适用范围

本文档介绍 OpenHarmony `global_resource_tool`（restool）组件的基本信息、项目定位和核心能力。

**适用对象**: 新加入的开发者、构建工程师、安全审计人员

---

## 项目定位

### 基本定义

**global_resource_tool**（简称 **restool**）是 OpenHarmony 的资源编译工具，负责：
- 编译资源文件（图片、JSON、XML 等）
- 创建资源索引（resources.index）
- 生成资源头文件（ResourceTable.h/js/txt/ts）
- 支持资源叠加编译（Overlay）
- 支持纹理压缩（ASTC/SUT）

### 在 OpenHarmony 中的位置

```
OpenHarmony SDK
└── toolchains/
    └── restool          <-- 本工具
```

**集成方式**:
- 随 SDK 分发，位于 `toolchains` 目录
- 被 IDE（DevEco Studio）调用
- 被构建系统（Hvigor/build）调用

### 项目边界

**属于本项目**:
- 资源编译核心逻辑
- 资源索引生成
- 资源头文件生成
- 命令行接口

**不属于本项目**:
- 资源运行时解析（在 frameworks 中）
- 资源加载（在 app 中）
- IDE 集成（在 DevEco Studio 中）
- 构建系统集成（在 Hvigor 中）

---

## 核心能力

### 1. 资源编译

**支持资源类型** (`include/resource_data.h:85-104`):
| 资源类型 | 说明 | 目录 |
|----------|------|------|
| ELEMENT | 元素资源 | element/ |
| RAW | 原始文件 | rawfile/ |
| MEDIA | 媒体文件 | media/ |
| PROF | 配置文件 | profile/ |
| INTEGER | 整型 | element/integer.json |
| STRING | 字符串 | element/string.json |
| STRARRAY | 字符串数组 | element/strarray.json |
| INTARRAY | 整型数组 | element/intarray.json |
| BOOLEAN | 布尔值 | element/boolean.json |
| COLOR | 颜色 | element/color.json |
| ID | ID 引用 | element/id.json |
| THEME | 主题 | element/theme.json |
| PLURAL | 复数 | element/plural.json |
| FLOAT | 浮点数 | element/float.json |
| PATTERN | 模式 | element/pattern.json |
| SYMBOL | 符号 | element/symbol.json |
| RES | 资源文件 | resfile/ |

### 2. 资源索引生成

生成 `resources.index` 文件，包含：
- 资源 ID 与资源路径的映射
- 限定词信息（语言、地区、分辨率等）
- 资源类型信息

### 3. 资源头文件生成

支持多种格式 (`src/resource_pack.cpp:143-150`):
- `.h` - C/C++ 头文件
- `.js` - JavaScript 模块
- `.txt` - 纯文本
- `.ts` - TypeScript 模块（API 23+）

### 4. 资源叠加编译

支持在已有 HAP/HSP 基础上叠加编译新资源。

### 5. 纹理压缩

支持 ASTC 和 SUT 纹理压缩 (`src/compression_parser.cpp`):
- 通过 `--compressed-config` 指定配置
- 动态加载 `libimage_transcoder_shared` 库

### 6. 多线程编译

支持多线程加速 (`src/thread_pool.cpp`):
- 通过 `--thread` 指定线程数
- 默认线程池大小: 8 (`include/resource_data.h:55`)

---

## 运行环境

### 支持平台

| 平台 | 支持状态 | 说明 |
|------|----------|------|
| Windows | ✅ | 通过 MinGW 交叉编译 |
| Linux | ✅ | 原生支持 |
| macOS | ✅ | 原生支持 |

### 编译依赖

**系统组件**:
- `zlib` - 压缩库

**第三方库** (`bundle.json:24-28`):
- `bounds_checking_function` - 安全边界检查
- `cJSON` - JSON 解析
- `libpng` - PNG 图片处理

### 运行时依赖

**可选依赖**:
- `libimage_transcoder_shared` - 纹理压缩库（仅在启用纹理压缩时需要）

---

## 关键概念

### 资源目录结构

```
resources/
├── base/                    # 默认资源
│   ├── element/            # 元素资源
│   ├── media/              # 媒体资源
│   ├── profile/            # 配置文件
│   └── rawfile/            # 原始文件
├── zh_CN/                  # 中文资源（限定词目录）
│   └── element/
├── en_US/                  # 英文资源
│   └── element/
└── ...
```

### 限定词（Qualifiers）

**支持的限定词类型** (`include/resource_data.h:69-83`):
| 类型 | 枚举值 | 说明 |
|------|--------|------|
| LANGUAGE | 0 | 语言（如 zh、en） |
| REGION | 1 | 地区（如 CN、US） |
| RESOLUTION | 2 | 分辨率（如 sdpi、mdpi） |
| ORIENTATION | 3 | 方向（vertical、horizontal） |
| DEVICETYPE | 4 | 设备类型（phone、tablet） |
| SCRIPT | 5 | 文字脚本 |
| NIGHTMODE | 6 | 夜间模式（dark、light） |
| MCC | 7 | 移动国家码 |
| MNC | 8 | 移动网络码 |
| INPUTDEVICE | 10 | 输入设备 |

### 资源 ID 分配

**ID 范围** (`src/cmd/package_parser.cpp:303`):
- 有效范围: `[0x01000000, 0x06FFFFFF)` 和 `[0x08000000, 0xFFFFFFFF)`
- 系统资源: `ohos.global.systemres` 使用 `RES_ID_SYS` 类型
- 应用资源: 其他使用 `RES_ID_APP` 类型

### 资源索引文件格式

**Header 结构** (`src/resource_table.cpp`):
- 魔数标识
- 版本信息
- 数据偏移量
- 数据池

---

## 版本信息

**当前版本** (`include/resource_data.h:53`):
```
RestoolV2 6.1.0.003
```

**版本历史**:
- API 18: 支持 `--thread` 选项
- API 19: 支持 `--ignored-file` 选项
- API 20: 支持新模块类型
- API 23: 支持 `.ts` 头文件格式、`--ignored-path` 选项

---

## 相关文档

- [目录结构](01_Directory_Structure.md) - 代码组织方式
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [对外接口](03_Public_API.md) - 命令行使用方法
- [GN 构建目标](07_GN_Targets.md) - 构建系统配置
