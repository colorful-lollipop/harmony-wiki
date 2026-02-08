# 04_GN_Targets - 构建目标与编译产物

> GN目标梳理、产物类型、依赖关系与安装路径

---

## 1. 构建文件总览

### 1.1 BUILD.gn文件清单

| 路径 | 类型 | 说明 |
|------|------|------|
| `common/BUILD.gn` | 生产 | 公共库 |
| `services/BUILD.gn` | 生产 | 服务库 |
| `sa_profile/BUILD.gn` | 生产 | SA配置文件 |
| `param/BUILD.gn` | 生产 | 系统参数 |
| `frameworks/js/napi/BUILD.gn` | 生产 | N-API模块 |
| `frameworks/ets/ani/BUILD.gn` | 生产 | ANI模块 |
| `frameworks/native/client/BUILD.gn` | 生产 | Native客户端 |
| `frameworks/native/ipc/BUILD.gn` | 生产 | IPC接口 |
| `services/external_adapters/security_command_adapter/BUILD.gn` | 生产 | Rust安全核心 |

### 1.2 配置gni文件

| 路径 | 说明 |
|------|------|
| `companion_device_auth.gni` | 主配置，feature flags，sanitizer设置 |
| `services/BUILD.gni` | 服务源文件列表，依赖定义 |
| `frameworks/native/client/BUILD.gni` | 客户端配置 |

---

## 2. 生产目标详解

### 2.1 companion_device_auth_common

**类型**: ohos_static_library

**路径**: `common/BUILD.gn`

**源文件**:
- `common/src/iam_para2str.cpp`

**产物**: `libcompanion_device_auth_common.a`

**用途**: 公共工具函数，日志格式转换

---

### 2.2 companiondeviceauthservice

**类型**: ohos_shared_library

**路径**: `services/BUILD.gn`

**关键属性**:
```gn
ohos_shared_library("companiondeviceauthservice") {
    sanitize = companion_device_auth_sanitize
    branch_protector_ret = "pac_ret"
    remove_configs = [ "//build/config/compiler:no_exceptions" ]
    
    sources = companion_device_auth_service_sources  # 81个cpp文件
    configs = [ ":service_config" ]
    deps = companion_device_auth_service_deps
    external_deps = companion_device_auth_service_external_deps
    
    version_script = "companion_device_auth_service_map"
}
```

**产物**: `libcompaniondeviceauthservice.z.so`

**安装路径**: `/system/lib/`

**依赖**:
- `//common:companion_device_auth_common`
- `//frameworks/native/ipc:companion_device_auth_stub`
- `//services/external_adapters/security_command_adapter:companiondeviceauthservice_rs`
- `//services/external_adapters/security_command_adapter:companiondeviceauthservice_rs_cpp_if`

**外部依赖**:
- access_token: libaccesstoken_sdk, libtokenid_sdk
- c_utils: utils
- hilog: libhilog
- hicollie: libhicollie
- ipc: ipc_core, ipc_single
- json: nlohmann_json_static
- safwk: system_ability_fwk
- samgr: samgr_proxy

---

### 2.3 companiondeviceauth (N-API)

**类型**: ohos_shared_library

**路径**: `frameworks/js/napi/BUILD.gn`

**关键属性**:
```gn
ohos_shared_library("companiondeviceauth") {
    sanitize = companion_device_auth_sanitize
    branch_protector_ret = "pac_ret"
    
    sources = [
        "src/companion_device_auth_entry.cpp",
        "src/companion_device_auth_napi_helper.cpp",
        "src/companion_device_auth_napi_impl.cpp",
        "src/napi_*_callback.cpp",
        "src/status_monitor.cpp",
    ]
    
    relative_install_dir = "module/useriam"
}
```

**产物**: `libcompaniondeviceauth.so`

**安装路径**: `/system/lib/module/useriam/`

**依赖**:
- `//frameworks/native/client:companion_device_auth_client`

---

### 2.4 companion_device_auth_client

**类型**: ohos_shared_library

**路径**: `frameworks/native/client/BUILD.gn`

**产物**: `libcompanion_device_auth_client.so`

**安装路径**: `/system/lib/`

**依赖**:
- `//frameworks/native/ipc:companion_device_auth_proxy`
- `//common:companion_device_auth_common`

**innerapi_tags**: ["platformsdk"]

---

### 2.5 companion_device_auth_ipc_interface

**类型**: idl_gen_interface

**路径**: `frameworks/native/ipc/BUILD.gn`

**源文件**:
- `idl/ICompanionDeviceAuth.idl`
- `idl/CompanionDeviceAuthTypes.idl`
- `idl/IIpc*Callback.idl`

**生成代码**:
- Proxy类 (客户端代理)
- Stub类 (服务端存根)
- Types类 (数据结构)

---

### 2.6 companiondeviceauthservice_rs

**类型**: ohos_rust_shared_ffi

**路径**: `services/external_adapters/security_command_adapter/BUILD.gn`

**关键属性**:
```gn
ohos_rust_shared_ffi("companiondeviceauthservice_rs") {
    sources = companion_device_auth_rust_service_relative_sources
    crate_name = "companiondeviceauthservice_rs"
}
```

**产物**: `libcompaniondeviceauthservice_rs.so`

**Rust源文件**: 86个.rs文件

**目录结构**:
```
rust/
├── lib.rs                      # 库入口
├── entry/                      # FFI入口
├── commands/                   # 命令解析
├── common/                     # 公共数据结构
├── impls/                      # 功能实现
├── jobs/                       # 任务机制
├── request/                    # 请求处理
├── traits/                     # 接口定义
└── utils/                      # 工具类
```

---

## 3. 产物清单

### 3.1 库文件

| 产物名 | 类型 | 安装路径 | 说明 |
|--------|------|----------|------|
| libcompaniondeviceauthservice.z.so | shared | /system/lib/ | 主服务库 |
| libcompanion_device_auth_client.so | shared | /system/lib/ | Native客户端 |
| libcompaniondeviceauth.so | shared | /system/lib/module/useriam/ | N-API模块 |
| libcompaniondeviceauth_ani.so | shared | /system/lib/ | ANI模块 |
| libcompaniondeviceauthservice_rs.so | shared | /system/lib/ | Rust安全核心 |
| libcompanion_device_auth_common.a | static | - | 公共库 |

### 3.2 配置文件

| 产物名 | 安装路径 | 说明 |
|--------|----------|------|
| 945.json | /system/profile/ | SA配置 |
| companiondeviceauth.cfg | /system/etc/init/ | 服务启动配置 |
| companion_device_auth.para | /system/param/ | 系统参数 |
| companion_device_auth.para.dac | /system/param/ | 参数权限 |

---

## 4. 条件编译特性

### 4.1 Feature Flags

从 `companion_device_auth.gni`:

| Flag | 默认值 | 说明 |
|------|--------|------|
| companion_device_auth_enable_auth_state_maintain_simulation | true | 认证状态模拟 |
| companion_device_auth_deploy_mode | "standard" | 部署模式 (standard/ap) |
| companion_device_auth_enable_extension | false | 扩展模式 (TEE支持) |
| companion_device_auth_enable_coverage | false | 覆盖率检测 |

### 4.2 可选依赖特性

从 `services/BUILD.gni`:

| 特性 | 条件 | 添加内容 |
|------|------|----------|
| HAS_USER_AUTH_FRAMEWORK | 有user_auth_framework部件 | fwk_comm适配代码 |
| HAS_SOFT_BUS_CHANNEL | 有dsoftbus部件 | SoftBus通道代码 |
| Account OS | 有os_account部件 | 真实用户ID管理 |

---

## 5. Sanitizer配置

从 `companion_device_auth.gni:34-42`:

```gn
companion_device_auth_sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
    blocklist = "${companion_device_auth_path}/cfi_blocklist.txt"
}
```

**安全特性**:
- 整数溢出检测
- 未定义行为检测
- 边界检测
- 控制流完整性(CFI)
- 跨DSO CFI

---

## 6. 构建命令

### 6.1 完整构建

```bash
# 构建所有目标
hb build //base/useriam/companion_device_auth/...
```

### 6.2 单独构建

```bash
# 构建服务
hb build //base/useriam/companion_device_auth/services:companiondeviceauthservice

# 构建N-API
hb build //base/useriam/companion_device_auth/frameworks/js/napi:companiondeviceauth

# 构建客户端
hb build //base/useriam/companion_device_auth/frameworks/native/client:companion_device_auth_client
```

### 6.3 测试构建

```bash
# 单元测试
hb build //base/useriam/companion_device_auth/test/unittest:companion_device_auth_unittest

# 模块测试
hb build //base/useriam/companion_device_auth/test/moduletest:companion_device_auth_moduletest
```

---

## 7. 依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│ companiondeviceauth (N-API)                                  │
└───────────────────────┬─────────────────────────────────────┘
                        │ depends on
┌───────────────────────▼─────────────────────────────────────┐
│ companion_device_auth_client                                 │
└───────────────────────┬─────────────────────────────────────┘
                        │ depends on
        ┌───────────────┴───────────────┐
        ▼                               ▼
┌───────────────────┐           ┌───────────────────┐
│ companion_device_ │           │ companion_device_ │
│ auth_proxy        │           │ auth_common       │
└───────────────────┘           └───────────────────┘
        │                               ▲
        │ depends on                    │ depends on
        ▼                               │
┌───────────────────┐                   │
│ companion_device_ │───────────────────┘
│ auth_ipc_interface│
└───────────────────┘
        ▲
        │ IPC
        ▼
┌─────────────────────────────────────────────────────────────┐
│ companiondeviceauthservice                                   │
├─────────────────────────────────────────────────────────────┤
│ ├── depends on: companion_device_auth_stub                 │
│ ├── depends on: companion_device_auth_common               │
│ └── depends on: companiondeviceauthservice_rs              │
└─────────────────────────────────────────────────────────────┘
```

---

*文档生成时间: 2025-02-06*
