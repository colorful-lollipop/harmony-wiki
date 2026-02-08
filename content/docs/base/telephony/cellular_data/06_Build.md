# GN 构建配置

## 1. 构建目标清单

### 1.1 根目录 BUILD.gn

| Target | Type | Output | Purpose |
|--------|------|--------|---------|
| `tel_cellular_data` | ohos_shared_library | libtel_cellular_data.so | 主服务动态库 |
| `tel_cellular_data_static` | ohos_static_library | libtel_cellular_data_static.a | 静态库（测试用） |

### 1.2 frameworks/native/BUILD.gn

| Target | Type | Output | Purpose |
|--------|------|--------|---------|
| `cellular_data_util_config` | config | - | 公共配置 |
| `cellulardata_interface_config` | config | - | 接口配置 |
| `cellular_data_interface` | idl_gen_interface | 生成代码 | IDL 接口 |
| `cellulardata_interface_stub` | ohos_source_set | - | Stub 代码 |
| `tel_cellular_data_api` | ohos_shared_library | libtel_cellular_data_api.so | 对外 API |

### 1.3 frameworks/js/BUILD.gn

| Target | Type | Output | Install Path |
|--------|------|--------|--------------|
| `data` | ohos_shared_library | libdata.so | system/lib/module/telephony/ |

### 1.4 frameworks/cj/BUILD.gn

| Target | Type | Output |
|--------|------|--------|
| `cj_cellular_data_ffi` | ohos_shared_library | libcj_cellular_data_ffi.so |

### 1.5 frameworks/ets/ani/BUILD.gn

| Target | Type | Output |
|--------|------|--------|
| `cellular_data_ani_cxx_gen` | rust_cxx | 生成 C++ 绑定 |
| `cellular_data_ani_cxx` | ohos_static_library | libcellular_data_ani_cxx.a |
| `cellular_data_ani` | ohos_rust_shared_library | libcellular_data_ani.so |
| `cellular_data_abc` | generate_static_abc | telephony_data.abc |

### 1.6 interfaces/kits/c/BUILD.gn

| Target | Type | Output | Install Path |
|--------|------|--------|--------------|
| `telephony_data` | ohos_shared_library | libtelephony_data.so | system/lib/ndk/ |

### 1.7 sa_profile/BUILD.gn

| Target | Type | Sources |
|--------|------|---------|
| `cellular_data_sa_profile` | ohos_sa_profile | 4007.json / 4007_dynamic.json |

## 2. 产物清单

### 2.1 系统库

| 产物 | Target | 路径 |
|------|--------|------|
| libtel_cellular_data.so | tel_cellular_data | system/lib/ |
| libtel_cellular_data_api.so | tel_cellular_data_api | system/lib/ |
| libdata.so | frameworks/js:data | system/lib/module/telephony/ |
| libtelephony_data.so | telephony_data | system/lib/ndk/ |
| libcj_cellular_data_ffi.so | cj_cellular_data_ffi | system/lib/ |
| libcellular_data_ani.so | cellular_data_ani | system/lib/ |
| telephony_data.abc | cellular_data_abc | system/framework/ |

### 2.2 静态库

| 产物 | Target |
|------|--------|
| libtel_cellular_data_static.a | tel_cellular_data_static |
| libcellular_data_ani_cxx.a | cellular_data_ani_cxx |

### 2.3 可执行文件

| 产物 | Target | 路径 |
|------|--------|------|
| tel_cellular_data_ui_test | unit_test | system/bin/ |

## 3. 依赖关系

### 3.1 内部依赖

```
root BUILD.gn
├── frameworks/native:cellulardata_interface_stub

frameworks/native/BUILD.gn
├── :cellular_data_interface (IDL 生成)
└── frameworks/native:BUILD.gn

frameworks/js/BUILD.gn
└── frameworks/native:tel_cellular_data_api

frameworks/cj/BUILD.gn
└── frameworks/native:tel_cellular_data_api

frameworks/ets/ani/BUILD.gn
└── frameworks/native:tel_cellular_data_api

interfaces/kits/c/BUILD.gn
└── frameworks/native:tel_cellular_data_api
```

### 3.2 外部依赖

| 组件 | 用途 | 在以下 targets 中使用 |
|------|------|---------------------|
| `c_utils:utils` | C 工具库 | 所有 targets |
| `hilog:libhilog` | 日志 | 所有 targets |
| `ipc:ipc_single` | IPC 通信 | 所有 targets |
| `samgr:samgr_proxy` | Service Manager | 所有 targets |
| `safwk:system_ability_fwk` | SA Framework | 所有 targets |
| `core_service:libtel_common` | Telephony 公共库 | 所有 targets |
| `core_service:tel_core_service_api` | Telephony API | frameworks/native |
| `telephony_data:tel_telephony_data` | 数据类型 | 所有 targets |
| `init:libbegetutil` | 初始化工具 | 所有 targets |
| `init:libbeget_proxy` | 初始化代理 | root BUILD.gn |
| `eventhandler:libeventhandler` | 事件处理 | root BUILD.gn |
| `common_event_service:cesfwk_innerkits` | 公共事件 | root BUILD.gn |
| `hisysevent:libhisysevent` | HiSysEvent | root BUILD.gn |
| `netmanager_base:net_conn_manager_if` | 网络连接 | root BUILD.gn |
| `netmanager_base:net_policy_manager_if` | 网络策略 | root BUILD.gn |
| `netmanager_base:net_stats_manager_if` | 网络统计 | root BUILD.gn |
| `netmanager_ext:networkslice_manager_if` | 网络切片 | root BUILD.gn |
| `data_share:datashare_consumer` | DataShare | root BUILD.gn |
| `hicollie:libhicollie` | HiCollie (可选) | 条件依赖 |
| `power_manager:powermgr_client` | 电源管理 (可选) | 条件依赖 |

## 4. 特性开关

### 4.1 全局参数

```gn
declare_args() {
  telephony_cellular_data_hicollie_able = true
  cellular_data_feature_base_power_improvement = false
}
```

### 4.2 条件定义

| 特性 | 定义 | 条件 |
|------|------|------|
| `OHOS_BUILD_ENABLE_TELEPHONY_EXT` | 增强 telephony | `global_parts_info.telephony_telephony_enhanced` |
| `OHOS_BUILD_ENABLE_TELEPHONY_VSIM` | 虚拟 SIM | `global_parts_info.telephony_telephony_enhanced` |
| `OHOS_BUILD_ENABLE_DATA_SERVICE_EXT` | 数据服务扩展 | `global_parts_info.communication_netmanager_enhanced` |
| `HICOLLIE_ENABLE` | HiCollie 支持 | `telephony_cellular_data_hicollie_able` |
| `ABILITY_POWER_SUPPORT` | 电源管理 | `powermgr_power_manager` 启用 |
| `BASE_POWER_IMPROVEMENT` | 电源优化 | `cellular_data_feature_base_power_improvement` |
| `CONFIG_DUAL_FRAMEWORK` | 双框架 | `is_double_framework` |
| `BINDER_IPC_32BIT` | 32位 IPC | `target_cpu == "arm"` |
| `CONFIG_STANDARD_SYSTEM` | 标准系统 | `is_standard_system` |
| `BUILD_PUBLIC_VERSION` | 公共版本 | `build_public_version` |

## 5. 编译产物路径

```
out/
├── standard/
│   ├── system/lib/
│   │   ├── libtel_cellular_data.so
│   │   ├── libtel_cellular_data_api.so
│   │   ├── libcj_cellular_data_ffi.so
│   │   └── libcellular_data_ani.so
│   ├── system/lib/module/telephony/
│   │   └── libdata.so
│   ├── system/lib/ndk/
│   │   └── libtelephony_data.so
│   ├── system/framework/
│   │   └── telephony_data.abc
│   └── system/bin/
│       └── tel_cellular_data_ui_test
```

## 6. 编译命令

### 6.1 全量编译

```bash
./build.sh --product-name <product> --ccache
```

### 6.2 模块编译

```bash
# 编译 cellular_data 模块
./build.sh --product-name <product> --parts telephony --subsystems telephony:cellular_data

# 只编译 JS N-API
./build.sh --product-name <product> --parts telephony:cellular_data --target frameworks/js:data
```

### 6.3 单独编译

```bash
# 使用 gn + ninja
gn gen out/standard --args="target_os=\"ohos\" target_cpu=\"arm64\""
ninja -C out/standard telephony_cellular_data
ninja -C out/standard frameworks/js:data
```

## 7. 特性编译

```bash
# 启用 HiCollie
gn gen out/standard --args="global_parts_info={ hiviewdfx_hicollie = true }"
ninja -C out/standard telephony_cellular_data

# 启用电源优化
gn gen out/standard --args="cellular_data_feature_base_power_improvement=true"
ninja -C out/standard telephony_cellular_data

# 启用增强 telephony
gn gen out/standard --args="global_parts_info={ telephony_telephony_enhanced = true }"
ninja -C out/standard telephony_cellular_data
```

## 8. 安全编译选项

```gn
sanitize = {
  cfi = true           # Control Flow Integrity
  cfi_cross_dso = true # Cross-DSO CFI
  debug = false
}
branch_protector_ret = "pac_ret"  # PAC-RET 分支保护

cflags_cc = [
  "-O2",
  "-D_FORTIFY_SOURCE=2",  # FORTIFY 强化
]
```
