# Wiki 导航

## 安全研究路线

```
🛡️ 安全研究员专用路线

1️⃣ 攻击面识别
   └── 09_AttackSurface.md ← 外部输入清单、信任边界
   
2️⃣ 风险评估
   └── 10_SecurityReview.md ← 漏洞分析、修复建议
   
3️⃣ 架构安全
   └── 03_Architecture.md ← 线程模型、数据流
   
4️⃣ 接口安全
   ├── 04_N-API.md ← N-API 安全考虑
   └── 05_Inner_API.md ← Native API 安全考虑
   
5️⃣ 构建安全
   └── 06_Build.md ← Feature Flags 安全影响
```

## 全站目录

### 必读文档

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README](README.md) | Wiki 使用说明与更新方式 | ⭐⭐⭐ |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境 | ⭐⭐⭐ |
| [SUMMARY](SUMMARY.md) | 本导航文档 | ⭐⭐⭐ |

### 架构与结构

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块职责 | ⭐⭐⭐ |
| [03_Architecture](03_Architecture.md) | 架构设计、数据流、线程模型 | ⭐⭐⭐ |

### API 接口

| 文档 | 描述 | 语言 |
|------|------|------|
| [04_N-API](04_N-API.md) | JS/ArkTS 对外接口 | JavaScript/ArkTS |
| [05_Inner_API](05_Inner_API.md) | Native Inner API | C/C++ |

### 构建与部署

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [06_Build](06_Build.md) | GN Targets、编译产物、Feature Flags | ⭐⭐⭐ |
| [07_Security](07_Security.md) | 安全风险评审与修复建议 | ⭐⭐⭐ |
| [08_Troubleshooting](08_Troubleshooting.md) | 常见问题与调试指南 | ⭐⭐ |

### 附录（可选）

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图谱 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 配置项与 Feature Flags 详解 |

### 安全研究（新增）

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [09_AttackSurface](09_AttackSurface.md) | 攻击面分析、外部输入清单 | ⭐⭐⭐ |
| [10_SecurityReview](10_SecurityReview.md) | 安全风险评估、漏洞清单 | ⭐⭐⭐ |

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图谱 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 配置项与 Feature Flags 详解 |

## 模块索引

### 按功能分类

| 功能 | 相关文档 | 关键文件 |
|------|----------|----------|
| IPC 核心 | 03_Architecture, 05_Inner_API | `ipc_skeleton.h`, `iremote_object.h` |
| 消息传递 | 03_Architecture, 04_N-API | `message_parcel.h`, `message_option.h` |
| 分布式 IPC | 03_Architecture, 05_Inner_API | `dbinder_service.h` |
| JS 接口 | 04_N-API | `napi_rpc_native_module.cpp` |
| 构建系统 | 06_Build | `BUILD.gn`, `config.gni` |
| 攻击面分析 | 09_AttackSurface | N/API 输入点分析 |
| 安全风险评估 | 10_SecurityReview | 漏洞清单与修复建议 |

### 按受众分类

| 受众 | 推荐文档 |
|------|----------|
| 新人学习者 | 01_Overview → 02_Directory_Structure → 03_Architecture → 04_N-API/05_Inner_API |
| 安全研究员 | 09_AttackSurface → 10_SecurityReview → 03_Architecture → 05_Inner_API |
| 系统开发者 | 05_Inner_API → 03_Architecture → 06_Build → 07_Security |
| 应用开发者 | 01_Overview → 04_N-API → 08_Troubleshooting |

## 版本信息

- **组件版本**：3.0
- **Wiki 最后更新**：2026-02-07
- **OpenHarmony 版本**：标准系统/小型系统/轻量系统
- **文档版本**：2.0（含新增安全文档）

## 贡献指南

发现文档错误或需要补充？请：

1. 在 `wiki/_work/NOTES.md` 中记录发现
2. 修改对应 Wiki 文档
3. 提交 PR 至 OpenHarmony 仓库

### 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-07 | 2.0 | 新增 09_AttackSurface.md、10_SecurityReview.md 安全研究文档 |
| 2026-02-06 | 1.0 | 初始 Wiki 版本 |

---

*本文档为 OpenHarmony IPC 组件工程 Wiki*
