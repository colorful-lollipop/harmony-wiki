# API 差异分析

## 5.1 概述

### 5.1.1 差异状态

**结论**: meshoptimizer 在 OpenHarmony 中**不存在 API 差异**。

由于该库采用**零 Patch 集成策略**，上游代码被完整复制，未进行任何修改，因此：

| 差异类型 | 状态 | 说明 |
|---------|------|------|
| 新增 API | ❌ 无 | 未添加任何 OHOS 特有 API |
| 废弃 API | ❌ 无 | 未废弃任何上游 API |
| 修改 API | ❌ 无 | 未修改任何 API 签名 |
| 条件编译 | ❌ 无 | 未使用 `#ifdef OHOS` 等宏 |

### 5.1.2 差异原因

meshoptimizer 无需 API 修改的根本原因：

| 原因 | 说明 |
|------|------|
| 纯算法设计 | 所有功能通过纯数学计算实现 |
| 无平台绑定 | 不涉及文件系统、网络等平台相关操作 |
| 标准 C++ | 使用标准 C++ 接口，无平台特定 API |
| 完善的抽象 | API 设计已考虑跨平台需求 |

---

## 5.2 头文件导出

### 5.2.1 导出头文件

| 头文件 | 路径 | 状态 | 说明 |
|--------|------|------|------|
| `meshoptimizer.h` | `src/meshoptimizer.h` | ✅ 完整导出 | 主头文件，包含所有公开 API |

### 5.2.2 头文件引用方式

```cpp
// 标准引用方式
#include "meshoptimizer.h"

// 效果：等价于引用
// third_party/meshoptimizer/src/meshoptimizer.h
```

---

## 5.3 API 清单

### 5.3.1 核心解码 API（主要用途）

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_decodeVertexBuffer` | 解码压缩的顶点缓冲区 | ✅ 无差异 |
| `meshopt_decodeIndexBuffer` | 解码压缩的索引缓冲区 | ✅ 无差异 |
| `meshopt_decodeIndexSequence` | 解码索引序列 | ✅ 无差异 |

### 5.3.2 编码 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_encodeVertexBuffer` | 编码顶点缓冲区 | ✅ 无差异 |
| `meshopt_encodeIndexBuffer` | 编码索引缓冲区 | ✅ 无差异 |
| `meshopt_encodeIndexSequence` | 编码索引序列 | ✅ 无差异 |
| `meshopt_encodeVertexBufferBound` | 计算顶点缓冲区编码上界 | ✅ 无差异 |
| `meshopt_encodeIndexBufferBound` | 计算索引缓冲区编码上界 | ✅ 无差异 |

### 5.3.3 索引与重映射 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_generateVertexRemap` | 生成顶点重映射表 | ✅ 无差异 |
| `meshopt_remapIndexBuffer` | 重映射索引缓冲区 | ✅ 无差异 |
| `meshopt_remapVertexBuffer` | 重映射顶点缓冲区 | ✅ 无差异 |

### 5.3.4 顶点缓存优化 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_optimizeVertexCache` | 优化顶点缓存效率 | ✅ 无差异 |
| `meshopt_analyzeVertexCache` | 分析顶点缓存效率 | ✅ 无差异 |

### 5.3.5 过度绘制优化 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_optimizeOverdraw` | 减少过度绘制 | ✅ 无差异 |
| `meshopt_analyzeOverdraw` | 分析过度绘制情况 | ✅ 无差异 |

### 5.3.6 顶点提取优化 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_optimizeVertexFetch` | 优化顶点提取 | ✅ 无差异 |
| `meshopt_analyzeVertexFetch` | 分析顶点提取效率 | ✅ 无差异 |

### 5.3.7 量化 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_quantizeUnorm` | 无符号归一化量化 | ✅ 无差异 |
| `meshopt_quantizeSnorm` | 有符号归一化量化 | ✅ 无差异 |
| `meshopt_quantizeHalf` | 半精度浮点量化 | ✅ 无差异 |
| `meshopt_quantizeFloat` | 单精度浮点量化 | ✅ 无差异 |

### 5.3.8 简化 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_simplify` | 网格简化 | ✅ 无差异 |
| `meshopt_simplifySloppy` | 快速简化 | ✅ 无差异 |
| `meshopt_simplifyWithAttributes` | 属性感知简化 | ✅ 无差异 |
| `meshopt_simplifyPoints` | 点云简化 | ✅ 无差异 |

### 5.3.9 索引生成 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_generateShadowIndexBuffer` | 生成阴影索引缓冲区 | ✅ 无差异 |
| `meshopt_stripify` | 转换为三角形带 | ✅ 无差异 |
| `meshopt_unstripify` | 从三角形带还原 | ✅ 无差异 |

### 5.3.10 高级功能 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_buildMeshlets` | 构建 Meshlet 数据 | ✅ 无差异 |
| `meshopt_buildMeshletsBound` | 计算 Meshlet 构建上界 | ✅ 无差异 |
| `meshopt_computeMeshletBounds` | 计算 Meshlet 边界 | ✅ 无差异 |
| `meshopt_optimizeMeshlet` | 优化单个 Meshlet | ✅ 无差异 |

### 5.3.11 滤波编解码 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_encodeFilterOct` | 八面体滤波编码 | ✅ 无差异 |
| `meshopt_decodeFilterOct` | 八面体滤波解码 | ✅ 无差异 |
| `meshopt_encodeFilterQuat` | 四元数滤波编码 | ✅ 无差异 |
| `meshopt_decodeFilterQuat` | 四元数滤波解码 | ✅ 无差异 |
| `meshopt_encodeFilterExp` | 指数滤波编码 | ✅ 无差异 |
| `meshopt_decodeFilterExp` | 指数滤波解码 | ✅ 无差异 |

### 5.3.12 工具 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_spatialSortRemap` | 空间排序重映射 | ✅ 无差异 |
| `meshopt_generateAdjacencyIndexBuffer` | 生成邻接索引缓冲区 | ✅ 无差异 |
| `meshopt_generateTessellationIndexBuffer` | 生成细分索引缓冲区 | ✅ 无差异 |
| `meshopt_generateProvokingIndexBuffer` | 生成引发顶点索引缓冲区 | ✅ 无差异 |

### 5.3.13 内存管理 API

| API 函数 | 功能描述 | 状态 |
|---------|---------|------|
| `meshopt_setAllocator` | 设置自定义内存分配器 | ✅ 无差异 |

---

## 5.4 数据结构

### 5.4.1 公开数据结构

| 数据结构 | 描述 | 状态 |
|---------|------|------|
| `meshopt_VertexStats` | 顶点统计信息 | ✅ 无差异 |
| `meshopt_OverdrawStats` | 过度绘制统计 | ✅ 无差异 |
| `meshopt_FetchStats` | 顶点提取统计 | ✅ 无差异 |
| `meshopt_Meshlet` | Meshlet 数据结构 | ✅ 无差异 |
| `meshopt_Bounds` | 边界信息 | ✅ 无差异 |
| `meshopt_Stream` | 顶点流描述 | ✅ 无差异 |

### 5.4.2 常量定义

| 常量 | 描述 | 状态 |
|------|------|------|
| `meshopt_SimplifyLockBorder` | 简化锁定边界选项 | ✅ 无差异 |
| `meshopt_SimplifyErrorAbsolute` | 绝对误差选项 | ✅ 无差异 |
| `meshopt_SimplifySparse` | 稀疏简化选项 | ✅ 无差异 |
| `meshopt_SimplifyPrune` | 剪枝选项 | ✅ 无差异 |
| `meshopt_EncodeExpSeparate` | 指数编码模式 | ✅ 无差异 |
| `meshopt_EncodeExpSharedVector` | 指数编码共享向量模式 | ✅ 无差异 |
| `meshopt_EncodeExpSharedComponent` | 指数编码共享组件模式 | ✅ 无差异 |
| `meshopt_EncodeExpClamped` | 指数编码钳制模式 | ✅ 无差异 |

---

## 5.5 版本兼容性

### 5.5.1 编码版本控制

meshoptimizer 提供版本控制 API 以确保数据兼容性：

```cpp
// 设置编码版本（用于生成兼容旧版本的数据）
void meshopt_encodeVertexVersion(int version);
void meshopt_encodeIndexVersion(int version);
```

| 用途 | 说明 |
|------|------|
| 生成跨版本兼容数据 | 设置较低的版本号 |
| 使用最新编码 | 使用默认或最高版本 |

### 5.5.2 版本支持

| OHOS 版本 | 上游版本 | 编码版本兼容性 |
|-----------|---------|---------------|
| 6.1 | v0.22 | 完全兼容 |

---

## 5.6 使用建议

### 5.6.1 API 选择指南

根据您的使用场景，选择合适的 API：

| 场景 | 推荐 API | 说明 |
|------|---------|------|
| glTF 加载 | `meshopt_decode*` | 解码压缩数据 |
| 资源打包 | `meshopt_encode*` | 编码压缩数据 |
| 实时渲染 | `meshopt_optimize*` | 优化网格渲染 |
| LOD 生成 | `meshopt_simplify*` | 生成多级细节 |

### 5.6.2 迁移到 OHOS

如果您已有使用 meshoptimizer 的代码，迁移到 OHOS 几乎无需修改：

```cpp
// 原始代码（其他平台）
#include "meshoptimizer.h"

// OHOS 代码
#include "meshoptimizer.h"  // 完全相同

// API 调用完全一致
meshopt_decodeVertexBuffer(/* ... */);  // 无需修改
```

---

## 5.7 总结

### 差异总结

| 差异类型 | 数量 | 影响 |
|---------|------|------|
| 新增 API | 0 | 无 |
| 修改 API | 0 | 无 |
| 废弃 API | 0 | 无 |
| 条件编译 | 0 | 无 |

### 核心结论

1. **API 100% 兼容上游** - 无任何修改
2. **迁移成本为零** - 现有 meshoptimizer 代码可直接使用
3. **文档通用** - 上游文档完全适用于 OHOS
4. **无 OH 特有 API** - 无需学习新接口

---

*文档版本: v1.0*
*最后更新: 2026-02-07*
