# Python 3.11.4 概览

## 库基本信息

| 属性 | 值 |
|-----|-----|
| **库名称** | Python (CPython) |
| **版本** | 3.11.4 |
| **许可证** | Python Software Foundation License V2 |
| **上游地址** | https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tgz |
| **维护者** | anguanglin@huawei.com |

## 原始功能简介

Python 是一种高级、解释型、通用的编程语言，强调代码的可读性和简洁性。CPython 是 Python 的参考实现，使用 C 语言编写，包含:

- **Python 解释器**: 执行 Python 字节码
- **标准库**: 丰富的内置模块 (文件 I/O、网络、正则表达式等)
- **C API**: 用于编写 C 扩展模块
- **开发工具**: 包括 IDLE、文档生成工具等

## OpenHarmony 中的定位和作用

### 在 OH 中的角色

Python 在 OpenHarmony 中主要承担**构建时工具链**的角色，而非运行时组件:

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 构建系统                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│   │  ArkCompiler │    │   hiperf     │    │   protobuf   │  │
│   │  (ETS 前端)   │    │  (性能分析)   │    │  (代码生成)   │  │
│   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘  │
│          │                   │                   │          │
│          └───────────────────┴───────────────────┘          │
│                              │                              │
│                              ▼                              │
│                    ┌──────────────────┐                     │
│                    │  Python 3.11.4   │                     │
│                    │  (构建时工具)     │                     │
│                    └──────────────────┘                     │
│                              │                              │
│                              ▼                              │
│                    ┌──────────────────┐                     │
│                    │   LLVM/Clang     │                     │
│                    └──────────────────┘                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 主要使用场景

| 场景 | 说明 | 相关组件 |
|-----|------|---------|
| **ArkCompiler ETS 前端** | 处理 TypeScript/ETS 语法分析和转换 | arkcompiler/ets_frontend |
| **性能分析工具** | hiperf 主机端脚本处理 | developtools/hiperf |
| **Protobuf 代码生成** | 生成 C++ 代码 | trace_streamer |
| **ArkGuard 混淆** | ArkTS 代码混淆工具 | arkguard |
| **构建脚本** | GN 构建系统中的 Python 脚本 | 多处 |

### 与 OH 系统类型的关系

根据 `bundle.json` 配置，Python 适配以下系统类型:

- **mini**: 最小系统，Python 作为构建工具
- **small**: 小型系统，Python 作为构建工具
- **standard**: 标准系统，Python 作为构建工具

注意: Python **不作为**系统运行时组件部署到设备。

## OH 定制化概述

### 主要 Patch

1. **cross_compile_support_ohos.patch** (OHOS 特有)
   - 添加 OpenHarmony 目标平台识别
   - 禁用部分与 OHOS 不兼容的模块

2. **cpython_mingw_v3.11.4.patch** (通用增强)
   - 添加 MinGW-w64 构建支持
   - 改进 Windows 开发环境支持

### 模块禁用

为适应 OHOS 环境，以下模块被禁用:

| 模块 | 功能 | 禁用原因 |
|-----|------|---------|
| `_uuid` | UUID 生成 | 依赖系统库 |
| `_socket` | 网络套接字 | TODO: 需确认 |
| `zlib` | 压缩/解压 | 可能与其他组件冲突 |
| `_ctypes` | C 类型接口 | TODO: 需确认 |
| `binascii` | 二进制/ASCII 转换 | TODO: 需确认 |

### 版本管理

```
Python 3.11.4 (上游版本)
    │
    ├── OpenHarmony 3.1 (当前集成版本)
    │
    ├── cross_compile_support_ohos.patch
    │   └── OHOS 交叉编译支持
    │
    └── cpython_mingw_v3.11.4.patch
        └── MinGW Windows 构建支持
```

## 文档导航

- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [03_Build_Integration.md](03_Build_Integration.md) - 构建适配说明
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中使用情况
- [05_API_Differences.md](05_API_Differences.md) - API 差异 (如有)
- [06_Security.md](06_Security.md) - 安全风险分析

## 参考链接

- [Python 官方文档](https://docs.python.org/3.11/)
- [Python 3.11 新特性](https://docs.python.org/3.11/whatsnew/3.11.html)
- [OpenHarmony 构建系统](https://gitee.com/openharmony/build)
