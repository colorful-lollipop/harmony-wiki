# ScreenLock 工程 Wiki

> OpenHarmony 锁屏应用工程文档
> 
> **BundleName**: `com.ohos.systemui`  
> **版本**: 1.0.0  
> **生成时间**: 2026-02-06

---

## 文档说明

本文档是 OpenHarmony `applications/standard/screenlock` 仓库的工程 Wiki，旨在帮助开发者快速理解项目架构、接口设计和安全风险。

### 覆盖范围

| 类别 | 覆盖内容 |
|------|----------|
| ✅ 项目概览 | 定位、边界、核心能力、运行环境 |
| ✅ 目录结构 | 13个模块的职责说明 |
| ✅ 架构设计 | 组件图、数据流、线程模型 |
| ✅ 系统API | 使用的@ohos系统服务（48个文件，77+处调用） |
| ✅ 权限清单 | 21个系统权限详细说明 |
| ✅ 安全风险 | 攻击面分析、可被利用点识别 |
| ✅ 构建配置 | Hvigor构建系统、模块配置 |

### 未覆盖范围

- ❌ 测试代码（test/ tests/ unittest/ 等目录）
- ❌ UI设计规范
- ❌ 国际化实现细节

### 如何更新本文档

1. 修改代码后同步更新对应Wiki页面
2. 在 `wiki/_work/NOTES.md` 中记录新发现的事实
3. 更新 `wiki/_work/PLAN.md` 跟踪进度
4. 重新生成相关Mermaid图表

### 阅读建议

**新人快速入门**:
1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [架构设计](03_Architecture.md) - 理解整体架构
4. [安全风险](08_Security.md) - 了解安全注意事项

**开发者参考**:
1. [对外API](04_External_API.md) - 系统服务使用清单
2. [内部API](05_Internal_API.md) - 模块接口说明
3. [附录-调用链](appendix/Callgraphs.md) - 关键调用链

---

## 项目基本信息

| 属性 | 值 |
|------|-----|
| 应用名称 | ScreenLock（锁屏应用） |
| BundleName | com.ohos.systemui |
| 应用类型 | 系统预置应用 |
| 版本号 | 1.0.0 (1000000) |
| 最低API版本 | 8 |
| 目标API版本 | 9 |
| 编译SDK版本 | 23 |
| UI语法 | ArkTS/ETS (ArkUI声明式) |
| 应用模型 | Stage模型 |
| 技术栈 | 纯ArkTS，无C++ N-API |

---

## 快速链接

- [完整导航](SUMMARY.md)
- [事实记录(内部)](_work/NOTES.md)
- [工作计划(内部)](_work/PLAN.md)

---

## 文档验证说明

### 验证状态

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 文件路径准确性 | ✅ 已验证 | 所有引用路径已验证存在（见`wiki/_work/NOTES.md`） |
| 权限清单 | ✅ 已验证 | 21个权限来自`phone/module.json5`，22个来自`pc/module.json5` |
| 系统API使用 | ✅ 已验证 | 28个@ohos模块，49个文件使用 |
| 安全风险证据 | ✅ 已验证 | 7个风险点均有代码证据支持 |
| 代码片段 | ✅ 已验证 | 所有代码片段来自实际源文件 |

### 关键证据位置

- **权限声明**: `product/phone/src/main/module.json5:21-85`
- **系统服务使用**: 49个源文件，163处import（见NOTES.md）
- **安全分析**: 基于`accountsModel.ts:239-258`等实际代码

---

*本文档由工程Agent生成，基于代码中的直接证据。*
*生成时间: 2026-02-06*
*验证方式: 静态代码分析 + 文件路径验证*
