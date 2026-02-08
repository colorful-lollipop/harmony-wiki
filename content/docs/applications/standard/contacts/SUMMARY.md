# 全站导航

本文档为 OpenHarmony Contacts 应用的工程 Wiki，提供完整的技术参考。

## 阅读指南

### 新人快速上手（推荐顺序）

1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [架构说明](01_Architecture.md) - 理解整体设计和数据流
3. [模块结构](02_Module_Structure.md) - 熟悉模块划分和职责

### 开发者参考

4. [API 参考](03_API_Reference.md) - 查找 Native API 调用模式
5. [构建系统](04_Build_System.md) - 了解构建配置和产物

### 安全与运维

6. [安全评审](05_Security_Review.md) - 安全风险和修复建议
7. [问题排查](06_Troubleshooting.md) - 常见问题和定位路径

---

## 文档目录

### 快速入口

| 文档 | 描述 | 适用角色 |
|------|------|----------|
| [README](README.md) | Wiki 说明和更新方式 | 所有开发者 |
| [项目概览](00_Overview.md) | 项目定位、功能、技术栈 | 新人、业务方 |
| [架构说明](01_Architecture.md) | 组件图、数据流、时序图 | 架构师、开发者 |
| [模块结构](02_Module_Structure.md) | 目录结构和模块职责 | 开发者 |

### 技术参考

| 文档 | 描述 | 适用角色 |
|------|------|----------|
| [API 参考](03_API_Reference.md) | Native API 调用清单和模式 | 开发者 |
| [构建系统](04_Build_System.md) | hvigor 配置和编译产物 | 开发者、CI/CD |

### 安全与运维

| 文档 | 描述 | 适用角色 |
|------|------|----------|
| [安全评审](05_Security_Review.md) | 攻击面、风险清单、修复建议 | 安全工程师 |
| [问题排查](06_Troubleshooting.md) | 常见问题和调试指南 | 开发者、运维 |

---

## 模块快速跳转

### 按功能模块

| 功能 | 路径 | 相关文档 |
|------|------|----------|
| 联系人管理 | `feature/contact/` | API 参考、模块结构 |
| 通话记录 | `feature/call/` | API 参考、模块结构 |
| 拨号盘 | `feature/dialpad/` | API 参考、模块结构 |
| 账号管理 | `feature/account/` | API 参考、模块结构 |
| 电话号码处理 | `feature/phonenumber/` | API 参考、模块结构 |

### 按代码层级

| 层级 | 路径 | 说明 |
|------|------|------|
| 入口层 | `entry/src/main/ets/` | Ability、页面、组件 |
| 功能层 | `feature/` | 业务功能模块 |
| 公共层 | `common/` | 工具类、权限管理 |
| 资源层 | `*/src/main/resources/` | 多语言资源 |

---

## 相关链接

### 内部文档

- [README](../README.md) - 项目英文说明
- [README_zh](../README_zh.md) - 项目中文说明
- [bundle.json](../bundle.json) - 模块配置

### 外部资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [ArkTS 语言介绍](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/arkts/arkts-introduction.md)
- [ArkUI 组件参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/ui/ui-overview.md)

---

## 贡献指南

### 文档更新

1. 克隆仓库并创建分支
2. 修改对应 Wiki 文件
3. 提交 PR 前检查链接有效性
4. 验证代码证据引用准确

### 质量标准

- 关键结论必须有代码证据（路径 + 符号）
- API 文档必须包含完整参数说明
- 安全评审必须包含风险级别和修复建议
