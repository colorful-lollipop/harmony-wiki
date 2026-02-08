# SUMMARY - 文档导航

> 新人阅读顺序：建议按以下顺序阅读，快速理解项目。

## 快速入门

1. **[00_Overview.md](00_Overview.md)** - 项目概览、核心能力、架构定位
2. **[README.md](README.md)** - 本 Wiki 的说明

## 核心文档（必读）

3. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构、模块职责
4. **[02_Architecture.md](02_Architecture.md)** - 整体架构、数据流、线程模型、状态机
5. **[03_Inner_API.md](03_Inner_API.md)** - SDK 接口、回调定义、错误码

## 进阶文档

6. **[04_Build_System.md](04_Build_System.md)** - GN 构建 Targets、编译产物、条件编译
7. **[05_Security_Review.md](05_Security_Review.md)** - 安全机制、攻击面、风险评估
8. **[06_Troubleshooting.md](06_Troubleshooting.md)** - 构建/运行/调试问题排查

## 附录

- **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
- **[appendix/Config_Flags.md](appendix/Config_Flags.md)** - 关键宏/Feature Flags

---

## 新人阅读路线图

```
┌─────────────────────────────────────────────────────────────────┐
│                     新人阅读路线                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: 项目认知                                                 │
│  ┌──────────────────────────────┐                               │
│  │  00_Overview.md              │ → 理解项目定位、核心能力         │
│  └──────────────────────────────┘                               │
│                                                                  │
│  Step 2: 结构理解                                                 │
│  ┌──────────────────────────────┐                               │
│  │  01_Directory_Structure.md  │ → 熟悉模块划分、代码位置          │
│  └──────────────────────────────┘                               │
│                                                                  │
│  Step 3: 架构掌握                                                 │
│  ┌──────────────────────────────┐                               │
│  │  02_Architecture.md          │ → 理解数据流、线程模型           │
│  └──────────────────────────────┘                               │
│                                                                  │
│  Step 4: 接口熟悉                                                 │
│  ┌──────────────────────────────┐                               │
│  │  03_Inner_API.md             │ → 掌握 SDK 接口、错误码          │
│  └──────────────────────────────┘                               │
│                                                                  │
│  Step 5: 构建与安全 (进阶)                                        │
│  ┌──────────────────────────────┐                               │
│  │  04_Build_System.md          │ → 熟悉编译配置                   │
│  │  05_Security_Review.md      │ → 了解安全边界                   │
│  │  06_Troubleshooting.md     │ → 掌握调试方法                   │
│  └──────────────────────────────┘                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 文档速查

| 需要... | 查看文档 |
|---------|----------|
| 了解项目做什么 | 00_Overview.md |
| 找某个功能的代码 | 01_Directory_Structure.md |
| 理解数据如何流转 | 02_Architecture.md |
| 查找 API 使用方式 | 03_Inner_API.md |
| 修改编译配置 | 04_Build_System.md |
| 了解安全机制 | 05_Security_Review.md |
| 解决构建/运行问题 | 06_Troubleshooting.md |

---

## 术语表

| 术语 | 含义 |
|------|------|
| Source | 主控端，控制其他设备相机 |
| Sink | 被控端，分享本地相机 |
| SA | System Ability，系统能力 |
| HDF | Hardware Driver Foundation，硬件驱动框架 |
| IPC | Inter-Process Communication，进程间通信 |
| N-API | Native API，本项目无 |
| SoftBus | 分布式软总线 |
