# 导航摘要

本文档提供 OpenHarmony Update Service Wiki 的完整导航。

## 文档索引

### 快速入门

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](README.md) | Wiki 使用说明 | ⭐⭐⭐ |
| [00_Overview.md](00_Overview.md) | 项目概览 | ⭐⭐⭐ |
| [SUMMARY.md](SUMMARY.md) | 本文档 | ⭐⭐⭐ |

### 架构设计

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [01_Architecture.md](01_Architecture.md) | 系统架构详解 | ⭐⭐⭐ |
| [03_Inner_API.md](03_Inner_API.md) | Inner API 与 IPC | ⭐⭐ |

### API 参考

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [02_N-API.md](02_N-API.md) | JS API 完整参考 | ⭐⭐⭐ |

### 构建部署

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [04_GN_Build.md](04_GN_Build.md) | GN 构建配置 | ⭐⭐ |

### 安全运维

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [05_Security.md](05_Security.md) | 安全风险分析 | ⭐⭐⭐ |
| [06_Troubleshooting.md](06_Troubleshooting.md) | 常见问题 | ⭐⭐ |

---

## 新人阅读路径

### 路径 1: API 使用者 (15分钟)

```
1. [00_Overview.md](00_Overview.md)     → 了解功能
2. [02_N-API.md](02_N-API.md)           → 查找 API
```

### 路径 2: 开发者 (30分钟)

```
1. [00_Overview.md](00_Overview.md)     → 了解功能
2. [01_Architecture.md](01_Architecture.md) → 理解架构
3. [02_N-API.md](02_N-API.md)           → API 实现
4. [04_GN_Build.md](04_GN_Build.md)     → 构建配置
```

### 路径 3: 安全审计 (45分钟)

```
1. [00_Overview.md](00_Overview.md)     → 了解功能
2. [01_Architecture.md](01_Architecture.md) → 架构
3. [05_Security.md](05_Security.md)   → 安全评审
4. [03_Inner_API.md](03_Inner_API.md)  → IPC 权限
```

---

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| SA | System Ability | 系统能力 |
| N-API | Node-API | Node.js 风格 API |
| OTA | Over-The-Air | 空中下载升级 |
| IPC | Inter-Process Communication | 进程间通信 |
| IDL | Interface Definition Language | 接口定义语言 |
