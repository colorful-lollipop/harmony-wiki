# MediaLibrary Wiki 索引

本文档为 MediaLibrary (medialibrary_standard) 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、N-API 接口、构建系统和安全风险。

## 文档覆盖范围

### 必读文档

| 文档 | 描述 | 阅读顺序 |
|-----|------|---------|
| [README](README.md) | Wiki 使用指南和覆盖范围说明 | ⭐ 必读 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、运行环境 | 1️⃣ |
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构和模块职责 | 2️⃣ |
| [02_Architecture](02_Architecture.md) | 架构图、数据流、线程模型 | 3️⃣ |
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口详细参考 | 4️⃣ |
| [04_Inner_API](04_Inner_API.md) | 内部 API 和模块依赖 | 5️⃣ |
| [05_Build_System](05_Build_System.md) | GN Targets 和编译产物 | 6️⃣ |
| [06_Security_Review](06_Security_Review.md) | 安全风险评审 | 7️⃣ |
| [07_Troubleshooting](07_Troubleshooting.md) | 常见问题定位 | 🔧 |

### 附录文档

| 文档 | 描述 |
|-----|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 关键配置项 |

---

## 新人阅读路线

### 快速入门 (10 分钟)
1. [README](README.md) - 了解 Wiki 使用方式
2. [00_Overview](00_Overview.md) - 理解项目定位

### 深入学习 (30 分钟)
1. 完成快速入门
2. [01_Directory_Structure](01_Directory_Structure.md) - 熟悉代码结构
3. [02_Architecture](02_Architecture.md) - 理解架构设计

### 开发参考 (按需)
- N-API 开发 → [03_N-API_Reference](03_N-API_Reference.md)
- 内部模块开发 → [04_Inner_API](04_Inner_API.md)
- 构建配置 → [05_Build_System](05_Build_System.md)
- 安全问题 → [06_Security_Review](06_Security_Review.md)
- 问题排查 → [07_Troubleshooting](07_Troubleshooting.md)

---

## 关键入口点

### N-API 入口
- 主模块: `multimedia.mediaLibrary` → `native_module_ohos_medialibrary.cpp`
- 用户文件管理: `userfileManager` → `native_module_ohos_userfile_manager.cpp`
- 照片访问: `photoAccessHelper` → `native_module_ohos_photoaccess_helper.cpp`

### 服务入口
- 资产管理: `MediaAssetsManagerService`
- 相册管理: `MediaAlbumsManagerService`

### 权限检查
- 权限验证: `AccessTokenKit::VerifyAccessToken`
- 主要权限: `READ_IMAGEVIDEO`, `WRITE_IMAGEVIDEO`

---

## 版本信息

- **项目版本**: 4.0
- **文档生成时间**: 基于代码分析自动生成
- **最后更新**: 随代码变更持续更新

---

## 贡献指南

### 文档更新
1. 所有文档修改位于 `wiki/` 目录
2. 遵循文档模板格式
3. 关键结论需附带代码证据（文件路径 + 符号）

### 证据规范
- API 接口 → 接口文件路径
- 架构设计 → BUILD.gn 或架构图
- 安全风险 → 安全相关代码路径

---

## 反馈与改进

如发现文档错误或遗漏，请：
1. 检查对应章节的代码证据
2. 在 NOTES.md 中记录发现
3. 提交 Issue 或 PR 完善文档
