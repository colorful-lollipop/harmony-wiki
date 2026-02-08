# i18n_lite Wiki 文档

## 项目概述

本 Wiki 为 OpenHarmony i18n_lite 国际化模块的工程文档，涵盖项目架构、API 接口、构建配置、安全分析等内容。

**i18n_lite** 是 OpenHarmony Globalization 子系统的核心组件，提供轻量级国际化能力，包括：

- 日期时间格式化
- 数字格式化
- 复数规则处理
- 区域信息管理
- JavaScript API 绑定

## 文档覆盖范围

### 已覆盖内容

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概述 | ✅ 完成 | 项目定位、核心能力、运行环境 |
| 目录结构 | ✅ 完成 | 模块职责划分 |
| N-API 接口 | ✅ 完成 | JS API 清单、参数校验 |
| C++ API | ✅ 完成 | 头文件、类结构、方法说明 |
| 架构设计 | ✅ 完成 | 组件图、数据流、线程模型 |
| GN 构建 | ✅ 完成 | Targets、依赖、产物映射 |
| 安全分析 | ✅ 完成 | 攻击面、风险清单 |
| 附录 | ✅ 完成 | 调用链、配置标志 |

### 未覆盖内容

- 测试用例分析（按规范要求忽略）
- 完整时序图（部分关键流程已覆盖）
- 第三方依赖详细分析（bounds_checking_function 已标注）

## 更新方式

### 何时更新 Wiki

当发生以下变更时，应同步更新 Wiki：

1. **API 变更**：新增、修改或删除 N-API/C++ API
2. **架构变更**：模块拆分、合并或依赖变更
3. **构建变更**：新增 BUILD.gn target、修改编译产物
4. **安全修复**：发现并修复安全问题
5. **重大重构**：影响模块职责或数据流的变更

### 更新步骤

```bash
# 1. 克隆仓库并创建分支
git checkout -b wiki-update

# 2. 修改 wiki/ 目录下的相关文档
# 使用 git diff 确认变更

# 3. 提交变更
git add wiki/
git commit -m "docs: update wiki for [变更说明]"

# 4. 创建 PR 进行 Code Review
```

### 版本管理

Wiki 版本与 i18n_lite 组件版本保持同步：

- **i18n_lite 1.0.0** → Wiki v1.0.0
- **i18n_lite 1.1.0** → Wiki v1.1.0

## 新人阅读建议

### 推荐阅读顺序

1. `index.md` - Wiki 导航和快速入口
2. `01_Overview.md` - 项目定位和核心能力
3. `02_API.md` - N-API 接口文档（按需查阅）
4. `03_Architecture.md` - 架构和数据流
5. `04_Build.md` - 构建配置和产物
6. `05_Security.md` - 安全注意事项
7. `06_Appendix.md` - 常见问题和附录

### 按角色推荐

| 角色 | 推荐阅读 |
|------|----------|
| 应用开发者 | `index.md` → `02_API.md` |
| 系统开发者 | 全部文档 |
| 安全审计 | `05_Security.md` → `03_Architecture.md` |
| 构建维护 | `04_Build.md` → `03_Architecture.md` |

## 文档规范

### 代码证据要求

所有关键结论必须可追溯到代码证据：

- **文件路径**：`frameworks/i18n/src/date_time_format.cpp:42`
- **符号名**：`DateTimeFormat::Format()`
- **宏定义**：`i18n_lite_support_i18n_product`

无法确认的信息必须标注 `TODO(需确认)` 并说明缺少的证据。

### 术语统一

| 术语 | 英文 | 说明 |
|------|------|------|
| 国际化 | i18n | Internationalization |
| 区域信息 | Locale | 语言/脚本/地区的组合 |
| 格式化 | Format | 按规则转换数据格式 |
| 复数规则 | Plural Rule | 不同数量的名词变化规则 |

## 相关资源

### 内部链接

- [项目概览](./index.md)
- [API 接口文档](./02_API.md)
- [架构设计](./03_Architecture.md)
- [构建配置](./04_Build.md)
- [安全分析](./05_Security.md)
- [附录](./06_Appendix.md)

### 外部资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Globalization 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/globalization.md)
- [i18n_lite 仓库](https://gitee.com/openharmony/global_i18n_lite)
- [边界检查函数库](https://gitee.com/openharmony/bounds_checking_function)

## 反馈与贡献

### 贡献指南

1. 发现文档错误？请提交 Issue 或 PR
2. 需要新增内容？请在 `wiki/_work/NOTES.md` 添加建议
3. 有更好的表达方式？欢迎改进文档质量

### 联系方式

- 仓库地址：https://gitee.com/openharmony/global_i18n_lite
- 组件负责人：Globalization 子系统团队

---

**文档生成时间**：2026-02-06  
**文档版本**：1.0.0  
**最后更新**：2026-02-06
