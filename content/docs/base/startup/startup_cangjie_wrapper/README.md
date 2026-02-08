# startup_cangjie_wrapper Wiki 文档说明

## 文档概述

本 Wiki 是 OpenHarmony **startup_cangjie_wrapper（启动恢复仓颉封装）** 组件的完整技术文档，旨在帮助开发者快速理解项目架构、API 设计、构建流程和安全风险。

## 文档更新时间

生成时间: 2026-02-06

---

## 文档覆盖范围

### 已覆盖 ✓

- [x] 项目概览和定位
- [x] 目录结构与模块职责
- [x] 系统架构（组件图、数据流、线程模型）
- [x] 仓颉 API 清单（32 个公开 API + 5 个隐藏 API）
- [x] 内部 API（FFI 函数、依赖关系）
- [x] GN 构建目标和依赖
- [x] 编译产物和安装路径
- [x] 安全风险评审（5 个可利用点）
- [x] 常见问题和定位路径

### 未覆盖 ⊗

- [ ] 测试用例和测试流程（按需求忽略）
- [ ] 详细的 FFI 实现细节（依赖外部 init 组件）
- [ ] 性能分析和优化建议
- [ ] 实际编译产物的文件名和路径（待验证）
- [ ] 仓颉运行时加载机制（待确认）

---

## 如何阅读本文档

### 新人入门（30分钟）

1. **[00_Overview.md](00_Overview.md)** - 项目概览和快速了解
2. **[01_Project_Positioning.md](01_Project_Positioning.md)** - 项目定位和边界
3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 目录结构详解

### 深入理解（2小时）

4. **[03_Architecture.md](03_Architecture.md)** - 架构说明
5. **[04_Cangjie_API.md](04_Cangjie_API.md)** - API 清单
6. **[05_Internal_API.md](05_Internal_API.md)** - 内部 API

### 构建与部署（1小时）

7. **[06_GN_Targets.md](06_GN_Targets.md)** - GN 目标
8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 编译产物

### 安全与运维（1小时）

9. **[08_Security_Review.md](08_Security_Review.md)** - 安全风险评审
10. **[09_Common_Issues.md](09_Common_Issues.md)** - 常见问题

### 附录（按需查阅）

- **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 调用链
- **[appendix/API_Reference.md](appendix/API_Reference.md)** - API 参考

---

## 文档质量标准

### 证据要求

- 所有关键结论必须有**代码证据**（路径 + 符号 + 行号）
- 禁止凭空猜测或假设
- 无法确认的内容必须标注 `TODO（需确认）`

### 术语规范

| 术语 | 规范 |
|------|------|
| 仓颉 | 统一使用"仓颉"（可备注 Cangjie） |
| 封装 | 使用"封装"或"Wrapper" |
| 启动恢复 | 使用"启动恢复"或"startup" |

### 链接规范

- 所有内部链接使用相对路径
- 外部链接使用完整 URL
- 链接文本简洁明确

---

## 如何更新文档

### 1. 更新代码证据

当代码变更时，需要更新：

- [ ] 更新 `wiki/_work/NOTES.md`（事实记录）
- [ ] 更新相关 .md 文件中的代码行号
- [ ] 验证证据是否仍然有效

### 2. 添加新内容

当需要添加新章节时：

- [ ] 在 `wiki/_work/PLAN.md` 中添加任务
- [ ] 先阅读相关代码
- [ ] 提取关键事实到 NOTES.md
- [ ] 撰写文档内容
- [ ] 更新 `wiki/SUMMARY.md` 导航

### 3. 修正错误

当发现文档错误时：

- [ ] 验证代码证据
- [ ] 修正错误内容
- [ ] 更新 NOTES.md
- [ ] 在文档末尾添加更新说明

### 4. 定期审查

建议每季度进行一次文档审查：

- [ ] 检查所有代码证据是否仍然有效
- [ ] 检查 TODO 是否有新进展
- [ ] 检查链接是否有效
- [ ] 检查内容是否过时

---

## 文档生成工具

本 Wiki 使用 **AI 辅助工具**生成，基于代码分析自动提取关键信息。

### 生成流程

1. **代码扫描**: 使用 grep、ast-grep 等工具扫描代码
2. **事实提取**: 提取文件路径、符号、行号等信息
3. **文档生成**: 基于事实生成结构化文档
4. **证据验证**: 确保所有结论都有代码证据

### 工作目录

- **NOTES.md**: 事实记录（路径、符号、疑问、TODO）
- **PLAN.md**: 任务拆解和进度追踪

---

## 文档维护

### 维护团队

- 主维护者: 待确认
- 贡献者: 欢迎通过 PR 贡献

### 反馈渠道

- GitHub Issues: [待添加]
- 邮件: [待添加]

### 版本历史

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，完成所有章节 |

---

## 相关资源

### 官方文档

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [仓颉语言规范](https://developer.openharmony.cn/cn/doc/cangjie-quickstart)
- [OpenHarmony 设备信息 API（ArkTS）](https://docs.openharmony.cn/application-dev/reference/apis/js-apis-device-info.md)

### 外部仓库

- [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - 仓颉互操作库
- [startup_init](https://gitcode.com/openharmony/startup_init) - init 组件
- [仓颉设备信息 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-device_info.md)

### 开发工具

- [OpenHarmony 源码下载](https://docs.openharmony.cn/development/get-source-code)
- [OpenHarmony 构建指南](https://docs.openharmony.cn/application-dev/quick-start/start-overview)
- [仓颉编译器](https://developer.openharmony.cn/cn/download/cangjie)

---

## 许可证

本文档遵循项目的 Apache License 2.0 许可证。

---

## 联系方式

如有疑问或建议，请通过以下方式联系：

- 提交 GitHub Issue
- 发送邮件至维护团队

---

*最后更新: 2026-02-06*
