# 关键配置开关

本文档描述分布式硬件管理框架的关键编译开关和运行时配置。

> **适用范围**: 需要了解或修改框架配置的开发者

---

## 编译时配置

### GN 开关 (distributedhardwarefwk.gni)

| 开关名称 | 类型 | 默认值 | 说明 |
|----------|------|--------|------|
| `distributed_hardware_fwk_low_latency` | `declare_args` | `false` | 低延迟/闭源模式 |
| `dhfwk_os_account` | `if-else` | `global_parts_info` 决定 | OS 账户支持 |

**证据**: `distributedhardwarefwk.gni:39-49`
```gni
declare_args() {
  distributed_hardware_fwk_low_latency = false
}

if (!defined(global_parts_info) ||
      defined(global_parts_info.account_os_account)) {
    dhfwk_os_account = true
  } else {
    dhfwk_os_account = false
  }
```

---

### 低延迟模式效果

当 `distributed_hardware_fwk_low_latency = true` 时：

| 效果 | 定义 | 文件位置 |
|------|------|----------|
| 添加 `DHARDWARE_LOW_LATENCY` 宏 | `services/.../BUILD.gn:146-148` |
| 添加 `DHARDWARE_OPEN_SOURCE` 宏 | `services/.../BUILD.gn:150-152` |
| 添加 `DHARDWARE_CHECK_RESOURCE` 宏 | `services/.../BUILD.gn:154-156` |
| 选择闭源配置文件 | `sa_profile/dhardware.cfg` |

---

### OS 账户支持效果

当 `dhfwk_os_account = true` 时：

| 效果 | 依赖 | 文件位置 |
|------|------|----------|
| 添加 `OS_ACCOUNT_PART` 宏 | - | `services/.../BUILD.gn:192` |
| 添加外部依赖 | `os_account:libaccountkits`, `os_account:os_account_innerkits` | `services/.../BUILD.gn:184-188` |

---

## 运行时配置

### SA 配置文件 (dhardware.cfg)

**路径**: `sa_profile/dhardware.cfg`

#### 基础配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | `dhardware` | 服务名 |
| `uid` | `dhardware` | 运行用户 |
| `gid` | `dhardware`, `input` | 运行组 |
| `ondemand` | `true` | 按需启动 |
| `apl` | `system_basic` | APL 等级 |

**证据**: `sa_profile/dhardware.cfg`
```json
{
  "services": [{
    "name": "dhardware",
    "path": ["/system/bin/sa_main", "/system/profile/dhardware.json"],
    "uid": "dhardware",
    "gid": ["dhardware", "input"],
    "ondemand": true,
    "apl": "system_basic"
  }]
}
```

---

### 权限配置

| 权限 | 说明 | 是否 ACL |
|------|------|----------|
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 | ❌ |
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | 软总线中心 | ❌ |
| `ohos.permission.CAMERA` | 相机 | ❌ |
| `ohos.permission.ACCESS_SERVICE_DM` | 设备管理访问 | ❌ |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 分布式硬件访问 | ❌ |
| `ohos.permission.ENABLE_DISTRIBUTED_HARDWARE` | 使能分布式硬件 | ❌ |
| `ohos.permission.REPORT_RESOURCE_SCHEDULE_EVENT` | 上报资源调度事件 | ❌ |
| `ohos.permission.MONITOR_DEVICE_NETWORK_STATE` | 监控网络状态 | ✅ |
| `ohos.permission.SYNC_PROFILE_DP` | 同步设备画像 | ✅ |

**证据**: `sa_profile/dhardware.cfg`
```json
"permission": [
  "ohos.permission.DISTRIBUTED_DATASYNC",
  "ohos.permission.CAMERA",
  ...
],
"permission_acls": [
  "ohos.permission.MONITOR_DEVICE_NETWORK_STATE",
  "ohos.permission.SYNC_PROFILE_DP"
]
```

---

## 安全加固配置

### 编译器标志

| 标志 | 值 | 说明 |
|------|-----|------|
| `-fstack-protector-strong` | 启用 | 堆栈保护 |
| `-D_FORTIFY_SOURCE=2` | 启用 | 运行时边界检查 |
| `-O2` | 优化级别 | 性能优化 |
| `-fpie` | 启用 | 位置无关代码 |

**证据**: `services/.../BUILD.gn:132-150`
```gn
cflags = [
  "-fstack-protector-strong",
  "-D_FORTIFY_SOURCE=2",
  "-O2",
]
```

### 链接标志

| 标志 | 值 | 说明 |
|------|-----|------|
| `-fpie` | 启用 | 位置无关可执行文件 |
| `-Wl,-z,relro` | 启用 | 只读重定位 |
| `-Wl,-z,now` | 启用 | 立即符号绑定 |

---

### Sanitizer 配置

| sanitizer | 值 | 说明 |
|-----------|-----|------|
| `boundary_sanitize` | `true` | 边界检查 |
| `integer_overflow` | `true` | 整数溢出检测 |
| `ubsan` | `true` | 未定义行为检测 |
| `cfi` | `true` | 控制流完整性 |
| `cfi_cross_dso` | `true` | 跨 DSO CFI |
| `branch_protector_ret` | `"pac_ret"` | ARM PAC 返回地址保护 |

**证据**: `services/.../BUILD.gn:20-28`
```gn
sanitize = {
  boundary_sanitize = true
  integer_overflow = true
  ubsan = true
  cfi = true
  cfi_cross_dso = true
  debug = false
}
branch_protector_ret = "pac_ret"
```

---

## 日志配置

### 日志标签

| 标签 | 值 | 用途 |
|------|-----|------|
| `DH_LOG_TAG` | `"distributedhardwaremanager_js"` | N-API 层 |
| `DH_LOG_TAG` | `"dhfwksvr"` | 服务层 |
| `LOG_DOMAIN` | `0xD004100` | 日志域 |

**证据**: `services/.../BUILD.gn:126-130`
```gn
defines = [
  "HI_LOG_ENABLE",
  "DH_LOG_TAG=\"dhfwksvr\"",
  "LOG_DOMAIN=0xD004100",
]
```

---

## 配置修改指南

### 添加新的编译开关

1. 在 `distributedhardwarefwk.gni` 中添加 `declare_args()`
2. 在目标 `BUILD.gn` 中添加条件编译
3. 更新本文档

### 修改 SA 配置

1. 编辑 `sa_profile/dhardware.cfg`
2. 更新权限声明
3. 更新 `bundle.json` 中的子组件声明

---

## 返回文档

- [返回主文档](../README.md)
- [GN 构建配置](../05_GN_Build.md)
- [安全评审](../07_Security_Review.md)
