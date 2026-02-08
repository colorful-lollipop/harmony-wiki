# OpenHiTLS Wiki 导航

**本文档重点说明 openHiTLS 在 OpenHarmony 中的集成与适配方式，原始库功能仅作简要介绍。**

---

## 文档列表

### 入门必读

| 文档 | 说明 |
|-----|------|
| [01_Overview.md](./01_Overview.md) | 库简介、OH 中的定位、版本信息 |
| [02_Patches.md](./02_Patches.md) | **核心文档** - Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 详解、编译配置、特性开关 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、curl 集成示例 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异（本库无差异） |
| [06_Security.md](./06_Security.md) | 安全分析、CVE 跟踪建议 |

### 工作文档

| 文档 | 说明 |
|-----|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 评估报告 - 完整的信息收集结果 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |

---

## 快速概览

### 这是什么库？

**openHiTLS** 是华为开发的开源密码学和 TLS 库，为 OpenHarmony 提供：
- 国密支持：SM2/SM3/SM4/TLCP
- 标准 TLS：TLS 1.3/1.2, DTLS
- 后量子算法：ML-DSA, ML-KEM, SLH-DSA
- 高性能：ARMv8/x86_64 汇编优化

### OH 集成特点

| 特点 | 说明 |
|-----|------|
| **无 Patch** | 原生支持 OH，无需代码修改 |
| **5 个组件** | bsl, crypto, pki, tls, auth 按需使用 |
| **NDK 可用** | 应用层可通过 NDK 调用 |
| **curl 集成** | 主要用于 curl 的国密 HTTPS 支持 |

### 一句话总结

> openHiTLS 是 OpenHarmony 的国密 TLS 解决方案，通过无侵入式 BUILD.gn 配置集成，为 curl 等模块提供 SM2/SM3/SM4/TLCP 国密能力。

---

## 阅读建议

### 如果你是...

**应用开发者** → 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 了解如何通过 NDK 使用

**系统开发者** → 阅读 [03_Build_Integration.md](./03_Build_Integration.md) 了解 BUILD.gn 配置

**升级维护者** → 阅读 [02_Patches.md](./02_Patches.md) 和 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md)

**安全工程师** → 阅读 [06_Security.md](./06_Security.md) 了解安全跟踪策略

---

## 相关链接

- 上游项目：https://openhitls.net / https://gitcode.com/openhitls
- OpenHarmony 文档：https://docs.openharmony.cn
- Mulan PSL v2：http://license.coscl.org.cn/MulanPSL2
