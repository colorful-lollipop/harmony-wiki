# 编译产物

**目的**: 描述 WLAN 组件的编译产物、安装路径和运行时加载关系

**适用范围**: 构建工程师、系统集成者、OTA 升级工程师

**生成时间**: 2026-02-06

---

## 编译产物概览

WLAN 组件编译后产生以下类型的产物：

| 产物类型 | 数量 | 说明 |
|---------|------|------|
| 共享库（.so） | 12+ | 主要运行时库 |
| NDK 库（.so + .h） | 1 | NDK 接口库 |
| NAPI 模块（.z.so） | 5 | JavaScript 模块 |
| 应用（.hap） | 1 | 门户认证应用 |
| SA Profile 配置（.json） | 4 | System Ability 配置 |
| 静态库（.a） | 9+ | 链接到共享库 |

---

## Shared Libraries（标准系统）

### 基础库

| 库名称 | 路径 | 大小估计 | 依赖 | 说明 |
|---------|------|----------|------|------|
| `libwifi_ext_lib.so` | `/system/lib/` | ~50KB | - | 国际化工具库 |

### Native SDK 库

| 库名称 | 路径 | 大小估计 | 依赖 | 说明 |
|---------|------|----------|------|------|
| `libwifi_sdk.so` | `/system/lib/` | ~500KB | `libwifi_ext_lib.so`, `wifi_utils` | C++ Native SDK，封装所有 IPC Proxy |

### NDK 库

| 库名称 | 路径 | 大小估计 | 依赖 | 说明 |
|---------|------|----------|------|------|
| `libwifi_ndk.so` | `/system/lib/` + `/ndk/` | ~100KB | `libwifi_sdk.so` | NDK 接口库（头文件在 `ndk/oh_wifi.h`）|

### 核心服务库

| 库名称 | 路径 | 大小估计 | 依赖 | 说明 |
|---------|------|----------|------|------|
| `libwifi_manager_service.so` | `/system/lib/` | ~3MB | 所有子服务静态库 | 核心管理服务 |
| `libwifi_device_ability.z.so` | `/system/lib/` | ~800KB | `libwifi_sdk.so` | SA 1120（设备管理）|
| `libwifi_scan_ability.z.so` | `/system/lib/` | ~600KB | `libwifi_sdk.so` | SA 1124（扫描服务）|
| `libwifi_hotspot_ability.z.so` | `/system/lib/` | ~700KB | `libwifi_sdk.so` | SA 1121（热点管理）|
| `libwifi_p2p_ability.z.so` | `/system/lib/` | ~500KB | `libwifi_sdk.so` | SA 1123（P2P 服务）|

### 子服务库

| 库名称 | 路径 | 大小估计 | 说明 |
|---------|------|----------|------|
| `libwifi_ap_service.so` | `/system/lib/` | ~400KB | AP 服务 |
| `libwifi_p2p_service.so` | `/system/lib/` | ~450KB | P2P 服务 |
| `libwifi_scan_service.so` | `/system/lib/` | ~350KB | 扫描服务 |
| `libwifi_sta_service.so` | `/system/lib/` | ~380KB | STA 服务 |

---

## NAPI 模块产物

### 主 NAPI 模块

| 库名称 | 模块路径 | 导出 JS 命名空间 | 加载时机 |
|---------|----------|-----------|----------|
| `wifi.z.so` | `/system/lib/module/wifi.z.so` | `@ohos/wifi` | 应用导入时 |
| `wifiext.z.so` | `/system/lib/module/wifiext.z.so` | `@ohos.wifiext` | 条件编译（FEATURE_AP_EXTENSION）|

### WiFi Manager NAPI 模块

| 库名称 | 模块路径 | 导出 JS 命名空间 | 说明 |
|---------|----------|-----------|------|
| `wifimanager.z.so` | `/system/lib/module/wifimanager.z.so` | `@ohos.wifiManager` | ArkTS WiFi Manager |
| `wifimanagerext.z.so` | `/system/lib/module/wifimanagerext.z.so` | `@ohos.wifiManagerExt` | WiFi Manager 扩展 |

### Cangjie FFI 模块

| 库名称 | 模块路径 | 导出 JS 命名空间 | 说明 |
|---------|----------|-----------|------|
| `libcj_wifi_ffi.so` | `/system/lib/module/libcj_wifi_ffi.so` | - | Cangjie 语言 FFI 绑定 |

---

## 应用产物

### 门户认证应用

| 文件名 | 路径 | 大小估计 | 说明 |
|---------|------|----------|------|
| `PortalLogin.hap` | `/system/app/PortalLogin/` | ~2MB | 门户认证应用 |

**应用结构**:
- ETS/JavaScript 源码（`entry/src/main/ets/`）
- 资源文件（`AppScope/resources/`）
- 应用配置（`app.json` 或 `module.json`）
- 签名（`signature/portallogin.p7b`）

**安装目录**:
```
/system/app/PortalLogin/
├── PortalLogin.hap
└── libs/ (依赖库)
```

---

## System Ability Profile 配置

### SA 配置文件

| SA ID | 配置文件 | 库文件 | 运行进程 | 说明 |
|-------|---------|---------|----------|------|
| 1120 | `/system/etc/sa_profile/1120.json` | `libwifi_device_ability.z.so` | `wifi_manager_service` | WiFi 设备管理服务 |
| 1121 | `/system/etc/sa_profile/1121.json` | `libwifi_scan_ability.z.so` | `wifi_manager_service` | WiFi 扫描服务 |
| 1123 | `/system/etc/sa_profile/1123.json` | `libwifi_p2p_ability.z.so` | `wifi_manager_service` | WiFi P2P 服务 |
| 1124 | `/system/etc/sa_profile/1124.json` | `libwifi_hotspot_ability.z.so` | `wifi_manager_service` | WiFi 热点服务 |

### Profile 配置结构
**证据**: `wifi/services/wifi_standard/sa_profile/1120.json`

```json
{
  "process": "wifi_manager_service",
  "start_on_demand": true,
  "distributed": false,
  "bundle_name": ["wifi_manager_service"],
  "permission": [
    "ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS",
    "ohos.permission.ACCESS_CERT_MANAGER",
    // ... 20+ 个权限
  ]
}
```

**关键字段**:
- `process`: SA 运行的进程名
- `start_on_demand`: 是否按需启动（true）
- `distributed`: 是否分布式（false）
- `bundle_name`: Bundle 名称列表
- `permission`: SA 需要的权限列表

---

## 运行时加载关系

### 加载顺序

```
1. Init 进程启动
   ↓
2. 加载 libwifi_manager_service.so
   ↓
3. 加载 System Ability（通过 SA Profile）
   ├─ Load SA 1120 (WifiDeviceMgrServiceImpl)
   ├─ Load SA 1124 (WifiScanMgrServiceImpl)
   ├─ Load SA 1121 (WifiHotspotMgrServiceImpl)
   └─ Load SA 1123 (WifiP2pServiceImpl) [条件]
   ↓
4. 加载依赖库
   ├─ libwifi_sdk.so (IPC Proxy)
   ├─ libwifi_ext_lib.so
   └─ libwifi_utils.so
   ↓
5. SA 服务调用 OnStart()
   ├─ 初始化各子服务（STA/AP/P2P/Scan）
   ├─ 加载 HAL 驱动
   └─ 调用 Publish() 向 SAMgr 注册
   ↓
6. 服务就绪，开始接受 IPC 调用
```

### IPC Binder 链接

```
JS Application
    ↓ 导入 @ohos.wifi
    ↓ 动态加载 wifi.z.so (NAPI 模块)
    ↓
wifi.z.so (NAPI 模块)
    ↓ 通过 IPC 调用 System Ability
    ↓
libwifi_sdk.so (Native SDK)
    ↓
WifiDeviceProxy
    ↓
libwifi_device_ability.z.so (SA 1120)
    ↓
WifiDeviceMgrServiceImpl
    ↓
WiFi Manager Service
    ↓
WiFi HAL/HDI
```

### 库依赖关系

```
libwifi_manager_service.so
├── 依赖：libwifi_toolkit.a (静态链接)
├── 依赖：libwifi_sta_service.a (静态链接)
├── 依赖：libwifi_scan_service.a (静态链接)
├── 依赖：libwifi_common_service.a (静态链接)
├── 依赖：libwifi_manager_service_static.a (静态链接)
└── 运行时依赖：libwifi_sdk.so
```

---

## 安装路径

### System 库
```
/system/lib/
├── libwifi_ext_lib.so
├── libwifi_sdk.so
├── libwifi_ndk.so
├── libwifi_manager_service.so
├── libwifi_device_ability.z.so
├── libwifi_scan_ability.z.so
├── libwifi_hotspot_ability.z.so
├── libwifi_p2p_ability.z.so
├── libwifi_ap_service.so
├── libwifi_p2p_service.so
├── libwifi_scan_service.so
└── libwifi_sta_service.so
```

### NDK 库
```
/system/lib/
└── libwifi_ndk.so

/system/ndk/
└── include/oh_wifi.h (NDK 头文件）
```

### NAPI 模块
```
/system/lib/module/
├── wifi.z.so
├── wifiext.z.so
├── wifimanager.z.so
├── wifimanagerext.z.so
└── libcj_wifi_ffi.so
```

### 应用
```
/system/app/
└── PortalLogin/
    ├── PortalLogin.hap
    └── libs/
```

### SA Profile 配置
```
/system/etc/sa_profile/
├── 1120.json
├── 1121.json
├── 1123.json
└── 1124.json
```

---

## 产物大小估算

| 类别 | 总大小估计 | 说明 |
|--------|----------|------|
| 核心服务库（.so） | ~6.5MB | WiFi Manager 服务及其子服务 |
| System Ability 库（.so） | ~2.5MB | 4 个 SA 库 |
| NAPI 模块（.z.so） | ~1.2MB | 5 个 JavaScript 模块 |
| 基础库（.so） | ~50KB | 国际化工具 |
| 应用（.hap） | ~2MB | 门户认证应用 |
| **总计** | ~12.5MB | 不包括依赖库和共享库 |

**注意**: 实际大小取决于编译配置（debug/release）、优化级别和目标架构。

---

## 版本信息

### 动态库版本
- `.so` 文件不包含版本号
- 版本信息存储在 bundle.json 中
- 运行时可通过系统包管理器查询

### 组件版本
**证据**: `wifi/bundle.json:2-4`

```json
{
  "name": "@ohos/wifi",
  "version": "3.1.0",
  "description": "The WLAN module provides basic WLAN functions...",
  "license": "Apache License 2.0",
  "component": {
    "name": "wifi",
    "subsystem": "communication"
  }
}
```

---

## 相关跳转

- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建系统详解
- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 架构说明

---

**注意**: 编译产物的大小和路径可能因系统配置而有所不同，实际安装位置以系统分区方案为准。
