# 项目定位与边界

## 目的

本文档定义 PrintSpooler 项目的职责范围、功能边界和约束条件，帮助开发者理解项目支持的功能范围和限制。

## 适用范围

本文档适用于：

- 架构设计人员
- 产品经理
- 功能开发人员
- 需要评估项目适用性的技术人员

## 关键结论

1. **核心定位**：OpenHarmony 打印管理服务，提供打印预览、设备发现、任务管理
2. **功能边界**：仅支持 IPP 无驱动打印，不支持需要单独驱动的打印机
3. **系统角色**：预置系统应用（`com.ohos.spooler`），作为打印服务的客户端
4. **协议支持**：支持 Mopria 协议，通过 Wifi P2P 和 mDNS 发现设备

---

## 项目定位

### 核心职责

PrintSpooler 作为 OpenHarmony 系统的打印管理中心，负责：

1. **打印预览界面** - 提供用户友好的打印参数设置和预览
2. **打印机发现** - 通过 WiFi P2P 和 mDNS 协议发现网络打印机
3. **打印任务管理** - 创建、下发、监听打印任务状态
4. **打印服务接口** - 作为 PrintExtensionAbility 实现打印扩展

### 不在范围内

以下功能不在当前项目范围内：

- ❌ 扫描功能 - 当前版本仅支持打印
- ❌ 非IPP 打印协议 - 不支持 USB 打印、蓝牙打印等
- ❌ 需要驱动的打印机 - 不支持需要单独安装驱动的设备
- ❌ 跨设备打印 - 当前版本未支持分布式打印

---

## 功能边界

### 支持的功能

#### 1. IPP 无驱动打印

- 支持 Mopria 协议的打印机
- 通过 IPP/IPPS 协议（端口 631）与打印机通信
- 支持配置打印机到 CUPS 服务

**证据**：
- README.md:13: "当前只支持对接支持ipp无驱动打印协议的打印机"
- PrintConstants.ts:16-18: SCHEME_IPP, SCHEME_IPPS 定义
- PrintConstants.ts:22-23: IPP_PORT, IPP_PATH 定义

#### 2. Wifi P2P 发现和连接

- 发现周围的 P2P 打印机
- 建立 P2P 连接获取打印机 IP
- 支持 Mopria 协议

**证据**：
- P2pDiscoveryChannel.ts - P2P 发现实现
- P2pPrinterConnection.ts - P2P 连接实现

#### 3. mDNS 服务发现

- 发现局域网内的 IPP 打印机
- 自动发现支持 `_ipp._tcp` 和 `_ipps._tcp` 服务的设备

**证据**：
- MdnsDiscovery.ts - mDNS 发现实现
- PrintConstants.ts:18-19: SERVICE_IPP, SERVICE_IPPS 定义

### 不支持的功能

#### 1. 其他打印协议

- ❌ USB 打印
- ❌ 蓝牙打印
- ❌ 云打印

#### 2. 需要驱动的打印机

- ❌ 需要安装驱动的传统打印机
- ❌ 不符合 Mopria 标准的设备

#### 3. 高级打印功能

- ❌ 双面打印机控制（仅支持双面打印参数设置）
- ❌ 多纸盒选择
- ❌ 打印机维护功能

---

## 技术约束

### 协议限制

| 协议 | 支持 | 说明 |
|------|------|------|
| IPP (TCP) | ✅ | 标准互联网打印协议，端口 631 |
| IPPS (TLS) | ✅ | 加密 IPP 协议 |
| Mopria | ✅ | 移动打印联盟标准 |
| USB | ❌ | 不支持 |
| 蓝牙 | ❌ | 不支持 |

### 设备类型支持

| 设备类型 | 支持 | 说明 |
|---------|------|------|
| 支持 Mopria 的网络打印机 | ✅ | 主要支持类型 |
| Wifi P2P 打印机 | ✅ | 通过 P2P 直接连接 |
| 需要驱动的打印机 | ❌ | 不支持 |

### 系统版本约束

- **最低 API 级别**：API 11
- **目标 API 级别**：API 18
- **运行时**：OpenHarmony

---

## 系统角色

### 在 OpenHarmony 中的位置

PrintSpooler 在系统中的角色：

```
[应用层]
    ↓
[PrintSpooler (com.ohos.spooler)]  ← 本项目
    ↓ (通过 print API)
[Print Service]  (系统打印服务)
    ↓
[CUPS / Printer Driver]  (打印服务)
```

### 与其他组件的关系

| 组件 | 关系 | 说明 |
|------|------|------|
| print_print_fwk | 客户端 | 通过 @kit.BasicServicesKit 调用打印框架 |
| cupsd | 配置客户端 | 通过 print.addPrinterToCups() 配置打印机 |
| Wifi Manager | 客户端 | 通过 @ohos.wifi 进行 P2P 操作 |
| Bundle Manager | 客户端 | 获取自身 Bundle 信息 |

---

## 未来扩展点

以下功能是潜在的扩展方向（当前未实现）：

1. **扫描功能** - 添加扫描支持
2. **更多打印协议** - 支持 USB、蓝牙等
3. **分布式打印** - 支持跨设备打印
4. **高级打印功能** - 双面控制、多纸盒等

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目整体介绍
- [架构设计](03_Architecture.md) - 系统架构和组件关系
- [对外 API](04_External_API.md) - 与打印服务的 API 交互
