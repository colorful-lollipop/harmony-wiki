# WLAN 组件概览

**目的**: 描述 WLAN 组件的项目定位、边界、核心能力、运行环境和关键概念

**适用范围**: OpenHarmony WLAN 组件开发者、系统开发者、安全审计人员

**生成时间**: 2026-02-06

---

## 项目定位

### 组件名称
- **完整路径**: `foundation/communication/wifi`
- **组件名**: `communication_wifi`
- **子系统**: `communication`
- **包名**: `@ohos/wifi`

### 功能定位
WLAN 组件为 OpenHarmony 系统提供无线局域网（WLAN）功能，包括：

1. **WLAN 基础功能**
   - STA（Station）模式：连接到外部 AP 的功能
   - AP（Access Point）模式：提供热点功能
   - 扫描功能：扫描可用的 WLAN 网络
   - 配置管理：保存和管理网络配置

2. **P2P（Peer-to-Peer）功能**
   - WiFi Direct：设备间直连
   - 组创建与管理：支持 GO（Group Owner）和 GC 模式
   - 设备发现：发现附近的 P2P 设备

3. **WLAN 消息通知**
   - 状态变化通知：WLAN 状态、连接状态、扫描状态等
   - 事件订阅：通过 `on()`/`off()` API 订阅事件

### 组件边界
**负责的边界**:
- ✅ WLAN 硬件接口管理（通过 HAL/HDI）
- ✅ WLAN 扫描与连接逻辑
- ✅ WLAN 配置管理
- ✅ P2P 功能管理
- ✅ 网络层适配（DHCP、网络策略）

**不负责的边界**:
- ❌ IP 路由策略（由 `netmanager` 负责）
- ❌ 网络接口驱动（由 HAL 层负责）
- ❌ 系统电源管理（由 `power_manager` 负责）
---

## 核心能力

### 支持的标准（SystemCapabilities）
根据 [bundle.json](../wifi/bundle.json:40-46)，组件支持以下系统能力：

| 系统能力 | 说明 |
|----------|------|
| `SystemCapability.Communication.WiFi.STA` | WiFi Station 模式 |
| `SystemCapability.Communication.WiFi.AP.Core` | WiFi AP 核心功能 |
| `SystemCapability.Communication.WiFi.P2P` | WiFi P2P 功能 |
| `SystemCapability.Communication.WiFi.Core` | WiFi 核心功能 |

### 功能特性（Feature Flags）
组件支持 40+ 个特性开关，可通过 GN 参数配置：

**核心功能**:
- `wifi_feature_with_p2p` - P2P 功能（默认：true）
- `wifi_feature_with_scan_control` - 扫描控制（默认：true）
- `wifi_feature_with_encryption` - 加密功能（默认：true）
- `wifi_feature_with_random_mac_addr` - 随机 MAC 地址（默认：true）

**增强功能**:
- `wifi_feature_wifi_pro_ctrl` - WiFi Pro 控制（默认：true）
- `wifi_feature_with_vap_manager` - VAP 管理器（默认：true）
- `wifi_feature_with_security_detect` - 安全检测（默认：true）
- `wifi_feature_with_portal_login` - 门户登录（默认：true）

**性能优化**:
- `wifi_feature_sta_ap_exclusion` - STA/AP 互斥（默认：true）
- `wifi_feature_with_local_random_mac` - 本地随机 MAC（默认：true）
- `wifi_feature_with_ipv6_selfcure` - IPv6 自愈（默认：true）

详细特性列表请参考：[05_GN_Targets.md](05_GN_Targets.md) 或 [appendix/Config_Flags.md](appendix/Config_Flags.md)

---

## 运行环境

### 支持的系统类型
根据 [bundle.json](../wifi/bundle.json:84-87)：

- **Standard System** - 完整功能
- **Small System** - 轻量级功能（`ohos_lite`）

### 进程模型
WLAN 服务以 `wifi_manager_service` 进程运行：

| 组件 | 进程名 | 说明 |
|------|---------|------|
| 核心服务 | `wifi_manager_service` | 所有 SA 服务运行在此进程中 |
| HAL 服务 | `wifi_hal_service` | 硬件抽象层服务（可选） |
| DHCP 服务 | `dhcp_service` | DHCP 客户端（关联服务） |

### System Ability（SA）架构
WLAN 组件提供 4 个 System Ability：

| SA ID | 名称 | 产物库 | 说明 |
|-------|------|---------|------|
| 1120 | WIFI_DEVICE_ABILITY_ID | libwifi_device_ability.z.so | STA 设备管理服务 |
| 1121 | WIFI_HOTSPOT_ABILITY_ID | libwifi_hotspot_ability.z.so | AP/热点管理服务 |
| 1123 | WIFI_P2P_ABILITY_ID | libwifi_p2p_ability.z.so | P2P 管理服务 |
| 1124 | WIFI_SCAN_ABILITY_ID | libwifi_scan_ability.z.so | 扫描服务 |

---

## 关键概念

### STA（Station）模式
设备连接到外部 AP 的模式：
- 扫描可用网络
- 连接/断开连接
- 管理已保存的网络配置
- 获取连接状态信息
- 处理 IP 配置（DHCP/静态）

### AP（Access Point）模式
设备作为热点提供服务：
- 启用/禁用热点
- 管理连接的站点
- 配置热点参数（SSID、密码、频段等）
- 管理热点黑名单

### P2P（Peer-to-Peer）
设备间直接连接模式：
- 设备发现
- 组创建（GO 模式）
- 连接管理
- 持久化组管理

### System Ability（SA）
OpenHarmony 系统能力的抽象：
- 提供跨进程 IPC 通信
- 支持服务动态加载/卸载
- 提供服务状态监控

### N-API（Node-API）
JavaScript 与 Native C++ 之间的绑定层：
- 导出 JS 接口到应用层
- 封装 IPC 调用
- 处理异步操作（Promise/Callback）

### IPC（Inter-Process Communication）
进程间通信机制：
- 基于 OpenHarmony Binder
- 使用 Proxy-Stub 模式
- 支持 Service Ability 跨进程调用

---

## 技术栈

### 开发语言
| 层级 | 语言 |
|------|------|
| 应用层 | ArkTS/JavaScript |
| NAPI 绑定层 | C++ (N-API) |
| 服务实现层 | C++ |
| HAL 适配层 | C |
| 配置与脚本 | GN（构建） |

### 依赖的系统服务
根据 [bundle.json](../wifi/bundle.json:90-137)，依赖以下系统服务：

**基础服务**:
- `ability_base` - 能力基础框架
- `ability_runtime` - 能力运行时
- `access_token` - 访问令牌管理
- `bundle_framework` - Bundle 框架
- `c_utils` - C 工具库
- `ipc` - IPC 框架
- `safwk` - SA 框架
- `samgr` - SA 管理器
- `hilog` - 日志系统

**网络相关**:
- `netmanager_base` - 网络管理基础
- `netmanager_ext` - 网络管理扩展
- `netstack` - 网络协议栈
- `dhcp` - DHCP 服务

**安全相关**:
- `certificate_manager` - 证书管理
- `security_guard` - 安全守护
- `huks` - 硬件密钥存储系统

**其他**:
- `power_manager` - 电源管理
- `time_service` - 时间服务
- `i18n` - 国际化
- `relational_store` - 关系数据库
- `eventhandler` - 事件处理器
- `hiviewdfx` - 系统监控
- `hisysevent` - 系统事件
- `hiappevent` - 应用事件
- `ffrt` - 异步函数运行时
- `bounds_checking_function` - 边界检查

### 第三方依赖
- `googletest` - 测试框架
- `wpa_supplicant` - WiFi 认证守护进程

---

## 相关跳转

### 基础文档
- [01_Directory_Structure.md](01_Directory_Structure.md) - 了解目录结构和模块职责
- [README_zh.md](../README_zh.md) - 官方中文 README
- [README.md](../README.md) - 官方英文 README

### 技术文档
- [02_Architecture.md](02_Architecture.md) - 深入了解架构设计
- [03_Public_API_NAPI.md](03_Public_API_NAPI.md) - 学习对外 API
- [04_Inner_API.md](04_Inner_API.md) - 了解内部接口

### 构建与部署
- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建系统详解
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物说明

### 安全与调试
- [07_Security_Review.md](07_Security_Review.md) - 安全风险评审
- [08_FAQ.md](08_FAQ.md) - 常见问题解答

---

## 参考资料

### 代码证据
本文档中的所有关键结论均基于代码库内的直接证据：
- 文件路径：`wifi/path/to/file.ext`
- 行号引用：`wifi/path/to/file.ext:line_number`
- 符号名：函数、类、宏等

### 限制说明
- **不含测试**：本文档不引用测试相关内容（`test/`、`tests/`、`unittest/` 等）
- **基于当前代码**：本文档基于代码库当前状态生成
- **持续更新**：代码变更后需同步更新文档

---

**下一步**: 阅读 [01_Directory_Structure.md](01_Directory_Structure.md) 了解详细目录结构和模块职责。
