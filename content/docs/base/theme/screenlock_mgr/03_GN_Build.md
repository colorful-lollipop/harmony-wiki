# 03_GN_Build - 构建系统

## 1. 构建概述

### 1.1 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建入口文件 |
| `screenlock.gni` | 全局变量定义 |

**根 BUILD.gn**:
```gn
// BUILD.gn:24-32
group("screenlock_mgr_packages") {
  if (is_standard_system) {
    deps = [
      ":screenlock_cfg",
      "sa_profile:screenlock_sa_profiles",
      "services:screenlock_server",
    ]
  }
}
```

---

### 1.2 全局变量 (screenlock.gni)

```gni
// screenlock.gni:14-24
screenlock_mgr_path = "//base/theme/screenlock_mgr"

window_base_path = "//foundation/window/window_manager"
graphic_base_path = "//foundation/graphic"

declare_args() {
  # Functional isolation, memory optimization
  screenlock_mgr_so_crop = false
  screenlock_mgr_wearable_enable_payment_app = false
}
```

---

## 2. Targets 清单

### 2.1 根目录 BUILD.gn

| Target | 类型 | 输出 | 依赖 |
|--------|------|------|------|
| `screenlock_cfg` | ohos_prebuilt_etc | screenlock.cfg | 无 |
| `screenlock_mgr_packages` | group | 包组 | :screenlock_cfg, sa_profile, services |

### 2.2 SA Profile (sa_profile/BUILD.gn)

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `screenlock_sa_profiles` | ohos_sa_profile | 3704.json | SA 配置 |

**SA 配置** (`sa_profile/3704.json`):
```json
{
    "process": "foundation",
    "systemability": [{
        "name": 3704,
        "libpath": "libscreenlock_server.z.so",
        "run-on-create": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

---

### 2.3 Services (services/BUILD.gn)

#### 2.3.1 screenlock_server (动态库)

| 属性 | 值 |
|------|-----|
| 类型 | ohos_shared_library |
| 输出 | libscreenlock_server.z.so |
| 编译选项 | CFI, PAC_RET, -fvisibility=hidden |

**关键 sources**:
```gn
sources = [
  "src/command.cpp",
  "src/commeventsubscriber.cpp",
  "src/dump_helper.cpp",
  "src/innerlistenermanager.cpp",
  "src/preferences_util.cpp",
  "src/screenlock_callback_proxy.cpp",
  "src/screenlock_get_info_callback.cpp",
  "src/screenlock_inner_listener_proxy.cpp",
  "src/screenlock_manager_stub.cpp",
  "src/screenlock_system_ability.cpp",
  "src/screenlock_system_ability_proxy.cpp",
  "src/strongauthmanager.cpp",
]
```

**public_configs**:
```gn
public_configs = [ ":screenlock_mgr_service_config" ]

config("screenlock_mgr_service_config") {
  visibility = [ ":*" ]
  include_dirs = [
    "include",
    "${screenlock_mgr_path}/frameworks/native/include",
    "${screenlock_mgr_path}/interfaces/inner_api/include",
  ]
}
```

**external_deps**:
```gn
external_deps = [
  "ability_base:want",
  "ability_runtime:ability_manager",
  "access_token:libaccesstoken_sdk",
  "access_token:libtokenid_sdk",
  "c_utils:utils",
  "common_event_service:cesfwk_innerkits",
  "eventhandler:libeventhandler",
  "ffrt:libffrt",
  "hilog:libhilog",
  "hitrace:hitrace_meter",
  "init:libbeget_proxy",
  "ipc:ipc_single",
  "os_account:os_account_innerkits",
  "preferences:native_preferences",
  "safwk:system_ability_fwk",
  "samgr:samgr_proxy",
  "time_service:time_client",
  "user_auth_framework:userauth_client",
  "window_manager:libdm",
  "window_manager:libwm",
]
```

**特性控制**:
```gn
if (screenlock_mgr_so_crop == true) {
  cflags += [ "-DIS_SO_CROP_H" ]
}

if (screenlock_mgr_wearable_enable_payment_app == true) {
  defines = [ "SUPPORT_WEAR_PAYMENT_APP" ]
  deps = [ "${screenlock_mgr_path}/watch:watch_screenlock_static" ]
}
```

**安全编译选项**:
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  cfi_vcall_icall_only = true
}
branch_protector_ret = "pac_ret"
```

---

#### 2.3.2 screenlock_server_static (静态库)

| 属性 | 值 |
|------|-----|
| 类型 | ohos_static_library |
| 输出 | libscreenlock_server_static.a |

**用途**: 测试和静态链接场景

---

### 2.4 N-API (frameworks/js/napi/BUILD.gn)

#### 2.4.1 screenlock (动态库)

| 属性 | 值 |
|------|-----|
| 类型 | ohos_shared_library |
| 输出 | libscreenlock.so |

**sources**:
```gn
sources = [
  "src/async_call.cpp",
  "src/napi_screenlock_ability.cpp",
  "src/screenlock_callback.cpp",
  "src/screenlock_js_util.cpp",
  "src/screenlock_system_ability_callback.cpp",
  "src/uv_queue.cpp",
]
```

**deps**:
```gn
deps = [
  "${screenlock_mgr_path}/interfaces/inner_api:screenlock_client_static",
]

external_deps = [
  "c_utils:utils",
  "eventhandler:libeventhandler",
  "hilog:libhilog",
  "hitrace:hitrace_meter",
  "ipc:ipc_single",
  "napi:ace_napi",
]
```

**安装路径**:
```gn
relative_install_dir = "module"
```

---

### 2.5 Native Client (interfaces/inner_api/BUILD.gn)

#### 2.5.1 screenlock_client (动态库)

| 属性 | 值 |
|------|-----|
| 类型 | ohos_shared_library |
| 输出 | libscreenlock_client.z.so |

**sources**:
```gn
sources = [
  "${screenlock_mgr_path}/frameworks/native/src/screenlock_callback_stub.cpp",
  "${screenlock_mgr_path}/frameworks/native/src/screenlock_inner_listener_stub.cpp",
  "${screenlock_mgr_path}/frameworks/native/src/screenlock_inner_listener_wapper.cpp",
  "${screenlock_mgr_path}/frameworks/native/src/screenlock_manager.cpp",
  "${screenlock_mgr_path}/frameworks/native/src/screenlock_manager_proxy.cpp",
  "${screenlock_mgr_path}/frameworks/native/src/screenlock_system_ability_stub.cpp",
]
```

**external_deps**:
```gn
external_deps = [
  "c_utils:utils",
  "eventhandler:libeventhandler",
  "hilog:libhilog",
  "hitrace:hitrace_meter",
  "ipc:ipc_single",
  "samgr:samgr_proxy",
]
```

**innerapi_tags**:
```gn
innerapi_tags = [
  "platformsdk",
  "sasdk",
]
```

**version_script**:
```gn
version_script = "screenlock_client.versionscript"
```

---

#### 2.5.2 screenlock_client_static (静态库)

| 属性 | 值 |
|------|-----|
| 类型 | ohos_static_library |
| 输出 | libscreenlock_client_static.a |

---

### 2.6 ETS/ANI (frameworks/ets/ani/BUILD.gn)

| Target | 类型 | 说明 |
|--------|------|------|
| `screenlock_ani` | ohos_shared_library | ANI 动态库 |
| `screenLock_static_etc` | ohos_static_library | 静态库 (测试用) |

---

### 2.7 Watch 模块 (watch/BUILD.gn)

| Target | 类型 | 说明 |
|--------|------|------|
| `watch_screenlock_static` | ohos_static_library | 可穿戴设备支持 (可选) |

**条件编译**:
```gn
if (screenlock_mgr_wearable_enable_payment_app == true) {
  defines = [ "SUPPORT_WEAR_PAYMENT_APP" ]
  deps = [ "${screenlock_mgr_path}/watch:watch_screenlock_static" ]
}
```

---

## 3. 编译产物清单

### 3.1 产物列表

| 产物 | 路径 | 安装位置 | 说明 |
|------|------|----------|------|
| `libscreenlock_server.z.so` | out/.../miscservices/screenlock_native | system/lib/ | SA 服务库 |
| `libscreenlock_client.z.so` | out/.../miscservices/screenlock_native | system/lib/ | IPC 客户端库 |
| `libscreenlock.so` | out/.../miscservices/screenlock_native | system/lib/module/ | N-API 库 |
| `libscreenlock_utils.z.so` | out/.../miscservices/screenlock_native | system/lib/ | 工具库 |
| `libscreenlockability.z.so` | out/.../miscservices/screenlock_native | system/lib/module/app/ | SA 能力库 |
| `screenlock.cfg` | out/... | system/init/ | 服务配置 |

> **证据来源**: README.md + bundle.json

---

## 4. 运行时加载关系

```
应用进程:
┌──────────────────────────────────────────────────────────┐
│  libace_napi.z.so (ACE N-API 运行时)                      │
│       │                                                  │
│       ▼                                                  │
│  libscreenlock.so (N-API 实现)                           │
│       │                                                  │
│       ▼                                                  │
│  libscreenlock_client.z.so (IPC Proxy)                  │
│       │                                                  │
│       ▼                                                  │
│  Binder IPC ──────────────────────────────────────────── │
                   │                                        │
                   ▼                                        │
Foundation 进程:                                           │
┌──────────────────────────────────────────────────────────┐
│  libscreenlock_server.z.so (SA 实现)                     │
│       │                                                  │
│       ├──> libscreenlockability.z.so                     │
│       │                                                  │
│       ├──> libscreenlock_utils.z.so                      │
│       │                                                  │
│       └──> 外部依赖 (ipc, safwk, samgr, 等)              │
└──────────────────────────────────────────────────────────┘
```

---

## 5. 构建配置

### 5.1 bundle.json 配置

```json
{
  "component": {
    "name": "screenlock_mgr",
    "subsystem": "theme",
    "syscap": ["SystemCapability.MiscServices.ScreenLock"],
    "features": [
      "screenlock_mgr_so_crop",
      "screenlock_mgr_wearable_enable_payment_app"
    ],
    "build": {
      "group_type": {
        "base_group": [],
        "fwk_group": [
          "//base/theme/screenlock_mgr/interfaces/inner_api:screenlock_client",
          "//base/theme/screenlock_mgr/frameworks/js/napi:screenlock",
          "//base/theme/screenlock_mgr/frameworks/ets/ani:screenlock_ani",
          "//base/theme/screenlock_mgr/frameworks/ets/ani:screenLock_static_etc"
        ],
        "service_group": [
          "//base/theme/screenlock_mgr:screenlock_mgr_packages"
        ]
      },
      "inner_kits": [{
        "name": "//base/theme/screenlock_mgr/interfaces/inner_api:screenlock_client",
        "header": {
          "header_files": ["screenlock_manager.h"],
          "header_base": "//base/theme/screenlock_mgr/interfaces/inner_api/include"
        }
      }]
    }
  }
}
```

---

## 6. 编译命令

### 6.1 标准编译

```bash
./build.sh --product-name <产品名> --build-target screenlock_native
```

### 6.2 启用可穿戴支付应用

```bash
./build.sh --product-name <产品名> \
  --build-target screenlock_native \
  --gn-args screenlock_mgr_wearable_enable_payment_app=true
```

### 6.3 启用代码裁剪

```bash
./build.sh --product-name <产品名> \
  --build-target screenlock_native \
  --gn-args screenlock_mgr_so_crop=true
```

---

## 7. 安全编译选项

所有动态库均启用以下安全选项:

| 选项 | 说明 |
|------|------|
| `-fdata-sections` | 数据段分离 |
| `-ffunction-sections` | 函数段分离 |
| `-fvisibility=hidden` | 隐藏符号 |
| `-fvisibility-inlines-hidden` | 内联函数符号隐藏 |
| CFI (Control Flow Integrity) | 控制流完整性保护 |
| PAC_RET | 指针认证码返回地址保护 |

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [02_Architecture](02_Architecture.md) | 系统架构 |
| [04_Security_Review](04_Security_Review.md) | 安全评审 |
