# FlatBuffers 概述

## 1.1 原始库简介

### 1.1.1 什么是 FlatBuffers

FlatBuffers 是由 Google 开发的高性能跨平台序列化库，其核心设计理念是**零拷贝（Zero-Copy）数据访问**。与传统序列化库（如 JSON、Protocol Buffers、MessagePack）不同，FlatBuffers 在序列化数据时即构建完整的内存结构，读取时无需解析或解包操作，直接通过偏移量访问数据。

#### 核心特性

- **内存效率优先**: 数据以平面二进制格式存储，无解析开销
- **直接访问**: 支持在不解包整个数据结构的情况下访问特定字段
- **强类型安全**: 通过 Schema 定义数据结构，编译时类型检查
- **向前向后兼容**: 字段添加、删除不影响旧版本数据的读取
- **跨平台支持**: 支持 C++、C#、Java、JavaScript、Go、Rust、Python 等 15+ 语言
- **多格式支持**: 既可作为序列化库，也可作为内存数据访问层

### 1.1.2 工作原理

FlatBuffers 的工作流程分为两个主要阶段：

#### 序列化阶段

1. **Schema 定义**: 使用 `.fbs` 文件定义数据结构（类似 Protocol Buffers）
2. **代码生成**: 使用 `flatc` 编译器生成目标语言的访问代码
3. **构建 Buffer**: 使用 `FlatBufferBuilder` 在内存中构造二进制数据
4. **完成编码**: 调用 `Finish()` 完成编码，生成可直接传输的二进制数据

#### 反序列化阶段

1. **读取 Buffer**: 直接访问二进制数据，无需解析
2. **字段访问**: 通过生成的访问器函数直接读取数据
3. **按需解析**: 只有在需要访问嵌套数据时才进行部分解析

### 1.1.3 典型应用场景

| 场景 | 优势 |
|-----|------|
| 游戏开发 | 快速数据加载，减少内存占用 |
| AI/ML 模型传输 | 高效模型文件加载，支持增量更新 |
| 跨语言数据交换 | 多语言支持，零拷贝访问 |
| 网络数据传输 | 二进制格式紧凑，解析开销低 |
| 配置文件 | Schema 约束，类型安全 |

---

## 1.2 OpenHarmony 中的定位

### 1.2.1 系统角色

FlatBuffers 在 OpenHarmony 系统中承担**高性能序列化基础设施**的角色，主要服务于 AI 推理和神经网络相关模块。作为第三方库，它被深度集成到以下核心子系统：

```
OpenHarmony 系统
├── AI 子系统
│   ├── Neural Network Runtime (NNRT)
│   │   └── 使用 FlatBuffers 进行模型数据序列化
│   └── MindSpore Lite
│       └── 使用 FlatBuffers 解析 MindIR 模型文件
└── 第三方库基础设施
    └── FlatBuffers (@ohos/flatbuffers)
```

### 1.2.2 选型理由

OpenHarmony 选择 FlatBuffers 作为 AI 模型序列化的主要原因：

#### 性能优势

| 指标 | FlatBuffers | Protobuf | JSON |
|-----|------------|----------|------|
| 解析速度 | 极快（零拷贝） | 快 | 慢 |
| 内存占用 | 低 | 中 | 高 |
| 数据大小 | 小 | 中 | 大 |
| 访问灵活性 | 高 | 高 | 中 |

#### AI 场景契合

- **模型加载**: AI 模型文件通常较大（数十 MB 至数 GB），FlatBuffers 的零拷贝特性可显著缩短加载时间
- **增量更新**: 支持部分字段更新，适合模型微调和在线学习场景
- **跨语言互操作**: 模型训练（Python）、推理（C++）需要跨语言数据交换
- **硬件加速器通信**: 与 NPU 通信需要高效的数据格式

### 1.2.3 版本与上游同步

| 属性 | 信息 |
|-----|------|
| **当前版本** | v25.2.10 |
| **上游版本** | v25.2.10 |
| **同步状态** | 完全同步 |
| **上游地址** | https://github.com/google/flatbuffers/archive/refs/tags/v25.2.10.tar.gz |
| **License** | Apache-2.0 |

OpenHarmony 保持与上游同版本同步，仅进行必要的适配修改，未进行功能性裁剪或增强。

---

## 1.3 目录结构分析

### 1.3.1 整体结构

```
third_party/flatbuffers/
├── .github/                 # GitHub 配置（CI/CD）
├── CMake/                   # CMake 构建配置
├── bazel/                   # Bazel 构建配置
├── include/                 # 头文件目录
│   └── flatbuffers/        # FlatBuffers 核心头文件
├── src/                     # 源码目录
├── tests/                   # 测试用例
├── samples/                 # 示例程序
├── grpc/                   # gRPC 集成（包含 Patch）
├── cangjie/                # OH 特有：仓颉语言绑定
├── CMakeLists.txt          # 上游 CMake 构建文件
├── BUILD.bazel             # 上游 Bazel 构建文件
├── BUILD.gn               # OH GN 构建适配
├── bundle.json             # OH 组件配置
├── README.OpenSource       # OH 开源声明
└── LICENSE.txt             # Apache-2.0 许可证
```

### 1.3.2 OH 特有内容

#### cangjie/ 目录

`cangjie/` 目录是 OpenHarmony 特有的仓颉（Cangjie）编程语言绑定，包含以下文件：

| 文件 | 功能 |
|-----|------|
| `decode.cj` | FlatBuffer 数据解码器实现 |
| `flatbuffer_object.cj` | 仓颉语言的 FlatBuffer 对象封装 |
| `table.cj` | Table 数据结构支持 |
| `builder.cj` | FlatBuffer 构建器支持 |
| `constants.cj` | 常量定义 |
| `exception.cj` | 异常处理机制 |
| `test/` | 测试用例和测试脚本 |

**说明**: 仓颉是华为自研的编程语言，正在逐步融入 OpenHarmony 生态。通过仓颉绑定，开发者可以使用仓颉语言直接访问和创建 FlatBuffers 序列化的数据。

#### grpc/ 目录

`grpc/` 目录包含 gRPC 集成的测试代码和 Patch 文件：

| 文件 | 类型 | 说明 |
|-----|------|------|
| `build_grpc_with_cxx14.patch` | Patch | C++14 标准修复 |
| `boringssl.patch` | Patch | BoringSSL 链接修复 |
| `README.md` | 文档 | gRPC 集成说明 |

### 1.3.3 关键源文件

FlatBuffers 的核心功能由以下模块实现：

| 模块 | 路径 | 功能 |
|-----|------|------|
| **FlatBufferBuilder** | `src/flatbuffers.cpp` | 二进制 Buffer 构建器 |
| **代码生成器** | `src/idl_parser.cpp` | Schema 解析器 |
| **代码生成器** | `src/idl_gen_text.cpp` | 文本格式生成 |
| **Reflection** | `reflection/` | 运行时反射支持 |

---

## 1.4 在 OH 生态中的位置

### 1.4.1 依赖关系

FlatBuffers 作为底层基础设施库，被以下 OH 核心模块依赖：

```
flatbuffers (第三方库)
    │
    ├── AI 子系统
    │   ├── MindSpore Lite ─────┐
    │   └── Neural Network Runtime
    │                          │
    └── 依赖链                  ▼
                    AI 模型数据序列化和反序列化
```

### 1.4.2 数据流向

#### MindSpore Lite 场景

```
训练环境 (Python)
    │
    ▼ 生成 MindIR
MindIR 文件 (FlatBuffers 格式)
    │
    ▼ 传输到设备
OpenHarmony 设备
    │
    ├── MindSpore Lite 加载
    │       │
    │       ▼
    │   FlatBuffers 反序列化
    │       │
    │       ▼
    └── AI 推理引擎
```

#### Neural Network Runtime 场景

```
模型服务器
    │
    ▼ 传输模型定义
FlatBuffers 序列化模型
    │
    ▼
NNRT (Neural Network Runtime)
    │
    ├── 反序列化获取模型结构
    ├── 调度执行
    └── 返回结果
```

---

## 1.5 小结

FlatBuffers 作为 Google 官方维护的高性能序列化库，在 OpenHarmony 生态中扮演着关键的 AI 数据基础设施角色。通过提供零拷贝序列化能力，它有效支撑了 MindSpore Lite 和 Neural Network Runtime 等核心 AI 模块的高效运行。

OpenHarmony 对 FlatBuffers 的集成策略是**最小化修改**，仅进行必要的构建系统适配和 OH 特有功能扩展（如仓颉语言支持），这种策略有利于后续与上游版本的同步和长期维护。

---

## 参考资料

- [FlatBuffers 官方文档](https://google.github.io/flatbuffers/)
- [FlatBuffers GitHub](https://github.com/google/flatbuffers)
- [OpenHarmony 第三方库规范](../README.md)
