# c-ares 库 Wiki

**OpenHarmony 第三方库文档**

---

## 概览

本文档说明 **c-ares** 库在 OpenHarmony 中的集成与适配。

**c-ares** 是一个现代异步 DNS 解析库，用于在 OpenHarmony 网络栈中执行高性能的域名解析。

---

## 文档导航

### 快速开始

| 你的需求 | 推荐阅读路径 |
|---------|------------|
| **了解 c-ares 基础** | [01_Overview.md](01_Overview.md) |
| **查看 OH Patch 列表** | [02_Patches.md](02_Patches.md) |
| **了解构建配置** | [03_Build_Integration.md](03_Build_Integration.md) |
| **查看依赖关系** | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| **API 参考手册** | [05_API_Differences.md](05_API_Differences.md) |
| **安全风险评估** | [06_Security.md](06_Security.md) |

### 深入学习

| 学习目标 | 推荐阅读路径 |
|---------|------------|
| **完整项目评估** | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) |
| **源码修改分析** | [_work/NOTES.md](_work/NOTES.md) |
| **开发工作计划** | [_work/PLAN.md](_work/PLAN.md) |

---

## 关键特性

### 标准 c-ares 功能

- ✅ **异步 DNS 查询**: 非阻塞的域名解析
- ✅ **并行查询**: 同时发起多个 DNS 请求
- ✅ **线程安全**: 支持多线程并发使用
- ✅ **查询缓存**: 内部缓存机制减少重复查询
- ✅ **DNSSEC 支持**: DNS 安全扩展
- ✅ **IPv6 支持**: 完整的 A/AAAA 记录处理
- ✅ **多种记录类型**: A, AAAA, SRV, MX, TXT, CNAME, NAPTR, TLSA, CAA 等

### OpenHarmony 增强功能

- 🚀 **NetSys 集成**: 通过 OH 网络系统服务获取 DNS 配置
- 🌐 **多网络支持**: netId 参数支持不同网络的 DNS 解析
- 💾 **系统缓存集成**: 与 OH DNS 缓存服务集成，跨进程共享
- 📊 **查询指标**: 详细的 DNS 查询性能指标和遥测

---

## 快速链接

### 相关资源

| 类型 | 链接 |
|-----|------|
| **上游仓库** | https://github.com/c-ares/c-ares |
| **上游文档** | https://c-ares.org/ |
| **OpenHarmony 仓库** | https://gitee.com/openharmony |
| **上游版本** | v1.34.6 (2025-12-08) |
| **OH 版本** | v1.34.5 |

### 使用示例

#### 基础用法

```c
#include <ares.h>

// 1. 初始化 DNS 通道
ares_channel_t *channel;
ares_init(&channel);

// 2. 设置回调
void callback(void *arg, int status, int timeouts,
               unsigned char *abuf, int alen)
{
  // 处理 DNS 响应
}

// 3. 发起查询
ares_query(channel, "example.com", ns_c_in,
           callback, NULL);
```

#### OpenHarmony 特定用法

```c
// 1. 初始化
ares_channel_t *channel;
ares_init(&channel);

// 2. 设置网络 ID（OH 特定）
int32_t netid = get_current_network_id();
ares_set_dns_netid(channel, netid);

// 3. 执行查询
ares_getaddrinfo(channel, "example.com", NULL,
                   NULL, addrinfo_callback, NULL);

// 4. 清理
ares_cleanup(channel);
```

---

## 贡献与反馈

### 问题报告

- **上游问题**: https://github.com/c-ares/c-ares/issues
- **OH 特定问题**: 通过 OpenHarmony 社区渠道报告

### 文档维护

本文档由 **OpenHarmony Wiki Agent** 自动生成和分析。
- **生成日期**: 2026-02-08
- **评估版本**: c-ares 1.34.5

---

## 版本历史

| 版本 | 日期 | 变更 |
|-----|------|-----|
| **v1.0** | 2026-02-08 | 初始版本，完成基础文档 |

---

**最后更新**: 2026-02-08
