# OpenHarmony libwebsockets Wiki

## 库概览

libwebsockets 是 OpenHarmony 系统中用于 **WebSocket 协议** 支持的核心第三方库。

### 关键信息

| 属性 | 内容 |
|-----|------|
| **库名称** | libwebsockets |
| **版本** | 4.3.3 |
| **原始许可证** | MIT License |
| **OH 许可证** | Apache-2.0 |
| **上游地址** | https://libwebsockets.org |

### 在 OpenHarmony 中的角色

libwebsockets 为 OpenHarmony 提供 **WebSocket 客户端和服务器** 功能支持，是 netstack 子系统的核心依赖之一。

**典型使用场景**:
- 应用层通过 WebSocket API 进行双向实时通信
- IDE Previewer 工具的网络通信功能
- 系统服务的 WebSocket 连接管理

### OH 适配特点

1. **精简的 Patch 集**: 仅 1 个 iOS 平台适配 patch
2. **完整的 GN 构建**: 支持标准系统和跨平台（iOS）
3. **双 target 设计**: `websockets`（系统用）和 `websockets_static`（IDE 工具用）

---

## 文档导航

| 文档 | 内容 |
|-----|-----|
| [01_Overview.md](01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](02_Patches.md) | **Patch 详细分析**（核心文档） |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配说明 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](05_API_Differences.md) | API/接口差异（如有） |
| [06_Security.md](06_Security.md) | 安全风险分析 |

---

## 快速参考

### 依赖关系（简化）

```
应用层 (JS/ArkTS/Cangjie/C)
    ↓
netstack WebSocket API
    ↓
libwebsockets ←── openssl, zlib
```

### 主要依赖者

1. **netstack** - WebSocket 功能实现
2. **IDE Previewer** - 开发工具网络通信

### 关键文件

- `BUILD.gn` - GN 构建配置
- `for_ios.patch` - iOS 平台适配
- `for_ios.sh` - iOS 构建脚本

---

## 维护信息

- **评估报告**: [ASSESSMENT.md](_work/ASSESSMENT.md)
- **分析笔记**: [NOTES.md](_work/NOTES.md)
- **任务计划**: [PLAN.md](_work/PLAN.md)

---

*最后更新: 2025-02-07*
