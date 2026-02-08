# OpenHarmony Photos 图库应用 - 工程文档

## 项目概述

本文档为 OpenHarmony 标准系统图库应用（Gallery）的完整工程文档，面向**新加入的开发者**，提供项目架构、模块职责、接口规范、构建系统、安全评估等方面的全面说明。

| 属性 | 值 |
|------|-----|
| **项目名称** | @ohos/photos |
| **版本** | 3.0 |
| **许可证** | Apache License 2.0 |
| **源码路径** | `applications/standard/photos/` |
| **文档版本** | 1.0 |
| **文档生成时间** | 2026-02-05 |

## 文档覆盖范围

### ✅ 已覆盖

- [x] 项目定位与核心能力
- [x] 目录结构与模块职责
- [x] MVP 架构说明（含数据流）
- [x] Ability 入口与外部调用接口（URI Scheme）
- [x] HAR 模块依赖关系
- [x] 构建系统（hvigor）配置
- [x] 权限与安全机制
- [x] 常见问题与排查路径

### ⚠️ 部分覆盖

- [ ] 完整 UI 组件清单（部分页面已覆盖）
- [ ] 详细的数据模型定义（核心模型已覆盖）

### ❌ 未覆盖

- [ ] 测试相关代码（根据规范忽略）
- [ ] 第三方依赖具体版本（见 `oh-package.json5`）
- [ ] 性能基准测试数据

## 文档使用方式

### 新人阅读顺序（推荐）

```
1. README.md (本文档) → 了解文档范围
2. SUMMARY.md → 全站导航
3. 00_Overview.md → 项目整体认知
4. 01_Architecture.md → 理解数据流和架构
5. 02_Module_Structure.md → 掌握模块划分
6. 03_External_API.md → 学习如何外部调用
7. 05_Build_System.md → 了解构建配置
```

### 按需查阅

- **外部调用**: 直接访问 `03_External_API.md`
- **模块职责**: 访问 `02_Module_Structure.md`
- **构建配置**: 访问 `05_Build_System.md`
- **安全问题**: 访问 `07_Security_Assessment.md`

## 代码证据规范

本文档所有关键结论均基于**代码证据**，遵循以下格式：

```
[结论陈述]
- 证据类型: 符号名/文件路径
- 位置: 文件路径:行号
- 代码片段: (可选) 关键代码摘要
```

示例：
> MainAbility 支持三种外部调用模式
> - 证据: `MainAbility.ts:92-166`
> - `parseWantParameter()` 函数解析 want 参数

## 文档更新方式

### 随代码更新的场景

| 场景 | 更新内容 | 负责人 |
|------|----------|--------|
| 新增 Ability | 更新 `03_External_API.md` | 开发者 |
| 新增 HAR 模块 | 更新 `02_Module_Structure.md` | 开发者 |
| 修改权限 | 更新 `07_Security_Assessment.md` | 开发者 |
| 构建配置变更 | 更新 `05_Build_System.md` | 开发者 |

### 定期审查

建议每季度审查文档与代码一致性，特别是：
- 模块依赖关系
- API 接口清单
- 安全评估部分

## 术语表

| 术语 | 定义 |
|------|------|
| **HAR** | Harmony Archive，静态共享包 |
| **Ability** | OpenHarmony 应用组件（Page/Service/Form等） |
| **URI Scheme** | 统一资源标识符方案，用于外部调用 |
| **MVP** | Model-View-Presenter 架构模式 |
| **UIAbility** | UI 界面能力，替代传统 PageAbility |
| **ExtensionAbility** | 扩展能力，包括 Service/Form 等 |

## 反馈与贡献

如发现文档错误或遗漏，请：

1. 检查 `wiki/_work/NOTES.md` 中的待确认项
2. 在对应文档页面添加 TODO 标注
3. 联系文档维护团队

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [ArkTS 开发者指南](https://docs.openharmony.cn/pages/v4.0/zh-cn/application-dev/arkts/arkts-get-started.md)
- [Ability 开发指南](https://docs.openharmony.cn/pages/v4.0/zh-cn/application-dev/ability/ability-guideline.md)
