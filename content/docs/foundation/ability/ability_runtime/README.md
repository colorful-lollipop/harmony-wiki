# Ability Runtime 工程 Wiki

## 概述

本文档是 OpenHarmony **ability_runtime**（元能力运行时）组件的工程 Wiki，旨在帮助开发者快速理解项目架构、模块职责、API 接口、构建系统和安全机制。

**注意**：本文档基于代码证据自动生成，如有更新延迟请以源码为准。

## 覆盖范围

### 已覆盖内容

- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- N-API 接口清单（JS API 面）
- Inner API（内部组件间接口）
- GN 构建系统与编译产物
- 安全风险评审

### 未覆盖内容

- 详细测试用例（遵循约束不引用测试代码）
- 第三方依赖库的内部实现细节
- 运行时性能调优参数

## 更新方式

当代码发生以下变更时，需要同步更新本文档：

1. **新增/删除 N-API 模块**：更新 `wiki/04_NAPI_Reference.md`
2. **修改模块结构**：更新 `wiki/02_Directory_Structure.md` 和 `wiki/03_Architecture.md`
3. **变更构建配置**：更新 `wiki/06_GN_Targets.md` 和 `wiki/07_Build_Artifacts.md`
4. **新增安全机制**：更新 `wiki/08_Security_Review.md`

### 手动更新步骤

```bash
# 1. 克隆或更新代码仓库
git clone https://gitee.com/openharmony/ability_runtime.git

# 2. 运行文档生成工具（如果有）
./generate_docs.sh

# 3. 或手动编辑对应的 md 文件

# 4. 提交变更
git add wiki/
git commit -m "docs: 更新工程文档"
```

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Ability Runtime 源码](https://gitee.com/openharmony/ability_ability_runtime)
- [Issue 反馈](https://gitee.com/openharmony/ability_ability_runtime/issues)

## 贡献指南

欢迎开发者贡献文档改进：

1.Fork 本仓库
2.创建特性分支：`git checkout -b docs-improvement`
3.修改 `wiki/` 目录下的文档
4.提交 PR 并描述修改内容

---

**最后更新**：2026-02-06
**生成工具**：Ability Runtime Wiki Generator
