# 构建与产物

> 最后更新：2026-02-07
> 版本：v3.0.0

## 7.1 GN Targets 清单

### 主要构建 Targets

| Target 名称 | 类型 | 输出产物 | 路径 |
|-------------|------|----------|------|
| **dslm_sdk** | ohos_shared_library | libdslm_sdk.z.so | `interfaces/inner_api/BUILD.gn` |
| **dslm_service** | ohos_shared_library | libdslm_service.z.so | `services/sa/BUILD.gn` |
| **dslm_service.rc** | ohos_resource | dslm_service.rc | `profile/BUILD.gn` |
| **service_dslm_obj** | ohos_source_set | 静态库 | `services/dslm/BUILD.gn` |
| **msg_obj** | ohos_source_set | 静态库 | `services/msg/BUILD.gn` |
| **dslm_oem_ext** | ohos_source_set | 静态库 | `oem_property/BUILD.gn` |
| **dslm_oem_standard** | ohos_static_library | libdslm_oem_standard.a | `oem_property/ohos/standard/BUILD.gn` |
| **dslm_oem_lite** | ohos_static_library | libdslm_oem_lite.a | `oem_property/ohos/lite/BUILD.gn` |
| **messenger_static** | ohos_static_library | libmessenger.a | `baselib/msglib/BUILD.gn` |
| **utils_static** | ohos_static_library | libutils.a | `baselib/utils/BUILD.gn` |

**证据**：`bundle.json:51-89`

---

## 7.2 SDK Targets

### dslm_sdk (Standard)

```gn
# interfaces/inner_api/BUILD.gn:101-138
ohos_shared_library("dslm_sdk") {
    sources = [
        "src/standard/device_security_level_callback_helper.cpp",
        "src/standard/device_security_level_callback_stub.cpp",
        "src/standard/device_security_level_defines.cpp",
        "src/standard/device_security_level_loader.cpp",
        "src/standard/device_security_level_proxy.cpp",
    ]
    
    include_dirs = [
        "include",
        "src/standard",
    ]
    
    external_deps = [
        "c_utils:utils",
        "hilog:libhilog",
        "ipc:ipc_core",
        "safwk:samgr_proxy",
        "//base/security/device_security_level/services/common:dslm_msg_obj",
        "//base/security/device_security_level/services/common:dslm_dfx_obj",
        "//base/security/device_security_level/services/dslm:service_dslm_obj",
        "//base/security/device_security_level/baselib/utils:utils_static",
        "//base/security/device_security_level/baselib/msglib:messenger_static",
        "//base/security/device_security_level/common:dslm_common",
    ]
    
    cflags_cc = [ "-fvisibility=hidden" ]
    ldflags = [ "-znow" ]
}
```

**输出产物**：`libdslm_sdk.z.so`

### dslm_sdk (Small)

```gn
# interfaces/inner_api/BUILD.gn
ohos_shared_library("dslm_sdk_small") {
    # 轻量版实现，源文件和依赖不同
}
```

### dslm_sdk (Mini)

```gn
# interfaces/inner_api/BUILD.gn
ohos_static_library("dslm_sdk_mini") {
    # 微型版实现，无 IPC
}
```

---

## 7.3 Service Targets

### dslm_service (SA)

```gn
# services/sa/BUILD.gn:148-209
ohos_shared_library("dslm_service") {
    sources = [
        "standard/dslm_callback_proxy.cpp",
        "standard/dslm_ipc_process.cpp",
        "standard/dslm_service.cpp",
    ]
    
    include_dirs = [
        "//base/security/device_security_level/services/common",
        "//base/security/device_security_level/services/dslm",
        "//base/security/device_security_level/oem_property/include",
        "//base/security/device_security_level/common/include",
        "//base/security/device_security_level/interfaces/inner_api/src/standard",
    ]
    
    external_deps = [
        "ipc:ipc_core",
        "safwk:safwk_core",
        "samgr:samgr_proxy",
        "hilog:libhilog",
        "//base/security/device_security_level/services/common:dslm_msg_obj",
        "//base/security/device_security_level/services/common:dslm_dfx_obj",
        "//base/security/device_security_level/services/dslm:service_dslm_obj",
        "//base/security/device_security_level/baselib/utils:utils_static",
        "//base/security/device_security_level/baselib/msglib:messenger_static",
        "//base/security/device_security_level/common:dslm_common",
        "//base/security/device_security_level/oem_property:dslm_oem_ext",
    ]
    
    shlib_type = "sa"  # 标记为 System Ability
}
```

**输出产物**：`libdslm_service.z.so`

**证据**：`services/sa/BUILD.gn:152`

---

## 7.4 编译产物清单

### 标准版产物 (Standard)

| 产物 | 类型 | 路径 | 说明 |
|------|------|------|------|
| `libdslm_sdk.z.so` | 动态库 | `out/.../sdk/` | SDK 动态库 |
| `libdslm_service.z.so` | 动态库 | `out/.../sa/` | SA 服务动态库 |
| `libdslm_oem_standard.a` | 静态库 | `out/.../lib/` | OEM 适配静态库 |
| `libmessenger.a` | 静态库 | `out/.../lib/` | 消息库 |
| `libutils.a` | 静态库 | `out/.../lib/` | 工具库 |

### 轻量版产物 (Small/Mini)

| 产物 | 类型 | 说明 |
|------|------|------|
| `libdslm_sdk_small.z.so` | 动态库 | Small SDK |
| `libdslm_sdk_mini.a` | 静态库 | Mini SDK (静态链接) |
| `libdslm_oem_lite.a` | 静态库 | Lite OEM 适配 |

---

## 7.5 依赖关系

### SDK 依赖图

```
dslm_sdk
├── c_utils:utils
├── hilog:libhilog
├── ipc:ipc_core
├── safwk:samgr_proxy
├── services/common:dslm_msg_obj
├── services/common:dslm_dfx_obj
├── services/dslm:service_dslm_obj
├── baselib/utils:utils_static
├── baselib/msglib:messenger_static
└── common:dslm_common
```

### Service 依赖图

```
dslm_service
├── ipc:ipc_core
├── safwk:safwk_core
├── samgr:samgr_proxy
├── hilog:libhilog
├── services/common:dslm_msg_obj
├── services/common:dslm_dfx_obj
├── services/dslm:service_dslm_obj
├── baselib/utils:utils_static
├── baselib/msglib:messenger_static
├── common:dslm_common
└── oem_property:dslm_oem_ext
```

---

## 7.6 Feature 开关

### 组件 Feature

| Feature 名称 | 定义位置 | 默认值 | 说明 |
|--------------|----------|--------|------|
| `device_security_level_feature_cred_level` | `bundle.json:19` | 启用 | 信任等级特性 |
| `device_security_level_feature_plugin_path` | `bundle.json:20` | 启用 | 插件路径特性 |
| `device_security_level_feature_secondary_session_name` | `bundle.json:21` | 启用 | 次级会话名称特性 |

**证据**：`bundle.json:18-22`

```json
"features": [
    "device_security_level_feature_cred_level",
    "device_security_level_feature_plugin_path",
    "device_security_level_feature_secondary_session_name"
]
```

### 平台配置

| 配置项 | Standard | Small | Mini |
|--------|----------|-------|------|
| **C++ 支持** | ✅ | ❌ | ❌ |
| **IPC (Binder)** | ✅ | ✅ | ❌ |
| **动态库** | ✅ | ✅ | ❌ |
| **静态库** | ❌ | ❌ | ✅ |

---

## 7.7 编译配置

### 编译器标志

```gn
# Standard 版本编译器标志
cflags_cc = [
    "-fvisibility=hidden",      # 隐藏符号
    "-fstack-protector-strong", # 栈保护
]

# 安全加固
if (enable branch_protector) {
    cflags_cc += [ "-fvisibility=hidden", "-fsanitize=cfi" ]
}

# 链接器标志
ldflags = [
    "-znow",                    # 禁用符号重定位
    "-zrelro",                 # 只读重定位
    "-pie",                    # 位置无关可执行文件
]
```

### Sanitizer 配置

| Sanitizer | 启用条件 | 说明 |
|-----------|----------|------|
| **CFI** | `enable_branch_protector` | 控制流完整性 |
| **UBSan** | `enable_ubsan` | 未定义行为检测 |
| **Integer Overflow** | `enable_overflow_sanitize` | 整数溢出检测 |
| **PacRet** | `enable_pac_ret` | 指针认证 |

**证据**：`services/sa/standard/BUILD.gn`

---

## 7.8 运行时加载

### SA 服务注册

```cpp
// services/sa/standard/dslm_service.cpp:38
REGISTER_SYSTEM_ABILITY_BY_ID(
    DslmService,                    // 服务类名
    DEVICE_SECURITY_LEVEL_MANAGER_SA_ID,  // SA ID: 3511
    true);                           // runOnCreate
```

**SA ID**: 3511

**证据**：`common/include/idevice_security_level.h:30`

### 配置文件

| 配置文件 | 路径 | 说明 |
|----------|------|------|
| `dslm_service.cfg` | `profile/` | 服务权限配置 |
| `dslm_service.rc` | `profile/` | 资源文件 |
| `dslm_service.xml` | `profile/` | SA 配置 |
| `dslm_finger*.cfg` | `oem_property/ohos/standard/` | 凭证配置文件 |

**dslm_service.cfg 示例**：

```json
{
    "uid": 3046,
    "gid": 3046,
    "apl": "system_basic",
    "selinux": {
        "context": "u:r:dslm_service:s0"
    },
    "permissions": [
        "ohos.permission.ACCESS_IDS",
        "ohos.permission.sec.ACCESS_UDID",
        "ohos.permission.ACCESS_SERVICE_DM",
        "ohos.permission.DISTRIBUTED_DATASYNC"
    ]
}
```

---

## 7.9 编译命令

### 全量编译

```bash
# 编译整个 device_security_level 模块
hb build -p //base/security/device_security_level
```

### 单独编译 SDK

```bash
# 编译 dslm_sdk
hb build -p //base/security/device_security_level/interfaces/inner_api:dslm_sdk
```

### 单独编译服务

```bash
# 编译 dslm_service
hb build -p //base/security/device_security_level/services/sa:dslm_service
```

### 清理编译

```bash
# 清理编译产物
hb clean -p //base/security/device_security_level
```

---

## 7.10 产物安装路径

### Standard 系统

| 产物 | 安装路径 |
|------|----------|
| SDK 动态库 | `/system/lib64/libdslm_sdk.z.so` |
| SA 动态库 | `/system/lib64/libdslm_service.z.so` |

### Small 系统

| 产物 | 安装路径 |
|------|----------|
| SDK 动态库 | `/system/lib/libdslm_sdk_small.z.so` |

### Mini 系统

| 产物 | 安装路径 |
|------|----------|
| SDK 静态库 | 静态链接到应用 |

---

## 下一章

- [08_Internals.md](./08_Internals.md) - 内部实现细节
