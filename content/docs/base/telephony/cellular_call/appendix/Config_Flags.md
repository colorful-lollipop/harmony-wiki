# 附录 - 配置标志说明

## 目的

本文档详细说明 Cellular Call 模块的**所有编译配置标志、Feature 开关和运行时配置**，供开发者查阅和配置参考。

---

## 编译时配置 (cellularcall.gni)

### Feature 标志

| 标志名 | 类型 | 默认值 | 宏定义 | 功能描述 | 证据位置 |
|--------|------|--------|--------|----------|----------|
| `cellular_call_dynamic_start` | bool | false | 无 | SA 动态启动开关 | `cellularcall.gni:15` |
| `cellular_call_tel_power_mode` | bool | false | `BASE_POWER_IMPROVEMENT_FEATURE` | 省电模式 | `cellularcall.gni:16` |
| `cellular_call_support_UT` | bool | true | 无 | 单元测试支持 | `cellularcall.gni:17` |
| `cellular_call_satellite` | bool | false | `CELLULAR_CALL_SATELLITE` | 卫星通话功能 | `cellularcall.gni:18` |
| `cellular_call_support_rtt` | bool | false | `SUPPORT_RTT_CALL` | RTT 实时文本通话 | `cellularcall.gni:19` |

---

## 配置详解

### 1. cellular_call_dynamic_start

**功能**: 控制 SA 的启动模式

| 模式 | 配置值 | SA 配置 | 适用场景 |
|------|--------|----------|----------|
| **静态启动** | `false` | `4006.json` | 设备启动时预加载 |
| **动态启动** | `true` | `4006_dynamic.json` | 按需启动，节省内存 |

**配置影响**:

```gn
# cellularcall.gni
cellular_call_dynamic_start = false  # 静态
# cellular_call_dynamic_start = true  # 动态

# BUILD.gn 条件
if (!cellular_call_dynamic_start) {
  sources = [ "4006.json" ]        # 静态配置
} else {
  sources = [ "4006_dynamic.json" ]  # 动态配置
}
```

**证据位置**: `sa_profile/BUILD.gn:18-22`

---

### 2. cellular_call_tel_power_mode

**功能**: 启用省电模式，优化通话功耗

| 模式 | 宏定义 | 效果 |
|------|--------|------|
| **关闭** | 无 | 标准功耗模式 |
| **开启** | `BASE_POWER_IMPROVEMENT_FEATURE` | 省电优化 |

**代码启用**:

```gn
# cellularcall.gni
if (cellular_call_tel_power_mode) {
  global_defines += [ "BASE_POWER_IMPROVEMENT_FEATURE" ]
}

# BUILD.gn 条件编译
#ifdef BASE_POWER_IMPROVEMENT_FEATURE
void StartOrderEventSubscriber();  // 省电模式专用
#endif
```

**证据位置**:
- `cellularcall.gni:25`
- `cellular_call_service.h:760-762`

---

### 3. cellular_call_support_UT

**功能**: 启用单元测试支持

| 值 | 效果 |
|----|------|
| `true` | 编译单元测试代码 |
| `false` | 不编译测试代码 |

**注意**: 此标志仅影响测试代码编译，不影响业务代码。

---

### 4. cellular_call_satellite ⭐

**功能**: 启用卫星通话功能（卫星通信专用）

| 值 | 效果 |
|----|------|
| `false` | 不包含卫星通话代码 |
| `true` | 包含完整卫星通话功能 |

**编译影响**:

```gn
# cellularcall.gni
if (cellular_call_satellite) {
  global_defines += [ "CELLULAR_CALL_SATELLITE" ]
}

# BUILD.gn 源文件
if (cellular_call_satellite) {
  sources += [
    "services/connection/src/cellular_call_connection_satellite.cpp",
    "services/control/src/satellite_control.cpp",
    "services/satellite_service_interaction/src/satellite_call_callback_stub.cpp",
    "services/satellite_service_interaction/src/satellite_call_client.cpp",
    "services/satellite_service_interaction/src/satellite_call_proxy.cpp",
  ]
}
```

**启用后新增功能**:

| 接口 | 功能 | 证据位置 |
|------|------|----------|
| `SatelliteControl` | 卫星通话控制 | `services/control/include/satellite_control.h` |
| `SatelliteCallConnection` | 卫星连接管理 | `services/connection/include/cellular_call_connection_satellite.h` |
| `SatelliteCallInterface` | 卫星通话接口 | `interfaces/innerkits/satellite/satellite_call_interface.h` |

**证据位置**:
- `cellularcall.gni:28-30`
- `BUILD.gn:58-66`

---

### 5. cellular_call_support_rtt

**功能**: 启用 RTT (Real-Time Text) 实时文本通话功能

| 值 | 效果 |
|----|------|
| `false` | 不支持 RTT |
| `true` | 支持 RTT 实时文本 |

**代码启用**:

```gn
# cellularcall.gni
if (cellular_call_support_rtt) {
  global_defines += [ "SUPPORT_RTT_CALL" ]
}

# BUILD.gn 条件
#ifdef SUPPORT_RTT_CALL
int32_t SetRttCapability(int32_t slotId, bool isEnable) override;
int32_t UpdateImsRttCallMode(int32_t slotId, int32_t callId, ImsRTTCallMode mode) override;
#endif
```

**RTT 相关接口**:

| 接口 | 功能 | 证据位置 |
|------|------|----------|
| `SetRttCapability()` | RTT 开关设置 | `cellular_call_service.h:691` |
| `UpdateImsRttCallMode()` | RTT 模式更新 | `cellular_call_service.h:701` |
| `ImsRTTCallMode` | RTT 模式枚举 | `ims_call_types.h` |

**证据位置**:
- `cellularcall.gni:32-34`
- `cellular_call_service.h:683-702`

---

## 构建配置 (BUILD.gn)

### 编译选项

| 选项 | 值 | 功能 | 证据位置 |
|------|------|------|----------|
| `sanitize.cfi` | `true` | 控制流完整性保护 | `BUILD.gn:19` |
| `sanitize.cfi_cross_dso` | `true` | 跨 DSO CFI 保护 | `BUILD.gn:20` |
| `branch_protector_ret` | `"pac_ret"` | PACRET 返回地址保护 | `BUILD.gn:23` |
| `-fstack-protector-all` | cflags_cc | 堆栈保护 | `BUILD.gn:117` |
| `-O2` | cflags_cc | 优化等级 | `BUILD.gn:118` |
| `-D_FORTIFY_SOURCE=2` | cflags_cc | FORTIFY_SOURCE | `BUILD.gn:119` |

### 设备特定配置

| 设备 | 标志 | 功能 | 证据位置 |
|------|------|------|----------|
| `rk3568` | `CALL_MANAGER_AUTO_START_OPTIMIZE` | CallManager 启动优化 | `BUILD.gn:123` |

```gn
if (device_name == "rk3568") {
  defines += [ "CALL_MANAGER_AUTO_START_OPTIMIZE" ]
}
```

---

## 组件配置 (bundle.json)

### SysCap

```
SystemCapability.Telephony.CellularCall
```

**证据位置**: `bundle.json:22`

### 构建组

| 组类型 | Targets |
|--------|---------|
| service_group | `//base/telephony/cellular_call:tel_cellular_call` |
| service_group | `//base/telephony/cellular_call/sa_profile:cellular_call_sa_profile` |

### Inner Kits

| Kit | Headers | 产物 |
|-----|---------|------|
| tel_ims_call_api | `interfaces/innerkits/ims/*`, `interfaces/innerkits/ims_common/*` | - |
| tel_satellite_call_api | `interfaces/innerkits/satellite/*` | - |

---

## SA 配置

### 静态配置 (4006.json)

```json
{
    "process": "telephony",
    "systemability": [{
        "name": 4006,
        "libpath": "libtel_cellular_call.z.so",
        "run-on-create": true,
        "depend": [4010],
        "distributed": false,
        "dump_level": 1
    }]
}
```

### 动态配置 (4006_dynamic.json)

与静态配置类似，但支持按需启动。

---

## 配置组合矩阵

| 场景 | dynamic_start | tel_power_mode | satellite | rtt | 说明 |
|------|---------------|----------------|-----------|-----|------|
| **标准手机** | false | false | false | false | 普通蜂窝通话 |
| **省电优化** | false | true | false | false | 功耗敏感设备 |
| **卫星通信** | false | false | true | false | 卫星电话设备 |
| **RTT 支持** | false | false | false | true | 无障碍通话 |
| **全功能** | true | true | true | true | 功能最全配置 |
| **轻量动态** | true | false | false | false | 最小内存占用 |

---

*最后更新：2026-02-06*
