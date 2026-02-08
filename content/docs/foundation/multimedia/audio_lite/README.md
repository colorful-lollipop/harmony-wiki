# Audio Lite Wiki - 文档说明

本文档为 `foundation/multimedia/audio_lite` 项目的工程 Wiki，旨在帮助新人快速完整理解项目。

## 覆盖范围

### ✅ 已覆盖

| 维度 | 内容 |
|------|------|
| 项目概述 | 定位、能力、运行环境 |
| 架构设计 | 组件图、数据流、线程模型、调用链 |
| 对外接口 | C++ API 清单、参数说明（本项目无 N-API） |
| 构建系统 | GN targets、产物、依赖关系 |
| 安全评审 | 攻击面、风险点、修复建议 |
| 问题定位 | 常见构建/运行/调试问题 |

### ❌ 未覆盖

| 维度 | 原因 |
|------|------|
| JS/N-API 接口 | 本项目为纯 C++ framework，不包含 N-API 绑定 |
| 测试代码 | 根据规范，不引用测试代码作为业务证据 |
| 第三方库实现 | codec、audio_hw 等 HAL 库在独立仓库 |

## 文档规范

### 质量标准

- 每篇文档包含：**目的、适用范围、关键结论、相关跳转链接**
- 关键结论必须可追溯到代码证据（文件路径 + 符号名）
- 术语统一：Samgr → 系统能力管理器，IPC → 进程间通信

### 证据标注格式

```markdown
**证据**：[文件路径:行号]
> 代码片段或说明
```

## 更新方式

### 何时更新 Wiki

| 触发条件 | 操作 |
|----------|------|
| 新增模块/接口 | 在对应章节添加 API 文档 |
| 修改 GN targets | 更新 Build System 章节 |
| 安全相关变更 | 更新 Security Review 章节 |
| 架构重构 | 更新 Architecture 章节 |

### 更新流程

1. 在 `wiki/_work/NOTES.md` 中记录发现
2. 修改对应 Wiki 文档
3. 更新 SUMMARY.md 的链接（如果新增文件）
4. 运行 `lsp_diagnostics` 验证代码引用正确性

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── index.md               # 首页概览
├── 01_Overview.md        # 项目概述
├── 02_Architecture.md    # 架构说明
├── 03_API_Reference.md    # API 参考
├── 04_Build_System.md    # 构建系统
├── 05_Security_Review.md # 安全评审
├── 06_Troubleshooting.md # 问题定位
└── _work/                # 工作区（不纳入版本控制）
    ├── NOTES.md          # 事实记录
    └── PLAN.md           # 工作计划
```

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| Samgr | System Ability Manager | OpenHarmony 系统能力管理器 |
| SA | Service Ability | 系统服务能力 |
| IPC | Inter-Process Communication | 进程间通信 |
| HAL | Hardware Abstraction Layer | 硬件抽象层 |
| Surface | - | OpenHarmony 图形/媒体缓冲区共享机制 |

## 反馈与贡献

如发现文档错误或遗漏，请：

1. 在 `wiki/_work/NOTES.md` 中记录问题
2. 标注「需要更新」的章节
3. 注明代码证据位置

---

*最后更新：2026-02-06*
