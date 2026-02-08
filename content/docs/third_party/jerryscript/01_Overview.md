# JerryScript 原始库简介

## 1. 库基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | JerryScript |
| **许可证** | Apache License 2.0 |
| **OH 版本** | v2.3.0 |
| **上游地址** | https://github.com/jerryscript-project/jerryscript.git |
| **OH 维护者** | pengbiao1@huawei.com |
| **所属组织** | JS Foundation |

## 2. 核心功能概述

JerryScript 是一个**超轻量级 JavaScript 引擎**，专为资源极度受限的嵌入式设备设计。

### 关键特性

- **极低内存占用**: 可在 <64KB RAM 和 <200KB Flash 的设备上运行
- **标准兼容**: 完整支持 ECMAScript 5.1 标准
- **紧凑二进制**: ARM Thumb-2 编译后仅约 160KB
- **C99 实现**: 最大程度保证可移植性
- **快照支持**: 可将 JavaScript 源码预编译为字节码
- **成熟 C API**: 易于嵌入应用程序

## 3. 技术架构

### 模块组成

```
JerryScript
├── jerry-core          # 核心引擎（解析器、字节码 VM、ECMA 实现）
├── jerry-ext           # 扩展模块（调试器、句柄作用域、参数处理等）
├── jerry-port          # 平台移植层（日期、I/O、调试器、内存管理等）
├── jerry-libm          # 内嵌数学库（替代系统 libm）
├── jerry-debugger      # 调试器实现
└── jerry-main          # 命令行工具
```

### 核心模块说明

| 模块 | 功能 |
|------|------|
| **jerry-core** | 核心 ECMAScript 引擎，包含解析器、虚拟机、垃圾回收 |
| **jerry-ext** | 提供扩展功能，如调试器传输、句柄作用域、参数验证工具 |
| **jerry-port** | 平台适配层，需要针对不同平台实现 I/O、内存等底层功能 |
| **jerry-libm** | 自包含的数学库，用于不支持标准 libm 的嵌入式系统 |
| **jerry-debugger** | 支持 TCP 和 WebSocket 的远程调试协议 |

## 4. 在 OpenHarmony 中的定位

### 核心作用

JerryScript 是 OpenHarmony 轻量级系统的**JavaScript 运行时核心**，主要服务于：

1. **轻量级 UI 引擎** (ace_engine_lite)
   - 执行 JS 应用代码
   - 提供 JS 与原生代码的绑定

2. **包管理服务** (bundle_framework_lite)
   - JS 字节码预编译
   - 应用包解析和处理

3. **网络代理管理** (netmanager_base)
   - 执行 PAC (Proxy Auto-Configuration) 代理脚本
   - 动态路由决策

### OH 特性支持

OpenHarmony 对 JerryScript 进行了扩展以支持以下 OH 特性：

| 特性 | 说明 |
|------|------|
| **ES2015 支持** | 部分支持 ES6/ES2015 特性（Promise, Proxy, TypedArray 等） |
| **外部上下文** | 支持多任务环境下的上下文切换 |
| **GC 控制** | 提供 EnableGC/DisableGC API，支持关键时段暂停 GC |
| **内存优化** | 2023 年引入字符串字面量缓存、小对象缓存等优化 |
| **快照增强** | 增强的快照保存/执行能力，支持版本检查 |
| **调试器扩展** | 支持 IDE 调试器接入 (ACE_DEBUGGER_CUSTOM) |

### 适配平台

JerryScript 在 OpenHarmony 中支持多种配置：

| 平台类型 | 内核 | 构建模式 | 主要目标 |
|----------|------|----------|----------|
| **轻量系统** | LiteOS-M | 静态链接 (lite_library) | IP Camera 等微控制器设备 |
| **轻量系统** | LiteOS-A | 共享库 (shared_library) | 小型 IoT 设备 |
| **标准系统** | Linux | 共享库/静态库 | 开发板、模拟器 |

## 5. 与其他 JS 引擎的对比

| 特性 | JerryScript | QuickJS | V8 |
|------|-------------|---------|-----|
| **内存占用** | <64KB | ~200KB | >1MB |
| **二进制大小** | ~160KB | ~300KB | >1MB |
| **标准支持** | ES5.1 (部分 ES6) | ES2020+ | ES2020+ |
| **嵌入式适用性** | ✅ 极佳 | ✅ 优秀 | ❌ 不适合 |
| **调试支持** | ✅ 有 | ✅ 有 | ✅ 强大 |

**JerryScript 的选择理由**:
- 超低内存占用适合 OH 轻量级系统
- 成熟稳定的 C API
- 活跃的开源社区
- Apache 2.0 许可证（OH 首选）

## 6. 性能特点

### 优化方向

1. **内存优先**: 所有关键数据结构都经过内存占用优化
2. **启动速度**: 快速初始化，适合实时系统
3. **字节码**: 支持预编译，消除解析开销
4. **快照**: 可将编译后的字节码持久化，加速应用加载

### 性能指标（官方数据）

- **ARM Thumb-2**: 160KB 二进制，<64KB RAM
- **x86-64**: ~180KB 二进制，<64KB RAM
- **启动时间**: <10ms（空引擎）
- **JS 执行速度**: 适合轻量级应用，非高性能场景

## 7. 上游社区

- **GitHub**: https://github.com/jerryscript-project/jerryscript
- **项目页面**: http://jerryscript.net
- **Wiki**: https://github.com/jerryscript-project/jerryscript/wiki
- **Mailing List**: jerryscript-dev@groups.io
- **IRC**: #jerryscript on freenode

### 发布周期

上游 JerryScript 相对活跃，通常每 2-3 个月发布一个版本。OpenHarmony 使用 v2.3.0 版本（2020 年），后续版本有多个安全修复和性能改进。

---

## 8. OpenHarmony 中的定制化概述

OpenHarmony 对 JerryScript 的修改主要集中在：

1. **构建系统**: 完全替换为 GN/Ninja，支持 OH 组件化
2. **内存管理**: 2023 年引入堆分配简化和字符串优化
3. **平台适配**: JUPITER/IAR 平台专门适配
4. **GC 增强**: 支持动态开关 GC，递归深度限制
5. **安全加固**: 默认禁用 eval()，CVE 修复

详细内容请参阅：
- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [03_Build_Integration.md](03_Build_Integration.md) - 构建适配详情
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景
