# TEE Client 构建系统

## 1. 构建系统概述

TEE Client 使用 GN (Generate Ninja) 作为构建系统，与 OpenHarmony 整体构建框架集成。

**构建命令**（以 RK3568 芯片为例）：
```bash
./build.sh --product-name rk3568 --ccache --build-target tee_client
```

**产物输出路径**：`out/rk3568/tee/tee_client`

## 2. GN 构建配置

### 2.1 根配置文件

| 文件 | 说明 |
|------|------|
| `tee_client.gni` | 全局构建参数配置 |

**tee_client.gni 内容**：
```gn
declare_args() {
    tee_client_features_tui = false  // TUI 功能开关
}
```

### 2.2 全局配置参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `tee_client_features_tui` | bool | false | 启用 TUI (Trusted User Interface) 功能 |
| `component_type` | string | "" | 组件类型，为 "system" 时启用完整功能 |

## 3. Targets 清单

### 3.1 Frameworks 层 Targets

#### libteec

| 属性 | 值 |
|------|-----|
| **Target 名称** | `libteec` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libteec.so` |
| **安装路径** | `/system/lib/` |
| **Part** | `tee_client` |
| **子系统** | `tee` |
| **InnerAPI 标签** | `ndk` |

**BUILD.gn 位置**：`frameworks/build/standard/BUILD.gn`

**Source 文件**：
```
libteec_client/tee_client.cpp
```

**Include 目录**：
```
../../../interfaces/inner_api
../../include
../../include/standard/teec_system/
../../include/standard/
../../libteec_client/
../../libteec_vendor/
../../authentication
```

**外部依赖**：
```
bounds_checking_function:libsec_shared
c_utils:utils
hilog:libhilog
ipc:ipc_single
safwk:system_ability_fwk
samgr:samgr_proxy
```

**链接选项**：
```
-Wl,-z,max-page-size=4096
-Wl,-z,separate-code
```

---

#### libteec_vendor

| 属性 | 值 |
|------|-----|
| **Target 名称** | `libteec_vendor` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libteec_vendor.so` |
| **安装路径** | `/vendor/lib/` |
| **Part** | `tee_client` |
| **子系统** | `tee` |
| **InnerAPI 标签** | `chipsetsdk_sp` |

**BUILD.gn 位置**：`frameworks/build/standard/BUILD.gn`

**Source 文件**：
```
../../libteec_vendor/load_sec_file.c
../../libteec_vendor/tee_client_api.c
../../libteec_vendor/tee_client_app_load.c
../../libteec_vendor/tee_client_socket.c
../../../services/authentication/tee_auth_common.c
```

**宏定义**：
```
LIB_TEEC_VENDOR
CONFIG_LOG_REPORT
```

**Include 目录**：
```
../../../interfaces/inner_api
../../include
../../include/standard/teec_vendor/
../../include/standard/
../../libteec_vendor/
../../../services/authentication
```

**外部依赖**：
```
bounds_checking_function:libsec_shared
c_utils:utils
hilog:libhilog
hisysevent:libhisysevent
```

---

### 3.2 Services 层 Targets

#### teecd

| 属性 | 值 |
|------|-----|
| **Target 名称** | `teecd` |
| **类型** | `ohos_executable` |
| **输出文件** | `teecd` |
| **安装路径** | `/vendor/bin/` |
| **Part** | `tee_client` |
| **子系统** | `tee` |

**BUILD.gn 位置**：`services/teecd/build/standard/BUILD.gn`

**Source 文件**：
```
../../../../frameworks/tee_file/tee_file.c
../../../authentication/tcu_authentication.c
../../../authentication/tee_auth_common.c
../../../authentication/tee_get_native_cert.c
../../src/fs_work_agent.c
../../src/late_init_agent.c
../../src/misc_work_agent.c
../../src/secfile_load_agent.c
../../src/tee_agent.c
../../src/tee_ca_auth.c
../../src/tee_ca_daemon.c
../../src/tee_load_dynamic.c
```

**宏定义**：
```
CONFIG_FSWORK_THREAD_ELEVATE_PRIO
DYNAMIC_DRV_DIR="/vendor/bin/tee_dynamic_drv/"
DYNAMIC_SRV_DIR="/vendor/bin/tee_dynamic_srv/"
CONFIG_LATE_INIT
ENABLE_FDSAN_CHECK
DYNAMIC_SRV_FEIMA_DIR="/vendor/etc/passthrough/teeos/dynamic_srv"
DYNAMIC_DRV_FEIMA_DIR="/vendor/etc/passthrough/teeos/dynamic_drv"
```

**Include 目录**：
```
../../../../interfaces/inner_api
../../include
../../include/standard
../../../../frameworks/include
../../../../frameworks/include/standard
../../../../frameworks/include/standard/teec_vendor
../../../../frameworks/libteec_vendor
../../../authentication
```

**外部依赖**：
```
bounds_checking_function:libsec_shared
c_utils:utils
hilog:libhilog
```

---

#### libcadaemon

| 属性 | 值 |
|------|-----|
| **Target 名称** | `libcadaemon` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libcadaemon.so` |
| **安装路径** | `/system/lib/` |
| **Part** | `tee_client` |
| **子系统** | `tee` |
| **条件** | `component_type == "system"` |

**BUILD.gn 位置**：`services/cadaemon/build/standard/BUILD.gn`

**Source 文件**：
```
../../../../frameworks/libteec_vendor/load_sec_file.c
../../../../frameworks/libteec_vendor/tee_client_api.c
../../../../frameworks/libteec_vendor/tee_client_app_load.c
../../../../frameworks/tee_file/tee_file.c
../../../authentication/tcu_authentication.c
../../../authentication/tee_auth_common.c
../../../authentication/tee_auth_system.cpp
../../../authentication/tee_get_native_cert.c
../../src/ca_daemon/cadaemon_service.cpp
../../src/ca_daemon/cadaemon_stub.cpp
../../src/tui_daemon/tee_tui_daemon_wrapper.cpp
../../src/tui_daemon/tui_file.cpp
```

**条件依赖**（当 `tee_client_features_tui == true`）：
```
../../../../services/cadaemon/build/standard:libcadaemon_tui
```

**宏定义**（当 `component_type == "system"`）：
```
ENABLE_FDSAN_CHECK
CONFIG_LOG_REPORT
```

**Include 目录**：
```
../../../../interfaces/inner_api
../../../../frameworks/include/
../../../../frameworks/include/standard/
../../../../frameworks/include/standard/teec_system
../../../../frameworks/libteec_vendor/
../../../authentication
../../src/ca_daemon
../../../../services/cadaemon/src/tui_daemon
```

**外部依赖**：
```
access_token:libaccesstoken_sdk
bounds_checking_function:libsec_shared
bundle_framework:appexecfwk_base
bundle_framework:bundlemgr_mini
c_utils:utils
hilog:libhilog
ipc:ipc_single
openssl:libcrypto_shared
safwk:system_ability_fwk
samgr:samgr_proxy
hisysevent:libhisysevent
```

**Sanitize**：
```
cfi: true
cfi_cross_dso: true
```

---

#### libcadaemon_tui

| 属性 | 值 |
|------|-----|
| **Target 名称** | `libcadaemon_tui` |
| **类型** | `ohos_shared_library` |
| **输出文件** | `libcadaemon_tui.so` |
| **条件** | `tee_client_features_tui == true` |

**Source 文件**：
```
../../../../frameworks/tee_file/tee_file.c
../../src/tui_daemon/tee_tui_daemon.cpp
../../src/tui_daemon/tui_event.cpp
```

**宏定义**：
```
FONT_HASH_VAL="8978e05044e7089ad6a9de38c505c8148305607983487435a916d2610700a7ca"
ENABLE_FDSAN_CHECK
SCENE_BOARD_ENABLE
```

**外部依赖**：
```
ability_base:want
bounds_checking_function:libsec_shared
c_utils:utils
call_manager:tel_call_manager_api
hilog:libhilog
image_framework:image_native
ipc:ipc_single
power_manager:powermgr_client
safwk:system_ability_fwk
samgr:samgr_proxy
init:libbegetutil
window_manager:libdm_lite
```

---

#### tlogcat

| 属性 | 值 |
|------|-----|
| **Target 名称** | `tlogcat` |
| **类型** | `ohos_executable` |
| **输出文件** | `tlogcat` |
| **安装路径** | `/system/bin/` |
| **Part** | `tee_client` |
| **子系统** | `tee` |

**BUILD.gn 位置**：`services/tlogcat/build/standard/BUILD.gn`

**Source 文件**：
```
../../../../frameworks/tee_file/tee_file.c
../../src/proc_tag.c
../../src/sys_hilog_cfg.c
../../src/tarzip.c
../../src/tlogcat.c
```

**宏定义**：
```
TEE_LOG_PATH_BASE="/data/log"
CONFIG_TLOGCAT_TAG
CONFIG_TEE_PRIVATE_LOGFILE
ENABLE_FDSAN_CHECK
```

**外部依赖**：
```
c_utils:utils
hilog:libhilog
zlib:libz
```

---

## 4. 启动配置 Targets

### 4.1 Init 配置文件

| Target | 类型 | 源文件 | 安装位置 |
|--------|------|--------|----------|
| `teecd.rc` | `ohos_prebuilt_etc` | `teecd.cfg` | `/system/etc/init/` |
| `cadaemon.rc` | `ohos_prebuilt_etc` | `cadaemon.cfg` | `/system/etc/init/` |
| `tlogcat.rc` | `ohos_prebuilt_etc` | `tlogcat.cfg` | `/system/etc/init/` |

### 4.2 SA Profile

| Target | 类型 | 源文件 | SA ID |
|--------|------|--------|-------|
| `cadaemon_profile` | `ohos_sa_profile` | `8001.json` | 8001 |

**8001.json 内容**：
```json
{
    "name": "CaDaemonService",
    "libpath": "/system/lib/libcadaemon.z.so",
    "run-on-create": true,
    "start-mode": "boot"
}
```

---

## 5. 产物清单与安装路径

### 5.1 编译产物总览

| 产物 | 类型 | 默认输出路径 | 安装路径 |
|------|------|--------------|----------|
| libteec.so | 共享库 | `out/rk3568/tee/tee_client` | `/system/lib/` |
| libteec_vendor.so | 共享库 | `out/rk3568/tee/tee_client` | `/vendor/lib/` |
| libcadaemon.so | 共享库 | `out/rk3568/tee/tee_client` | `/system/lib/` |
| libcadaemon_tui.so | 共享库 | `out/rk3568/tee/tee_client` | `/system/lib/` (条件) |
| teecd | 可执行文件 | `out/rk3568/tee/tee_client` | `/vendor/bin/` |
| tlogcat | 可执行文件 | `out/rk3568/tee/tee_client` | `/system/bin/` |
| teecd.rc | 配置文件 | `out/rk3568/tee/tee_client` | `/system/etc/init/` |
| cadaemon.rc | 配置文件 | `out/rk3568/tee/tee_client` | `/system/etc/init/` |
| tlogcat.rc | 配置文件 | `out/rk3568/tee/tee_client` | `/system/etc/init/` |
| 8001.json | SA 配置 | `out/rk3568/tee/tee_client` | `/system/profile/` |

### 5.2 运行时加载关系

```
CA Application
    │
    ├── libteec.so (/system/lib/)
    │       │
    │       ├── 依赖 ipc_single
    │       ├── 依赖 hilog
    │       └── 通过 IPC 与 cadaemon 通信
    │
    └── (可选) libteec_vendor.so (/vendor/lib/)
            │
            ├── 通过 Unix Socket 与 teecd 通信
            └── 用于芯片组件

cadaemon (/system/lib/libcadaemon.so)
    │
    ├── 依赖 safwk (System Ability 框架)
    ├── 依赖 access_token
    └── 作为 SA 8001 注册到 samgr

teecd (/vendor/bin/teecd)
    │
    ├── 依赖 frameworks/tee_file
    └── 依赖 authentication 模块
```

---

## 6. 依赖关系图

```
tee_client.gni
    │
    ├── tee_client_features_tui = false
    │
    ├── frameworks/build/standard/BUILD.gn
    │       │
    │       ├── libteec (shared_library)
    │       │   ├── sources: libteec_client/tee_client.cpp
    │       │   ├── deps: ipc, hilog, c_utils, safwk, samgr
    │       │   └── output: libteec.so
    │       │
    │       └── libteec_vendor (shared_library)
    │           ├── sources: libteec_vendor/*.c, auth/*.c
    │           ├── deps: hilog, hisysevent
    │           └── output: libteec_vendor.so
    │
    ├── services/teecd/build/standard/BUILD.gn
    │       │
    │       └── teecd (executable)
    │           ├── sources: tee_file/*.c, auth/*.c, src/*.c
    │           ├── deps: hilog, c_utils
    │           └── output: teecd
    │
    ├── services/cadaemon/build/standard/BUILD.gn
    │       │
    │       ├── libcadaemon (shared_library)
    │       │   ├── sources: libteec_vendor/*.c, src/*.cpp, tui/*.cpp
    │       │   ├── deps: libteec_vendor sources, access_token, ipc
    │       │   ├── conditional: libcadaemon_tui (if tui enabled)
    │       │   └── output: libcadaemon.so
    │       │
    │       └── libcadaemon_tui (shared_library, conditional)
    │           ├── sources: tui/*.cpp
    │           ├── deps: window_manager, power_manager
    │           └── output: libcadaemon_tui.so
    │
    └── services/tlogcat/build/standard/BUILD.gn
            │
            └── tlogcat (executable)
                ├── sources: tee_file/*.c, src/*.c
                ├── deps: hilog, zlib
                └── output: tlogcat
```

---

## 7. Feature Flags

### 7.1 tee_client_features_tui

**作用**：启用 TUI (Trusted User Interface) 功能

**默认值**：`false`

**启用方式**：
```gn
# 在产品配置或gn文件中
tee_client_features_tui = true
```

**影响**：
- 编译 `libcadaemon_tui.so`
- cadaemon 创建 TUI 专用线程
- 启用 TUI 相关功能（屏幕锁定、显示管理）

---

## 8. 编译产物推送与调试

### 8.1 推送命令

```bash
# 推送 cadaemon 配置
hdc file send cadaemon.json /system/profile/
hdc file send cadaemon.cfg /system/etc/init/

# 推送系统库
hdc file send libteec.so /system/lib/
hdc file send libcadaemon.so /system/lib/

# 推送日志服务
hdc file send tlogcat /system/bin/

# 推送芯片库
hdc file send libteec_vendor.so /vendor/lib/
hdc file send teecd /vendor/bin/
```

### 8.2 服务启动

```bash
# 重启 cadaemon 服务
hdc shell restart caDaemon

# 重启 teecd 服务
hdc shell restart teecd

# 查看服务状态
hdc shell ps -A | grep -E "cadaemon|teecd|tlogcat"
```

---

## 9. 条件编译说明

### 9.1 component_type 条件

当 `component_type == "system"` 时：
- 启用完整的 cadaemon 功能
- 启用 CFI (Control Flow Integrity)  sanitize
- 包含更多认证模块

### 9.2 tee_client_features_tui 条件

当 `tee_client_features_tui == true` 时：
- 编译 libcadaemon_tui.so
- cadaemon 创建 TUI 线程
- 启用 TUI 事件监听

---

## 10. bundle.json 配置

**文件位置**：`bundle.json`

```json
{
  "name": "@ohos/tee_client",
  "component": {
    "name": "tee_client",
    "subsystem": "tee",
    "syscap": ["SystemCapability.Tee.TeeClient = false"],
    "features": ["tee_client_features_tui"],
    "adapted_system_type": ["small", "standard"],
    "rom": "250KB",
    "ram": "5995KB",
    "deps": {
      "components": [
        "ability_base",
        "access_token",
        "call_manager",
        "c_utils",
        "hilog",
        "image_framework",
        "power_manager",
        "safwk",
        "samgr",
        "window_manager",
        "ipc",
        "dsoftbus",
        "device_manager",
        "bounds_checking_function",
        "zlib",
        "openssl",
        "bundle_framework",
        "hisysevent",
        "init"
      ]
    },
    "build": {
      "group_type": {
        "service_group": [
          "//base/tee/tee_client/services/cadaemon/build/standard:libcadaemon",
          "//base/tee/tee_client/services/cadaemon/build/standard/init:cadaemon.rc",
          "//base/tee/tee_client/services/cadaemon/build/standard/sa_profile:cadaemon_profile",
          "//base/tee/tee_client/services/teecd/build/standard:teecd",
          "//base/tee/tee_client/services/teecd/build/standard/init:teecd.rc",
          "//base/tee/tee_client/services/tlogcat/build/standard:tlogcat",
          "//base/tee/tee_client/services/tlogcat/build/standard/init:tlogcat.rc"
        ]
      },
      "inner_kits": [
        {
          "name": "//base/tee/tee_client/frameworks/build/standard:libteec",
          "header": {...}
        },
        {
          "name": "//base/tee/tee_client/frameworks/build/standard:libteec_vendor",
          "header": {...}
        }
      ]
    }
  }
}
```
