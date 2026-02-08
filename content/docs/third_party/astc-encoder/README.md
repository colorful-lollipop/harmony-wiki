# astc-encoder Wiki

## 概览

本文档是 OpenHarmony 第三方库 `astc-encoder` 的 Wiki，重点记录该库在 OpenHarmony 中的**集成方式、定制化内容和使用场景**。

> **注意**：原始库功能文档请参考上游 [ARM astc-encoder](https://github.com/ARM-software/astc-encoder)。本文档仅关注 OpenHarmony 相关的适配和差异。

---

## 库基本信息

| 属性 | 值 |
|------|-----|
| **原始库名称** | Arm ASTC Encoder (astcenc) |
| **版本** | 4.7.0 |
| **许可证** | Apache 2.0 |
| **上游地址** | https://github.com/ARM-software/astc-encoder.git |
| **OH 组件名** | astc-encoder |
| **OH 子系统** | thirdparty |
| **OH 包名** | @ohos/astc-encoder |

---

## OpenHarmony 适配概述

### 无 Patch 设计

与许多需要大量 Patch 的第三方库不同，**astc-encoder 在 OpenHarmony 中没有 Patch 文件**。这得益于：

1. **原生跨平台设计**：代码本身支持多平台（Windows/Linux/macOS/Android/iOS）
2. **Apache 2.0 许可证**：与 OH 完全兼容，无需许可证修改
3. **清晰的模块化**：核心编解码器与平台相关代码分离
4. **标准 C++**：无特殊编译器依赖

### OH 特有定制

虽然没有传统 Patch，但存在以下定制：

| 定制类型 | 说明 |
|---------|------|
| **条件编译** | `ASTC_CUSTOMIZED_ENABLE` - 启用 graphic_2d_ext 和 HMOS SDK 支持 |
| **平台宏** | `SUT_PATH_X64` - ARM64/模拟器特定路径处理 |
| **构建标记** | `BUILD_HMOS_SDK` - HMOS SDK 构建标识 |
| **附加源文件** | `astcenccli_platform_dependents.cpp` - 平台相关实现 |

---

## 文档导航

### 核心文档

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 中的作用和定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch，重点说明原因） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 详解、构建配置、编译选项 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、集成示例 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异分析（本库无 API 修改） |
| [06_Security.md](./06_Security.md) | 安全分析、CVE 状态、升级建议 |

### 工作文档

- `_work/ASSESSMENT.md` - 项目评估报告（Phase 0 输出）
- `_work/NOTES.md` - 分析过程记录
- `_work/PLAN.md` - 任务进度跟踪

---

## 快速参考

### 在 OH 模块中依赖此库

```gn
deps = [ "//third_party/astc-encoder:astc_encoder_shared" ]
```

或使用 external_deps（推荐）：

```gn
external_deps = [
  "astc-encoder:astc_encoder_shared",
]
```

### 头文件引用

```cpp
#include "astcenc.h"
```

### 编译输出路径

```
out/{product}/thirdparty/astc-encoder/libastc_encoder_shared.z.so
```

---

## 主要使用场景

根据代码分析，astc-encoder 在 OpenHarmony 中主要用于：

1. **图库缩略图压缩** - 生成 ASTC 格式的图库预览
2. **应用预置图压缩** - 压缩应用资源图片
3. **GPU 直接解码** - ASTC 码流可由 GPU 直接解码，减少 CPU 负载

---

## 版本历史

| 版本 | 时间 | 说明 |
|------|------|------|
| 4.7.0 | 2024-01 | 当前版本，修复了解压器舍入行为 |

详见上游 [ChangeLog-4x.md](../Docs/ChangeLog-4x.md)

---

## 维护者

- **OH 维护者**: wangyonglang@huawei.com
- **上游项目**: Arm Limited and contributors

---

## 许可证

本库遵循 Apache 2.0 许可证。详见 [LICENSE.txt](../LICENSE.txt)
