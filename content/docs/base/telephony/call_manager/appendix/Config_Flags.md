# 配置开关

**目的**: 汇总所有编译开关和运行时配置

---

## 编译 Feature Flags

### 全局开关

| 开关名称 | 类型 | 默认值 | 说明 |
|----------|------|--------|------|
| `call_manager_feature_hfp_async_enable` | bool | false | HFP 异步启用 |
| `call_manager_feature_not_support_multicall` | bool | false | 禁用多方通话 |
| `call_manager_feature_support_dsoftbus` | bool | true | 支持分布式软总线 |
| `call_manager_feature_support_rtt` | bool | false | 支持 RTT |
| `call_manager_feature_support_hearing_aid` | bool | true | 支持助听器 |
| `call_manager_sos_no_ringback_tone` | bool | false | SOS 无回铃音 |
| `call_manager_watch_call_blocking` | bool | false | 手表呼叫阻断 |

**证据**: `callmanager.gni:15-21`

---

## 条件编译宏

### 功能开关

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `SUPPORT_DSOFTBUS` | `call_manager_feature_support_dsoftbus=true` | 启用分布式通信 |
| `HFP_ASYNC_ENABLE` | `call_manager_feature_hfp_async_enable=true` | HFP 异步模式 |
| `NOT_SUPPORT_MULTICALL` | `call_manager_feature_not_support_multicall=true` | 禁用多方通话 |
| `SUPPORT_RTT_CALL` | `call_manager_feature_support_rtt=true` | RTT 实时文本 |
| `SUPPORT_HEARING_AID` | `call_manager_feature_support_hearing_aid=true` | 助听器支持 |
| `CALL_MANAGER_SOS_NO_RINGBACK_TONE` | `call_manager_sos_no_ringback_tone=true` | SOS 无回铃音 |
| `CALL_MANAGER_WATCH_CALL_BLOCKING` | `call_manager_watch_call_blocking=true` | 手表阻断 |

### 平台相关

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `SUPPORT_MUTE_BY_DATABASE` | watch platform | 手表静音数据库 |
| `ABILITY_BLUETOOTH_SUPPORT` | bluetooth enabled | 蓝牙功能 |
| `ABILITY_SMS_SUPPORT` | sms_mms enabled | 短信功能 |
| `ABILITY_CELLULAR_SUPPORT` | cellular_call enabled | 蜂窝通话 |
| `ABILITY_POWER_SUPPORT` | power_manager enabled | 电源管理 |
| `CELLULAR_DATA_SUPPORT` | cellular_data enabled | 蜂窝数据 |

### 可选功能

| 宏定义 | 条件 | 说明 |
|--------|------|------|
| `ABILITY_SCREENLOCKMGR_SUPPORT` | screenlock_mgr enabled | 屏幕锁管理 |
| `SUPPORT_VIBRATOR` | sensors_miscdevice enabled | 振动器 |
| `HICOLLIE_ENABLE` | hicollie enabled | 性能分析 |
| `TELEPHONY_CUST_SUPPORT` | telephony_enhanced | 定制化支持 |
| `OHOS_BUILD_ENABLE_TELEPHONY_CUST` | telephony_cust | 构建定制化 |
| `OHOS_SUBSCRIBE_MOTION_ENABLE` | msdp_motion | 运动订阅 |
| `OHOS_SUBSCRIBE_USER_STATUS_ENABLE` | msdp_user_status | 用户状态 |

**证据**: `callmanager.gni:174-326`

---

## 运行时配置

### SA 配置

```json
// sa_profile/4005.json
{
    "process": "telecom",
    "systemability": [{
        "name": 4005,
        "libpath": "libtel_call_manager.z.so",
        "run-on-create": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

### 日志配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `TELEPHONY_LOG_TAG` | "CallManager" | 日志 Tag |
| `LOG_DOMAIN` | 0xD001F10 | 日志域 |
| `LOG_TAG` | "CallManager" | 打印 Tag |

**证据**: `BUILD.gn:67-70`

---

## 性能配置

### CFI 防护

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `sanitize.cfi` | true | 启用 CFI |
| `sanitize.cfi_cross_dso` | true | 跨 DSO CFI |
| `sanitize.cfivcall_icall_only` | true | 仅虚拟调用 |
| `branch_protector_ret` | "pac_ret" | PAC 返回保护 |

**证据**: `BUILD.gn:18-24`

### 栈保护

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `-fstack-protector-all` | enabled | 全量栈保护 |
| `-D_FORTIFY_SOURCE=2` | enabled | FORTIFY_SOURCE |

**证据**: `BUILD.gn:50-52`

---

## 相关文档

- [构建系统](../03_Build_System.md)
- [安全评审](../04_Security_Review.md)
