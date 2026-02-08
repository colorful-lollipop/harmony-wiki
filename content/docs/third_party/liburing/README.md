# liburing Wiki 项目

本文档是 OpenHarmony 第三方库 `liburing` 的 Wiki 文档，详细记录了该库在 OpenHarmony 中的集成、适配和使用情况。

---

## 快速导航

### 核心文档

| 文档 | 内容概述 | 推荐阅读顺序 |
|-----|---------|------------|
| [01_Overview.md](./01_Overview.md) | 库概览、OH 定位、功能介绍 | 1️⃣ |
| [02_Patches.md](./02_Patches.md) | Patch 分析 (本库无 Patch) | 2️⃣ |
| [03_Build_Integration.md](./03_Build_Integration.md) | GN 构建适配详解 | 3️⃣ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景 | 4️⃣ |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异分析 | 5️⃣ |
| [06_Security.md](./06_Security.md) | 安全风险与建议 | 6️⃣ |

### 项目工作文档

- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估报告
- [_work/NOTES.md](./_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](./_work/PLAN.md) - 任务进度跟踪

---

## 库概览

**liburing** 是 Linux [io_uring](https://kernel.dk/io_uring.pdf) 异步 I/O 子系统的官方用户态封装库。

### 关键信息

| 属性 | 值 |
|-----|-----|
| **原始库** | liburing |
| **上游版本** | 2.7 |
| **上游地址** | https://github.com/axboe/liburing |
| **许可证** | MIT / LGPL (双许可) |
| **OH 组件名** | @ohos/liburing |
| **Patch 数量** | **0** (无修改) |

### OpenHarmony 适配特点

1. ✅ **零 Patch 集成** - 完全使用上游原始代码
2. ✅ **GN 构建适配** - 使用 BUILD.gn 替代 Makefile
3. ⚠️ **依赖待确认** - 当前未发现直接依赖者
4. ✅ **精简编译** - 仅编译核心功能（4 个源文件）

### 核心功能

```
┌─────────────────────────────────────────────────────────┐
│                     liburing 功能架构                      │
├─────────────────────────────────────────────────────────┤
│  文件 I/O    │  read, write, readv, writev, pread, pwrite  │
├─────────────────────────────────────────────────────────┤
│  网络 I/O    │  accept, connect, send, recv, sendmsg, recvmsg │
├─────────────────────────────────────────────────────────┤
│  文件系统    │  openat, close, statx, mkdirat, unlinkat...    │
├─────────────────────────────────────────────────────────┤
│  高级特性    │  splice, tee, poll, epoll_ctl, uring_cmd        │
├─────────────────────────────────────────────────────────┤
│  缓冲管理    │  register_buffers, provided buffers             │
├─────────────────────────────────────────────────────────┤
│  同步原语    │  futex_wait, futex_wake, futex_waitv            │
└─────────────────────────────────────────────────────────┘
```

### 适用场景

liburing 适合以下高性能 I/O 场景：

- 🔥 **高并发网络服务** - 单线程处理数万连接
- 💾 **高速存储访问** - NVMe SSD 等低延迟设备
- 📡 **实时数据处理** - 需要确定性的 I/O 延迟
- 🔗 **零拷贝传输** - splice、sendfile 等场景

---

## 文档阅读建议

### 如果您是...

**应用开发者**:
> 阅读 [01_Overview.md](./01_Overview.md) 了解功能，参考上游文档编写代码

**系统集成工程师**:
> 重点阅读 [03_Build_Integration.md](./03_Build_Integration.md) 了解 GN 适配细节

**安全工程师**:
> 直接阅读 [06_Security.md](./06_Security.md) 了解 CVE 和升级策略

**维护者**:
> 建议按顺序阅读所有文档，特别关注 [02_Patches.md](./02_Patches.md) 和 [04_Usage_in_OH.md](./04_Usage_in_OH.md)

---

## 上游参考

- **GitHub**: https://github.com/axboe/liburing
- **技术论文**: https://kernel.dk/io_uring.pdf
- **邮件列表**: io-uring@vger.kernel.org
- **归档**: https://lore.kernel.org/io-uring/

---

## 贡献与反馈

如需更新本文档，请：
1. 确保遵循"证据优先原则"，所有结论需有代码/配置支撑
2. 更新相应的章节文档
3. 同步更新本 README 的导航信息
4. 在 _work/NOTES.md 中记录修改历史

---

*本文档最后更新: 2026-02-08*
