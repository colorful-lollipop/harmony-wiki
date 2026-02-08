# 文档导航

## 快速索引

### 概览与架构
- [首页/概览](00_Overview.md) - 项目定位、核心能力、目录结构
- [系统架构](01_Architecture.md) - 组件图、数据流、线程模型

### 接口文档
- [JS N-API 接口](10_NAPI_JS.md) - FileShare/FileURI/Backup JS API
- [NDK 接口](11_NAPI_NDK.md) - Native 开发接口
- [内部 API](20_Inner_API.md) - 模块间接口、依赖方向

### 实现细节
- [备份服务 SA](02_Service_SA.md) - Service Ability 实现、IPC 模式
- [工具库](03_Utils.md) - Utils 模块职责与依赖

### 构建与部署
- [GN 构建配置](04_GN_Build.md) - Targets、产物映射、编译开关

### 安全评审
- [安全风险评审](05_Security_Review.md) - 攻击面、风险点、修复建议

## 完整目录

```
wiki/
├── README.md              # 本文档说明
├── SUMMARY.md             # 导航索引
├── 00_Overview.md        # 项目概览
├── 01_Architecture.md    # 系统架构
├── 02_Service_SA.md      # 备份服务 SA
├── 03_Utils.md           # 工具库
├── 04_GN_Build.md        # GN 构建配置
├── 10_NAPI_JS.md         # JS N-API
├── 11_NAPI_NDK.md        # NDK 接口
├── 20_Inner_API.md       # 内部 API
└── 05_Security_Review.md # 安全评审
```

## 阅读顺序建议

### 场景 1：新人入门
```
1. README.md → 00_Overview.md → 01_Architecture.md → 10_NAPI_JS.md
```

### 场景 2：API 使用
```
1. 10_NAPI_JS.md (或 11_NAPI_NDK.md) → 查看具体 API
```

### 场景 3：开发调试
```
1. 01_Architecture.md → 02_Service_SA.md → 03_Utils.md
```

### 场景 4：安全审计
```
1. 05_Security_Review.md
```

## 双路线导航

### 路线 A：新人学习路线 👨‍💻

适合刚接触此项目的开发者，快速理解项目定位和基本用法：

```
Step 1: README.md → 了解 Wiki 结构
Step 2: 00_Overview.md → 理解项目是什么、能做什么
Step 3: 01_Architecture.md → 理解系统如何组织
Step 4: 10_NAPI_JS.md → 学习 JS API 使用
Step 5: 04_GN_Build.md → 了解构建方式（可选）
```

**预计时间**: 30-60 分钟  
**目标**: 能够描述项目架构，找到核心代码位置

### 路线 B：安全研究路线 🔒

适合安全研究员，快速识别攻击面和安全风险：

```
Step 1: README.md → 了解 Wiki 结构
Step 2: 01_Architecture.md → 理解架构和信任边界
Step 3: 05_Security_Review.md → 查看已识别的风险点
Step 4: 02_Service_SA.md → 分析服务层攻击面
Step 5: 03_Utils.md → 检查工具库安全机制
Step 6: 10_NAPI_JS.md / 11_NAPI_NDK.md → 分析接口输入验证
```

**预计时间**: 60-120 分钟  
**目标**: 识别所有外部输入入口，评估可利用性

---

## 文档状态

| 文档 | 状态 | 最后更新 | 代码证据 |
|------|------|----------|----------|
| README.md | ✅ 完成 | 2026-02-07 | 有 |
| SUMMARY.md | ✅ 完成 | 2026-02-07 | 有 |
| 00_Overview.md | ✅ 完成 | 2026-02-07 | 有 |
| 01_Architecture.md | ✅ 完成 | 2026-02-07 | 有 |
| 02_Service_SA.md | ✅ 完成 | 2026-02-07 | 有 |
| 03_Utils.md | ✅ 完成 | 2026-02-07 | 有 |
| 04_GN_Build.md | ✅ 完成 | 2026-02-07 | 有 |
| 10_NAPI_JS.md | ✅ 完成 | 2026-02-07 | 有 |
| 11_NAPI_NDK.md | ✅ 完成 | 2026-02-07 | 有 |
| 20_Inner_API.md | ✅ 完成 | 2026-02-07 | 有 |
| 05_Security_Review.md | ✅ 完成 | 2026-02-07 | 有 |

## 符号说明

| 符号 | 含义 |
|------|------|
| ✅ | 文档已完成，含代码证据 |
| 🔒 | 涉及安全相关内容 |
| 📊 | 包含架构图或数据流 |
| 🔧 | 涉及实现细节 |
