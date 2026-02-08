# 项目概览

> 轻量级分布式组件管理框架核心文档，帮助快速理解项目定位与核心能力。

## 一句话定义

**dmsfwk_lite（轻量级分布式组件管理框架）是 OpenHarmony Small System 的核心系统服务，负责跨设备启动 FA（Feature Ability）并支持分布式场景下的应用协同。**

**证据来源**：`README_zh.md:11`「轻量级分布式组件管理负责跨设备启动FA的能力，支持分布式场景下的应用协同」。

## 项目定位

### 在 OpenHarmony 系统中的位置

dmsfwk_lite 属于 OpenHarmony 系统架构中的 **Ability 子系统**，作为系统服务运行在用户态。其通过 SAMgr（System Ability Manager）注册为系统级服务，为上层应用提供分布式能力支撑。

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (App Layer)                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Feature Ability (FA)                     │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                   应用框架层 (Framework)                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Ability Lite Framework                   │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                   系统服务层 (System Services)               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           dmsfwk_lite (本项目)                       │   │
│  │   跨设备 FA 启动、分布式调度                          │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌───────────────────┐  ┌───────────────────┐              │
│  │    SAMgr Lite     │  │   Bundle Manager  │              │
│  └───────────────────┘  └───────────────────┘              │
├─────────────────────────────────────────────────────────────┤
│                   基础设施层 (Foundation)                      │
│  ┌───────────────────┐  ┌───────────────────┐              │
│  │     DSoftBus      │  │    HUKS           │              │
│  └───────────────────┘  └───────────────────┘              │
├─────────────────────────────────────────────────────────────┤
│                      内核层 (Kernel)                          │
└─────────────────────────────────────────────────────────────┘
```

**证据来源**：`bundle.json:16`「subsystem: "ability"」表明其属于 Ability 子系统。

### 核心职责

| 职责 | 描述 | 代码证据 |
|------|------|---------|
| **远程 FA 启动** | 接收本地应用的远程启动请求，在远端设备上启动指定的 FA | `source/dmslite_famgr.c:85` |
| **跨设备通信** | 基于 DSoftBus 与远端设备建立会话并传递启动消息 | `source/dmslite_session.c:33-34` |
| **消息序列化** | 采用 TLV 格式封装启动消息，确保跨设备数据一致性 | `include/dmslite_tlv_common.h:32-58` |
| **权限校验** | 基于应用签名验证远程调用者身份，防止越权访问 | `source/dmslite_permission.c:63-116` |
| **FA 生命周期管理** | 协调本地与远端 FA 的创建、运行、销毁流程 | `source/dmslite_famgr.c:39-43` |

## 功能边界

### 支持的能力

| 能力 | 说明 | 限制条件 |
|------|------|---------|
| **远程 FA 启动** | 支持启动远端设备上的 Feature Ability | 仅支持 FA，不支持 Page Ability |
| **同步/异步调用** | 支持阻塞式同步调用和非阻塞式异步回调 | 异步回调依赖 listener 实现 |
| **数据传递** | 支持通过 Want.data 传递启动参数 | 数据长度受 MAX_DMS_MSG_LENGTH (1024) 限制 |
| **签名校验** | 基于应用证书签名进行跨设备身份验证 | 需要应用已安装且签名一致 |
| **会话管理** | 自动建立、维护、关闭跨设备通信会话 | 60 秒超时，超时后自动关闭 |

**证据来源**：`README_zh.md:72`「支持远程启动FA」；`include/dmslite_tlv_common.h:30`「MAX_DMS_MSG_LENGTH 1024」。

### 不支持的能力

| 能力 | 说明 | 替代方案 |
|------|------|---------|
| **Page Ability 远程启动** | 不支持跨设备启动 Page Ability（页面级 Ability） | 使用 FA 作为分布式入口 |
| **跨设备服务调用** | 不支持直接调用远端设备的 Service Ability | 通过 FA 间接实现业务协同 |
| **文件传输** | 不提供跨设备文件传输能力 | 使用分布式文件系统服务 |
| **实时音视频** | 不提供音视频流传输能力 | 使用分布式媒体服务 |

## 运行环境

### 系统要求

| 要求 | 说明 | 来源 |
|------|------|------|
| **操作系统** | OpenHarmony 操作系统 | `README_zh.md:68` |
| **系统类型** | Small System（轻量系统） | `bundle.json:18` |
| **内核类型** | LiteOS-A 或 Linux | `BUILD.gn:17` |
| **组网要求** | 主从设备在同一局域网，可互相 ping 通 | `README_zh.md:66` |

### 依赖服务

| 服务 | 用途 | 是否必需 |
|------|------|---------|
| **SAMgr Lite** | 系统服务注册与发现 | 必须 |
| **DSoftBus** | 设备间通信 | 必须 |
| **Bundle Manager** | 应用包信息查询 | 必须 |
| **Ability Manager** | Ability 生命周期管理 | 必须 |
| **HUKS** | 安全密钥管理 | 可选（签名校验依赖） |

**证据来源**：`BUILD.gn:60-68` 依赖配置。

## 快速开始

### 环境准备

```bash
# 确认设备在同一局域网
ping <remote_device_ip>

# 确认目标设备已安装目标 FA
hdc shell bm dump <bundle_name>
```

### 远程启动 FA 示例

以下代码展示如何在主设备上启动远端设备的 FA：

```cpp
import ohos.aafwk.ability.Ability;
import ohos.aafwk.content.Want;
import ohos.bundle.ElementName;

// 构造 Want 参数
Want want = new Want();
ElementName name = new ElementName(
    remote_device_id,           // 远端设备 ID
    "ohos.dms.remote_bundle_name", // 远端包名
    "remote_ability_name"       // 远端 Ability 名
);
want.setElement(name);
want.setFlags(Want.FLAG_ABILITYSLICE_MULTI_DEVICE);  // 设置分布式标记

// 启动远端 FA
startAbility(want);
```

**证据来源**：`README_zh.md:91-108` 官方示例代码。

### 关键参数说明

| 参数 | 类型 | 说明 | 必填 |
|------|------|------|------|
| **remote_device_id** | string | 目标设备的设备标识符 | 是 |
| **bundle_name** | string | 目标 FA 所在的应用包名 | 是 |
| **ability_name** | string | 目标 FA 的能力名称 | 是 |
| **FLAG_ABILITYSLICE_MULTI_DEVICE** | flag | 使能分布式启动的标志位 | 是 |

### 调用流程概览

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│ 主设备   │         │ dmsfwk   │         │ DSoftBus │         │ 从设备   │
│  应用    │ ──────> │  lite    │ ──────> │          │ ──────> │  dmsfwk  │
│          │         │          │         │          │         │  lite    │
└──────────┘         └──────────┘         └──────────┘         └──────────┘
     │                   │                   │                   │
     │  1. startAbility  │                   │                   │
     │                   │                   │                   │
     │              2. 权限校验               │                   │
     │                   │                   │                   │
     │              3. 消息封装               │                   │
     │                   │                   │                   │
     │              4. 会话建立               │                   │
     │                   │                   │                   │
     │              5. 消息发送               │                   │
     │                   │                   │                   │
     │                   │                   │              6. 消息解析
     │                   │                   │                   │
     │                   │                   │              7. FA 启动
```

## 版本与兼容性

| 版本 | 发布日期 | 主要变更 |
|------|---------|---------|
| 3.1 | - | 支持 Small System 完整分布式能力 |
| 3.0 | - | 初始版本发布 |

**证据来源**：`bundle.json:5`「version: "3.1"」。

## 相关资源

| 资源 | 链接 |
|------|------|
| 官方源码仓库 | [dmsfwk_lite](https://gitee.com/openharmony/ability_dmsfwk_lite) |
| Ability 子系统文档 | [Ability 文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/ability.md) |
| API 参考 | [04_Interface.md](./04_Interface.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| 安全分析 | [05_AttackSurface.md](./05_AttackSurface.md) |

---

## 快速导航

| 下一步 | 推荐阅读 |
|-------|---------|
| 理解架构 | [02_Architecture.md](./02_Architecture.md) |
| 查看接口 | [04_Interface.md](./04_Interface.md) |
| 安全分析 | [05_AttackSurface.md](./05_AttackSurface.md) |
| 代码导航 | [03_CodeMap.md](./03_CodeMap.md) |

---

*文档版本：v1.0*  
*最后更新：2026-02-07*
