# OAID 构建与产物

## 构建系统概览

OAID 项目使用 **GN (Generate Ninja)** 构建系统，这是 OpenHarmony 的标准构建工具。

```
┌─────────────────────────────────────────────────────────────┐
│                    构建流程                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  BUILD.gn ──► GN 生成 ──► build.ninja ──► Ninja 编译 ──► 产物 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## GN 目标清单

### 根目标

**文件**: `BUILD.gn`

```gn
# 根目标：oaid_native_packages
group("oaid_native_packages") {
  deps = [
    "etc/init:oaidservice.cfg",                    # 启动配置
    "interfaces/innerkits:oaid_client",            # 客户端库
    "interfaces/kits/js/napi/oaid:oaid",          # N-API 模块
    "profile:cloud_oaid_sa_profiles",              # SA Profile
    "services:oaid_service",                       # 服务主体
    "services:oaid_service_config_json",           # 配置文件
  ]
}
```

### 模块目标详解

#### 1. 服务模块

**文件**: `services/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `oaid_service` | `ohos_shared_library` | `liboaid_service.z.so` | 服务动态库 |
| `oaid_service_config_json` | `ohos_prebuilt_etc` | `oaid_service_config.json` | 配置文件 |

**关键配置**:
```gn
ohos_shared_library("oaid_service") {
  sources = [
    "oaid_manager/src/oaid_service.cpp",
    "oaid_manager/src/oaid_service_stub.cpp",
    # ... 其他源文件
  ]
  
  deps = [
    "//foundation/ability/ability_runtime:ability_manager",
    "//foundation/ability/ability_runtime:ability_connect",
    "//foundation/distributeddatamgr/kv_store:distributedkvstore",
    # ... 其他依赖
  ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_core",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "eventhandler:libeventhandler",
  ]
}
```

#### 2. 内部接口模块

**文件**: `interfaces/innerkits/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `oaid_client` | `ohos_shared_library` | `liboaid_client.z.so` | 客户端库 |

**导出头文件**:
```gn
ohos_shared_library("oaid_client") {
  sources = [
    "src/oaid_service_client.cpp",
    "src/oaid_service_proxy.cpp",
    "src/oaid_remote_config_observer_stub.cpp",
  ]
  
  # 导出给外部使用的头文件
  public_configs = [ "oaid_client_public_config" ]
}

config("oaid_client_public_config") {
  include_dirs = [ "include" ]
}
```

#### 3. N-API 模块

**文件**: `interfaces/kits/js/napi/oaid/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `oaid` | `ohos_shared_library` | `liboaid_napi.z.so` | N-API 动态库 |

**模块配置**:
```gn
ohos_shared_library("oaid") {
  sources = [
    "src/oaid.cpp",
    "src/oaid_init.cpp",
  ]
  
  deps = [
    "//domains/advertising/oaid/interfaces/innerkits:oaid_client",
  ]
  
  external_deps = [
    "napi:libnapi",
    "hilog:libhilog",
  ]
  
  # N-API 模块名
  relative_install_dir = "module/identifier"
}
```

#### 4. 工具模块

**文件**: `utils/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `oaid_utils` | `source_set` | - | 工具类源码集合 |

#### 5. 配置模块

**文件**: `etc/init/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `oaidservice.cfg` | `ohos_prebuilt_etc` | `oaidservice.cfg` | 服务启动配置 |

**启动配置内容**:
```json
{
    "uid": "oaid_service",
    "gid": ["oaid_service", "shell"],
    "sandbox": 0,
    "permission": [
        "ohos.permission.DISTRIBUTED_DATASYNC",
        "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
        "ohos.permission.PERMISSION_USED_STATS",
        "ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS"
    ],
    "secon": "u:r:oaid_service:s0"
}
```

#### 6. Profile 模块

**文件**: `profile/BUILD.gn`

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `cloud_oaid_sa_profiles` | `ohos_sa_profile` | `6101.json` | SA Profile |

**SA Profile 内容** (`6101.json`):
```json
{
    "process": "oaid_service",
    "name": "OAIDService",
    "systemability": [
        {
            "name": 6101,
            "libpath": "liboaid_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

---

## 编译产物

### 产物列表

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|---------|------|
| `liboaid_service.z.so` | 共享库 | `/system/lib/` | OAID 服务库 |
| `liboaid_client.z.so` | 共享库 | `/system/lib/` | 客户端库 |
| `liboaid_napi.z.so` | 共享库 | `/system/lib/module/identifier/` | N-API 模块 |
| `oaid_service_config.json` | 配置文件 | `/etc/advertising/oaid/` | 服务配置 |
| `oaidservice.cfg` | 启动配置 | `/system/etc/init/` | SA 启动配置 |
| `6101.json` | SA Profile | `/system/profile/` | SA 管理配置 |

### 产物依赖关系

```
liboaid_napi.z.so
    │
    ├──► liboaid_client.z.so
    │       │
    │       └──► libipc_core.so
    │       └──► libhilog.so
    │
    └──► libnapi.so
    └──► libhilog.so

liboaid_service.z.so
    │
    ├──► libability_manager.so
    ├──► libability_connect.so
    ├──► libdistributedkvstore.so
    ├──► libaccess_token.so
    ├──► libbundle_framework.so
    ├──► libhilog.so
    ├──► libipc_core.so
    ├──► libsystem_ability_fwk.so
    ├──► libsamgr_proxy.so
    └──► libeventhandler.so
```

---

## 构建配置选项

### Feature 开关

**文件**: `oaid.gni`

```gni
# OAID 功能开关
oaid_enable_fuzztest = true  # 启用模糊测试
oaid_enable_debug = false    # 调试模式
```

### 条件编译

```gn
# 根据 Feature 开关控制编译
if (oaid_enable_fuzztest) {
  deps += [ "//domains/advertising/oaid/test/fuzztest:fuzztest" ]
}
```

---

## 依赖关系

### 外部依赖

| 依赖组件 | 用途 | 路径 |
|---------|------|------|
| `ability_runtime` | Ability 管理 | `//foundation/ability/ability_runtime` |
| `access_token` | 权限管理 | `//foundation/ability/access_token` |
| `bundle_framework` | Bundle 管理 | `//foundation/ability/bundle_framework` |
| `kv_store` | 分布式存储 | `//foundation/distributeddatamgr/kv_store` |
| `ipc` | IPC 通信 | `//foundation/communication/ipc` |
| `safwk` | SA 框架 | `//foundation/systemabilitymgr/safwk` |
| `samgr` | SA 管理 | `//foundation/systemabilitymgr/samgr` |
| `hilog` | 日志系统 | `//foundation/distributedschedule/hilog` |
| `c_utils` | C++ 工具 | `//foundation/distributedschedule/c_utils` |
| `eventhandler` | 事件处理 | `//foundation/distributedschedule/eventhandler` |
| `config_policy` | 配置策略 | `//foundation/distributedschedule/config_policy` |

### Bundle 依赖

**文件**: `bundle.json`

```json
{
  "deps": {
    "components": [
      "ability_runtime",
      "access_token",
      "bundle_framework",
      "cJSON",
      "c_utils",
      "config_policy",
      "hilog",
      "kv_store",
      "ipc",
      "napi",
      "safwk",
      "samgr",
      "eventhandler",
      "libuv",
      "openssl"
    ]
  }
}
```

---

## 版本信息

### 版本脚本

**文件**: `interfaces/innerkits/liboaidclient.versionscript`

```ld
{
  global:
    # 导出的符号
    _ZN4OHOS5Cloud17OAIDServiceClient*;  # OAIDServiceClient 相关符号
    _ZN4OHOS5Cloud17OAIDServiceProxy*;   # OAIDServiceProxy 相关符号
  local:
    *;  # 其他符号私有
}
```

### 版本号

**文件**: `bundle.json`

```json
{
  "version": "3.2",
  "component": {
    "name": "oaid",
    "subsystem": "advertising",
    "rom": "300KB",
    "ram": "1024KB"
  }
}
```

---

## 构建命令

### 全量构建

```bash
# 构建 OAID 模块
gn gen out --args="target_os=\"ohos\""
ninja -C out domains/advertising/oaid:oaid_native_packages
```

### 单独构建

```bash
# 构建服务
ninja -C out domains/advertising/oaid/services:oaid_service

# 构建客户端
ninja -C out domains/advertising/oaid/interfaces/innerkits:oaid_client

# 构建 N-API
ninja -C out domains/advertising/oaid/interfaces/kits/js/napi/oaid:oaid
```

### 测试构建

```bash
# 构建模糊测试
ninja -C out domains/advertising/oaid/test/fuzztest:fuzztest
```

---

## 安装路径

```
/system/
├── lib/
│   ├── liboaid_service.z.so
│   ├── liboaid_client.z.so
│   └── module/
│       └── identifier/
│           └── liboaid_napi.z.so
├── etc/
│   └── init/
│       └── oaidservice.cfg
├── profile/
│   └── 6101.json
└── bin/
    └── oaid_service (如为可执行文件)

/etc/advertising/oaid/
└── oaid_service_config.json
```

---

## 相关文档

- [项目概览](01_Overview.md)
- [代码地图](03_CodeMap.md)
- [内部实现](08_Internals.md)
