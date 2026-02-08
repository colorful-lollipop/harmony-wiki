# 文档导航

本文档为 SELinux Adapter 组件的工程 Wiki，提供新人快速上手与开发者深度参考。

## 快速入门

### 新人阅读顺序（推荐）

```
1. README.md           → 了解 Wiki 覆盖范围与使用方式
2. 01_Overview.md     → 项目定位、核心能力、技术栈
3. 02_Architecture.md → 组件架构、数据流、线程模型
4. 03_API.md          → Inner API 接口详情
5. 04_Build.md        → GN 构建配置与产物
```

## 完整文档列表

### 核心文档

| 文档 | 标题 | 主要内容 |
|------|------|----------|
| [README.md](README.md) | Wiki 说明 | 覆盖范围、更新方式、阅读建议 |
| [01_Overview.md](01_Overview.md) | 项目概述 | 定位、边界、核心能力、运行环境 |
| [02_Architecture.md](02_Architecture.md) | 架构设计 | 组件图、数据流、线程模型、依赖关系 |
| [03_API.md](03_API.md) | Inner API | 5 个核心库的 C 接口详解 |
| [04_Build.md](04_Build.md) | 构建系统 | GN Targets、依赖、编译产物 |
| [05_Security.md](05_Security.md) | 安全评审 | 攻击面、信任边界、风险分析 |
| [06_Troubleshooting.md](06_Troubleshooting.md) | 故障排查 | 常见问题、调试方法、日志解读 |

### 附录

| 文档 | 标题 | 说明 |
|------|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 | 入口→核心逻辑调用链 |

### 工作文档

| 文档 | 标题 | 说明 |
|------|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估 | 项目画像、受众分析、文档策略 |
| [_work/NOTES.md](_work/NOTES.md) | 事实记录 | 代码证据汇总 |
| [_work/PLAN.md](_work/PLAN.md) | 任务计划 | 任务进度追踪 |

## 模块速查

| 模块 | 路径 | 对应产物 |
|------|------|----------|
| 策略加载 | `framework/policycoreutils/src/load_policy.cpp` | libload_policy.so |
| 文件标签恢复 | `framework/policycoreutils/src/selinux_restorecon.c` | librestorecon.so |
| HAP 上下文 | `framework/policycoreutils/src/hap_restorecon.cpp` | libhap_restorecon.so |
| 参数检查 | `framework/policycoreutils/src/param_checker.c` | libparaperm_checker.so |
| 服务检查 | `framework/policycoreutils/src/service_checker.cpp` | libservice_checker.so |

## 反馈与贡献

- 发现文档错误或遗漏：请提交 Issue
- 贡献新文档：创建 PR 并更新 SUMMARY.md
