# OpenHarmony SDK C 工程 Wiki

> **生成时间**: 2025-02-06  
> **仓库路径**: interface_sdk_c  
> **文档版本**: 1.0

---

## Wiki 简介

本 Wiki 提供 OpenHarmony SDK C（C API 声明仓库）的完整工程文档，包括：

- **项目定位与架构**: 理解仓库在整个 OpenHarmony 系统中的位置和作用
- **目录结构与模块**: 详细的模块划分和职责说明
- **N-API 接口文档**: 对外 JavaScript 接口的完整梳理
- **内部架构**: 核心模块间的依赖关系和数据流
- **构建系统**: GN 构建目标和产物说明
- **安全分析**: 基于代码证据的安全风险评审
- **常见问题**: 构建、运行、调试的常见问题及解决方案

---

## 文档覆盖范围

### ✅ 已覆盖内容

| 类别 | 覆盖范围 | 状态 |
|------|----------|------|
| 模块清单 | 约 30+ 个主要模块 | ✅ 完整 |
| N-API 接口 | 基础 N-API + 领域 N-API | ✅ 完整 |
| GN 构建目标 | ndk_targets.gni 中定义的目标 | ✅ 完整 |
| 头文件清单 | 100+ 个公开头文件 | ✅ 完整 |
| 目录结构 | 两级目录结构 | ✅ 完整 |

### 📋 文档清单

1. [SUMMARY.md](./SUMMARY.md) - 全站导航与阅读路线
2. [00_Overview.md](./00_Overview.md) - 项目概览与定位
3. [01_Directory_Structure.md](./01_Directory_Structure.md) - 目录结构与模块职责
4. [02_Architecture.md](./02_Architecture.md) - 架构说明（组件图/数据流/线程模型）
5. [03_NAPI_Reference.md](./03_NAPI_Reference.md) - 对外 N-API 接口文档
6. [04_Internal_API.md](./04_Internal_API.md) - 内部 API 与模块接口
7. [05_GN_Build.md](./05_GN_Build.md) - GN 构建目标与编译配置
8. [06_Build_Artifacts.md](./06_Build_Artifacts.md) - 编译产物与安装路径
9. [07_Security_Analysis.md](./07_Security_Analysis.md) - 安全风险评审
10. [08_Common_Issues.md](./08_Common_Issues.md) - 常见问题与定位

### 📁 附录

- [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 关键调用链
- [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 关键配置项

---

## 如何更新本 Wiki

### 自动更新

本 Wiki 基于代码自动生成，当以下文件变更时需要重新生成：

- `ndk_targets.gni` - 新增/修改 NDK 构建目标
- `**/BUILD.gn` - 构建配置变更
- `**/*.ndk.json` - API 定义变更
- `**/*.h` - 头文件接口变更

### 手动更新流程

1. **同步代码**: 拉取最新代码到本地
2. **重新扫描**: 使用脚本重新扫描目录结构和 API
3. **对比更新**: 对比新旧文档，标记变更点
4. **审核发布**: 技术审核后发布更新

---

## 关键代码证据

本 Wiki 中的所有结论均基于以下代码文件：

| 文件类型 | 路径 | 作用 |
|----------|------|------|
| 根文档 | `README.md` | 项目说明和目录结构 |
| 构建配置 | `ndk_targets.gni` | NDK 构建目标列表 |
| 用户指南 | `docs/user_guide.md` | C API 使用指南 |
| 构建指南 | `docs/howto_add.md` | C API 构建添加指南 |
| 命名规范 | `docs/capi_naming.md` | C API 接口编码规范 |
| N-API 定义 | `arkui/napi/libnapi.ndk.json` | 基础 N-API 符号列表 |
| N-API 头文件 | `arkui/napi/native_api.h` | 基础 N-API 接口声明 |

---

## 注意事项

1. **接口稳定性**: 所有 C API 接口需保持前向兼容性
2. **测试目录**: Wiki 内容已排除 test/tests/unittest 等测试目录
3. **证据链**: 关键结论均有代码文件路径和行号支持
4. **更新频率**: 建议随版本发布同步更新 Wiki

---

## 贡献与反馈

如发现 Wiki 内容有误或需要补充，请：

1. 检查对应的代码文件确认问题
2. 提交 Issue 描述问题或缺失内容
3. 提供代码证据（文件路径 + 行号）

---

**OpenHarmony SDK C Wiki 生成于 2025-02-06**
