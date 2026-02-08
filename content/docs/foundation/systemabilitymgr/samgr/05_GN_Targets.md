# GN 构建目标

## 配置文件总览

| 文件 | 用途 |
|------|------|
| `config.gni` | 根配置，定义特性开关 |
| `services/samgr/var.gni` | 服务模块变量配置 |
| `bundle.json` | 组件描述和构建分组 |

## 特性开关

### config.gni

```gn
# 延迟加载 DBinder 服务
samgr_enable_delay_dbinder = true

# 扩展加载超时
samgr_enable_extend_load_timeout = false

# 启用 mksh 依赖
samgr_mksh_enable = true
```

### var.gni

```gn
# 启用 hicollie 看门狗
hicollie_able = true

# 启用 preferences 数据存储
preferences_enable = true

# 启用资源调度
ressched_able = true

# 设备管理器支持
support_device_manager = false

# 通用事件服务支持
support_common_event = false

# SoftBus 支持
support_softbus = true

# 访问令牌支持
samgr_support_access_token = true

# Penglai TEE 模式
support_penglai_mode = false
```

## 构建目标详解

### 1. samgr (可执行文件)

**定义**: `services/samgr/native/BUILD.gn:38`

```gn
ohos_executable("samgr") {
    sanitize = {
        cfi = true                    # 控制流完整性
        cfi_cross_dso = true          # 跨 DSO CFI
        cfi_no_nvcall = true          # 无虚调用 CFI
        blocklist = "../../../cfi_blocklist.txt"
    }
    branch_protector_ret = "pac_ret"  # 指针认证
    
    sources = [  # 26 个源文件
        "//foundation/systemabilitymgr/samgr/services/samgr/native/source/main.cpp",
        "//foundation/systemabilitymgr/samgr/services/samgr/native/source/system_ability_manager.cpp",
        ...
    ]
    
    deps = [
        "//foundation/systemabilitymgr/samgr/interfaces/innerkits/common:samgr_common",
        "//foundation/systemabilitymgr/samgr/interfaces/innerkits/dynamic_cache:dynamic_cache",
        "//foundation/systemabilitymgr/samgr/interfaces/innerkits/rust:samgr_rust",
        "//foundation/systemabilitymgr/samgr/interfaces/innerkits/samgr_proxy:samgr_proxy",
    ]
}
```

**输出**: `system/bin/samgr`

**条件编译源文件**:

| 源文件 | 条件 |
|--------|------|
| `device_networking_collect.cpp` | `support_device_manager` |
| `common_event_collect.cpp` | `support_common_event` |
| `device_switch_collect.cpp` | `support_common_event` |
| `device_timed_collect_tool.cpp` | `preferences_enable` |

**宏定义**:

| 宏 | 条件 | 说明 |
|----|------|------|
| `SAMGR_USE_FFRT` | 始终 | 使用 FFRT |
| `WITH_SELINUX` | `build_selinux` | SELinux 支持 |
| `HICOLLIE_ENABLE` | `hicollie_able` | 看门狗 |
| `SUPPORT_DEVICE_MANAGER` | `support_device_manager` | 设备管理 |
| `SUPPORT_COMMON_EVENT` | `support_common_event` | 通用事件 |
| `PREFERENCES_ENABLE` | `preferences_enable` | 偏好存储 |
| `SAMGR_ENABLE_EXTEND_LOAD_TIMEOUT` | `samgr_enable_extend_load_timeout` | 扩展超时 |
| `SAMGR_ENABLE_DELAY_DBINDER` | `samgr_enable_delay_dbinder` | 延迟 DBinder |
| `SUPPORT_PENGLAI_MODE` | `support_penglai_mode` | Penglai 模式 |

### 2. samgr_proxy (共享库)

**定义**: `interfaces/innerkits/samgr_proxy/BUILD.gn:39`

```gn
ohos_shared_library("samgr_proxy") {
    version_script = "libsamgr_proxy.versionscript"
    defines = [ "SAMGR_PROXY" ]
    
    sources = [  # 9 个源文件
        "//foundation/systemabilitymgr/samgr/frameworks/native/source/system_ability_manager_proxy.cpp",
        "//foundation/systemabilitymgr/samgr/services/samgr/native/source/service_registry.cpp",
        ...
    ]
    
    deps = [
        "//foundation/systemabilitymgr/samgr/interfaces/innerkits/dynamic_cache:dynamic_cache",
    ]
    
    innerapi_tags = [
        "chipsetsdk_sp",
        "platformsdk",
        "sasdk",
    ]
    
    install_images = [
        system_base_dir,
        updater_base_dir,
    ]
}
```

**输出**: `system/lib/libsamgr_proxy.so`

**符号版本控制**: `libsamgr_proxy.versionscript`

### 3. samgr_common (共享库)

**定义**: `interfaces/innerkits/common/BUILD.gn`

```gn
ohos_shared_library("samgr_common") {
    version_script = "libsamgr_common.versionscript"
    
    sources = [
        "//foundation/systemabilitymgr/samgr/services/common/src/parse_util.cpp",
        "//foundation/systemabilitymgr/samgr/services/dfx/source/hisysevent_adapter.cpp",
        "//foundation/systemabilitymgr/samgr/services/dfx/source/samgr_xcollie.cpp",
    ]
    
    external_deps = [
        "libxml2:libxml2",
        "c_utils:utils",
        "hilog:libhilog",
        "hisysevent:libhisysevent",
        "hicollie:libhicollie",
        ...
    ]
    
    sanitize = {
        integer_overflow = true   # 整数溢出检测
        ubsan = true              # 未定义行为检测
    }
    
    innerapi_tags = [ "platformsdk" ]
}
```

**输出**: `system/lib/libsamgr_common.so`

### 4. dynamic_cache (静态库)

**定义**: `interfaces/innerkits/dynamic_cache/BUILD.gn`

```gn
ohos_static_library("dynamic_cache") {
    sources = [ "./src/dynamic_cache.cpp" ]
    
    external_deps = [
        "c_utils:utils",
        "hilog:libhilog",
        "ipc:ipc_core",
    ]
}
```

**输出**: `libdynamic_cache.a` (链接到其他目标)

### 5. samgr_rust (Rust 库)

**定义**: `interfaces/innerkits/rust/BUILD.gn`

```gn
# C++ 包装静态库
ohos_static_library("samgr_rust_cpp") {
    sources = [
        "src/cxx/status_change_wrapper.cpp",
        "src/cxx/system_ability_manager_wrapper.cpp",
    ]
}

# Rust 共享库
ohos_rust_shared_library("samgr_rust") {
    sources = [ "src/lib.rs" ]
    
    rustflags = [
        "-Zstack-protector=all",  # 栈保护
    ]
}
```

**输出**:
- `libsamgr_rust_cpp.a`
- `system/lib/libsamgr.so` (Rust 共享库)

### 6. samgr_etc (配置文件)

**定义**: `etc/BUILD.gn`

```gn
ohos_prebuilt_etc("samgr.para") {
    source = "samgr.para"
    install_images = [ system_base_dir ]
    part_name = "samgr"
    install_enable = true
    install_dir = "etc/param/"
}

ohos_prebuilt_etc("samgr_init") {
    source = "samgr_standard.cfg"
    install_images = [ system_base_dir ]
    part_name = "samgr"
    install_enable = true
    install_dir = "etc/init/"
}
```

**输出**:
- `system/etc/param/samgr.para`
- `system/etc/param/samgr.para.dac`
- `system/etc/init/samgr_standard.cfg`

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│                        samgr (executable)                   │
├─────────────────────────────────────────────────────────────┤
│  deps:                                                      │
│    ├── samgr_common (shared)                                │
│    ├── samgr_proxy (shared)                                 │
│    ├── dynamic_cache (static)                               │
│    └── samgr_rust (shared)                                  │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ samgr_common  │    │ samgr_proxy   │    │ samgr_rust    │
│   (shared)    │    │   (shared)    │    │   (shared)    │
├───────────────┤    ├───────────────┤    ├───────────────┤
│ external:     │    │ external:     │    │ deps:         │
│  - libxml2    │    │  - ipc_single │    │  - samgr_     │
│  - hisysevent │    │  - c_utils    │    │    rust_cpp   │
│  - hicollie   │    │  - hilog      │    │  - samgr_     │
└───────────────┘    └───────────────┘    │    proxy      │
                                          └───────────────┘
```

## 外部依赖

### samgr 可执行文件依赖

| 组件 | 目标 | 用途 |
|------|------|------|
| access_token | libaccesstoken_sdk | 访问令牌验证 |
| c_utils | utils | 通用工具 |
| config_policy | configpolicy_util | 配置策略 |
| ffrt | libffrt | 快速函数运行时 |
| hilog | libhilog | 日志系统 |
| hisysevent | libhisysevent | 系统事件 |
| hitrace | hitrace_meter | 性能跟踪 |
| init | libbeget_proxy, libbegetutil | 启动服务 |
| ipc | ipc_core, libdbinder | IPC 通信 |
| json | nlohmann_json_static | JSON 解析 |
| safwk | system_ability_ondemand_reason | SA 框架 |
| qos_manager | qos, concurrent_task_client | QoS 管理 |
| selinux_adapter | libservice_checker | SELinux (可选) |
| hicollie | libhicollie | 看门狗 (可选) |
| device_manager | devicemanagersdk | 设备管理 (可选) |
| common_event_service | cesfwk_innerkits | 通用事件 (可选) |
| preferences | native_preferences | 偏好存储 (可选) |
| toybox | toybox | 工具箱 (musl) |
| mksh | sh | Shell (musl, 可选) |

### samgr_proxy 依赖

| 组件 | 目标 | 用途 |
|------|------|------|
| c_utils | utils | 通用工具 |
| hilog | libhilog | 日志 |
| init | libbegetutil | 启动工具 |
| ipc | ipc_single | IPC 单例 |
| json | nlohmann_json_static | JSON 解析 |

## 构建分组 (bundle.json)

### fwk_group (框架组)

```json
[
    "//foundation/systemabilitymgr/samgr/interfaces/innerkits/common:samgr_common",
    "//foundation/systemabilitymgr/samgr/interfaces/innerkits/dynamic_cache:dynamic_cache",
    "//foundation/systemabilitymgr/samgr/interfaces/innerkits/samgr_proxy:samgr_proxy"
]
```

### service_group (服务组)

```json
[
    "//foundation/systemabilitymgr/samgr/etc:samgr_etc",
    "//foundation/systemabilitymgr/samgr/services/samgr/native:samgr"
]
```

## 安全特性

| 特性 | 应用目标 | 说明 |
|------|----------|------|
| CFI | samgr, samgr_proxy, samgr_common, dynamic_cache, samgr_rust_cpp | 控制流完整性 |
| PAC-RET | 所有 native 目标 | 指针认证 |
| CFI Cross-DSO | 所有目标 | 跨 DSO CFI |
| Integer Overflow | samgr_common | 整数溢出检测 |
| UBSan | samgr_common | 未定义行为检测 |
| Stack Protector | samgr_rust | Rust 栈保护 |
| CFI Blocklist | - | `cfi_blocklist.txt` |

## 常用构建命令

```bash
# 构建 samgr 可执行文件
hb build //foundation/systemabilitymgr/samgr/services/samgr/native:samgr

# 构建 samgr_proxy 共享库
hb build //foundation/systemabilitymgr/samgr/interfaces/innerkits/samgr_proxy:samgr_proxy

# 构建所有 samgr 目标
hb build //foundation/systemabilitymgr/samgr/...

# 启用特性开关构建
hb build --gn-args="support_common_event=true"
```
