# 项目概述

> 本文档描述 DRM Framework 的项目定位、核心功能、设计原则。

## 项目定位

| 属性 | 值 |
|------|-----|
| **部件名称** | @ohos/drm_framework |
| **版本** | 3.1 |
| **子系统** | multimedia |
| **系统能力** | SystemCapability.Multimedia.Drm.Core |
| **License** | Apache License 2.0 |

## 核心功能

DRM 框架提供完整的数字版权管理能力，主要包括：

### 1. DRM 证书管理

| 功能 | 描述 | API |
|------|------|-----|
| 生成证书请求 | 生成发往证书服务器的请求数据 | `generateKeySystemRequest()` |
| 处理证书响应 | 处理证书服务器返回的响应 | `processKeySystemResponse()` |
| 查询证书状态 | 获取当前设备的证书状态 | `getCertificateStatus()` |

**代码位置**: `media_key_system_napi.cpp`, `native_mediakeysystem.h`

### 2. DRM 许可证管理

| 功能 | 描述 | API |
|------|------|-----|
| 生成许可证请求 | 生成发往许可证服务器的请求 | `generateMediaKeyRequest()` |
| 处理许可证响应 | 处理许可证服务器返回的许可证 | `processMediaKeyResponse()` |
| 离线许可证 | 许可证的存储、恢复、清除 | `getOfflineMediaKeyIds()`, `restoreOfflineMediaKeys()` |

**代码位置**: `key_session_napi.cpp`, `native_mediakeysession.h`

### 3. DRM 节目授权

- 底层 DRM 插件根据许可证对 DRM 节目进行授权
- 授权状态查询：`checkMediaKeyStatus()`
- 密钥过期处理：通过事件回调通知

### 4. DRM 节目解密

| 功能 | 描述 | API |
|------|------|-----|
| 获取解密模块 | 获取用于解密的 MediaDecryptModule | `getDecryptModule()` |
| 安全解码器检测 | 检查是否需要硬件安全解码器 | `requireSecureDecoderModule()` |

**代码位置**: `media_decrypt_module_service.cpp`

## 模块架构

DRM 框架由 8 大核心模块组成：

```
┌─────────────────────────────────────────────────────────────┐
│                     DRM Framework                             │
├─────────────────┬─────────────────┬─────────────────────────┤
│ MediaKeySystem  │ MediaKeySession │ MediaDecryptModule      │
│ Factory         │                 │                         │
├─────────────────┼─────────────────┼─────────────────────────┤
│ MediaKeySystem  │ MediaKeySession │ MediaDecryptModule      │
│ Client          │ Client          │ Client                  │
├─────────────────┼─────────────────┼─────────────────────────┤
│ MediaKeySystem  │ MediaKeySession │ MediaDecryptModule      │
│ Service         │ Service         │ Service                 │
├─────────────────┼─────────────────┼─────────────────────────┤
│                 HDI Interface (IMediaKeySystem, ...)        │
├─────────────────┴─────────────────┴─────────────────────────┤
│                    DRM Plugin (厂商实现)                      │
└─────────────────────────────────────────────────────────────┘
```

### 模块职责

| 模块 | 职责 |
|------|------|
| MediaKeySystemFactory | DRM 方案枚举、MediaKeySystem 实例创建 |
| MediaKeySystem | DRM 证书管理、配置属性、离线密钥管理 |
| MediaKeySession | 许可证请求/处理、密钥状态管理 |
| MediaDecryptModule | 媒体数据解密 |

## 运行环境

### 系统依赖

```json
// bundle.json 依赖
{
  "ability_base", "ability_runtime", "access_token",
  "curl", "safwk", "napi", "samgr", "hitrace",
  "ipc", "hisysevent", "c_utils", "hilog",
  "hidumper", "hicollie", "hdf_core", "eventhandler",
  "bundle_framework", "drivers_interface_drm",
  "memmgr", "hiappevent", "json", "init",
  "data_share", "os_account", "runtime_core",
  "netmanager_base"
}
```

### 硬件要求

- 支持 DRM 的显示子系统
- 可选: 硬件安全模块 (DRM 插件要求)

## 设计原则

1. **分层架构**: 应用层 → Native框架 → SA服务 → HDI → 插件
2. **进程隔离**: DRM 服务运行在独立进程 (SA 3012)
3. **插件化**: 支持多 DRM 方案 (通过 HDI 接口)
4. **安全优先**: 敏感操作在安全环境执行

## 相关文档

- [目录结构](02_Directory_Structure.md) - 代码组织
- [架构设计](05_Architecture.md) - 详细架构
- [安全评审](08_Security_Review.md) - 安全分析
