# GN 目标梳理

## 目的

描述 hdc 项目的 GN 构建系统，包括所有 targets、类型、依赖、配置和输出产物。

## 适用范围

本文档适用于：
- 理解构建系统结构
- 修改构建配置
- 添加新的 targets
- 了解编译产物和安装位置

## 相关跳转

- [目录结构](./02_Directory_Structure.md) - 模块职责
- [构建产物](./07_Build_Artifacts.md) - 安装路径和加载关系

---

## BUILD.gn 文件清单

### 根构建文件

**文件**：`/Volumes/lexar/code/d/work/oh/developtools/hdc/BUILD.gn` (584 行)

**主要目标**：

| Target | 类型 | 输出 | 用途 |
|--------|------|------|------|
| `hdc` | ohos_executable | hdc | PC 端工具 |
| `hdcd_system` | ohos_prebuilt_executable | system/hdcd | 设备端守护进程（system 镜像）|
| `hdcd_updater` | ohos_prebuilt_executable | updater/hdcd | 设备端守护进程（updater 镜像）|
| `hdc_register` | ohos_shared_library | libhdc_register.so | JDWP 注册库 |
| `hdc_updater` | ohos_static_library | libhdc_updater.a | 升级静态库 |
| `serialize_structs` | ohos_static_library | libserialize_structs.a | Rust CFFI 桥接 |
| `lib` | ohos_rust_static_library | libhdc.rlib | Rust 库 |
| `hdc_hash_gen` | action | hdc_hash_gen.h | Hash 头生成 |

### 子目录 BUILD.gn 文件

| 目录 | 文件 | Targets |
|--------|------|--------|
| `credential/` | BUILD.gn | hdc_credential_exec, hdc_credential (group) |
| `sudo/` | BUILD.gn | exec_sudo, sudo (group) |
| `hdcd_user_permit/` | BUILD.gn | exec_hdcd_user_permit, hdcd_user_permit (group) |
| `hdc_rust/` | BUILD.gn | serialize_structs, hdcd, cffi_host, hdc_library_host, hdc_rust (executable) |
| `src/daemon/etc/` | BUILD.gn | daemon_etc (group), hdc_credential.cfg, hdcd.cfg, hdc.para, hdc.para.dac |

---

## 模板 Targets

### hdcd_source_set 模板

**定义**：`BUILD.gn:80-193`

**用途**：创建 Daemon 源集模板

**参数**：
- `hdcd_uv_thread_size`：默认 4（Daemon UV 线程数）
- `image_name`：system/updater

**Sources**（共约 37 个文件）：
- Daemon 源文件：`src/daemon/*.cpp` (13 个)
- Common 源文件：`hdc_common_sources` (25 个，包括 connect_validation.cpp)
- 配置目标：`src/daemon/etc:daemon_etc`

**Defines**：
```gn
defines = [
  "HARMONY_PROJECT",
  "USE_CONFIG_UV_THREADS",
  "SIZE_THREAD_POOL=$hdcd_uv_thread_size",
  "HDC_HILOG",
  "OPENSSL_SUPPRESS_DEPRECATED",
  "HDC_SUPPORT_ENCRYPT_TCP",
  "MEMORY_POOL_ENABLE"
]
```

**外部依赖**：
```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "c_utils:utils",
  "hilog:libhilog",
  "init:libbegetutil",
  "libuv:uv",
  "lz4:liblz4_static",
  "openssl:libcrypto_shared",
  "openssl:libssl_shared",
  "hicollie:libhicollie",
]
```

### build_hdc 模板

**定义**：`BUILD.gn:263-372`

**用途**：构建 Daemon 可执行文件和预构建安装

**输出**：
- `hdcd_${image_name}_exe`：编译的可执行文件
- `hdcd_${image_name}`：预构建安装目标

**安装路径**：`system/bin/` (system 镜像) 或 `updater/bin/` (updater 镜像)

---

## 主 Targets 详解

### 1. hdc - Host 工具

**定义**：`BUILD.gn:380-491`

**类型**：`ohos_executable`

**输出**：`hdc` (PC 端可执行文件)

**Sources**（19 个文件）：
- Host 源文件：`src/host/*.cpp` (15 个)
- Common 源文件：`hdc_common_sources` (25 个)
- 条件源文件（如果 `is_ohos`）：
  - `src/common/credential_message.cpp`
  - `src/common/connect_validation.cpp`
  - `src/common/hdc_huks.cpp`
  - `src/common/password.cpp`
  - `src/common/command_event_report.cpp`
  - `src/host/system_depend.cpp`

**Defines**：
```gn
defines = [
  "HDC_HOST",
  "HARMONY_PROJECT",
  "USE_CONFIG_UV_THREADS",
  "SIZE_THREAD_POOL=$hdc_uv_thread_size",
  "OPENSSL_SUPPRESS_DEPRECATED",
  "HDC_SUPPORT_ENCRYPT_TCP",
  "MEMORY_POOL_ENABLE"
]
```

**条件 Defines**（平台相关）：
```gn
if (is_mac) {
  defines += [ "HOST_MAC" ]
}
if (is_linux) {
  defines += [ "HOST_LINUX" ]
}
if (is_mingw) {
  defines += [
    "HOST_MINGW",
    "WIN32_LEAN_AND_MEAN"
  ]
}
if (is_ohos) {
  defines += [
    "IS_RELEASE_VERSION",
    "HDC_SUPPORT_ENCRYPT_PRIVATE_KEY",
    "HOST_OHOS",
    "HDC_SUPPORT_REPORT_COMMAND_EVENT"
  ]
}
```

**外部依赖**：
```gn
external_deps = [
  "bounds_checking_function:libsec_static",
  "libusb:libusb",
  "libuv:uv_static",
  "lz4:liblz4_static",
  "openssl:libcrypto_static",
  "openssl:libssl_static",
]
```

**条件依赖**（如果 `is_ohos`）：
```gn
external_deps += [
  "huks:libhukssdk",
  "c_utils:utils",
  "init:libbegetutil",
]
```

**特殊配置**（`is_mingw`）：
```gn
static_link = false
libs = [ "setupapi" ]
ldflags = [
  "-Wl,--whole-archive",
  "-lpthread",
  "-Wl,--no-whole-archive",
]
```

**特殊配置**（`is_linux`）：
```gn
static_link = false
ldflags = [
  "-Wl,--whole-archive",
  "-lpthread",
  "-latomic",
  "-ldl",
  "-lrt",
  "-Wl,--no-whole-archive",
]
```

### 2. hdcd_system - Device Daemon (System 镜像)

**定义**：`BUILD.gn:374-391`

**类型**：`ohos_prebuilt_executable`

**输出**：`system/hdcd`

**依赖**：`hdcd_system_source` (使用 `hdcd_source_set` 模板，image_name=system)

**安装**：`system/bin/`

**附加 Defines**（如果 `build_selinux && image_name == "system"`）：
```gn
defines += [
  "SURPPORT_SELINUX",
  "HDC_TRACE",
  "HDC_STATISTIC_REPORT_ENABLE",
  "HDC_HICOLLIE_ENABLE",
]
external_deps += [
  "selinux:libselinux",
  "hitrace:hitrace_meter",
  "hisysevent:libhisysevent",
]
```

### 3. hdcd_updater - Device Daemon (Updater 镜像)

**定义**：`BUILD.gn:401-429`

**类型**：`ohos_prebuilt_executable`

**输出**：`updater/hdcd`

**依赖**：`hdcd_updater_source` (使用 `hdcd_source_set` 模板，image_name=updater)

**安装**：`updater/bin/`

**附加 Defines**：
```gn
defines += [ "UPDATER_MODE" ]
```

### 4. hdc_register - JDWP 注册库

**定义**：`BUILD.gn:542-567`

**类型**：`ohos_shared_library`

**输出**：`libhdc_register.so`

**Sources**：
- `src/register/hdc_connect.cpp`
- `src/register/hdc_jdwp.cpp`

**Defines**：
```gn
defines = [
  "JS_JDWP_CONNECT",
  "HDC_HILOG",
]
```

**外部依赖**：
```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "c_utils:utils",
  "hilog:libhilog",
  "init:libbeget_proxy",
  "init:libbegetutil",
  "libuv:uv",
]
```

**内部 API 标签**：`innerapi_tags = [ "platformsdk" ]`

### 5. serialize_structs - Rust CFFI 桥接

**定义**：`BUILD.gn:196-236`

**类型**：`ohos_static_library`

**输出**：`libserialize_structs.a`

**条件**：`product_name != "ohos-sdk"`

**Sources**（20 个文件）：
- `hdc_rust/src/cffi/base.h`, `bridge.cpp/h`
- `hdc_rust/src/cffi/cmd.cpp/h`
- `hdc_rust/src/cffi/getparameter.cpp/h`
- `hdc_rust/src/cffi/log.cpp/h`
- `hdc_rust/src/cffi/mount.cpp/h`, `mount_wrapper.cpp`
- `hdc_rust/src/cffi/oh_usb.cpp/h`
- `hdc_rust/src/cffi/sendmsg.cpp/h`
- `hdc_rust/src/cffi/serial_struct.cpp/h`, `serial_struct_define.h`
- `hdc_rust/src/cffi/sys_para.cpp/h`
- `hdc_rust/src/cffi/transfer.cpp`
- `hdc_rust/src/cffi/uart.cpp/h`, `uart_wrapper.cpp`
- `hdc_rust/src/cffi/usb_util.cpp/h`, `usb_wrapper.cpp`
- `hdc_rust/src/cffi/utils.cpp/h`

**外部依赖**：
```gn
external_deps = [
  "bounds_checking_function:libsec_static",
  "hilog:libhilog",
  "init:libbegetutil",
  "lz4:liblz4_static",
]
```

**条件 Defines**（如果 `is_mac`）：
```gn
defines = [ "HOST_MAC" ]
```

**条件 Defines**（如果 `build_selinux`）：
```gn
external_deps += [ "selinux:libselinux" ]
defines += [ "SURPPORT_SELINUX" ]
```

### 6. hdcd - Rust Daemon

**定义**：`hdc_rust/BUILD.gn:324-353`

**类型**：`ohos_rust_executable`

**输出**：`hdcd`

**依赖**：
- `serialize_structs`
- `//third_party/rust/crates/env_logger:lib`
- `//third_party/rust/crates/humantime:lib`
- `//third_party/rust/crates/log:lib`
- `//third_party/rust/crates/nix:lib`

**外部依赖**：
```gn
external_deps = [
  "hilog:hilog_rust",
  "rust_libc:lib",
  "rust_rust-openssl:lib",
]
```

**条件依赖**（如果 `product_name != "ohos-sdk"`）：
```gn
deps += [
  ":serialize_structs",
  ":lib",
]
external_deps += [
  "ylong_runtime:ylong_runtime",
]
```

**条件依赖**（如果 `!defined(ohos_lite)`）：
```gn
external_deps += [ "faultloggerd:panic_handler" ]
```

**条件依赖**（如果 `is_emulator && product_name != "ohos-sdk"`）：
```gn
features = [ "emulator" ]
```

**配置**：
```gn
rustflags = [ "-Cforce-frame-pointers=yes" ]
```

---

## Feature Flags

### 全局配置（hdc.gni）

**定义**：`/Volumes/lexar/code/d/work/oh/developtools/hdc/hdc.gni`

**主要 flags**：

| Flag | 默认值 | 说明 |
|-------|--------|------|
| `hdc_debug` | false | 调试构建（HDC_DEBUG 宏）|
| `hdc_host_hide_debug_win` | true | Windows 下隐藏调试窗口 |
| `hdc_support_uart` | true | 启用 UART 传输支持 |
| `hdc_test_coverage` | false | 启用测试覆盖率 |
| `hdc_jdwp_test` | false | 启用 JDWP 测试 |
| `js_jdwp_connect` | true | 启用 JavaScript JDWP 连接 |
| `hdc_version_check` | false | 启用版本检查 |
| `support_hdcd_user_permit` | false | 启用用户权限助手 |
| `hdc_support_account_constraint` | true | 启用账户约束 |
| `hdc_feature_support_sudo` | false | 启用 sudo 功能 |
| `hdc_feature_support_credential` | false | 启用凭证管理 |
| `hdc_feature_support_report_command_event` | false | 启用命令事件上报 |
| `hdc_feature_support_usr_symlink` | false | 启用用户符号链接 |

### 条件编译配置（BUILD.gn）

| 条件 | Defines | 说明 |
|--------|---------|------|
| `is_mac` | `HOST_MAC` | macOS 平台 |
| `is_linux` | `HOST_LINUX` | Linux 平台 |
| `is_mingw` | `HOST_MINGW`, `WIN32_LEAN_AND_MEAN` | Windows/MinGW 平台 |
| `is_ohos` | `IS_RELEASE_VERSION`, `HARMONY_PROJECT` | OpenHarmony 构建 |
| `build_variant == "user"` | `IS_RELEASE_VERSION` | 用户版本构建 |
| `build_selinux` | `SURPPORT_SELINUX` | SELinux 支持 |
| `image_name == "system"` | `HDC_TRACE`, `HDC_STATISTIC_REPORT_ENABLE` | System 镜像特性 |
| `image_name == "updater"` | `UPDATER_MODE` | Updater 镜像模式 |
| `is_emulator` | `HDC_EMULATOR` | 模拟器模式 |
| `hdc_version_check` | `HDC_VERSION_CHECK` | 版本检查功能 |

---

## Target 依赖关系

### 完整依赖图

```
hdc_target (group)
├── hdc (executable)
│   ├── hdc_hash_gen (action)
│   └── external_deps: libsec_static, libusb, uv_static, lz4, openssl
│
├── hdc_register (shared_library)
│   └── external_deps: libsec_shared, c_utils, hilog, init, libuv
│
├── hdcd_system (prebuilt_executable)
│   ├── hdcd_system_source (source_set) [uses hdcd_source_set template]
│   │   ├── hdc_hash_gen (action)
│   │   ├── daemon_etc (group)
│   │   └── external_deps: libsec_shared, c_utils, hilog, init, libuv, lz4, openssl, hicollie
│   │   └── conditional: selinux, hitrace, hisysevent (if build_selinux && system image)
│   └── hdcd_system_exe (executable)
│
└── hdcd_updater (prebuilt_executable)
    ├── hdcd_updater_source (source_set) [uses hdcd_source_set template]
    │   ├── hdc_hash_gen (action)
    │   ├── daemon_etc (group)
    │   └── external_deps: libsec_shared, c_utils, hilog, init, libuv, lz4, openssl
    └── hdcd_updater_exe (executable) [has UPDATER_MODE define]
```

---

## 配置文件安装

### src/daemon/etc/ 目标

**定义**：`src/daemon/etc/BUILD.gn`

**Targets**：

| Target | 类型 | 源文件 | 安装路径 |
|--------|------|---------|----------|
| `hdc_credential.cfg` | ohos_prebuilt_etc | `init/hdc_credential.cfg` |
| `hdcd.cfg` | ohos_prebuilt_etc | `init/hdcd.cfg` (user) 或 `init/hdcd.root.cfg` (root) |
| `hdcd.root.cfg` | ohos_prebuilt_etc | 同上（选择安装）|
| `hdc.para` | ohos_prebuilt_etc | `param/hdc.para` (user) 或 `param/hdc.root.para` (root) |
| `hdc.root.para` | ohos_prebuilt_etc | 同上（选择安装）|
| `hdc.para.dac` | ohos_prebuilt_etc | `param/hdc.para.dac` |

**安装镜像**：
- `hdcd.cfg` 和 `hdc.para` 系列：`install_images = [ "system" ]`
- `hdcd.root.cfg` 和 `hdc.root.para` 系列：`install_images = [ "updater" ]`

---

## 关键结论

1. **三部分构建输出**：
   - PC 端：`hdc` 可执行文件
   - Device 端 System 镜像：`system/hdcd`
   - Device 端 Updater 镜像：`updater/hdcd`

2. **Feature Flags 控制**：
   - 主要功能通过 `hdc_feature_support_*` flags 控制
   - 平台相关 features 通过编译选项控制
   - 条件编译依赖 `is_*` 变量

3. **复杂的条件编译**：
   - 根据 `image_name` 选择不同的 Defines
   - 根据 `build_selinux` 添加 SELinux 依赖
   - 根据 `product_name` 控制编译范围

4. **Rust 迁移进行中**：
   - `hdc_rust/` 提供完整的 Rust 实现
   - 通过 CFFI 桥接与 C++ 代码互操作
   - 使用 Rust 特定 features（emulator）

---

## 待确认事项

**TODO(需确认)**：
1. 各 targets 的实际编译产物大小
2. Feature flags 对最终二进制大小的影响
3. Rust 实现的迁移完成度
