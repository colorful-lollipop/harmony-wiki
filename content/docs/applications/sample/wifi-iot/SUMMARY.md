# Wifi IoT Sample 应用 Wiki 导航

> **新人阅读顺序**: 按章节顺序阅读，可快速了解项目全貌

---

## 📚 文档导航

### [1. 项目概览](01_Project_Overview.md)
- 项目定位与边界
- 核心能力与运行环境
- 关键概念说明
- 适用场景

### [2. 目录结构与模块职责](02_Directory_Structure.md)
- 顶层目录树
- 各模块职责说明
- 文件组织规范
- 依赖关系图

### [3. 架构说明](03_Architecture.md)
- 组件图与模块关系
- 数据流向
- 线程模型（CMSIS-OS2）
- 关键时序图（SAMGR 服务注册与调用）

### [4. 对外 API 文档](04_N-API_External.md)
> 注：本项目为纯 C 代码，无 JS/N-API 绑定

- SAMGR_Lite 服务接口
- GPIO 硬件操作接口
- Demo SDK 接口
- API 清单与使用示例

### [5. 内部 API](05_Inner_API.md)
- 模块间接口定义
- SAMGR 内部接口
- 依赖方向分析
- 稳定性标注

### [6. GN 构建系统](06_GN_Build.md)
- BUILD.gn 配置结构
- Target 清单（按模块分组）
- 依赖关系
- 编译选项与宏定义

### [7. 编译产物](07_Build_Artifacts.md)
- 静态库产物（.a）
- 最终可执行文件
- 安装路径说明
- 运行时加载关系

### [8. 安全风险评审](08_Security_Review.md)
- 攻击面分析
- 信任边界
- 安全机制评估
- 风险点与修复建议

### [9. 常见问题与调试](09_QA_Troubleshooting.md)
- 常见构建问题
- 运行时调试技巧
- 日志分析方法

---

## 📂 附录

### [附录 A: 关键调用链](appendix/Callgraphs.md)
- 服务注册流程
- 特性调用流程
- 广播消息流程
- GPIO 操作流程

### [附录 B: 配置宏与 Feature Flags](appendix/Config_Flags.md)
- 构建时配置
- 运行时配置
- 关键常量定义

---

## 🔗 快速导航

| 需求 | 链接 |
|------|------|
| 快速了解项目 | [项目概览](01_Project_Overview.md) |
| 理解代码结构 | [目录结构](02_Directory_Structure.md) |
| 学习 SAMGR_Lite | [架构说明](03_Architecture.md) |
| 调用对外接口 | [对外 API](04_N-API_External.md) |
| 了解构建流程 | [GN 构建](06_GN_Build.md) |
| 产物安装说明 | [编译产物](07_Build_Artifacts.md) |
| 安全评估 | [安全评审](08_Security_Review.md) |
| 排查问题 | [常见问题](09_QA_Troubleshooting.md) |

---

## 📊 项目统计

| 指标 | 数值 |
|------|------|
| C 源文件数 | 16 个 |
| 总代码行数 | ~1765 行 |
| GN Targets | 4 个 |
| 依赖组件 | 5 个 |
| 模块数量 | 4 个 |

---

## 🎯 按角色阅读

### 新手开发者
1. 项目概览 → 2. 目录结构 → 3. 架构说明 → 4. 对外 API

### 构建工程师
1. 项目概览 → 6. GN 构建 → 7. 编译产物

### 安全评审员
1. 项目概览 → 3. 架构说明 → 8. 安全风险评审

### 维护开发者
1. 项目概览 → 2. 目录结构 → 5. 内部 API → 9. 常见问题
