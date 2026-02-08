# WLAN Wiki 导航

**目的**: 提供全站导航和新人阅读顺序

**生成时间**: 2026-02-06

---

## 快速导航

| 文档 | 说明 | 预计阅读时间 |
|------|------|-----------|
| [00_Overview.md](00_Overview.md) | 项目概览、定位、核心能力 | 30 分钟 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | 30 分钟 |
| [02_Architecture.md](02_Architecture.md) | 架构说明、数据流、时序 | 60 分钟 |
| [03_Public_API_NAPI.md](03_Public_API_NAPI.md) | 对外 N-API 文档 | 120 分钟 |
| [04_Inner_API.md](04_Inner_API.md) | 内部 API 文档 | 60 分钟 |
| [05_GN_Targets.md](05_GN_Targets.md) | GN 目标梳理 | 60 分钟 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物说明 | 45 分钟 |
| [07_Security_Review.md](07_Security_Review.md) | 安全风险评审 | 90 分钟 |
| [08_FAQ.md](08_FAQ.md) | 常见问题与解答 | 30 分钟 |

**总计**: 约 8-9 小时完整学习

---

## 新人学习路径

### 路径一：快速入门（1 小时）
**目标**: 快速了解 WLAN 组件是什么，能做什么，如何集成

1. **[00_Overview.md](00_Overview.md)** (30 分钟)
   - 了解项目定位和边界
   - 理解核心能力（STA/AP/P2P）
   - 掌握关键概念和术语

2. **[01_Directory_Structure.md](01_Directory_Structure.md)** (30 分钟)
   - 熟悉目录结构
   - 了解各模块的职责
   - 知道源文件的大致位置

---

### 路径二：理解架构（1 小时）
**目标**: 深入理解 WLAN 组件如何工作

1. **[02_Architecture.md](02_Architecture.md)** (60 分钟)
   - 理解分层架构
   - 学习 IPC/SA 通信机制
   - 掌握数据流和时序
   - 了解线程模型

---

### 路径三：学习 API（2 小时）
**目标**: 学习如何使用 WLAN 提供的接口

1. **[03_Public_API_NAPI.md](03_Public_API_NAPI.md)** (120 分钟)
   - 学习所有可用的 JS API
   - 了解 API 参数、返回值、错误码
   - 掌握事件订阅机制
   - 阅读示例代码

2. **[04_Inner_API.md](04_Inner_API.md)** (可选，60 分钟)
   - 了解内部 API 结构
   - 理解模块间接口
   - 如需参与 WLAN 组件开发

---

### 路径四：构建与部署（1 小时）
**目标**: 了解如何构建和部署 WLAN 组件

1. **[05_GN_Targets.md](05_GN_Targets.md)** (60 分钟)
   - 理解 GN 构建系统
   - 了解 feature flags
   - 掌握目标依赖关系

2. **[06_Build_Artifacts.md](06_Build_Artifacts.md)** (45 分钟)
   - 了解编译产物
   - 知道安装路径
   - 理解运行时加载关系

---

### 路径五：安全与调试（2 小时）
**目标**: 了解安全考虑和常见问题

1. **[07_Security_Review.md](07_Security_Review.md)** (90 分钟)
   - 了解安全风险
   - 掌握权限验证机制
   - 学习安全最佳实践

2. **[08_FAQ.md](08_FAQ.md)** (30 分钟)
   - 查看常见问题
   - 学习调试技巧
   - 了解定位路径

---

### 路径六：深度学习（可选）
**目标**: 针对特定需求深入学习

1. **[appendix/Callgraphs.md](appendix/Callgraphs.md)** (可选)
   - 深入理解关键调用链

2. **[appendix/Config_Flags.md](appendix/Config_Flags.md)** (可选)
   - 详细了解 feature flags

---

## 文档分类

### 入门类
- 📖 [00_Overview.md](00_Overview.md) - 从这里开始
- 📁 [01_Directory_Structure.md](01_Directory_Structure.md) - 了解代码组织

### 架构类
- 🏗 [02_Architecture.md](02_Architecture.md) - 系统设计详解

### API 类
- 📚 [03_Public_API_NAPI.md](03_Public_API_NAPI.md) - JS 开发必读
- 🔧 [04_Inner_API.md](04_Inner_API.md) - 组件开发参考

### 构建类
- 🔨 [05_GN_Targets.md](05_GN_Targets.md) - 构建系统说明
- 📦 [06_Build_Artifacts.md](06_Build_Artifacts.md) - 部署指南

### 安全与调试类
- 🔒 [07_Security_Review.md](07_Security_Review.md) - 安全风险分析
- ❓ [08_FAQ.md](08_FAQ.md) - 问题排查手册

### 附录类
- 📋 [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链参考
- ⚙️ [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置参考

---

## 按角色导航

### 应用开发者
1. [00_Overview.md](00_Overview.md)
2. [01_Directory_Structure.md](01_Directory_Structure.md)
3. [03_Public_API_NAPI.md](03_Public_API_NAPI.md)

### 系统开发者
1. [00_Overview.md](00_Overview.md)
2. [01_Directory_Structure.md](01_Directory_Structure.md)
3. [02_Architecture.md](02_Architecture.md)
4. [04_Inner_API.md](04_Inner_API.md)
5. [05_GN_Targets.md](05_GN_Targets.md)
6. [06_Build_Artifacts.md](06_Build_Artifacts.md)

## 安全研究路径

### 快速审计路径（2 小时）
**目标**: 快速识别攻击面、信任边界和潜在风险点

1. **[00_Overview.md](00_Overview.md)** (15 分钟)
   - 了解项目定位和核心能力
   - 识别对外暴露面（N-API、IPC、配置文件）

2. **[03_Public_API_NAPI.md](03_Public_API_NAPI.md)** (45 分钟)
   - 识别所有外部输入入口（JS API 参数、事件回调）
   - 了解权限校验点分布

3. **[07_Security_Review.md](07_Security_Review.md)** (60 分钟)
   - 攻击面全景分析
   - 信任边界与数据流
   - 已知风险点与利用路径
   - 修复建议

---

### 深度审计路径（4 小时）
**目标**: 全面安全评估，发现潜在漏洞

1. **[00_Overview.md](00_Overview.md)** (15 分钟)
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** (15 分钟)
   - 定位核心安全相关文件
3. **[02_Architecture.md](02_Architecture.md)** (60 分钟)
   - 深入理解信任边界跨越点
   - 分析 IPC 通信安全
   - 理解权限检查时机
4. **[03_Public_API_NAPI.md](03_Public_API_NAPI.md)** (45 分钟)
5. **[04_Inner_API.md](04_Inner_API.md)** (45 分钟)
   - 了解内部接口权限模型
6. **[07_Security_Review.md](07_Security_Review.md)** (90 分钟)

---

## 按角色导航

### 应用开发者
1. [00_Overview.md](00_Overview.md)
2. [01_Directory_Structure.md](01_Directory_Structure.md)
3. [03_Public_API_NAPI.md](03_Public_API_NAPI.md)

### 系统开发者
1. [00_Overview.md](00_Overview.md)
2. [01_Directory_Structure.md](01_Directory_Structure.md)
3. [02_Architecture.md](02_Architecture.md)
4. [04_Inner_API.md](04_Inner_API.md)
5. [05_GN_Targets.md](05_GN_Targets.md)
6. [06_Build_Artifacts.md](06_Build_Artifacts.md)

### 安全审计人员
1. [00_Overview.md](00_Overview.md) - 了解项目定位和暴露面
2. [03_Public_API_NAPI.md](03_Public_API_NAPI.md) - 识别外部输入入口
3. [07_Security_Review.md](07_Security_Review.md) - 安全风险深度分析
4. [02_Architecture.md](02_Architecture.md) - 理解信任边界（补充）

### 架构师
阅读所有文档，重点关注：
- [02_Architecture.md](02_Architecture.md)
- [05_GN_Targets.md](05_GN_Targets.md)
- [appendix/Callgraphs.md](appendix/Callgraphs.md)

---

## 搜索提示

### 按主题查找
- **API 使用**: 搜索 "enableWifi"、"scan"、"connect" 等
- **权限**: 搜索 "permission"、"VerifyPermission"
- **架构**: 搜索 "SystemAbility"、"Proxy"、"Stub"
- **构建**: 搜索 "wifi_feature_"、"BUILD.gn"
- **安全**: 搜索 "security"、"vulnerability"

### 按文件类型查找
- **头文件** (`.h`)：查看接口定义和数据结构
- **实现文件** (`.cpp`)：查看业务逻辑
- **配置文件** (`.gni`、`.cfg`)：查看编译和运行时配置
- **脚本文件**：查看构建和初始化逻辑

---

## 更新日志

| 日期 | 更新内容 |
|------|---------|
| 2026-02-06 | 创建 SUMMARY.md，提供完整导航和新人阅读路径 |

---

**提示**: 建议按"新人学习路径"顺序阅读文档，可快速掌握 WLAN 组件的知识体系。
