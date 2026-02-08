# 03_Build_Integration - 构建系统适配

## 3.1 构建架构概览

### 构建系统选择

| 项目 | 选择 | 说明 |
|------|------|------|
| **主构建系统** | GN (Ohos.gni) | OHOS 官方构建系统 |
| **上游构建系统** | CMake | Google Benchmark 原生构建系统 |
| **构建兼容性** | ✅ 完整支持 | GN 配置完整映射 CMake 功能 |

### 构建流程

```
┌─────────────────────────────────────────────────────────────────┐
│                     OpenHarmony 构建流程                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   source code (v1.9.4)                                          │
│         │                                                       │
│         ▼                                                       │
│   ┌─────────────┐                                               │
│   │  BUILD.gn   │  ◄── OHOS GN 构建配置                        │
│   │  (OH 适配)  │                                               │
│   └─────────────┘                                               │
│         │                                                       │
│         ▼                                                       │
│   ┌─────────────┐                                               │
│   │  ohos_     │                                               │
│   │ static_     │  ◄── 生成静态库 libbenchmark.a                │
│   │  library    │                                               │
│   └─────────────┘                                               │
│         │                                                       │
│         ▼                                                       │
│   ┌─────────────┐                                               │
│   │  benchmark │                                               │
│   │   test     │  ◄── 被各模块的 benchmarktest 链接             │
│   └─────────────┘                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 3.2 BUILD.gn 配置详解

### 文件位置

```
third_party/benchmark/
├── BUILD.gn                    # OH 构建配置（主文件）
├── CMakeLists.txt             # 上游构建配置（保留）
└── include/
    └── benchmark/             # 公共头文件
```

### BUILD.gn 完整内容

```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/ohos.gni")

# 公共头文件配置 - 供外部模块引用
config("benchmark_config") {
  include_dirs = [ "include" ]
}

# 内部构建配置 - 仅本库使用
config("benchmark_private_config") {
  visibility = [ ":*" ]
  include_dirs = [ "include" ]
}

# 核心 benchmark 静态库
ohos_static_library("benchmark") {
  sources = [
    "src/benchmark.cc",
    "src/benchmark_api_internal.cc",
    "src/benchmark_name.cc",
    "src/benchmark_register.cc",
    "src/benchmark_runner.cc",
    "src/check.cc",
    "src/colorprint.cc",
    "src/commandlineflags.cc",
    "src/complexity.cc",
    "src/console_reporter.cc",
    "src/counter.cc",
    "src/csv_reporter.cc",
    "src/json_reporter.cc",
    "src/perf_counters.cc",
    "src/reporter.cc",
    "src/statistics.cc",
    "src/string_util.cc",
    "src/sysinfo.cc",
    "src/timers.cc",
  ]

  public_configs = [ ":benchmark_config" ]
  configs = [ ":benchmark_private_config" ]
}

# benchmark_main 简化库（自动生成 main 函数）
ohos_static_library("benchmark_main") {
  include_dirs = [ "include" ]

  sources = [ "src/benchmark_main.cc" ]

  deps = [ ":benchmark" ]
}
```

## 3.3 配置项详解

### 关键配置参数

| 配置项 | 类型 | 说明 |
|--------|------|------|
| **`include_dirs`** | list | 头文件搜索路径，指向 `include/` |
| **`sources`** | list | 源文件列表，共 20 个文件 |
| **`public_configs`** | list | 对外可见的构建配置 |
| **`configs`** | list | 内部构建配置 |
| **`deps`** | list | 依赖关系 |

### 头文件导出结构

```
include/
└── benchmark/
    ├── benchmark.h           # 主头文件，包含所有宏定义
    ├── benchmark_api.h       # API 接口声明
    ├── benchmark_register.h  # 测试注册接口
    └── export.h              # 导出声明
```

### 头文件导出配置 (bundle.json)

```json
"inner_kits": [
    {
        "name":"//third_party/benchmark:benchmark",
        "header":{
            "header_files":[
                "benchmark.h",
                "export.h"
            ],
            "header_base":"//third_party/benchmark/include"
        }
    }
]
```

**导出路径**: `//third_party/benchmark:benchmark`

## 3.4 编译选项对比

### OH vs 上游 (CMake)

| 功能 | OH (BUILD.gn) | 上游 (CMake) | 差异 |
|------|---------------|--------------|------|
| **静态库类型** | ohos_static_library | add_library(static) | ✅ 等价 |
| **头文件路径** | include_dirs | target_include_directories | ✅ 等价 |
| **源文件** | sources | srcs | ✅ 等价 |
| **依赖声明** | deps | target_link_libraries | ✅ 等价 |
| **配置可见性** | visibility | N/A | OH 特有 |
| **公共配置** | public_configs | PUBLIC | ✅ 等价概念 |

### OH 特有配置

```gn
# 配置可见性控制 - 仅本 target 可见
config("benchmark_private_config") {
  visibility = [ ":*" ]  # 对所有模块隐藏
}

# 公共配置 - 链接时自动传递
public_configs = [ ":benchmark_config" ]
```

## 3.5 构建产物

### 生成的静态库

| 产物 | 目标 | 说明 |
|------|------|------|
| **libbenchmark.a** | benchmark | 核心 benchmark 库 |
| **libbenchmark_main.a** | benchmark_main | 含 main 函数的简化库 |

### 链接方式

```gn
# 方式 1: 直接链接 benchmark
ohos_benchmarktest("my_test") {
    sources = [ "test.cpp" ]
    deps = [ "//third_party/benchmark" ]
}

# 方式 2: 使用 benchmark_main (无需手动 BENCHMARK_MAIN)
ohos_benchmarktest("my_test") {
    sources = [ "test.cpp" ]
    deps = [ "//third_party/benchmark:benchmark_main" ]
}
```

## 3.6 常见构建问题

### 问题 1: 头文件找不到

**症状**:
```
fatal error: 'benchmark/benchmark.h' file not found
```

**解决方案**:
```gn
ohos_benchmarktest("my_test") {
    sources = [ "test.cpp" ]
    # 确保 benchmark_config 被应用
    configs = [ "//third_party/benchmark:benchmark_config" ]
}
```

### 问题 2: 符号未定义

**症状**:
```
undefined reference to 'benchmark::State::range(int)'
```

**解决方案**:
```gn
ohos_benchmarktest("my_test") {
    sources = [ "test.cpp" ]
    # 链接完整的 benchmark 库
    deps = [ "//third_party/benchmark" ]
}
```

## 3.7 构建性能优化

### 增量构建

Benchmark 库支持 OH 的增量构建机制:
- 修改源文件仅重新编译受影响的部分
- 头文件变更触发相关源文件重编

### 分布式构建

支持 OH 分布式构建系统:
- 多线程并行编译
- 缓存中间产物

### 构建时间参考

| 操作 | 预计时间 |
|------|----------|
| **完整构建** | ~30 秒 (Release) |
| **增量构建** | ~5 秒 |
| **清理构建** | ~40 秒 |

## 3.8 构建系统兼容性矩阵

| OH 系统类型 | 兼容性 | 备注 |
|-------------|--------|------|
| **small** | ✅ 支持 | 轻量设备 |
| **standard** | ✅ 支持 | 标准设备 |
| **full** | ✅ 支持 | 全功能设备 |

## 3.9 与上游构建的差异总结

| 差异维度 | OH 实现 | 上游实现 | 说明 |
|----------|---------|----------|------|
| **构建系统** | GN | CMake | OH 标准选择 |
| **库类型** | 静态库 | 静态库 + 动态库 | OH 仅使用静态库 |
| **配置可见性** | visibility | N/A | GN 特有特性 |
| **配置传递** | public_configs | PUBLIC | 概念等价 |
| **输出位置** | OH out目录 | build/ | 构建系统差异 |
