# SUMMARY - astc-encoder Wiki 阅读指南

## 推荐阅读顺序

### 快速了解（5 分钟）
1. [README.md](./README.md) - 概览和导航
2. [01_Overview.md](./01_Overview.md) - 库简介和 OH 定位

### 深入了解（15 分钟）
3. [02_Patches.md](./02_Patches.md) - 为什么这个库没有 Patch？
4. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 详解
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 谁在用它？

### 运维参考（按需阅读）
6. [05_API_Differences.md](./05_API_Differences.md) - API 参考
7. [06_Security.md](./06_Security.md) - 安全与升级

---

## 文档分类

### 按读者角色

| 角色 | 推荐阅读 |
|------|---------|
| **新接触此库** | README → 01_Overview → 02_Patches |
| **需要集成使用** | 03_Build_Integration → 04_Usage_in_OH → 05_API_Differences |
| **负责升级维护** | 02_Patches → 03_Build_Integration → 06_Security |
| **排查问题** | 04_Usage_in_OH → 03_Build_Integration → _work/ASSESSMENT.md |

### 按任务类型

| 任务 | 参考文档 |
|------|---------|
| 了解库做什么 | 01_Overview.md |
| 理解为什么无 Patch | 02_Patches.md |
| 在 BUILD.gn 中添加依赖 | 03_Build_Integration.md, 04_Usage_in_OH.md |
| 编写使用代码 | 05_API_Differences.md, 上游 astcenc.h |
| 评估升级影响 | 06_Security.md, 02_Patches.md |
| 排查构建问题 | 03_Build_Integration.md, _work/ASSESSMENT.md |

---

## 关键结论速览

### 关于 Patch
❌ **无 Patch 文件**
- 这是一个**干净集成**的第三方库
- 所有 OH 适配通过 BUILD.gn 条件编译完成
- 维护成本低，升级风险小

### 关于构建
✅ **标准 GN 构建**
- 提供 `astc_encoder_shared` 目标
- 静态库 + 共享库两层结构
- 支持条件编译扩展

### 关于使用
📍 **图像框架专用**
- 主要用于 `image_framework` 子系统
- 核心场景：图库缩略图、应用预置图压缩
- 依赖模块数：约 5 个

### 关于 API
🔒 **无 API 修改**
- 完全使用上游原始 API
- 通过 `astcenc.h` 头文件访问
- 调用方式与上游一致

---

## 外部参考

### 上游文档
- [GitHub 仓库](https://github.com/ARM-software/astc-encoder)
- [ASTC 格式概述](../Docs/FormatOverview.md)
- [编码指南](../Docs/Encoding.md)
- [4.x 版本变更日志](../Docs/ChangeLog-4x.md)

### OH 相关代码
- `//third_party/astc-encoder/BUILD.gn`
- `//foundation/multimedia/image_framework/interfaces/innerkits/BUILD.gn`
- `//foundation/multimedia/image_framework/plugins/common/libs/image/libextplugin/BUILD.gn`

---

## 文档更新记录

| 日期 | 更新内容 |
|------|---------|
| 2026-02-07 | 初始版本创建 |
