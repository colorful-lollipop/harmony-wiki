# 构建配置

> 目的：提供设备互信认证模块的 GN 构建配置参考，包括 targets、依赖关系、编译产物和特征开关。
>
> 适用范围：需要编译本模块或理解构建系统的开发者。

---

## 1. 根构建文件

### 1.1 BUILD.gn 位置

**路径**：`/base/security/device_auth/BUILD.gn`

### 1.2 主构建组

```gn
# SDK 构建组
group("deviceauth_sdk_build") {
  if (os_level == "standard" || os_level == "small") {
    deps = [ "services:deviceauth_sdk" ]
  }
}

# 服务构建组
group("deviceauth_service_build") {
  if (os_level == "standard" || os_level == "small") {
    deps = [
      "services:deviceauth_service",
      "services/sa/sa_profile:deviceauth_sa_profile",
    ]
  }
}

# 核心库构建组
group("deviceauth_build") {
  deps = [ "services:deviceauth" ]
}

# N-API 构建组
group("deviceauth_napi_build") {
  if (os_level == "standard") {
    deps = [ "interfaces/kits/napi:deviceauth_napi" ]
  }
}
```

**证据**：`BUILD.gn:16-39`

---

## 2. Targets 清单

### 2.1 核心库 Targets

| Target 名称 | 类型 | 输出 | OS 级别 |
|-------------|------|------|--------|
| `deviceauth` | `static_library` | `libdeviceauth.a` | mini/small |
| `deviceauth` | `ohos_static_library` | `libdeviceauth.a` | standard |

### 2.2 SDK Targets

| Target 名称 | 类型 | 输出 | OS 级别 | 安装路径 |
|-------------|------|------|---------|----------|
| `deviceauth_sdk` | `shared_library` | `libdeviceauth_sdk.so` | small | - |
| `deviceauth_sdk` | `ohos_shared_library` | `libdeviceauth_sdk.so` | standard | `system/lib/` |

### 2.3 服务 Targets

| Target 名称 | 类型 | 输出 | OS 级别 | 安装路径 |
|-------------|------|------|---------|----------|
| `deviceauth_service` | `executable` | `deviceauth_service` | small | - |
| `deviceauth_service` | `ohos_shared_library` | `libdeviceauth_service.so` | standard | `system/lib/` |
| `pre_deviceauth_service` | `ohos_prebuilt_etc` | `deviceauth_service.cfg` | standard | `system/etc/init/` |

### 2.4 N-API Target

| Target 名称 | 类型 | 输出 | OS 级别 | 安装路径 |
|-------------|------|------|---------|----------|
| `deviceauth_napi` | `ohos_shared_library` | `libdeviceauth_napi.so` | standard | `system/lib/module/security/` |

### 2.5 HAL Targets

| Target 名称 | 类型 | 输出 | OS 级别 |
|-------------|------|------|--------|
| `deviceauth_hal_liteos` | `static_library` | `libdeviceauth_hal_liteos.a` | mini |
| `deviceauth_hal_linux` | `static_library` | `libdeviceauth_hal_linux.a` | small |
| `deviceauth_hal_linux` | `ohos_static_library` | `libdeviceauth_hal_linux.a` | standard |

### 2.6 SA Profile Target

| Target 名称 | 类型 | 输出 | 说明 |
|-------------|------|------|------|
| `deviceauth_sa_profile` | `ohos_sa_profile` | `4701.json` | SA ID 配置文件 |

**证据**：`services/BUILD.gn`

---

## 3. 依赖关系

### 3.1 内部依赖

```
deviceauth_sdk
  └── deps_adapter:deviceauth_hal_linux

deviceauth_service
  ├── :deviceauth (static)
  ├── :pre_deviceauth_service
  └── deps_adapter:deviceauth_hal_linux

deviceauth_napi
  ├── deps_adapter:deviceauth_hal_linux
  └── services:deviceauth_sdk
```

### 3.2 外部依赖

**标准系统依赖**：

| 组件 | 用途 |
|------|------|
| `ability_base` | 基础能力 |
| `access_token` | 访问控制 |
| `bounds_checking_function` | 安全函数 |
| `cJSON` | JSON 解析 |
| `common_event_service` | 公共事件 |
| `c_utils` | 通用工具 |
| `dsoftbus` | 软总线 |
| `hilog` | 日志 |
| `hisysevent` | 安全事件 |
| `hitrace` | 追踪 |
| `huks` | 密钥管理 |
| `init` | 初始化 |
| `json` | JSON |
| `mbedtls` | 加密库 |
| `napi` | N-API |
| `openssl` | OpenSSL |
| `os_account` | 账号管理 |
| `samgr` | SA 管理 |
| `safwk` | SA 框架 |
| `ipc` | IPC |
| `netmanager_base` | 网络管理 |
| `memmgr` | 内存管理 |
| `eventhandler` | 事件处理 |

**证据**：`bundle.json:47-73`

---

## 4. 编译产物

### 4.1 产物清单

| 产物 | 类型 | 源 Target | 安装路径 |
|------|------|-----------|----------|
| `libdeviceauth_sdk.so` | 共享库 | `services:deviceauth_sdk` | `system/lib/` |
| `libdeviceauth_service.so` | 共享库 | `services:deviceauth_service` | `system/lib/` |
| `libdeviceauth_napi.so` | 共享库 | `interfaces/kits/napi:deviceauth_napi` | `system/lib/module/security/` |
| `libhichainsdk.so` | 共享库 | `frameworks/deviceauth_lite:hichainsdk` | `system/lib/` |
| `deviceauth_service.cfg` | 配置文件 | `pre_deviceauth_service` | `system/etc/init/` |
| `4701.json` | SA 配置 | `services/sa/sa_profile:deviceauth_sa_profile` | - |

### 4.2 符号导出

使用版本脚本控制导出符号：

| 脚本路径 | 用途 |
|----------|------|
| `services/device_auth.map` | SDK 和服务符号导出 |
| `interfaces/kits/napi/libdeviceauth_napi.map` | N-API 符号导出 |

**主要导出符号**：
- `InitDeviceAuthService` - 初始化服务
- `DestroyDeviceAuthService` - 销毁服务
- `GetGmInstance` - 获取群组管理实例
- `GetGaInstance` - 获取群组认证实例
- `GetCredMgrInstance` - 获取凭证管理实例
- `GetCredAuthInstance` - 获取凭证认证实例

**证据**：`BUILD.gn`，`bundle.json`

---

## 5. 特征开关

### 5.1 bundle.json 中的特性

```json
"features": [
  "device_auth_session_v1_enabled",
  "device_auth_session_v2_enabled",
  "device_auth_account_enabled",
  "device_auth_pseudonym_enabled",
  "device_auth_p2p_lite_protocol_enabled",
  "device_auth_p2p_standard_protocol_enabled",
  "device_auth_p2p_lite_protocol_legacy_enabled",
  "device_auth_account_lite_protocol_enabled",
  "device_auth_account_standard_protocol_enabled",
  "device_auth_storage_path",
  "device_auth_hichain_thread_stack_size",
  "device_auth_enable_posix_interface",
  "device_auth_enable_soft_bus_channel"
]
```

### 5.2 构建特征配置

**标准系统配置** (`default_config/standard/config.gni`)：

| 特征 | 默认值 | 说明 |
|------|--------|------|
| `enable_session_mini` | `true` | 支持 mini 会话 |
| `enable_session_v2` | `true` | 支持会话 v2 |
| `enable_session_v1` | `true` | 支持会话 v1 |
| `enable_extend_plugin` | `true` | 扩展插件 |
| `enable_account` | `true` | 账户相关认证 |
| `enable_pseudonym` | `true` | 匿名 ID |
| `enable_p2p_bind_lite_protocol` | `true` | P2P 绑定轻量协议 |
| `enable_p2p_bind_dl_speke_protocol` | `true` | DL-SPEKE 协议 |
| `enable_p2p_bind_standard_protocol` | `true` | EC-SPEKE 标准协议 |
| `enable_identity_service` | `true` | 身份服务 |

### 5.3 预处理器宏

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `ENABLE_PSEUDONYM` | `enable_pseudonym == true` | 启用匿名 ID |
| `ENABLE_P2P_BIND_ISO` | `enable_p2p_bind_lite_protocol` | ISO 协议 |
| `ENABLE_P2P_BIND_DL_SPEKE` | `enable_p2p_bind_dl_speke_protocol` | DL-SPEKE |
| `ENABLE_P2P_BIND_EC_SPEKE` | `enable_p2p_bind_standard_protocol` | EC-SPEKE |
| `ENABLE_SAVE_TRUSTED_INFO` | `enable_session_v2` | 保存信任信息 |
| `DEV_AUTH_PLUGIN_ENABLE` | `enable_extend_plugin` | 插件支持 |
| `DEV_AUTH_HIVIEW_ENABLE` | 标准构建 | HiView 集成 |

---

## 6. 构建命令示例

### 6.1 标准系统

```bash
# 生成标准系统构建
gn gen out/standard --args='os_level="standard"'

# 编译 device_auth 模块
hb build deviceauth -p
```

### 6.2 小型系统

```bash
# 生成小型系统构建
gn gen out/small --args='os_level="small"'

# 编译 device_auth 模块
hb build deviceauth -p
```

### 6.3 编译 N-API

```bash
# 确保 os_level 为 standard
gn gen out/standard --args='os_level="standard"'

# 编译 N-API
hb build deviceauth_napi -p
```

---

## 7. 相关跳转

| 内容 | 文档 |
|------|------|
| 项目概览 | [01_Overview.md](./01_Overview.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| 配置开关 | [appendix/Config_Flags.md](./appendix/Config_Flags.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*本文档最后更新：2026-02-06*
