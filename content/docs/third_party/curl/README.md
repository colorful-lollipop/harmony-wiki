# Curl - OpenHarmony 集成文档

> **curl 8.8.0** | **OpenHarmony 第三方库** | **版本 3.1**
>
> 本文档详细说明 OpenHarmony 对 curl 的集成、Patch、特殊适配、构建配置、依赖关系和 API 差异。

---

## 📖 文档概述（Overview）

### 文档目的

OpenHarmony 对 [curl](http://curl.haxx.se/) 库进行了深度定制化，本文档旨在：

1. **记录 OH 特定适配**：Patch、构建配置、特殊功能
2. **说明使用方式**：OH 模块如何集成和依赖 curl
3. **提供维护指南**：版本升级、安全更新、问题排查
4. **降低学习曲线**：帮助开发者快速理解 OH 定制化的内容

### 文档结构

```
wiki/
├── README.md                    # 本文件：文档导航和快速查找
├── SUMMARY.md                   # 快速查找指南（推荐首先阅读）
├── 01_Overview.md                # curl 库简介和在 OH 中的定位
├── 02_Patches.md               # ⭐ Patch 详细分析（核心文档）
├── 03_Build_Integration.md      # BUILD.gn 构建适配和配置
├── 04_Usage_in_OH.md           # 依赖关系、主要使用场景、依赖图
├── 05_API_Differences.md         # 新增/修改的 API 完整列表和使用示例
├── 06_Security.md               # 安全风险分析、CVE 状态、升级建议
└── _work/
    ├── ASSESSMENT.md          # 项目评估结果（Phase 0 分析）
    ├── NOTES.md               # 分析过程记录和技术债务
    └── PLAN.md               # 文档任务计划和进度
```

---

## 🎯 Curl 在 OpenHarmony 中的定位（Positioning）

### 核心功能

| 功能 | 说明 | OH 模块 |
|------|------|----------|
| **HTTP/HTTPS 客户端** | 提供完整的 HTTP/HTTPS 请求能力 | Ace Engine、媒体应用、网络框架 |
| **流媒体协议支持** | 支持范围请求、分片下载 | Media Foundation |
| **SSL/TLS 后端** | 支持 OpenSSL 和国密 OpenHiTLS | 所有安全通信模块 |
| **HTTP/2 多路复用** | 支持连接复用和流式传输 | 高性能场景 |

### OH 定制化范围

OpenHarmony 对 curl 进行了**7 个主要 Patch**，新增以下功能：

1. **网络诊断框架**（Patches 1-5）
   - SSL/TLS 详细错误信息
   - 连接 IP/端口记录
   - DNS 解析状态追踪
   - DNS 缓存来源判断（netsys 缓存/实时查询）
   - I/O 时间戳和传输字节数统计

2. **性能优化**（Patches 4, 6）
   - DNS 缓存主动清除接口
   - HTTP/2 连接均衡复用策略（跨多 IP 负载均衡）

3. **安全增强**（Patch 7）
   - Cookie 加密/解密 Hook 机制
   - 与华为 HUKS（Hardware User Key Store）集成
   - 支持 Cookie 的机密性保护

### 依赖关系概览

```
┌─────────────────────────────────────────────────┐
│          OpenHarmony Application Layer         │
│  ┌───────────┐  ┌───────────┐            │
│  │ Ace Engine │  │  Media     │            │
│  └─────┬─────┘  └─────┬─────┘            │
│        │                 │      │                │
│        ▼                 ▼      ▼                │
│  ┌─────────────────────────────────────┐     │
│  │         curl (libcurl)          │     │
│  │  + 7 OH-specific Patches       │     │
│  └─────────────────────────────────────┘     │
│                                                │
│  ┌────────────────────────────────────────┐    │
│  │        External Dependencies         │    │
│  │  c-ares │  nghttp2 │         │    │
│  │     └───┐     └───────┐      │    │
│  │           │             │        │      │    │
│  │           ▼             ▼        │      │    │
│  │     ┌─────────────────────────┐ │      │    │
│  │     │ OpenSSL │ OpenHiTLS │ │      │    │
│  │     └──────────┬──────────────┘ │      │    │
│  │                │               │      │    │
│  │                ▼               │      │    │
│  │           zlib │ brotli      │      │    │
│  └──────────┬───────────────────────┘      │    │
└────────────┼────────────────────────────────┘
             │
             ▼
┌───────────────────────────────────────┐
│     OpenHarmony System Services       │
│  netsys (DNS) │ HUKS (Crypto) │
└───────────────────────────────────────┘
```

**关键特性**：
- ✅ 所有 OH Patch 基于 `USE_ARES` 宏保护
- ✅ 支持标准系统（Linux/Android/Windows）和 LiteOS（M/A 内核）
- ✅ 支持国密 TLS 后端（OpenHiTLS）
- ✅ 完全兼容上游 curl API，仅做扩展

---

## 🚀 快速开始（Quick Start）

### 我想做什么？

| 场景 | 推荐文档 | 预估时间 |
|-------|----------|----------|
| **新手入门** | SUMMARY.md → 01_Overview.md | 15 分钟 |
| **使用 curl API** | 05_API_Differences.md | 20 分钟 |
| **诊断网络问题** | 02_Patches.md（诊断部分）+ 05_API | 30 分钟 |
| **集成 curl 到新模块** | 03_Build_Integration.md | 30 分钟 |
| **升级 curl 版本** | 02_Patches.md（升级建议）+ 06_Security.md | 90 分钟 |
| **了解依赖关系** | 04_Usage_in_OH.md | 20 分钟 |

### 常见问题（FAQ）

**Q: OH 的 curl 版本是上游的哪个版本？**
> A: 基于 curl 8.8.0，但应用了 7 个 OH 特有 Patch。版本追踪见 `bundle.json`（组件版本 3.1）。

**Q: 如何确定哪些 Patch 已生效？**
> A: 所有 Patch 都受 `USE_ARES` 宏保护。在标准系统构建时，检查 `curl -V` 输出中的 "Features:" 部分，查找 c-ares。

**Q: HTTP/2 是否支持？**
> A: 是，通过 nghttp2 库支持。HTTP/3 支持可通过 `netstack_feature_http3=true` 构建（需 quiche/boringssl）。

**Q: 如何启用国密 TLS（OpenHiTLS）？**
> A: 在 `BUILD.gn` 中，确保 `support_gmssl` 为 true（需要 `global_parts_info.thirdparty_openhitls` 存在）。

**Q: 新增的诊断 API 如何使用？**
> A: 参考 `05_API_Differences.md` 中的使用示例，通过 `curl_easy_getinfo()` 获取 SSL 错误、DNS 状态、连接信息等。

**Q: 是否可以禁用某些 OH Patch？**
> A: 所有 Patch 基于编译时宏（`USE_ARES`），无法在运行时禁用。如需标准上游行为，请构建时不启用相关宏。

**Q: 升级到新版本时如何处理 Patch？**
> A: 参考 `02_Patches.md` 中的升级建议，逐个验证 Patch 在新版本上的兼容性，可能需要手动解决冲突。

**Q: LiteOS 构建有什么区别？**
> A: LiteOS（M/A 内核）禁用了大量特性（如 LDAP、多种协议），使用 mbedTLS 而非 OpenSSL，更强调代码大小优化（`-Os`、`-ffunction-sections`）。

---

## 📚 阅读路线（Reading Paths）

### 路径 1：快速了解（15 分钟）
```
SUMMARY.md
    ↓
01_Overview.md（了解 curl 是什么）
    ↓
结束 ← 了解基本概念
```

### 路径 2：深入学习（2 小时）
```
02_Patches.md（核心文档，最详细）
    ↓
05_API_Differences.md（学习如何使用新增 API）
    ↓
结束 ← 掌握 OH 特定功能
```

### 路径 3：集成实践（1 小时）
```
03_Build_Integration.md
    ↓
04_Usage_in_OH.md（了解如何被其他模块使用）
    ↓
结束 ← 理解集成方式
```

### 路径 4：维护参考（30 分钟）
```
06_Security.md（了解安全风险和升级策略）
    ↓
查看 _work/ASSESSMENT.md（项目评估）
    ↓
结束 ← 具备维护能力
```

---

## 🔧 技术特点（Technical Highlights）

### Patch 分组

| 分组 | Patch 数 | 主要功能 |
|------|----------|----------|
| 诊断与监控 | 5 | SSL 错误、连接 IP、DNS 状态、I/O 统计 |
| 性能优化 | 2 | DNS 缓存清除、HTTP/2 均衡复用 |
| 安全增强 | 1 | Cookie 加密 Hook |

### 构建系统差异

| 特性 | 标准上游 | OH 定制版 |
|------|----------|----------|
| 构建系统 | CMake/autotools | GN（OH 原生） |
| DNS 解析器 | 可选（c-ares）| 强制使用 c-ares |
| TLS 后端 | OpenSSL | OpenSSL + OpenHiTLS（国密，可选）|
| 配置方式 | configure 脚本 | BUILD.gn + config 头文件 |
| 平台支持 | 通用 | 标准系统 + LiteOS-M/A |
| HTTP/3 | 可选构建 | 通过 `netstack_feature_http3` 控制 |

### API 扩展统计

| 类型 | 数量 | 示例 |
|------|------|------|
| CURLINFO 选项 | 20+ | `CURLINFO_SSL_ERROR`、`CURLINFO_DNS_STATUS` |
| CURLOPT 选项 | 10+ | `CURLOPT_BALANCED_CONNECTION`、`CURLOPT_ARES_SOCKET_FUNCTION` |
| 函数接口 | 2 | `curl_multi_clear_dns_cache()`、`curl_mime_init_with_boundary()` |
| 枚举类型 | 1 | `dns_status_type`（INIT/GET_IP/GET_CNAME/INVALID） |

---

## 📊 关键指标（Key Metrics）

### 代码规模
- **总 Patch 文件**: 7 个
- **修改源文件**: 13+ 个（跨 7 个 Patch）
- **新增代码行数**: 约 800+ 行（估算）
- **新增 API 数量**: 20+ 个 CURLINFO + 10+ 个 CURLOPT

### 文档完整度
- [x] 01_Overview.md - curl 简介 ✅
- [x] SUMMARY.md - 导航指南 ✅
- [ ] 02_Patches.md - Patch 详细分析 ⏳
- [ ] 03_Build_Integration.md - 构建集成 ⏳
- [ ] 04_Usage_in_OH.md - 依赖使用 ⏳
- [ ] 05_API_Differences.md - API 差异 ⏳
- [ ] 06_Security.md - 安全风险 ⏳

---

## 📝 维护与贡献（Maintenance & Contributing）

### 文档维护

本文档由 OpenHarmony 第三方库 Wiki 维护团队负责更新。

### 贡献指南

如发现文档错误、遗漏内容或有改进建议：

1. 在 OpenHarmony 项目中提 Issue
2. Fork 本文档仓库
3. 创建分支进行修改
4. 提交 Pull Request

### 更新频率

- **小更新**（笔误、链接修正）：随时
- **大更新**（新增文档、重大变更）：季度或随 OH curl 版本升级
- **同步更新**（上游 curl 更新影响 OH）：立即更新相关文档

---

## 📮 相关资源（Resources）

### 内部资源
- **工作文件**: `_work/ASSESSMENT.md`、`_work/NOTES.md`、`_work/PLAN.md`
- **构建配置**: `BUILD.gn`、`customized/include/*.h`
- **Patch 文件**: `0001-*.patch` 至 `0007-*.patch`

### 外部资源
- **curl 官方文档**: https://curl.se/libcurl/c/
- **curl GitHub 仓库**: https://github.com/curl/curl
- **curl Wiki**: https://github.com/curl/curl/wiki
- **Everything curl**: https://everything.curl.dev/
- **OpenHarmony curl fork**: https://gitee.com/openharmony/third_party_curl

### 上游 PR
- [PR #17743 - OpenHarmony SOVERSION 支持](https://github.com/curl/curl/pull/17743)（2025-06-25）
- 说明：为 OpenHarmony 添加 SOVERSION 默认启用支持

---

## 🎓 学习路径建议（Learning Path Recommendations）

### 角色导向（Role-Based）

| 角色 | 推荐阅读顺序 | 重点文档 |
|------|--------------|----------|
| **新手开发者** | SUMMARY → 01_Overview → 05_API | 了解基础、学会使用 |
| **应用开发者** | 04_Usage_in_OH → 05_API | 了解集成、直接使用 |
| **系统工程师** | 03_Build → 02_Patches → 06_Security | 深度理解、维护能力 |
| **安全工程师** | 06_Security → 02_Patches | 了解风险、制定策略 |
| **维护者** | ASSESSMENT → 02_Patches → PLAN | 全面掌握、规划升级 |

### 技能导向（Skill-Based）

| 技能 | 推荐文档 | 练习建议 |
|------|----------|----------|
| **网络编程** | 01_Overview、05_API | 使用 curl 发起 HTTP 请求、处理响应 |
| **故障诊断** | 02_Patches（诊断部分） | 使用新增的诊断 API 排查问题 |
| **跨平台开发** | 03_Build | 了解不同平台的构建差异 |
| **TLS 集成** | 03_Build（OpenHiTLS 部分） | 配置和使用国密 TLS |

---

**开始阅读**: 从 [SUMMARY.md](SUMMARY.md) 开始选择适合你的路径 🚀

---

**文档版本**: v1.0
**最后更新**: 2026-02-07
**维护者**: OpenHarmony 第三方库 Wiki 团队
