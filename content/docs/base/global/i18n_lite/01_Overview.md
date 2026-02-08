# 项目概述

## 1.1 项目定位

### 1.1.1 在 OpenHarmony 中的位置

i18n_lite 是 **OpenHarmony Globalization 子系统**的核心组件，隶属于 `base/global/` 目录层级。

```
OpenHarmony
├── base/
│   ├── global/              # Globalization 子系统
│   │   ├── i18n_lite/      # 轻量级国际化组件 (本文档)
│   │   └── resource_lite/  # 资源管理
│   └── ...
├── foundation/
│   └── arkui/               # UI 框架
└── ...
```

**定位说明**：
- 为 Mini System 和 Small System 提供轻量级国际化能力
- 相比 full i18n 模块，资源占用更小，适合资源受限设备
- 提供 C++ API 和 JavaScript API (ACELite) 两种接入方式

### 1.1.2 核心目标

| 目标 | 说明 | 实现方式 |
|------|------|----------|
| **轻量化** | 最小化内存和存储占用 | 精简数据文件、静态链接 |
| **本地化** | 支持多语言/区域格式习惯 | 区域数据文件 (i18n.dat) |
| **易用性** | 提供简洁的 API 接口 | N-API 和 C++ 头文件封装 |
| **可扩展** | 支持新增语言和区域 | 数据驱动架构 |

## 1.2 核心能力

### 1.2.1 能力清单

| 能力 | 描述 | 头文件 | 实现类 |
|------|------|--------|--------|
| **日期时间格式化** | 按区域习惯格式化日期和时间 | `date_time_format.h` | `DateTimeFormat` |
| **数字格式化** | 按区域习惯格式化数字和百分比 | `number_format.h` | `NumberFormat` |
| **复数规则** | 处理不同语言的复数形式 | `plural_format.h` | `PluralFormat` |
| **区域信息管理** | 管理语言/脚本/地区信息 | `locale_info.h` | `LocaleInfo` |
| **度量单位格式化** | 按区域格式化度量单位 | `measure_format.h` | `MeasureFormat` |
| **周信息** | 获取周相关数据 | `week_info.h` | `WeekInfo` |

### 1.2.2 能力详细说明

#### 日期时间格式化

支持多种格式化模式：

```cpp
enum AvailableDateTimeFormatPattern {
    HOUR12_MINUTE_SECOND,      // 12小时制 时:分:秒
    HOUR24_MINUTE_SECOND,      // 24小时制 时:分:秒
    HOUR_MINUTE,               // 时:分
    FULL,                      // 完整格式 (如 "Friday December 18, 2020")
    MEDIUM,                    // 中等格式 (如 "Dec 18, 2020")
    SHORT,                     // 短格式 (如 "12/18/2020")
    // ... 更多模式
};
```

#### 数字格式化

支持数字和百分比格式化：

```cpp
enum NumberFormatType {
    DECIMAL,   // 十进制格式
    PERCENT,   // 百分比格式
};
```

#### 复数规则

支持多种复数规则类型：

```cpp
enum PluralRuleType {
    ZERO,   // 零
    ONE,    // 一
    TWO,    // 二
    FEW,    // 少量
    MANY,   // 大量
    OTHER,  // 其他
};
```

不同语言的复数规则示例：
- **英语**：one, other
- **中文**：other (无复数变化)
- **阿拉伯语**：zero, one, two, few, many, other

## 1.3 目录结构

```
i18n_lite/
├── frameworks/i18n/                    # 核心框架实现
│   ├── include/                        # 内部头文件
│   │   ├── data_resource.h            # 资源数据管理
│   │   ├── date_time_data.h          # 日期时间数据
│   │   ├── date_time_format_impl.h   # 日期时间格式化实现
│   │   ├── i18n_memory_adapter.h     # 内存适配器
│   │   ├── i18n_pattern.h            # 格式化模式
│   │   ├── measure_format_impl.h     # 度量格式化实现
│   │   ├── number_data.h             # 数字数据
│   │   ├── number_format_impl.h      # 数字格式化实现
│   │   ├── plural_format_impl.h      # 复数格式化实现
│   │   ├── plural_rules.h            # 复数规则
│   │   ├── str_util.h               # 字符串工具
│   │   └── week_info.h              # 周信息
│   ├── src/                           # 实现代码
│   │   ├── data_resource.cpp         # 资源数据管理实现
│   │   ├── date_time_data.cpp       # 日期时间数据实现
│   │   ├── date_time_format.cpp      # 日期时间格式化入口
│   │   ├── date_time_format_impl.cpp # 日期时间格式化实现
│   │   ├── locale_info.cpp           # 区域信息实现
│   │   ├── measure_format.cpp        # 度量格式化入口
│   │   ├── measure_format_impl.cpp   # 度量格式化实现
│   │   ├── number_data.cpp          # 数字数据实现
│   │   ├── number_format.cpp         # 数字格式化入口
│   │   ├── number_format_impl.cpp   # 数字格式化实现
│   │   ├── plural_format.cpp         # 复数格式化入口
│   │   ├── plural_format_impl.cpp   # 复数格式化实现
│   │   ├── plural_rules.cpp         # 复数规则实现
│   │   ├── str_util.cpp             # 字符串工具实现
│   │   └── week_info.cpp            # 周信息实现
│   ├── BUILD.gn                       # 构建配置
│   └── i18n.dat                      # 区域数据文件 (二进制)
│
├── interfaces/kits/                    # API 接口
│   ├── i18n/                          # C++ API 头文件
│   │   └── include/
│   │       ├── date_time_format.h    # 日期时间格式化 API
│   │       ├── locale_info.h         # 区域信息 API
│   │       ├── measure_format.h      # 度量格式化 API
│   │       ├── number_format.h       # 数字格式化 API
│   │       ├── plural_format.h       # 复数格式化 API
│   │       ├── types.h               # 类型定义
│   │       └── week_info.h           # 周信息 API
│   │
│   └── js/builtin/                     # JavaScript API (ACELite)
│       ├── include/
│       │   └── locale_module.h       # JS 区域模块
│       ├── src/
│       │   └── locale_module.cpp     # JS API 实现
│       ├── BUILD.gn                   # JS 构建配置
│       └── CMakeLists.txt            # CMake 配置
│
├── tools/i18n-dat-tool/               # 区域数据生成工具
│   ├── src/main/                      # 工具源码
│   │   ├── java/                     # Java 实现
│   │   ├── python/                   # Python 实现
│   │   └── resources/               # 工具资源
│   ├── resources/                     # 区域数据模板
│   │   ├── locales.json              # 区域列表
│   │   ├── date-patterns.json        # 日期模式
│   │   ├── number-format.json        # 数字格式
│   │   ├── plural.json              # 复数规则
│   │   └── ... 更多数据文件
│   └── pom.xml                       # Maven 配置
│
├── bundle.json                        # 组件配置
├── i18n_lite.gni                     # GN 标志定义
└── LICENSE                           # Apache 2.0 许可证
```

### 目录职责说明

| 目录 | 职责 | 稳定性 |
|------|------|--------|
| `frameworks/i18n/include/` | 内部实现头文件 | 不稳定 |
| `frameworks/i18n/src/` | 核心实现代码 | 不稳定 |
| `interfaces/kits/i18n/include/` | 公共 C++ API 头文件 | 稳定 |
| `interfaces/kits/js/builtin/` | JS API 实现 | 稳定 |
| `tools/i18n-dat-tool/` | 数据生成工具 | 内部使用 |

## 1.4 关键概念

### 1.4.1 Locale (区域)

区域是语言、脚本和地区的组合，用于表示特定的语言文化习惯。

```cpp
// 格式: language_script_REGION
LocaleInfo locale("zh", "Hans", "CN");  // 简体中文 (中国)
LocaleInfo locale("en", "US");           // 英语 (美国)
LocaleInfo locale("en", "", "GB");      // 英语 (英国)
```

**组成要素**：
- **language**: ISO 639 语言代码 (2-3 字符，如 "zh", "en")
- **script**: ISO 15924 脚本代码 (4 字符，如 "Hans", "Latn")
- **region**: ISO 3166 国家/地区代码 (2 字符，如 "CN", "US")

### 1.4.2 格式化模式

#### 日期时间模式

| 模式 | 示例 | 说明 |
|------|------|------|
| `FULL` | "Friday December 18, 2020" | 完整格式 |
| `MEDIUM` | "Dec 18, 2020" | 中等格式 |
| `SHORT` | "12/18/2020" | 短格式 |

#### 数字模式

| 模式 | 示例 (en_US) | 说明 |
|------|--------------|------|
| `DECIMAL` | "1,234.56" | 十进制 |
| `PERCENT` | "50%" | 百分比 |

### 1.4.3 复数规则

复数规则定义了不同数量对应的名词形式。CLDR 定义了六种规则：

| 规则 | 适用语言示例 | 示例 |
|------|-------------|------|
| `ZERO` | 阿拉伯语 | 0 books |
| `ONE` | 英语、德语 | 1 book |
| `TWO` | 阿拉伯语、斯洛文尼亚语 | 2 books |
| `FEW` | 俄语、波兰语 | few books |
| `MANY` | 阿拉伯语 | many books |
| `OTHER` | 所有语言 | other books |

## 1.5 运行环境

### 1.5.1 适配系统类型

| 系统类型 | 说明 | 支持状态 |
|----------|------|----------|
| **Mini System** | 轻量级设备 (如智能手表) | ✅ 支持 |
| **Small System** | 小型设备 (如智能音箱) | ✅ 支持 |
| **Standard System** | 标准设备 (如手机) | ❌ 不支持 (使用 full i18n) |

### 1.5.2 依赖组件

#### 内部依赖

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| `utils_lite` | 组件依赖 | 基础工具库 |
| `bounds_checking_function` | 第三方依赖 | 内存安全函数 |

#### 运行时依赖

| 依赖 | 说明 | 位置 |
|------|------|------|
| `i18n.dat` | 区域数据文件 | 系统分区 `system/i18n/` |

### 1.5.3 内存与存储

| 指标 | 估算值 | 说明 |
|------|--------|------|
| **静态库大小** | ~500KB | 未压缩的 global_i18n.a |
| **数据文件** | ~83KB | i18n.dat 二进制文件 |
| **运行时内存** | 取决于使用 | 按需加载数据 |

## 1.6 版本与兼容性

### 1.6.1 版本历史

| 版本 | 发布日期 | 主要变更 |
|------|----------|----------|
| 1.0.0 | 2021-2022 | 初始版本，支持核心 i18n 功能 |

### 1.6.2 API 兼容性

| API 头文件 | 版本 | 稳定性 |
|------------|------|--------|
| `date_time_format.h` | 1.0 | 稳定 |
| `number_format.h` | 1.0 | 稳定 |
| `plural_format.h` | 1.0 | 稳定 |
| `locale_info.h` | 1.0 | 稳定 |
| `locale_module.h` | 1.0 | 稳定 |

## 1.7 相关资源

### 内部链接

- [API 接口文档](./02_API.md)
- [架构设计](./03_Architecture.md)
- [构建配置](./04_Build.md)
- [安全分析](./05_Security.md)

### 外部资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [CLDR Unicode 语言环境数据](http://cldr.unicode.org/)
- [BCP 47 语言标签](https://tools.ietf.org/html/bcp47)

---

*最后更新：2026-02-06*
