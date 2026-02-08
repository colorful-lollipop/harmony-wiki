# NetManager Cangjie Wrapper Wiki 文档

## 项目简介

`netmanager_cangjie_wrapper` 是 OpenHarmony 网络管理子系统的仓颉（Cangjie）语言封装层，为仓颉应用提供网络连接管理和 HTTP 数据请求能力。

**关键信息：**
- **项目类型**：仓颉语言 FFI 包装器
- **子系统**：communication
- **组件名**：netmanager_cangjie_wrapper
- **API 级别**：22+
- **系统能力**：`SystemCapability.Communication.NetManager.Core` 和 `SystemCapability.Communication.NetStack`
- **当前状态**：Beta 特性

## 文档覆盖范围

本 Wiki 文档涵盖以下内容：

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | 定位、边界、核心能力 |
| 系统架构 | ✅ 完成 | 组件图、数据流、依赖关系 |
| 对外 API | ✅ 完成 | Cangjie 接口定义 |
| FFI 接口 | ✅ 完成 | 外部函数声明 |
| GN 构建配置 | ✅ 完成 | Targets 和编译产物 |
| 安全评审 | ✅ 完成 | 风险分析和建议 |

## 文档结构

```
wiki/
├── README.md              # 本文档，覆盖范围和更新方式
├── SUMMARY.md             # 全站导航 + 新人阅读路线
├── index.md              # 首页
├── 01_Overview.md        # 项目定位与核心能力
├── 02_Architecture.md    # 系统架构说明
├── 03_API_Reference.md   # Cangjie API 参考
├── 04_FFI_Interface.md    # FFI 接口与数据类型
├── 05_GN_Build.md        # 构建配置与产物
├── 06_Security_Review.md # 安全风险评审
└── appendix/
    └── Callgraphs.md     # 关键调用链
```

## 阅读建议

**新人快速上手：**
1. 先阅 `index.md` 了解项目定位
2. 阅读 `01_Overview.md` 理解核心能力
3. 查看 `02_Architecture.md` 理解架构
4. 根据需求查看 `03_API_Reference.md`

**开发者扩展：**
1. `04_FFI_Interface.md` - 了解 FFI 接口定义
2. `05_GN_Build.md` - 构建配置细节

## 关键依赖仓库

本项目的实现依赖以下仓库：

| 仓库 | 用途 |
|------|------|
| `communication_netmanager_base` | 网络管理 C 接口 |
| `communication_netstack` | HTTP 栈 C 接口 |
| `arkcompiler_cangjie_ark_interop` | 仓颉互操作框架 |
| `hiviewdfx_cangjie_wrapper` | 日志接口 |

## 更新日志

| 日期 | 更新内容 | 更新人 |
|------|---------|--------|
| 2024-02-06 | 初始 Wiki 文档 | Wiki Generator |

## 反馈与贡献

如发现文档错误或需要补充，请：
1. 在代码中添加注释
2. 提交 Issue 反馈
3. 直接修改 Wiki 并提交 PR
