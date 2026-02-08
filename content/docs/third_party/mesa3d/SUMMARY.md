# 阅读路线

本文档旨在帮助开发者理解 **Mesa3D 在 OpenHarmony 中的集成与适配**。

## 推荐阅读顺序

### 入门路线

1. **[README.md](README.md)** - 快速了解 Mesa3D 在 OH 中的定位和作用
2. **[01_Overview.md](01_Overview.md)** - 了解 Mesa3D 原始功能和 OH 适配目标

### 核心技术路线

3. **[02_Patches.md](02_Patches.md)** - 深入理解 OH 对 Mesa3D 的所有修改（核心内容）
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 理解构建系统适配细节

### 实践路线

5. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解如何在 OH 中使用 Mesa3D
6. **[05_API_Differences.md](05_API_Differences.md)** - 如需进行二次开发，参考 API 差异
7. **[06_Security.md](06_Security.md)** - 安全评估和升级建议

## 快速索引

### 我想...

| 需求 | 跳转文档 |
|------|---------|
| 了解 Mesa3D 是什么 | 01_Overview.md |
| 查看 OH 做了哪些修改 | 02_Patches.md |
| 理解构建配置 | 03_Build_Integration.md |
| 查找依赖关系 | 04_Usage_in_OH.md |
| 进行二次开发 | 05_API_Differences.md |
| 评估安全风险 | 06_Security.md |

### 关键信息速查

| 主题 | 关键文件 |
|------|---------|
| OH 特有代码 | `src/egl/drivers/dri2/platform_ohos.c`, `include/vulkan/vulkan_ohos.h` |
| 构建脚本 | `ohos/build_ohos64.py`, `ohos/meson_cross_process64.py` |
| 构建配置 | `BUILD.gn`, `ohos/BUILD.gn` |
| Patch 文件 | `.gitlab-ci/container/patches/` |

## 文档更新日志

| 日期 | 更新内容 |
|------|---------|
| 2026-02-08 | 初始版本发布 |
