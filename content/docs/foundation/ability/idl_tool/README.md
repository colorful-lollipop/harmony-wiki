# OpenHarmony IDL Tool Wiki

---

## 文档覆盖范围

本文档系统为 **OpenHarmony IDL Tool** 项目提供完整的工程文档，帮助新人快速理解项目架构、模块职责、构建系统和使用方式。

### 覆盖的模块

- [x] 项目定位与边界
- [x] 目录结构与模块职责
- [x] 架构设计与数据流
- [x] 内部 API 与依赖关系
- [x] GN 构建目标与编译产物
- [x] 安全风险评审
- [x] 常见问题与调试指南

### 未覆盖范围

- ~~N-API（JS API）~~ - 本项目是代码生成工具，不对外提供 JS API
- ~~运行时服务实现~~ - 本项目生成代码，不提供服务
- ~~权限与鉴权机制~~ - 纯代码生成器，无运行时权限检查

---

## 生成方式

### 自动生成

本文档基于以下内容自动生成：

1. **代码静态分析**
   - IDL 源代码扫描
   - AST 模块结构分析
   - Parser/Lexer 实现分析
   - Codegen 后端分析
   - Metadata 系统分析
   - GN 构建系统分析

2. **证据提取**
   - 所有关键结论基于实际代码证据
   - 提供文件路径和符号名称
   - 必要时标注行号或代码片段

3. **一致性校验**
   - 术语统一性检查
   - 链接有效性验证
   - 无测试代码引用

---

## 手动更新指南

当代码发生变更时，需要更新本文档：

### 更新优先级

| 变更类型 | 优先级 | 需要更新的文档 |
|---------|--------|---------------|
| 新增 IDL 语法特性 | 高 | 02_Architecture.md, 04_Internal_API.md |
| 修改 AST 结构 | 高 | 02_Architecture.md |
| 新增代码生成后端 | 高 | 02_Architecture.md, 06_Build_Artifacts.md |
| 修改 GN 构建配置 | 中 | 05_GN_Targets.md, 06_Build_Artifacts.md |
| 新增支持的类型 | 中 | 00_Overview.md, 04_Internal_API.md |
| 安全相关修改 | 高 | 07_Security_Review.md |
| 文档结构调整 | 低 | README.md, SUMMARY.md |

### 更新步骤

1. 阅读变更的代码
2. 确定受影响的模块和文档
3. 更新相关文档章节
4. 验证文档中的证据引用
5. 检查 SUMMARY.md 中的链接
6. 更新本文档的"生成时间"

---

## 生成时间

**初始版本**: 2026-02-06
**上次更新**: 2026-02-06

---

## 维护者

本 Wiki 由自动化工具生成，基于代码静态分析。

如有疑问或建议，请参考：
- 主仓库：[ability_idl_tool](https://gitee.com/openharmony/ability_idl_tool)
- 开发指南：[IDL 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/IDL/)
