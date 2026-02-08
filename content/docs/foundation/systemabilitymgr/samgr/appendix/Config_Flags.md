# 配置选项

## 构建配置 (config.gni)

### samgr_enable_delay_dbinder

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | 延迟加载 DBinder 服务 |

**影响代码**:
```cpp
// system_ability_manager.h:277-279
#ifndef SAMGR_ENABLE_DELAY_DBINDER
    dBinderService_ = DBinderService::GetInstance();
#endif
```

**建议**:
- 标准系统保持 `true`，节省启动时间
- 如果需要早期分布式支持，设置为 `false`

---

### samgr_enable_extend_load_timeout

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | 扩展 SA 加载超时时间 |

**影响代码**:
```cpp
// 影响 LoadSystemAbility 超时
// 默认: 4秒
// 启用后: 12秒 (手表设备)
```

**建议**:
- 手表设备设置为 `true`
- 其他设备保持 `false`

---

### samgr_mksh_enable

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | 启用 mksh 依赖 |

**影响**:
```gn
if (samgr_mksh_enable) {
    external_deps += [ "mksh:sh" ]
}
```

---

## 服务变量 (var.gni)

### hicollie_able

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | 启用 hicollie 看门狗 |

**影响代码**:
```cpp
// services/samgr/native/BUILD.gn:121-124
if (hicollie_able) {
    external_deps += [ "hicollie:libhicollie" ]
    defines += [ "HICOLLIE_ENABLE" ]
}
```

```cpp
// 代码中使用
#ifdef HICOLLIE_ENABLE
    XCollie::GetInstance()....
#endif
```

**建议**:
- 生产环境保持 `true`，用于检测 ANR
- 测试环境可设置为 `false` 减少依赖

---

### preferences_enable

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | 启用 preferences 数据存储 |

**影响代码**:
```cpp
// services/samgr/native/BUILD.gn:147-151
if (preferences_enable) {
    sources += [ "device_timed_collect_tool.cpp" ]
    external_deps += [ "preferences:native_preferences" ]
    defines += [ "PREFERENCES_ENABLE" ]
}
```

**功能**:
- 支持 TimedEvent 持久化
- 设备状态收集的数据存储

---

### support_device_manager

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | 设备管理器支持 |

**影响代码**:
```cpp
// services/samgr/native/BUILD.gn:126-130
if (support_device_manager) {
    sources += [ "device_networking_collect.cpp" ]
    external_deps += [ "device_manager:devicemanagersdk" ]
    defines += [ "SUPPORT_DEVICE_MANAGER" ]
}
```

**功能**:
- 支持设备网络状态收集
- 分布式设备发现

**建议**:
- 分布式系统设置为 `true`
- 单体设备设置为 `false`

---

### support_common_event

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | 通用事件服务支持 |

**影响代码**:
```cpp
// services/samgr/native/BUILD.gn:132-145
if (support_common_event) {
    sources += [
        "common_event_collect.cpp",
        "device_switch_collect.cpp",
    ]
    defines += [
        "SUPPORT_COMMON_EVENT",
        "SUPPORT_SWITCH_COLLECT",
    ]
}
```

**功能**:
- 支持通用事件触发 OnDemand 加载
- 设备开关事件监听

---

### support_softbus

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | true |
| **说明** | SoftBus 通信支持 |

**影响**:
- 分布式服务发现
- 跨设备 RPC

**建议**:
- 分布式系统保持 `true`
- 单体设备可设置为 `false` 减少依赖

---

### support_penglai_mode

| 属性 | 值 |
|------|-----|
| **类型** | boolean |
| **默认值** | false |
| **说明** | Penglai TEE 模式支持 |

**影响代码**:
```cpp
// services/samgr/native/BUILD.gn:161-163
if (support_penglai_mode) {
    defines += [ "SUPPORT_PENGLAI_MODE" ]
}
```

```cpp
// 代码中使用
bool SamgrUtil::CheckPengLaiPermission(int32_t saId) {
#ifdef SUPPORT_PENGLAI_MODE
    // TEE 权限检查
#endif
}
```

**建议**:
- 仅在支持 Penglai TEE 的系统启用

---

## 系统参数

### samgr.max_services

| 属性 | 值 |
|------|-----|
| **类型** | int |
| **默认值** | 1000 |
| **说明** | 最大服务数量限制 |

**代码定义**:
```cpp
// system_ability_manager.h
static constexpr size_t MAX_SERVICES = 1000;
```

**影响**:
- abilityMap_ 大小限制
- onDemandAbilityMap_ 大小限制

---

### samgr.load_timeout

| 属性 | 值 |
|------|-----|
| **类型** | int (ms) |
| **默认值** | 4000 |
| **说明** | SA 加载默认超时 |

**代码使用**:
```cpp
// 手表设备使用 12000ms
#ifdef SAMGR_ENABLE_EXTEND_LOAD_TIMEOUT
    static constexpr int32_t LOAD_TIMEOUT = 12000;
#else
    static constexpr int32_t LOAD_TIMEOUT = 4000;
#endif
```

---

## 编译选项

### 安全相关

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `cfi` | 控制流完整性 | true |
| `cfi_cross_dso` | 跨 DSO CFI | true |
| `branch_protector_ret` | 返回地址保护 | pac_ret |
| `integer_overflow` | 整数溢出检测 | true (common) |
| `ubsan` | 未定义行为检测 | true (common) |

### 性能相关

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `SAMGR_USE_FFRT` | 使用 FFRT 调度 | true |
| `SAMGR_ENABLE_DELAY_DBINDER` | 延迟 DBinder 初始化 | true |

---

## 配置优先级

```
1. 运行时参数 (samgr.para) - 最高优先级
2. 编译时定义 (BUILD.gn defines)
3. 变量配置 (var.gni) - 较低优先级
4. 根配置 (config.gni) - 最低优先级
```

## 配置修改方法

### 方法 1: 修改 .gni 文件

```gn
# config.gni
samgr_enable_delay_dbinder = false
```

### 方法 2: 构建时覆盖

```bash
hb build --gn-args="samgr_enable_delay_dbinder=false"
```

### 方法 3: 运行时修改 (部分参数)

```bash
# 修改系统参数
param set samgr.max_services 2000
```

## 配置验证

```cpp
// 代码中检查配置
#ifdef HICOLLIE_ENABLE
    HILOGI("Hicollie enabled");
#else
    HILOGI("Hicollie disabled");
#endif
```

```bash
# 构建时检查
grep -r "define.*HICOLLIE_ENABLE" out/
```
