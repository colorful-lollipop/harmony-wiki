# 用户证书管理工程 Wiki

> 本 Wiki 文档由 OpenHarmony 工程 Agent 自动生成
> 生成时间: 2026-02-06 12:32:12
> 项目路径: applications/standard/user_certificate_manager

---

## 文档说明

### 覆盖范围

本 Wiki 文档覆盖以下内容：

1. **项目概述** - 用户证书管理部件的定位、边界、核心能力
2. **目录结构** - 模块职责和文件组织
3. **架构设计** - 组件图、数据流、时序图
4. **外部 API** - 与证书管理服务的接口清单
5. **内部 API** - 模块间接口和依赖
6. **构建系统** - GN Targets、编译产物
7. **安全评审** - 威胁模型、可被利用点、修复建议
8. **常见问题** - 构建、运行、调试相关问题

### 未覆盖范围

以下内容不在本 Wiki 覆盖范围内：

- **测试代码** - 所有测试相关文件 (`test/`, `ohosTest/`) 被忽略
- **第三方依赖实现** - `@ohos.security.certManager`、`@ohos.userIAM.userAuth` 等系统 API 的实现细节不在本项目范围内
- **部署/运维指南** - 生产和运维相关内容
- **性能优化建议** - 性能调优和基准测试

### 如何随代码更新文档

当项目代码发生变化时，建议按以下方式更新 Wiki：

1. **重大功能变更** - 更新相关章节，特别是：
   - `00_Overview.md` - 功能范围变化
   - `03_Architecture.md` - 架构调整
   - `04_External_API.md` - 新增/修改 API

2. **新增文件/模块** - 更新：
   - `02_Directory_Structure.md` - 新增目录和模块说明

3. **安全修复** - 更新：
   - `08_Security_Review.md` - 添加/删除风险点

4. **定期审查** - 建议每个季度进行一次全面审查，确认文档与代码同步

### 文档生成方法

本 Wiki 由 OpenHarmony 工程 Agent 基于以下信息生成：

- 源代码文件 (`certmanager/src/main/ets/`)
- 配置文件 (`BUILD.gn`, `bundle.json`, `module.json`)
- 构建脚本 (`build-profile.json5`, `oh-package.json5`)
- 权限声明 (`module.json` 中的 requestPermissions)

所有关键结论均可追溯到具体代码位置（文件路径 + 行号）。

---

## 文档结构

请参考 `SUMMARY.md` 获取完整的文档导航和新人阅读顺序。

---

## 贡献与反馈

如有疑问或建议，请通过以下方式反馈：

- 提交 Issue 到项目仓库
- 提交 PR 改进 Wiki 文档
- 联系项目维护者

---

**最后更新**: 2026-02-06
**文档版本**: 1.0.0
