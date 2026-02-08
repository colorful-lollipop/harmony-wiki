# 01_Overview.md - 原始库简介与 OH 定位

## 1. 原始库基本信息

### 1.1 概述

**astc-encoder**（astcenc）是 ARM 官方提供的 ASTC（Adaptive Scalable Texture Compression）纹理压缩编解码器参考实现。

| 属性 | 详情 |
|------|------|
| **全称** | Arm Adaptive Scalable Texture Compression Encoder |
| **版本** | 4.7.0 |
| **许可证** | Apache 2.0 License |
| **上游仓库** | https://github.com/ARM-software/astc-encoder.git |
| **维护者** | Arm Limited and contributors |
| **OH 维护者** | wangyonglang@huawei.com |

### 1.2 ASTC 格式简介

ASTC 是由 **ARM 和 AMD** 联合开发的纹理压缩标准：

- **标准地位**：已被采纳为 OpenGL、OpenGL ES 的官方 Khronos 扩展，Vulkan 的标准可选功能
- **技术优势**：在相同比特率下提供更好的图像质量，支持灵活的格式和比特率选择
- **比特率范围**：0.89 bits/pixel 至 8 bits/pixel
- **块大小支持**：支持所有 ASTC 规范定义的块大小（4x4 到 12x12）

### 1.3 核心功能

#### 压缩能力
- **输入格式**：BMP、JPEG、PNG、TGA（LDR）；EXR、HDR（HDR）
- **容器格式**：支持 DDS、KTX 容器
- **输出格式**：ASTC、KTX
- **配置文件**：
  - LDR Profile：2D 低动态范围
  - HDR Profile：2D LDR + HDR
  - Full Profile：2D/3D + LDR/HDR

#### 质量预设
| 预设 | 速度 | 质量 | 适用场景 |
|------|------|------|---------|
| `-fastest` | 最快 | 较低 | 快速预览 |
| `-fast` | 快 | 中等 | 开发调试 |
| `-medium` | 中等 | 良好 | 常规使用 |
| `-thorough` | 慢 | 很好 | 生产发布 |
| `-verythorough` | 很慢 | 优秀 | 高质量需求 |
| `-exhaustive` | 极慢 | 最优 | 极致质量 |

#### SIMD 优化
- x86-64：SSE2、SSE4.1、AVX2
- ARM：NEON

---

## 2. OpenHarmony 中的定位

### 2.1 引入目的

根据 `README_zh.md`：

> OpenHarmony 上引入 ASTC 主要用于**图库缩略图**和**其他应用预置图**的压缩。ASTC 码流可以直接由 GPU 解码显示，降低传输数据量和 CPU 解码耗时。

### 2.2 系统架构位置

```
应用层
    ↓
图像框架 (image_framework)
    ├── 接口层 (interfaces/innerkits) ← 使用 astc-encoder
    ├── 插件层 (plugins/libextplugin)  ← 使用 astc-encoder
    └── 实现层 (frameworks/innerkitsimpl)
    ↓
astc-encoder (third_party)
    ↓
GPU 驱动（直接解码 ASTC）
```

### 2.3 核心价值

| 价值点 | 说明 |
|--------|------|
| **降低 CPU 负载** | GPU 直接解码 ASTC，无需 CPU 参与 |
| **减少内存占用** | 压缩纹理占用更少显存/内存 |
| **减少带宽** | 传输压缩数据，减少 I/O 带宽 |
| **节省电量** | 内存带宽降低带来功耗节省 |

---

## 3. 代码结构

### 3.1 目录组织

```
astc-encoder/
├── Docs/                    # 文档
│   ├── FormatOverview.md    # ASTC 格式概述
│   ├── Encoding.md          # 编码指南
│   ├── ChangeLog-4x.md      # 4.x 版本变更日志
│   └── ...
├── Source/                  # 源代码（核心）
│   ├── astcenc.h            # 公开 API 头文件
│   ├── astcenc_*.cpp        # 核心编解码器源文件（22 个）
│   ├── astcenccli_*.cpp     # CLI 工具源文件
│   ├── UnitTest/            # 单元测试
│   └── Fuzzers/             # Fuzz 测试
├── Test/                    # 测试图像和数据
├── Utils/                   # 工具脚本
├── CMakeLists.txt           # 上游构建配置
├── BUILD.gn                 # OH 构建配置 ⭐
├── bundle.json              # OH 组件配置 ⭐
├── README.OpenSource        # OH 开源说明 ⭐
└── LICENSE.txt              # 许可证
```

### 3.2 核心源文件

| 文件名 | 功能 |
|--------|------|
| `astcenc.h` | 公开 API 头文件 |
| `astcenc_entry.cpp` | 入口点和上下文管理 |
| `astcenc_compress_symbolic.cpp` | 压缩核心算法 |
| `astcenc_decompress_symbolic.cpp` | 解压核心算法 |
| `astcenc_image.cpp` | 图像数据结构 |
| `astcenc_mathlib.cpp` | 数学库 |
| `astcenc_block_sizes.cpp` | 块大小处理 |
| `astcenc_color_quantize.cpp` | 颜色量化 |
| `astcenc_weight_align.cpp` | 权重对齐 |

### 3.3 OH 特有文件

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | GN 构建配置，定义静态库和共享库目标 |
| `bundle.json` | OH 组件元数据 |
| `README.OpenSource` | OH 开源合规说明 |
| `Source/astcenccli_platform_dependents.cpp` | 平台相关实现（线程、时间） |

---

## 4. 版本信息

### 4.1 当前版本

- **版本号**：4.7.0
- **发布时间**：2024年1月
- **状态**：稳定维护版本

### 4.2 4.7.0 主要变更

根据上游 ChangeLog：

- **Bug 修复**：修复了解压器舍入行为以匹配 Khronos 规范
- **新功能**：增加 `ASTCENC_FLG_USE_DECODE_UNORM8` 标志支持
- **新功能**：命令行工具支持 `-decode_unorm8` 选项
- **新功能**：支持进度报告回调

### 4.3 版本策略

- **主分支**（main）：活跃开发分支，用于最新主要版本系列
- **4.x 分支**：当前稳定分支（本库使用）
- **3.x 分支**：旧版本分支，仅接受 Bug 修复
- **1.x/2.x**：已停止维护

---

## 5. 许可证合规

### 5.1 许可证类型

**Apache License 2.0**

### 5.2 合规要点

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 许可证文件 | ✅ | LICENSE.txt 存在 |
| 源码头声明 | ✅ | 源文件包含 SPDX 标识 |
| 与 OH 兼容 | ✅ | Apache 2.0 完全兼容 |
| 专利授权 | ✅ | Apache 2.0 包含专利授权条款 |

### 5.3 版权声明示例

源文件头部标准格式：
```cpp
// SPDX-License-Identifier: Apache-2.0
// ----------------------------------------------------------------------------
// Copyright 2020-2024 Arm Limited
//
// Licensed under the Apache License, Version 2.0 (the "License"); ...
```

---

## 6. 相关资源

### 6.1 官方文档
- [ASTC Developer Guide](https://developer.arm.com/documentation/102162/latest/)
- [Khronos ASTC 规范](https://www.khronos.org/registry/DataFormat/specs/1.3/dataformat.1.3.html#ASTC)
- [GitHub Releases](https://github.com/ARM-software/astc-encoder/releases)

### 6.2 OH 内部参考
- `//third_party/astc-encoder/BUILD.gn`
- `//third_party/astc-encoder/bundle.json`
- `//foundation/multimedia/image_framework/`（主要使用者）

### 6.3 相关标准
- OpenGL ES 3.2（ASTC 为必需扩展）
- Vulkan 1.0+（ASTC 为可选功能）

---

## 7. 总结

astc-encoder 是一个**成熟、稳定、高质量**的纹理压缩库：

1. **行业标准**：ASTC 为 Khronos 官方标准，广泛支持
2. **开源友好**：Apache 2.0 许可证，与 OH 完全兼容
3. **设计良好**：模块化架构，易于集成
4. **性能优秀**：多 SIMD 优化级别，多线程支持
5. **OH 集成简洁**：无 Patch，通过 BUILD.gn 完成适配

**在 OpenHarmony 中的关键价值**：为图像框架提供高效的 GPU 可解码纹理压缩能力，显著降低系统资源消耗。
