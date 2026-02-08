# 阅读路线建议

本文档提供 EGL 库 Wiki 的阅读路线，帮助不同角色的读者快速找到所需信息。

## 读者类型与推荐阅读顺序

### 1. OpenHarmony 图形开发者

**目标**：了解如何在 OH 中使用 EGL API 或适配新平台

| 顺序 | 文档 | 重点内容 |
|------|------|----------|
| 1 | [01_Overview.md](./01_Overview.md) | EGL 功能定位与 OH 作用 |
| 2 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 头文件引用方式、使用场景 |
| 3 | [03_Build_Integration.md](./03_Build_Integration.md) | 构建配置细节 |

### 2. 图形驱动开发者

**目标**：为 OH 实现 EGL 驱动或添加新扩展

| 顺序 | 文档 | 重点内容 |
|------|------|----------|
| 1 | [02_Patches.md](./02_Patches.md) | OH 特有扩展规格 |
| 2 | [03_Build_Integration.md](./03_Build_Integration.md) | 编译宏与平台类型 |
| 3 | [01_Overview.md](./01_Overview.md) | 整体架构背景 |

### 3. 构建系统维护者

**目标**：理解 EGL 库的构建配置与依赖关系

| 顺序 | 文档 | 重点内容 |
|------|------|----------|
| 1 | [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 配置详解 |
| 2 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系图 |
| 3 | [02_Patches.md](./02_Patches.md) | 条件编译宏 |

### 4. 升级维护人员

**目标**：了解 EGL 库在 OH 中的定制化内容，评估升级影响

| 顺序 | 文档 | 重点内容 |
|------|------|----------|
| 1 | [03_Build_Integration.md](./03_Build_Integration.md) | OH 定制化配置 |
| 2 | [02_Patches.md](./02_Patches.md) | 扩展与平台适配 |
| 3 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 影响范围分析 |

## 文档依赖关系图

```
README.md (入口)
    │
    ├──► 01_Overview.md (基础概念)
    │         │
    │         └──► 03_Build_Integration.md (构建细节)
    │                   │
    │                   └──► 02_Patches.md (扩展规格)
    │
    └──► 04_Usage_in_OH.md (使用场景)
              │
              └──► 依赖 01_Overview.md 和 03_Build_Integration.md
```

## 关键信息速查

### Q: 这个库是做什么的？
**A**: 答：EGL-Registry 是 Khronos 官方的 EGL 头文件和扩展注册表。在 OH 中，它定义了 EGL API 接口规范，供图形驱动和渲染框架使用。

### Q: EGL 有多少 OH 特有的修改？
**A**: 答：没有代码 Patch。有 1 个 OH 特有扩展（EGL_OHOS_image_native_buffer）和平台类型适配。

### Q: 如何在代码中使用 EGL？
**A**: 答：包含头文件 `#include <EGL/egl.h>`，构建时依赖 `//third_party/EGL:libEGL` inner kit。

### Q: 升级上游版本需要注意什么？
**A**: 答：保留 BUILD.gn 和 OH 扩展定义，确保 `eglplatform.h` 中的 `OHOS_PLATFORM` 类型定义不被覆盖。

## 文档更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2024-XX-XX | 初始版本 |
