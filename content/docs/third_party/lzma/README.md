# LZMA SDK - OpenHarmony Wiki

## 库概览

**LZMA SDK** 是 OpenHarmony 第三方库中的重要压缩/解压基础设施，主要用于系统崩溃分析和调试功能。

---

## 快速导航

| 文档 | 描述 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | **核心文档**：Patch 详细分析 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异 (如有) |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

---

## 关键信息

### 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | 7 Zip - LZMA SDK |
| **版本** | 25.01 (上游) / 3.1 (OH 组件) |
| **许可证** | Public Domain |
| **上游地址** | https://7-zip.org/a/lzma2501.7z |
| **OH 组件** | @ohos/lzma |
| **子系统** | thirdparty |

### 在 OH 中的作用

```
┌─────────────────────────────────────────────────────────┐
│                 Faultloggerd 崩溃分析系统                 │
│                     (base/hiviewdfx/)                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ELF 文件  ──▶  压缩的 Unwind 信息  ──▶  LZMA 解压  ──▶  栈回溯 │
│                                                         │
│   用途：应用崩溃时获取调用栈，用于问题诊断和修复              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Patch 数量

| Patch 文件 | 状态 | 类型 |
|-----------|------|------|
| `add-linux-makefile-for-Format7zR.patch` | ✅ 已应用 | 构建适配 |

**说明**: 仅有一个构建相关 Patch，改动小，维护成本低。

### 架构支持

| 架构 | 设备端 | 主机端 | 特殊优化 |
|------|--------|--------|----------|
| ARM | ✅ | ❌ | - |
| ARM64 | ✅ | ✅ | PAC-RET + 汇编优化 |
| RISC-V 64 | ✅ | ✅ | - |
| X86_64 | ❌ | ✅ | 汇编优化 |

---

## 阅读建议

### 如果你是...

**系统开发者 (想了解如何使用)**
→ 直接阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 了解依赖方式

**构建工程师 (需修改编译配置)**
→ 阅读 [03_Build_Integration.md](./03_Build_Integration.md) 和 [02_Patches.md](./02_Patches.md)

**安全审计 (关注 CVE 和漏洞)**
→ 阅读 [06_Security.md](./06_Security.md)

**维护者 (需升级上游版本)**
→ 按顺序阅读：[02_Patches.md](./02_Patches.md) → [03_Build_Integration.md](./03_Build_Integration.md) → [04_Usage_in_OH.md](./04_Usage_in_OH.md)

---

## 相关链接

- [上游项目](https://www.7-zip.org/sdk.html)
- [OpenHarmony faultloggerd](https://gitee.com/openharmony/base_hiviewdfx_faultloggerd)
- [ASSESSMENT.md](./_work/ASSESSMENT.md) - 详细评估报告

---

*最后更新: 2025-02-08*
