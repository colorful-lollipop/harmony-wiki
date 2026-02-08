# 06 - GN 构建配置与编译产物

## 目的

本文档描述 Cellular Call 模块的 **GN 构建配置、Targets 清单、条件编译选项和编译产物**，帮助开发者理解构建系统和产物分布。

## 适用范围

- **读者对象**：构建系统开发者、集成工程师
- **前置知识**：了解 GN 构建系统、OpenHarmony 构建流程
- **使用场景**：
  - 理解构建配置
  - 添加新源文件
  - 配置编译选项
  - 分析构建产物

---

## 6.1 构建入口

### 6.1.1 根构建文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 模块主构建入口 |
| `cellularcall.gni` | 全局配置参数 |
| `bundle.json` | 组件配置描述 |

### 6.1.2 构建文件位置

```
cellular_call/
├── BUILD.gn                                    ✅ 主构建入口
├── cellularcall.gni                           ✅ 全局配置
├── bundle.json                                 ✅ 组件描述
│
├── interfaces/innerkits/ims/BUILD.gn          ✅ IMS 接口构建
├── interfaces/innerkits/satellite/BUILD.gn     ✅ 卫星接口构建
│
├── sa_profile/BUILD.gn                         ✅ SA 配置构建
│
└── test/BUILD.gn                              ⚠️ 测试构建（不分析）
```

---

## 6.2 全局配置 (cellularcall.gni)

### 6.2.1 配置参数

**文件位置**: `cellularcall.gni`

```gn
declare_args() {
  cellular_call_dynamic_start = false      # SA 动态启动开关
  cellular_call_tel_power_mode = false    # 省电模式开关
  cellular_call_support_UT = true         # UT 测试支持
  cellular_call_satellite = false          # 卫星通话开关
  cellular_call_support_rtt = false        # RTT 实时文本开关
}

global_defines = []

if (cellular_call_tel_power_mode) {
  global_defines += [ "BASE_POWER_IMPROVEMENT_FEATURE" ]
}

if (cellular_call_satellite) {
  global_defines += [ "CELLULAR_CALL_SATELLITE" ]
}

if (cellular_call_support_rtt) {
  global_defines += [ "SUPPORT_RTT_CALL" ]
}
```

### 6.2.2 Feature 开关列表

| 变量名 | 默认值 | 宏定义 | 功能 | 证据位置 |
|--------|--------|--------|------|----------|
| `cellular_call_dynamic_start` | false | - | SA 动态启动 | `cellularcall.gni:15` |
| `cellular_call_tel_power_mode` | false | `BASE_POWER_IMPROVEMENT_FEATURE` | 省电模式 | `cellularcall.gni:16`, `cellularcall.gni:25` |
| `cellular_call_support_UT` | true | - | 单元测试 | `cellularcall.gni:17` |
| `cellular_call_satellite` | false | `CELLULAR_CALL_SATELLITE` | 卫星通话 | `cellularcall.gni:18`, `cellularcall.gni:29` |
| `cellular_call_support_rtt` | false | `SUPPORT_RTT_CALL` | RTT 实时文本 | `cellularcall.gni:19`, `cellularcall.gni:33` |

---

## 6.3 主构建 Target (BUILD.gn)

### 6.3.1 Target 定义

**文件位置**: `BUILD.gn`

```gn
ohos_shared_library("tel_cellular_call") {
  # 目标名称: tel_cellular_call
  # 类型: ohos_shared_library (动态库)
  
  sanitize = {
    cfi = true              # CFI 堆栈保护
    cfi_cross_dso = true    # 跨 DSO CFI
    debug = false
  }
  branch_protector_ret = "pac_ret"  # PACRET 返回地址保护
}
```

### 6.3.2 源文件列表

**基础源文件** (50+ 文件):

| 分类 | 源文件 | 路径 |
|------|--------|------|
| **Common** | `base_request.cpp` | `services/common/src/` |
| | `cellular_call_hisysevent.cpp` | `services/common/src/` |
| | `cellular_call_rdb_helper.cpp` | `services/common/src/` |
| | `mmi_code_message.cpp` | `services/common/src/` |
| | `supplement_request_cs.cpp` | `services/common/src/` |
| | `supplement_request_ims.cpp` | `services/common/src/` |
| **Connection** | `base_connection.cpp` | `services/connection/src/` |
| | `cellular_call_connection_cs.cpp` | `services/connection/src/` |
| | `cellular_call_connection_ims.cpp` | `services/connection/src/` |
| **Control** | `control_base.cpp` | `services/control/src/` |
| | `cs_control.cpp` | `services/control/src/` |
| | `ims_control.cpp` | `services/control/src/` |
| | `ims_video_call_control.cpp` | `services/control/src/` |
| **IMS Interaction** | `ims_call_callback_stub.cpp` | `services/ims_service_interaction/src/` |
| | `ims_call_client.cpp` | `services/ims_service_interaction/src/` |
| | `ims_call_proxy.cpp` | `services/ims_service_interaction/src/` |
| **Manager** | `cellular_call_callback.cpp` | `services/manager/src/` |
| | `cellular_call_handler.cpp` | `services/manager/src/` |
| | `cellular_call_register.cpp` | `services/manager/src/` |
| | `cellular_call_service.cpp` | `services/manager/src/` |
| | `cellular_call_stub.cpp` | `services/manager/src/` |
| **Utils** | `cellular_call_config.cpp` | `services/utils/src/` |
| | `cellular_call_dump_helper.cpp` | `services/utils/src/` |
| | `cellular_call_supplement.cpp` | `services/utils/src/` |
| | `config_request.cpp` | `services/utils/src/` |
| | `emergency_utils.cpp` | `services/utils/src/` |
| | `mmi_code_utils.cpp` | `services/utils/src/` |
| | `module_service_utils.cpp` | `services/utils/src/` |
| | `standardize_utils.cpp` | `services/utils/src/` |
| **Other** | `telephony_ext_wrapper.cpp` | `services/telephony_ext_wrapper/src/` |

**条件编译源文件** (当 `cellular_call_satellite=true` 时):

| 源文件 | 路径 |
|--------|------|
| `cellular_call_connection_satellite.cpp` | `services/connection/src/` |
| `satellite_control.cpp` | `services/control/src/` |
| `satellite_call_callback_stub.cpp` | `services/satellite_service_interaction/src/` |
| `satellite_call_client.cpp` | `services/satellite_service_interaction/src/` |
| `satellite_call_proxy.cpp` | `services/satellite_service_interaction/src/` |

### 6.3.3 包含目录 (include_dirs)

```gn
include_dirs = [
  "interfaces/innerkits/ims",
  "interfaces/innerkits/ims_common",
  "interfaces/innerkits/satellite",
  "services/common/include",
  "services/manager/include",
  "services/control/include",
  "services/connection/include",
  "services/utils/include",
  "services/telephony_ext_wrapper/include",
]
```

### 6.3.4 宏定义 (defines)

```gn
defines = [
  "TELEPHONY_LOG_TAG = \"CellularCall\"",
  "LOG_DOMAIN = 0xD001F11",
  "LOG_TAG = \"CellularCall\"",
]
defines += global_defines
```

### 6.3.5 外部依赖 (external_deps)

```gn
external_deps = [
  "ability_base:want",
  "ability_base:zuri",
  "ability_runtime:dataobs_manager",
  "cJSON:cjson",
  "c_utils:utils",
  "call_manager:tel_call_manager_api",
  "common_event_service:cesfwk_innerkits",
  "core_service:libtel_common",
  "core_service:tel_core_service_api",
  "data_share:datashare_common",
  "data_share:datashare_consumer",
  "eventhandler:libeventhandler",
  "ffrt:libffrt",
  "graphic_surface:surface",
  "hilog:libhilog",
  "hisysevent:libhisysevent",
  "hitrace:hitrace_meter",
  "init:libbegetutil",
  "ipc:ipc_single",
  "json:nlohmann_json_static",
  "libphonenumber:phonenumber_standard",
  "resource_management:global_resmgr",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
  "telephony_data:tel_telephony_data",
]
```

### 6.3.6 编译器选项

```gn
cflags_cc = [
  "-fstack-protector-all",   # 堆栈保护
  "-O2",                     # 优化等级
  "-D_FORTIFY_SOURCE=2",    # FORTIFY_SOURCE 检查
]
```

---

## 6.4 子 Targets

### 6.4.1 SA Profile Target

**文件**: `sa_profile/BUILD.gn`

```gn
ohos_sa_profile("cellular_call_sa_profile") {
  if (!cellular_call_dynamic_start) {
    sources = [ "4006.json" ]          # 静态配置
  } else {
    sources = [ "4006_dynamic.json" ]  # 动态配置
  }
  part_name = "cellular_call"
}
```

### 6.4.2 IMS Interface Target

**文件**: `interfaces/innerkits/ims/BUILD.gn`

```gn
# 生成: libtel_ims_call_api.z.so 或静态库
# 包含 IMS Call 相关接口
```

### 6.4.3 Satellite Interface Target

**文件**: `interfaces/innerkits/satellite/BUILD.gn`

```gn
# 条件编译: 当 cellular_call_satellite=true 时
# 生成: libtel_satellite_call_api.z.so 或静态库
```

---

## 6.5 编译产物

### 6.5.1 主产物清单

| 产物名 | 类型 | 路径 | 说明 |
|--------|------|------|------|
| `libtel_cellular_call.z.so` | 动态库 | `out/.../libs/` | 主实现库 |
| `libtel_ims_call_api.z.so` | 动态库 | `out/.../libs/` | IMS 接口库 |
| `libtel_satellite_call_api.z.so` | 动态库 | `out/.../libs/` | 卫星接口库（条件） |
| `4006.json` | 配置文件 | `out/.../` | SA 静态配置 |
| `4006_dynamic.json` | 配置文件 | `out/.../` | SA 动态配置 |

### 6.5.2 安装路径

```
/system/lib/ld
  └── libtel_cellular_call.z.so

/system/lib/module
  ├── libtel_ims_call_api.z.so
  └── libtel_satellite_call_api.z.so (条件)

/system/etc/sa/
  ├── 4006.json (或 4006_dynamic.json)
  └── ...
```

### 6.5.3 SA 产物配置

**静态配置** (`sa_profile/4006.json`):

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

---

## 6.6 条件编译汇总

### 6.6.1 编译开关矩阵

| Feature | 条件变量 | 影响的源文件 | 影响的代码 |
|---------|----------|--------------|-----------|
| **卫星通话** | `cellular_call_satellite=true` | 5 个 .cpp 文件 | `CELLULAR_CALL_SATELLITE` 宏 |
| **RTT** | `cellular_call_support_rtt=true` | 头文件条件编译 | `SUPPORT_RTT_CALL` 宏 |
| **省电** | `cellular_call_tel_power_mode=true` | 条件代码 | `BASE_POWER_IMPROVEMENT_FEATURE` 宏 |
| **安全守护** | `security_guard` 组件存在 | 可选依赖 | `SECURITY_GUARDE_ENABLE` 宏 |
| **电话扩展** | `telephony_enhanced` 组件存在 | 可选代码 | `OHOS_BUILD_ENABLE_TELEPHONY_EXT` 宏 |
| **设备优化** | `device_name=="rk3568"` | 条件代码 | `CALL_MANAGER_AUTO_START_OPTIMIZE` 宏 |

### 6.6.2 产物映射表

| Target | 类型 | 输出 | 依赖 | 条件 |
|--------|------|------|------|------|
| `tel_cellular_call` | shared_library | `.z.so` | 20+ external_deps | 无 |
| `cellular_call_sa_profile` | sa_profile | `.json` | 无 | 无 |
| `tel_ims_call_api` | - | 接口库 | 无 | 无 |
| `tel_satellite_call_api` | - | 接口库 | 无 | `cellular_call_satellite` |

---

## 6.7 组件配置 (bundle.json)

### 6.7.1 组件信息

```json
{
    "name": "@ohos/cellular_call",
    "version": "4.0",
    "component": {
        "name": "cellular_call",
        "subsystem": "telephony",
        "syscap": ["SystemCapability.Telephony.CellularCall"],
        "features": [
            "cellular_call_dynamic_start",
            "cellular_call_tel_power_mode",
            "cellular_call_support_UT",
            "cellular_call_satellite",
            "cellular_call_support_rtt"
        ],
        "adapted_system_type": ["standard"],
        "rom": "1MB",
        "ram": "650KB"
    }
}
```

### 6.7.2 构建组配置

```json
{
    "build": {
        "group_type": {
            "service_group": [
                "//base/telephony/cellular_call:tel_cellular_call",
                "//base/telephony/cellular_call/sa_profile:cellular_call_sa_profile"
            ]
        },
        "inner_kits": [
            {
                "header": {
                    "header_base": [
                        "//base/telephony/cellular_call/interfaces/innerkits/ims",
                        "//base/telephony/cellular_call/interfaces/innerkits/ims_common"
                    ]
                },
                "name": "//base/telephony/cellular_call/interfaces/innerkits/ims:tel_ims_call_api"
            },
            {
                "header": {
                    "header_base": "//base/telephony/cellular_call/interfaces/innerkits/satellite"
                },
                "name": "//base/telephony/cellular_call/interfaces/innerkits/satellite:tel_satellite_call_api"
            }
        ]
    }
}
```

---

## 相关跳转

| 目标 | 链接 |
|------|------|
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| 接口规范 | [04_Interfaces.md](./04_Interfaces.md) |
| 安全评审 | [07_Security_Review.md](./07_Security_Review.md) |
| 常见问题 | [08_Troubleshooting.md](./08_Troubleshooting.md) |

---

*最后更新：2026-02-06*
