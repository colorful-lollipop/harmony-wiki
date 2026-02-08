# 文档导航

本文档提供 State Registry 模块的完整技术参考，强烈建议按以下顺序阅读以获得最佳学习体验。

## 阅读路线

### 新人学习路线

```
1. 阅读 [00_Overview.md](00_Overview.md)          → 理解模块定位和核心能力
   ↓
2. 阅读 [01_Directory_Structure.md](01_Directory_Structure.md)  → 熟悉代码组织
   ↓
3. 阅读 [02_Architecture.md](02_Architecture.md)  → 掌握整体架构
   ↓
4. 阅读 [03_JS_API.md](03_JS_API.md)              → 学习 API 使用方式
   ↓
5. 如需深入 Native 实现 → [04_Native_API.md](04_Native_API.md)
   ↓
6. 了解构建配置 → [05_GN_Build.md](05_GN_Build.md)
   ↓
7. 查看产物清单 → [06_Build_Artifacts.md](06_Build_Artifacts.md)
   ↓
8. 安全关注点 → [07_Security_Review.md](07_Security_Review.md)
```

### 安全研究路线

```
1. 快速了解 → [00_Overview.md](00_Overview.md) (重点看能力边界和权限)
   ↓
2. 攻击面分析 → [05_AttackSurface.md](05_AttackSurface.md) (必读)
   ↓
   ├─ 外部输入入口
   ├─ 信任边界图
   ├─ 敏感操作清单
   └─ 攻击路径分析
   ↓
3. 接口审计 → [04_Native_API.md](04_Native_API.md) (IPC 接口码、调用链)
   ↓
4. 深度安全评估 → [07_Security_Review.md](07_Security_Review.md) (风险分析)
   ↓
5. 架构理解 → [02_Architecture.md](02_Architecture.md) (数据流、组件交互)
```

## 全站文档列表

### 入门与概览

| 文档 | 说明 |
|------|------|
| [README.md](README.md) | Wiki 使用指南、覆盖范围、更新方式 |
| [00_Overview.md](00_Overview.md) | 项目定位、能力边界、运行环境 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构、模块职责 |

### 架构与 API

| 文档 | 说明 |
|------|------|
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型 |
| [03_JS_API.md](03_JS_API.md) | N-API 接口、JS API 清单 |
| [04_Native_API.md](04_Native_API.md) | Inner API、内部模块接口 |

### 构建与部署

| 文档 | 说明 |
|------|------|
| [05_GN_Build.md](05_GN_Build.md) | GN Targets、构建配置 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 产物清单、安装路径 |

### 安全与运维

| 文档 | 说明 |
|------|------|
| [05_AttackSurface.md](05_AttackSurface.md) | **攻击面分析** - 外部输入、信任边界、敏感操作 |
| [07_Security_Review.md](07_Security_Review.md) | 安全风险评估、修复建议 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 常见问题、调试方法 |

## 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键配置开关 |

## 相关链接

- **项目仓库**：https://gitee.com/openharmony/telephony_state_registry
- **OpenHarmony**：https://gitee.com/openharmony
- **Telephony 子系统**：https://gitee.com/openharmony/telephony
- **API 参考**：https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-telephony-kit/
