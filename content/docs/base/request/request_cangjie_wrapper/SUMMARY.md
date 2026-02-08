# 文档导航

本文档为 OpenHarmony `request_cangjie_wrapper` 子系统的工程 Wiki，提供项目定位、架构设计、API 接口、构建方式及安全风险的完整文档。

---

## 全站导航

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README](README.md) | 项目概览、文档导航、更新说明 | ⭐⭐⭐ |
| [SUMMARY](SUMMARY.md) | 全站导航、双路线推荐 | ⭐⭐⭐ |
| [01_Overview](01_Overview.md) | 项目定位、能力边界、运行环境 | ⭐⭐⭐ |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、层次结构 | ⭐⭐ |
| [03_CodeMap](03_CodeMap.md) | 目录结构、核心文件定位 | ⭐⭐ |
| [04_Interface](04_Interface.md) | N-API 清单、IPC 接口、配置 | ⭐⭐⭐ |
| [05_AttackSurface](05_AttackSurface.md) | 外部输入、敏感操作、信任边界 | ⭐⭐⭐ |
| [06_SecurityReview](06_SecurityReview.md) | 安全风险、修复建议 | ⭐⭐ |
| [07_Build](07_Build.md) | GN targets、编译产物、Feature 开关 | ⭐ |
| [08_Internals](08_Internals.md) | 核心类、生命周期、内部 API | ⭐ |

---

## 新人学习路线

> **目标**: 快速理解项目并上手使用

### 第一阶段：理解项目 (15 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | 项目是什么？能做什么？ | 5 min | [01_Overview.md](01_Overview.md#11-项目定义) |
| 2 | 核心能力边界 | 5 min | [01_Overview.md](01_Overview.md#12-能力边界) |
| 3 | 运行环境要求 | 5 min | [01_Overview.md](01_Overview.md#13-运行环境) |

### 第二阶段：架构认知 (15 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | 模块划分与依赖 | 5 min | [02_Architecture.md](02_Architecture.md#21-组件图) |
| 2 | 数据流转路径 | 5 min | [02_Architecture.md](02_Architecture.md#22-数据流) |
| 3 | 任务生命周期 | 5 min | [02_Architecture.md](02_Architecture.md#23-生命周期) |

### 第三阶段：API 掌握 (30 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | Task 任务管理 | 10 min | [04_Interface.md](04_Interface.md#41-task-类) |
| 2 | Config 配置说明 | 10 min | [04_Interface.md](04_Interface.md#42-config-类) |
| 3 | 事件回调机制 | 10 min | [04_Interface.md](04_Interface.md#43-事件回调) |

### 第四阶段：快速开始 (15 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | 权限申请 | 5 min | [01_Overview.md](01_Overview.md#14-快速开始) |
| 2 | 创建下载任务 | 5 min | [01_Overview.md](01_Overview.md#14-快速开始) |
| 3 | 订阅进度事件 | 5 min | [01_Overview.md](01_Overview.md#14-快速开始) |

### 推荐阅读顺序

```
1. README.md → 了解文档结构
2. 01_Overview.md → 理解项目定位
3. 03_CodeMap.md → 熟悉代码结构
4. 04_Interface.md → 掌握 API 使用
5. 02_Architecture.md → 深入架构设计
```

---

## 安全研究路线

> **目标**: 识别攻击面、评估风险、定位漏洞

### 第一阶段：攻击面识别 (20 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | 外部输入清单 | 10 min | [05_AttackSurface.md](05_AttackSurface.md#21-外部输入清单) |
| 2 | 敏感操作分析 | 5 min | [05_AttackSurface.md](05_AttackSurface.md#22-敏感操作) |
| 3 | 信任边界图 | 5 min | [05_AttackSurface.md](05_AttackSurface.md#23-信任边界) |

### 第二阶段：风险评估 (30 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | 输入验证缺陷 | 10 min | [06_SecurityReview.md](06_SecurityReview.md#31-输入验证) |
| 2 | 内存安全问题 | 5 min | [06_SecurityReview.md](06_SecurityReview.md#32-内存安全) |
| 3 | 权限与鉴权 | 5 min | [06_SecurityReview.md](06_SecurityReview.md#33-权限鉴权) |
| 4 | 并发安全 | 5 min | [06_SecurityReview.md](06_SecurityReview.md#34-并发安全) |
| 5 | 逻辑漏洞 | 5 min | [06_SecurityReview.md](06_SecurityReview.md#35-逻辑漏洞) |

### 第三阶段：代码审计 (45 分钟)

| 步骤 | 内容 | 时间 | 文档章节 |
|------|------|------|----------|
| 1 | N-API 入口点 | 10 min | [04_Interface.md](04_Interface.md#41-n-api-入口) |
| 2 | 文件操作路径 | 15 min | [NOTES.md](NOTES.md#1-外部输入点) |
| 3 | 网络请求处理 | 10 min | [NOTES.md](NOTES.md#7-调用链分析) |
| 4 | 回调处理机制 | 10 min | [08_Internals.md](08_Internals.md#32-回调管理) |

### 推荐审计清单

```
□ 检查 Config.url 输入验证 [证据: agent.cj:596]
□ 检查 FileSpec.path 路径处理 [证据: agent.cj:358]
□ 检查 Config.saveas 写入路径 [证据: agent.cj:687]
□ 检查 Config.headers HTTP 头 [证据: agent.cj:661]
□ 检查 token 安全机制 [证据: agent.cj:804]
□ 验证 FFI 接口参数传递 [证据: ffi.cj:793-819]
```

---

## 按角色阅读

| 角色 | 推荐阅读 | 优先级顺序 |
|------|----------|-----------|
| **Cangjie 应用开发者** | 概览 → API → 快速开始 | 01 → 04 → 01(快速开始) |
| **系统集成开发者** | 概览 → 架构 → 构建 | 01 → 02 → 07 |
| **安全审计人员** | 攻击面 → 安全评审 → 接口 | 05 → 06 → 04 |
| **贡献者** | 架构 → 代码地图 → 内部实现 | 02 → 03 → 08 |
| **架构师** | 概览 → 架构 → 安全 | 01 → 02 → 06 |

---

## 快速参考

### 核心类速查

| 类名 | 文件:行号 | 职责 |
|------|-----------|------|
| `Task` | agent.cj:1661 | 任务主入口 |
| `Config` | agent.cj:578 | 任务配置 |
| `FileSpec` | agent.cj:342 | 文件规格 |
| `Progress` | agent.cj:1076 | 进度信息 |
| `HttpResponse` | agent.cj:1140 | HTTP 响应 |

### 错误码速查

| 错误码 | 常量名 | 含义 |
|--------|--------|------|
| 13400001 | EXCEPTION_FILEIO | 文件 IO 错误 |
| 13400002 | EXCEPTION_FILEPATH | 路径错误 |
| 13400003 | EXCEPTION_SERVICE | 服务错误 |
| 13499999 | EXCEPTION_OTHERS | 其他错误 |

### 依赖组件

| 组件 | 用途 |
|------|------|
| `cangjie_ark_interop` | Cangjie 注解、FFI、异常 |
| `arkui_cangjie_wrapper` | 基础类型、CString 转换 |
| `hiviewdfx_cangjie_wrapper` | 日志 (hilog) |
| `ability_cangjie_wrapper` | Ability 上下文 |
| `request` | 底层 FFI 接口 |

---

## 版本信息

| 项目 | 版本 | 更新日期 |
|------|------|----------|
| Wiki | 1.0 | 2026-02-07 |
| 代码 | 6.1 | - |
| API Level | 22 | - |

---

## 外部链接

- [API 参考文档 (外部)](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/BasicServicesKit/cj-apis-request-agent.md)
- [开发指南 (外部)](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/basic-services/request/cj-app-file-upload-download.md)
- [上游 request_request 仓库](https://gitcode.com/openharmony/request_request/blob/master/README.md)
