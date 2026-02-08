# lwIP - OpenHarmony Wiki

## 库概览

**lwIP** (lightweight IP) 是 OpenHarmony 的基础 TCP/IP 协议栈，为上层的分布式软总线、网络服务、IoT 应用提供网络通信能力。

| 属性 | 值 |
|------|-----|
| **上游版本** | STABLE-2_2_1_RELEASE |
| **OH 组件版本** | 3.1 |
| **许可证** | BSD 3-Clause |
| **上游地址** | https://savannah.nongnu.org/projects/lwip/ |

## OH 适配概述

OpenHarmony 对 lwIP 进行了三项重要增强，使其更适合 IoT 和分布式场景：

| 特性 | 说明 | 配置宏 |
|------|------|--------|
| **网络容器** | 支持网络命名空间隔离，实现多应用网络隔离 | `LOSCFG_NET_CONTAINER` |
| **低功耗模式** | 定时器空闲时允许 CPU 睡眠，延长电池寿命 | `LWIP_LOWPOWER` |
| **分布式网络** | 支持跨设备网络代理，为软总线提供底层支持 | `LWIP_ENABLE_DISTRIBUTED_NET` |

## 文档导航

### 入门阅读

1. **[原始库简介](01_Overview.md)** - 了解 lwIP 的基本功能和在 OH 中的定位
2. **[Patch 详细分析](02_Patches.md)** - 深入了解 OH 特有功能的实现细节

### 开发指南

3. **[构建适配说明](03_Build_Integration.md)** - 如何在项目中使用 lwIP（GN 构建、配置选项）
4. **[依赖关系与使用](04_Usage_in_OH.md)** - 谁在使用 lwIP，如何正确使用
5. **[API/接口差异](05_API_Differences.md)** - OH 特有 API 参考

## 快速开始

### 在项目中使用 lwIP

```gn
# BUILD.gn
ohos_executable("my_app") {
  sources = [ "main.c" ]
  deps = [
    "//third_party/lwip:liblwip",  # 链接 lwIP 库
  ]
}
```

```c
// main.c
#include "lwip/sockets.h"
#include "lwip/netdb.h"

int main() {
    // 使用标准 BSD Socket API
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    // ... 你的网络代码
    return 0;
}
```

### 启用 OH 特有功能

```c
// lwipopts.h

// 启用低功耗模式（适用于电池供电设备）
#define LWIP_LOWPOWER 1

// 启用分布式网络（适用于分布式场景）
#define LWIP_ENABLE_DISTRIBUTED_NET 1
```

## 关键信息

### 无传统 Patch 文件

⚠️ **注意**: OH 的 lwIP 没有使用传统的 `.patch` 文件方式管理修改，而是将 OH 特有功能**直接集成**到代码库中，通过**条件编译宏**控制。

### 代码统计

- **总代码量**: ~36,600 行
- **OH 新增代码**: ~2,300 行（网络容器、低功耗、分布式网络）
- **修改文件数**: 68 个文件
- **条件编译宏**: `LOSCFG_NET_CONTAINER`, `LWIP_LOWPOWER`, `LWIP_ENABLE_DISTRIBUTED_NET`

### 主要依赖者

- **dsoftbus**: 分布式软总线
- **liteos_m**: 轻量级内核
- **uniproton**: 实时内核
- **Hisilicon SDK**: 芯片厂商封装

## 贡献与维护

- **维护者**: heqianmo@huawei.com
- **上游社区**: https://savannah.nongnu.org/projects/lwip/
- **OH 问题反馈**: OpenHarmony 官方渠道

## 许可证

```
BSD 3-Clause License

Copyright (c) 2001-2004 Swedish Institute of Computer Science.
All rights reserved.

Copyright (c) 2021-2023 Huawei Device Co., Ltd.
All rights reserved.
```
