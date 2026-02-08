# 构建配置

## 1. GN 构建概述

user_auth_framework 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统。

**关键配置文件**:
- `user_auth_framework.gni` - 全局变量定义
- `bundle.json` - 组件配置
- `*/BUILD.gn` - 各模块构建配置

> **证据**: `bundle.json` build 配置

---

## 2. 全局配置 (user_auth_framework.gni)

**文件**: `user_auth_framework.gni`

```gn
declare_args() {
  user_auth_framework_enable_dynamic_load = false
  user_auth_framework_path = "//base/useriam/user_auth_framework"
  screenlock_client_enable = true
  user_auth_framework_has_ext = false
}
```

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `user_auth_framework_enable_dynamic_load` | false | 是否启用动态加载模式 |
| `user_auth_framework_path` | - | 模块路径 |
| `screenlock_client_enable` | true | 是否启用锁屏客户端 |
| `user_auth_framework_has_ext` | false | 是否有扩展部分 |

---

## 3. 组件配置 (bundle.json)

**文件**: `bundle.json`

```json
{
  "component": {
    "name": "user_auth_framework",
    "subsystem": "useriam",
    "adapted_system_type": ["standard"],
    "syscap": ["SystemCapability.UserIAM.UserAuth.Core"],
    "features": [
      "user_auth_framework_enabled",
      "user_auth_framework_enable_dynamic_load"
    ],
    "build": {
      "group_type": {
        "base_group": [],
        "fwk_group": [...],
        "service_group": [...]
      },
      "inner_kits": [...],
      "test": [...]
    }
  }
}
```

---

## 4. 关键 Targets

### 4.1 服务层 Targets

| Target | 类型 | 路径 | 产物 |
|--------|------|------|------|
| `userauthservice` | ohos_shared_library | `services:BUILD.gn` | `libuserauthservice.so` |
| `userauth_services_ipc` | ohos_source_set | `services/ipc:BUILD.gn` | 链接入服务 |
| `userauth_service_core` | ohos_source_set | `services/core:BUILD.gn` | 链接入服务 |
| `userauth_service_context` | ohos_source_set | `services/context:BUILD.gn` | 链接入服务 |
| `userauth_service_base` | ohos_source_set | `services/base:BUILD.gn` | 链接入服务 |
| `userauth_service_load_mode` | ohos_source_set | `services/load_mode:BUILD.gn` | 条件链接 |
| `userauth_service_remote_connect` | ohos_source_set | `services/remote_connect:BUILD.gn` | 条件链接 |

### 4.2 框架层 Targets

| Target | 类型 | 路径 | 产物 |
|--------|------|------|------|
| `userauth` | ohos_shared_library | `frameworks/js/napi/user_auth:BUILD.gn` | `module/useriam/libuserauth.so` |
| `useraccessctrl` | ohos_shared_library | `frameworks/js/napi/user_access_ctrl:BUILD.gn` | `module/useriam/libuseraccessctrl.so` |
| `userauthicon` | ohos_shared_library | `frameworks/js/napi/user_auth_icon:BUILD.gn` | - |
| `userauthextensionability_napi` | ohos_shared_library | `frameworks/js/napi/user_auth_extension/user_auth_extension:BUILD.gn` | - |
| `userauthextensionmodule` | ohos_shared_library | `frameworks/js/napi/user_auth_extension/module_loader:BUILD.gn` | - |
| `user_auth_framework_ani` | group | `frameworks/ets/ani:BUILD.gn` | - |

### 4.3 Native 库 Targets

| Target | 类型 | 路径 | 产物 |
|--------|------|------|------|
| `userauth_client` | ohos_shared_library | `frameworks/native/client:BUILD.gn` | `libuserauth_client.so` |
| `userauth_executors` | ohos_shared_library | `frameworks/native/executors:BUILD.gn` | `libuserauth_executors.so` |
| `userauth_client_ipc` | ohos_source_set | `frameworks/native/ipc:BUILD.gn` | 链接入 client |
| `userauth_service_ipc` | ohos_source_set | `frameworks/native/ipc:BUILD.gn` | 链接入服务 |
| `attributes` | ohos_source_set | `frameworks/native/common:BUILD.gn` | 链接入 |
| `dfx` | ohos_source_set | `frameworks/native/common:BUILD.gn` | 链接入 |
| `iam_utils` | ohos_source_set | `common:BUILD.gn` | 链接入 |

### 4.4 ANI/ETS Targets

| Target | 类型 | 路径 | 产物 |
|--------|------|------|------|
| `user_auth` | generate_static_abc | `frameworks/ets/ani/user_auth:BUILD.gn` | `system/framework/user_auth.abc` |
| `user_access_ctrl_ani` | group | `frameworks/ets/ani/user_access_ctrl:BUILD.gn` | - |
| `user_auth_ani` | group | `frameworks/ets/ani/user_auth:BUILD.gn` | - |
| `userauth_ani` | taihe_shared_library | `frameworks/ets/ani/user_auth:BUILD.gn` | `libuserauth_ani.so` |
| `cj_userauth_ffi` | ohos_shared_library | `frameworks/cj/user_auth:BUILD.gn` | `libcj_userauth_ffi.so` |

---

## 5. 产物清单

### 5.1 动态库 (.so)

| 产物 | 路径 | 大小 | 说明 |
|------|------|------|------|
| `libuserauthservice.so` | - | ~3380KB | 主服务 |
| `libuserauth.so` | `module/useriam/` | - | N-API |
| `libuseraccessctrl.so` | `module/useriam/` | - | 访问控制 |
| `libuserauth_client.so` | - | - | Native Client |
| `libuserauth_executors.so` | - | - | 执行器框架 |
| `libuserauth_ani.so` | - | - | ANI 库 |
| `libcj_userauth_ffi.so` | - | - | Cangjie FFI |

### 5.2 静态库 (.a)

| 产物 | 链接位置 | 说明 |
|------|----------|------|
| `userauth_service_core` | libuserauthservice.so | 核心业务逻辑 |
| `userauth_service_context` | libuserauthservice.so | 上下文管理 |
| `userauth_service_base` | libuserauthservice.so | 基础服务 |
| `userauth_services_ipc` | libuserauthservice.so | IPC 层 |

### 5.3 字节码 (.abc)

| 产物 | 路径 | 说明 |
|------|------|------|
| `user_auth.abc` | `system/framework/` | ArkTS API |
| `user_access_ctrl.abc` | `system/framework/` | 访问控制 API |
| `user_auth_extension.abc` | `system/framework/` | 扩展能力 API |
| `user_auth_icon.abc` | `system/framework/` | 图标组件 |

### 5.4 配置文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `901.json` | `sa_profile/default/` | UserAuthService SA 配置 |
| `921.json` | `sa_profile/default/` | UserIdmService SA 配置 |
| `931.json` | `sa_profile/default/` | CoAuthService SA 配置 |
| `useriam.cfg` | `sa_profile/default/` | Init 配置 |
| `useriam.para` | `param/` | 系统参数 |

---

## 6. 依赖关系

### 6.1 服务依赖

```
userauthservice
├── userauth_services_ipc
│   ├── userauth_service_ipc (IDL 生成)
│   ├── userauth_service_context
│   │   ├── userauth_service_core
│   │   │   ├── userauth_client
│   │   │   ├── attributes
│   │   │   └── dfx
│   │   └── userauth_service_load_mode
│   └── userauth_service_remote_connect
├── common:iam_utils
└── external libs...
```

### 6.2 N-API 依赖

```
userauth (JS N-API)
├── userauth_client
├── common:user_auth_common
└── external libs (napi, ipc, hilog, etc.)
```

---

## 7. 条件编译

### 7.1 动态加载模式

当 `user_auth_framework_enable_dynamic_load = true` 时:

- 使用 `services/remote_connect/dynamic/` 源码
- 启用 `ENABLE_DYNAMIC_LOAD` 定义
- SA profiles 使用 `dynamic_load/` 目录

### 7.2 锁屏客户端

当 `screenlock_client_enable = true` 时:

- 依赖 `screenlock_mgr`, `eventhandler`, `ffrt`, `preferences`, `window_manager`

---

## 8. 编译命令

### 8.1 完整构建

```bash
hb build -f
```

### 8.2 模块构建

```bash
# 构建单个模块
hb build //base/useriam/user_auth_framework/services:userauthservice

# 构建 N-API
hb build //base/useriam/user_auth_framework/frameworks/js/napi/user_auth:userauth
```

### 8.3 清理构建

```bash
hb build -c clean
```

---

## 9. 构建产物验证

### 9.1 输出目录结构

```
out/ohos-arm-release/
├── system/
│   ├── lib/
│   │   ├── libuserauthservice.so
│   │   ├── libuserauth_client.so
│   │   └── libuserauth_executors.so
│   └── framework/
│       ├── user_auth.abc
│       └── user_access_ctrl.abc
├── system/etc/
│   └── sa/
│       ├── 901.json
│       ├── 921.json
│       └── 931.json
└── preload/
    └── ...
```

---

## 10. 相关文档

- [架构说明](01_Architecture.md)
- [N-API 接口](02_NAPI.md)
- [附录: 配置开关](appendix/Config_Flags.md)
