# meshoptimizer - OpenHarmony 第三方库文档

## 库概览

**meshoptimizer** 是一个高效的网格优化库，通过多种技术手段减少 3D 网格数据的存储大小并提升渲染性能。该库主要用于图形学和游戏开发领域，特别是在处理包含大量多边形的 3D 模型时，可以显著降低渲染负担并提高运行效率。

### 基础信息

| 属性 | 值 |
|------|-----|
| **库名称** | meshoptimizer |
| **上游版本** | v0.22 |
| **OH 版本** | 6.1 |
| **许可证** | MIT License |
| **上游地址** | https://github.com/zeux/meshoptimizer |
| **所属子系统** | thirdparty |
| **ROM 大小** | 125KB |
| **RAM 大小** | 938KB |

### OpenHarmony 适配概述

meshoptimizer 在 OpenHarmony 中的集成**极为简洁**：

- ✅ **无任何 Patch**：上游版本原生代码可直接使用，无需针对 OHOS 进行代码修改
- ✅ **标准 BUILD.gn 配置**：只需通过 GN 构建系统配置即可完成集成
- ✅ **纯算法实现**：完全跨平台，不依赖任何操作系统特定 API
- 🎯 **主要用途**：支持 glTF 格式的 EXT_meshopt_compression 扩展解码

---

## 文档导航

### 核心文档

| 文档 | 描述 | 必读 |
|------|------|------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议和文档结构 | ✅ 是 |
| [01_Overview.md](./01_Overview.md) | 库功能概述和 OH 定位 | ✅ 是 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（无 Patch 情况说明） | ✅ 是 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统集成详解 | ⚠️ 按需 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 在 OH 中的使用方式和场景 | ⚠️ 按需 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 | ❌ 否 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | ⚠️ 按需 |

### 工作文档

| 文档 | 描述 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估报告（内部工作文档） |

---

## 快速开始

### 依赖声明

在模块的 BUILD.gn 文件中添加依赖：

```gn
deps = ["//third_party/meshoptimizer:meshoptimizer"]
```

### 头文件引用

```cpp
#include "meshoptimizer.h"
```

### 基础解码示例

meshoptimizer 在 OH 中的主要用途是解码 glTF 的 EXT_meshopt_compression 扩展：

```cpp
// 解码顶点缓冲区
int result = meshopt_decodeVertexBuffer(
    vertices,    // 目标顶点缓冲区
    vertex_count, // 顶点数量
    vertex_size,  // 顶点大小（字节）
    encoded_data, // 编码后的数据
    encoded_size  // 编码数据大小
);

// 解码索引缓冲区
int result = meshopt_decodeIndexBuffer(
    indices,     // 目标索引缓冲区
    index_count,  // 索引数量
    encoded_data, // 编码后的数据
    encoded_size  // 编码数据大小
);
```

---

## 技术特性

### 核心优化算法

1. **顶点缓存优化 (Vertex Cache Optimization)**
   - 重新排序三角形以最大化 GPU 顶点缓存命中率
   - 显著减少顶点着色器重复调用

2. **过度绘制优化 (Overdraw Optimization)**
   - 重新排序三角形以减少不必要的像素着色器调用
   - 提升渲染效率

3. **顶点提取优化 (Vertex Fetch Optimization)**
   - 优化顶点缓冲区布局以提高内存访问效率
   - 减少带宽消耗

4. **顶点量化 (Vertex Quantization)**
   - 将高精度浮点数转换为低精度表示
   - 显著减少内存占用

5. **网格简化 (Mesh Simplification)**
   - 生成多级细节 (LOD)
   - 在保持视觉质量的同时减少多边形数量

6. **缓冲压缩 (Buffer Compression)**
   - 专有的顶点/索引压缩算法
   - 解码性能高达 1-3 GB/s

### 性能指标

| 指标 | 典型值 |
|------|-------|
| 顶点压缩率 | 2-4x（相比量化后数据） |
| 索引压缩率 | 5-6x（相比原始 16 位索引） |
| 解码性能 | 1-3 GB/s |
| ROM 占用 | 125KB |
| RAM 占用 | 938KB |

---

## 在 OpenHarmony 中的定位

### 当前状态

meshoptimizer **已被集成到 OHOS thirdparty 子系统**，但**尚未被其他模块显式依赖**。该库目前处于"备用就绪"状态，等待需要 3D 网格优化的模块进行引用。

### 预期使用场景

根据 OpenHarmony 的技术规划，meshoptimizer 预期将在以下场景中发挥作用：

1. **3D 模型渲染模块**
   - 高效加载和渲染复杂 3D 模型
   - 支持 glTF 格式的高效传输

2. **图形引擎集成**
   - 作为底层优化库供图形引擎调用
   - 支持实时 LOD 生成和切换

3. **资源管理系统**
   - 优化 3D 资源的存储和传输
   - 降低应用包体积

### 目标开发者

- 系统应用开发者
- 3D 图形引擎相关开发者
- 需要处理 3D 模型渲染的模块维护者

---

## 版本信息

### 上游版本 v0.22 发布说明

meshoptimizer v0.22 是 OHOS 集成的上游基准版本，包含以下主要特性：

- 完整的网格优化算法套件
- 高性能编解码器
- Mesh Shading 支持
- 高级压缩滤波器
- WebAssembly 绑定

### OHOS 版本差异

| OHOS 版本 | 说明 |
|----------|------|
| 6.1 | 当前 OHOS 集成的版本号，对应上游 v0.22 |

---

## 相关资源

### 官方资源

- **上游仓库**: https://github.com/zeux/meshoptimizer
- **gltfpack 工具**: https://github.com/zeux/meshoptimizer/releases
- **glTF EXT_meshopt_compression 规范**: https://github.com/KhronosGroup/glTF/blob/main/extensions/2.0/Vendor/EXT_meshopt_compression/README.md

### OpenHarmony 相关

- **主干代码获取**: 参考 [README_zh.md](../README_zh.md)
- **构建方法**: 参考 [README_zh.md](../README_zh.md)
- **开发者指南**: 参考 [README_zh.md](../README_zh.md)

---

## 贡献与维护

### 维护状态

| 状态 | 说明 |
|------|------|
| ✅ 已集成 | 库已成功集成到 OHOS 构建系统 |
| ⏳ 待使用 | 等待其他模块进行依赖引用 |
| 🔄 可升级 | 上游有更新时可考虑同步升级 |

### 升级建议

由于该库当前无 Patch，建议在后续升级时：
1. 优先考虑直接同步上游代码
2. 验证 OHOS 构建兼容性
3. 仅在发现 OHOS 特定问题时才考虑添加 Patch

---

*文档版本: v1.0*
*最后更新: 2026-02-07*
