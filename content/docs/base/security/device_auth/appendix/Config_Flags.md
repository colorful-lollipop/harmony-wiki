# 配置开关

> 目的：汇总设备互信认证模块的所有配置宏和 feature flags，用于定制化编译。
>
> 适用范围：需要裁剪功能或定制编译的开发者。

---

## 1. bundle.json 特性开关

### 1.1 功能特性

| 特性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `device_auth_session_v1_enabled` | bool | true | 启用会话 v1 |
| `device_auth_session_v2_enabled` | bool | true | 启用会话 v2 |
| `device_auth_account_enabled` | bool | true | 启用账户相关认证 |
| `device_auth_pseudonym_enabled` | bool | true | 启用匿名 ID |
| `device_auth_p2p_lite_protocol_enabled` | bool | true | 启用 P2P 轻量协议 |
| `device_auth_p2p_standard_protocol_enabled` | bool | true | 启用 P2P 标准协议 |
| `device_auth_p2p_lite_protocol_legacy_enabled` | bool | true | 启用 P2P 轻量遗留协议 |
| `device_auth_account_lite_protocol_enabled` | bool | true | 启用账户轻量协议 |
| `device_auth_account_standard_protocol_enabled` | bool | true | 启用账户标准协议 |

### 1.2 配置特性

| 特性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `device_auth_storage_path` | string | - | 自定义存储路径 |
| `device_auth_hichain_thread_stack_size` | number | - | 线程栈大小 |
| `device_auth_enable_posix_interface` | bool | false | 启用 POSIX 接口 |
| `device_auth_enable_soft_bus_channel` | bool | true | 启用 SoftBus 通道 |

---

## 2. GN 构建变量

### 2.1 OS 级别

| 变量 | 可选值 | 说明 |
|------|--------|------|
| `os_level` | `standard` | 标准系统 |
| `os_level` | `small` | 小型系统 |
| `os_level` | `mini` | 轻量系统 |

### 2.2 构建类型

| 变量 | 可选值 | 说明 |
|------|--------|------|
| `build_type` | `debug` | 调试构建 |
| `build_type` | `release` | 发布构建 |

### 2.3 特征配置

| 变量 | 位置 | 默认值 | 说明 |
|------|------|--------|------|
| `deviceauth_feature_config` | `deviceauth_env.gni` | `//base/security/device_auth/default_config` | 特征配置路径 |
| `device_auth_enable_soft_bus_channel` | `deviceauth_env.gni` | `true` | SoftBus 通道 |
| `device_auth_use_customized_key_adapter` | `deviceauth_env.gni` | `false` | 自定义密钥适配 |

---

## 3. 预处理器宏

### 3.1 协议宏

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `ENABLE_PSEUDONYM` | `enable_pseudonym == true` | 启用匿名 ID |
| `ENABLE_P2P_BIND_ISO` | `enable_p2p_bind_lite_protocol` | ISO 绑定协议 |
| `ENABLE_P2P_BIND_DL_SPEKE` | `enable_p2p_bind_dl_speke_protocol` | DL-SPEKE 绑定 |
| `ENABLE_P2P_BIND_EC_SPEKE` | `enable_p2p_bind_standard_protocol` | EC-SPEKE 绑定 |
| `ENABLE_P2P_AUTH_ISO` | `enable_p2p_auth_lite_protocol` | ISO 认证协议 |
| `ENABLE_P2P_AUTH_EC_SPEKE` | `enable_p2p_auth_standard_protocol` | EC-SPEKE 认证 |
| `ENABLE_ISO` | account 或 lite 协议启用 | ISO 协议 |
| `ENABLE_EC_SPEKE` | account 或 standard 协议启用 | EC-SPEKE |
| `ENABLE_AUTH_CODE_IMPORT` | 启用 ISO | 认证码导入 |
| `ENABLE_PUB_KEY_EXCHANGE` | 启用 EC-SPEKE | 公钥交换 |
| `ENABLE_SAVE_TRUSTED_INFO` | `enable_session_v2` | 保存信任信息 |

### 3.2 会话版本宏

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `P2P_PAKE_DL_PRIME_LEN_384` | DL 协议 | DL 素数长度 384 |
| `P2P_PAKE_DL_PRIME_LEN_256` | DL 协议 | DL 素数长度 256 |
| `P2P_PAKE_EC_TYPE` | standard 协议 | EC 曲线类型 |

### 3.3 模块宏

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `DEV_AUTH_PLUGIN_ENABLE` | `enable_extend_plugin` | 插件系统 |
| `DEV_AUTH_HIVIEW_ENABLE` | standard 构建 | HiView 集成 |
| `DEV_AUTH_SERVICE_BUILD` | 服务构建 | 服务构建标识 |
| `DEV_AUTH_IS_ENABLE` | identity service | 身份服务 |
| `DEV_AUTH_ENABLE_CE` | standard HAL | CE 支持 |
| `DEV_AUTH_USE_JEMALLOC` | musl 内存分配 | jemalloc 使用 |
| `DEV_AUTH_ENABLE_CFI` | 标准 sanitizer | CFI 保护 |

### 3.4 功能宏

| 宏定义 | 说明 |
|--------|------|
| `HILOG_ENABLE` | 启用日志 |
| `SET_THREAD_NAME` | Linux 设置线程名 |

---

## 4. 默认配置

### 4.1 标准系统配置

**文件**：`default_config/standard/config.gni`

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `enable_session_mini` | `true` | mini 会话 |
| `enable_session_v2` | `true` | 会话 v2 |
| `enable_session_v1` | `true` | 会话 v1 |
| `enable_extend_plugin` | `true` | 扩展插件 |
| `enable_account` | `true` | 账户相关 |
| `enable_pseudonym` | `true` | 匿名 ID |
| `enable_p2p_bind_lite_protocol` | `true` | P2P 轻量绑定 |
| `enable_p2p_bind_dl_speke_protocol` | `true` | DL-SPEKE |
| `enable_p2p_bind_standard_protocol` | `true` | EC-SPEKE |
| `enable_p2p_auth_lite_protocol` | `true` | P2P 轻量认证 |
| `enable_p2p_auth_standard_protocol` | `true` | P2P 标准认证 |
| `enable_identity_service` | `true` | 身份服务 |
| `device_auth_enable_run_on_demand_qos` | `true` | 按需 QoS |

---

## 5. 编译选项

### 5.1 构建标志

| 选项 | 值 | 说明 |
|------|-----|------|
| `-O2` | 优化级别 | 优化 |
| `-ftrapv` | 整数溢出检测 | 整数溢出陷阱 |
| `-Wall` | 警告 | 全部警告 |
| `-Werror` | 警告转错误 | 警告视为错误 |
| `-Wextra` | 警告 | 额外警告 |
| `-fstack-protector-all` | 栈保护 | 全栈保护 |
| `-FPIC` | 位置无关代码 | 生成 PIC |
| `-D_FORTIFY_SOURCE=2` | FORTIFY | 运行时检查 |
| `-Wformat=2` | 格式化安全 | 格式化检查 |

### 5.2 Sanitizer 配置（标准系统）

| Sanitizer | 启用 | 说明 |
|-----------|------|------|
| `cfi` | true | 控制流完整性 |
| `cfi_cross_dso` | true | 跨 DSO CFI |
| `integer_overflow` | true | 整数溢出 |
| `boundary_sanitize` | true | 边界检查 |
| `ubsan` | true | 未定义行为 |
| `branch_protector_ret` | `"pac_ret"` | 返回地址保护 |

---

## 6. 使用示例

### 6.1 禁用不需要的协议

```bash
# 生成构建配置
gn gen out/standard --args='
  os_level = "standard"
  enable_p2p_bind_dl_speke_protocol = false
  enable_p2p_auth_lite_protocol = false
'
```

### 6.2 自定义存储路径

```bash
gn gen out/standard --args='
  os_level = "standard"
  device_auth_storage_path = "/data/app/device_auth"
'
```

### 6.3 启用调试构建

```bash
gn gen out/standard --args='
  os_level = "standard"
  build_type = "debug"
'
```

---

## 7. 相关跳转

| 内容 | 文档 |
|------|------|
| 构建配置 | [05_Build_Config.md](../05_Build_Config.md) |
| 架构设计 | [02_Architecture.md](../02_Architecture.md) |
| 安全评审 | [06_Security_Review.md](../06_Security_Review.md) |

---

*本文档最后更新：2026-02-06*
