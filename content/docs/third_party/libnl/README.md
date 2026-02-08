# libnl - OpenHarmony Wiki

## 库概览

libnl（Netlink Library）是一个提供基于 netlink 协议的 Linux 内核接口 API 的库。它为应用程序与 Linux 内核进行网络配置通信提供了高级抽象接口。

| 属性 | 值 |
|------|-----|
| **上游版本** | 3.11.0 |
| **许可证** | LGPL V2.1 |
| **上游地址** | https://github.com/thom311/libnl |
| **OH 组件** | @ohos/libnl |
| **所属子系统** | thirdparty |

## 在 OpenHarmony 中的作用

libnl 在 OpenHarmony 中主要用于：

1. **WLAN 子系统**: 提供 netlink 接口用于网络配置、路由管理、邻居发现等
2. **Wi-Fi 连接管理**: wpa_supplicant 依赖 libnl 进行网络接口监控和配置
3. **网络驱动**: 与内核网络子系统通信的基础库

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](01_Overview.md) | 原始库简介和 OH 定位 |
| [02_Patches.md](02_Patches.md) | **核心文档**：Patch 详细分析 |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建适配说明 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](05_API_Differences.md) | API/接口差异（如有） |
| [06_Security.md](06_Security.md) | 安全风险分析 |

## 快速链接

- [官方文档](https://www.infradead.org/~tgr/libnl/)
- [官方 API 文档](https://www.infradead.org/~tgr/libnl/doc/api/)
- [OpenHarmony 贡献指南](https://gitee.com/openharmony/docs/blob/HEAD/zh-cn/contribute/参与贡献.md)

## 相关仓库

- [third_party_wpa_supplicant](https://gitee.com/openharmony/third_party_wpa_supplicant)
- [drivers_peripheral](https://gitee.com/openharmony/drivers_peripheral)
