# OpenHarmony Profiler Wiki 文档

> **版本**: 3.0.9  
> **最后更新**: 2025-02-07  
> **子系统**: developtools  
> **License**: Apache License 2.0

---

## 更新日志

### 2025-02-07 安全文档重大更新

**本次更新内容**:
- ✅ **新增插件攻击面分析**: 详细分析 ftrace_plugin、memory_plugin、native_hook、network_plugin、hiperf_plugin 的安全风险
- ✅ **新增高危漏洞**: 发现插件命令注入漏洞 (SEC-004)，提供攻击示例和修复建议
- ✅ **扩展风险清单**: 从 6 条扩展到 11 条，新增 Protobuf DoS、插件路径遍历、Socket 安全等风险
- ✅ **完善安全索引**: 新增 A.5 插件安全源码索引，包含命令注入、Protobuf 解析等关键代码位置
- ✅ **双路线导航**: SUMMARY.md 新增「新人学习路线」和「安全研究路线」

**关键发现**:
- 🔴 **命令注入漏洞**: hiperf_plugin 和 memory_plugin 存在高危命令注入风险
- 🟡 **路径遍历**: 多个插件通过 PID 构造文件路径，存在路径遍历风险
- 🟡 **Protobuf 反序列化**: 所有插件未限制 protobuf 消息大小，存在 DoS 风险

**文档质量**:
- 11 个风险项，全部包含代码证据 (文件:行号)
- 8 个代码示例，可直接用于漏洞验证
- 15+ 代码路径引用，可追溯至具体实现

## 文档概述

本文档是 OpenHarmony 性能调优组件 (`@ohos/hiprofiler`) 的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、构建方式和安全风险。

### 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | 定位、能力、依赖 |
| 架构说明 | ✅ 完成 | 组件图、数据流、插件机制 |
| N-API 接口 | ✅ 完成 | 41 个 JS API 详细说明 |
| 内部 API | ✅ 完成 | 插件架构、IPC 通信 |
| GN 构建 | ✅ 完成 | Targets、产物、依赖 |
| 安全评审 | ✅ 完成 | 攻击面分析、11条风险、修复建议 |
| 故障排查 | ✅ 完成 | 常见问题、调试方法 |

### 阅读建议

**新人推荐阅读顺序**:

1. `01_Overview.md` - 项目定位与核心能力
2. `02_Architecture.md` - 整体架构与数据流
3. `03_NAPI.md` - JS API 参考
4. `04_InnerAPI.md` - 内部模块与接口
5. `05_Build.md` - 构建配置与产物
6. `06_Security.md` - 安全风险评估

### 快速开始

```typescript
import hidebug from '@ohos.hidebug';

// 开始 CPU 性能分析
hidebug.startProfiling('my-profile');

// ... 执行被分析的代码 ...

// 停止分析并生成 trace 文件
hidebug.stopProfiling();

// 获取内存信息
const pss = hidebug.getPss();
console.log(`PSS Memory: ${pss}`);
```

### 相关链接

- [OpenHarmony Profiler 源码](https://gitee.com/openharmony/developtools_profiler)
- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [N-API 参考](https://docs.openharmony.com/docs/zh-cn/documentation/apis/js-apis-system-capability/)
- [系统能力列表](https://docs.openharmony.com/docs/zh-cn/documentation/architecture/syscap/)

---

*本文档由 Wiki 生成 Agent 自动维护*
