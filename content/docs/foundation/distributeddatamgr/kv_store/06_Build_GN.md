# GN 构建指南

## 概述

KV Store 使用 GN (Generate Ninja) 作为构建系统，配置文件分布在多个目录层级。

## 根构建文件

### BUILD.gn

**路径**：`//foundation/distributeddatamgr/kv_store/BUILD.gn`

**证据**：`BUILD.gn` 根构建文件定义了测试组

```gn
group("distributedtest") {
  testonly = true
  deps = [ "test/distributedtest/single_kvstore_client:distributedtest" ]
}

group("build_native_test") {
  testonly = true
  deps = [
    "frameworks/innerkitsimpl/distributeddatafwk/test:unittest",
    "frameworks/libs/distributeddb/test:unittest",
  ]
}
```

### kv_store.gni

**路径**：`//foundation/distributeddatamgr/kv_store/kv_store.gni`

**证据**：`kv_store.gni` 定义了路径变量和条件编译开关

```gn
kv_store_base_path = "//foundation/distributeddatamgr/kv_store"
kv_store_native_path = "${kv_store_base_path}/frameworks/native"
kv_store_api_path = "${kv_store_base_path}/interfaces/inner_api"

declare_args() {
  if (device_company != "qemu") {
    qemu_disable = true
  } else {
    qemu_disable = false
  }
}
```

## 组件配置

### bundle.json

**路径**：`//foundation/distributeddatamgr/kv_store/bundle.json`

**证据**：`bundle.json` 定义了组件元数据

```json
{
  "name": "@ohos/kv_store",
  "version": "3.1.0",
  "component": {
    "name": "kv_store",
    "subsystem": "distributeddatamgr",
    "syscap": [
      "SystemCapability.DistributedDataManager.KVStore.Core",
      "SystemCapability.DistributedDataManager.KVStore.DistributedKVStore"
    ],
    "features": [
      "kv_store_cloud",
      "kv_store_device"
    ],
    "adapted_system_type": ["standard"]
  }
}
```

## Targets 清单

### 核心库 Targets

| Target | 类型 | 路径 | 输出 | 职责 |
|--------|------|------|------|------|
| `distributeddb` | shared_library | `frameworks/libs/distributeddb/` | `libdistributeddb.z.so` | 分布式数据库核心 |
| `distributeddb_client` | shared_library | `frameworks/libs/distributeddb/` | `libdistributeddb_client.z.so` | 客户端库 |
| `gaussdb_rd` | static_library | `frameworks/libs/distributeddb/gaussdb_rd/` | `libgaussdb_rd.a` | GaussDB 读副本 |
| `customtokenizer` | static_library | `frameworks/libs/distributeddb/` | `libcustomtokenizer.a` | 自定义分词器 |

### 接口 Targets

| Target | 类型 | 路径 | 输出 | 职责 |
|--------|------|------|------|------|
| `distributeddata` | headers | `interfaces/jskits/distributeddata/` | 头文件 | JS API 声明 |
| `distributedkvstore` | headers | `interfaces/jskits/distributedkvstore/` | 头文件 | KV Store JS API |
| `distributeddata_inner` | headers | `interfaces/innerkits/distributeddata/` | 头文件 | Inner API 声明 |
| `distributeddata_client` | headers | `interfaces/innerkits/distributeddata/` | 头文件 | 客户端 Inner API |
| `distributeddb` | headers | `interfaces/innerkits/distributeddata/` | 头文件 | DB Inner API |
| `distributeddata_mgr` | headers | `interfaces/innerkits/distributeddatamgr/` | 头文件 | 管理器 Inner API |

### Native Targets

| Target | 类型 | 路径 | 输出 | 职责 |
|--------|------|------|------|------|
| `dbm_kv_store` | static_library | `frameworks/native/dbm_kv_store/` | `libdbm_kv_store.a` | DBM 风格 KV |
| `kv_store` | static_library | `frameworks/native/kv_store/` | `libkv_store.a` | Native KV |

### ETS/ArkTS Targets

| Target | 类型 | 路径 | 输出 | 职责 |
|--------|------|------|------|------|
| `distributedkvstore_ani_pack` | - | `frameworks/ets/taihe/kv_store/` | - | ArkTS 接口包 |

## 关键 Targets 详解

### distributeddb (核心库)

**路径**：`frameworks/libs/distributeddb/BUILD.gn`

**证据**：`BUILD.gn:98-140`

```gn
ohos_shared_library("distributeddb") {
  sources = distributeddb_src

  configs = [ ":distrdb_config" ]
  public_configs = [ ":distrdb_public_config" ]

  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "jsoncpp:jsoncpp",
    "zlib:shared_libz",
  ]

  public_external_deps = [
    "openssl:libcrypto_shared",
    "sqlite:sqlite",
  ]

  subsystem_name = "distributeddatamgr"
  part_name = "kv_store"
}
```

**编译配置** (`distrdb_config`)：

```gn
config("distrdb_config") {
  defines = [
    "_LARGEFILE64_SOURCE",
    "_FILE_OFFSET_BITS=64",
    "SQLITE_HAS_CODEC",
    "SQLITE_ENABLE_JSON1",
    "USING_HILOG_LOGGER",
    "USE_SQLITE_SYMBOLS",
    // ...
  ]
}
```

**依赖关系**：

```
distributeddb
    ├── gaussdb_rd:gaussdb_rd (静态库)
    ├── openssl:libcrypto_shared (动态库)
    ├── sqlite:sqlite (动态库)
    └── c_utils:utils (动态库)
```

### jskits Targets

#### distributedkvstore (JS API)

**路径**：`interfaces/jskits/distributedkvstore/BUILD.gn`

```gn
ohos_js_ptest("distributedkvstore") {
  part_name = "kv_store"
  subsystem_name = "distributeddatamgr"

  source_out_dir = "default"

  deps = [
    "//foundation/distributeddatamgr/kv_store/frameworks/jskitsimpl/distributedkvstore:js_kv_store",
  ]
}
```

**实现依赖**：`frameworks/jskitsimpl/distributedkvstore/BUILD.gn`

```gn
ohos_shared_library("js_kv_store") {
  sources = [
    "src/entry_point.cpp",
    "src/js_single_kv_store.cpp",
    "src/js_device_kv_store.cpp",
    // ...
  ]

  external_deps = [
    "hilog:libhilog",
    "napi:napi_suffix",
    "//foundation/distributeddatamgr/kv_store/frameworks/libs/distributeddb:distributeddb",
  ]
}
```

## 编译产物

### 产物清单

| 产物 | 路径 | 类型 | 加载方式 |
|-----|------|------|---------|
| `libdistributeddb.z.so` | `out/.../libs/` | 动态库 | 运行时加载 |
| `libdistributeddb_client.z.so` | `out/.../libs/` | 动态库 | 运行时加载 |
| `libgaussdb_rd.a` | `out/.../objs/` | 静态库 | 链接时链接 |
| `libcustomtokenizer.a` | `out/.../objs/` | 静态库 | 链接时链接 |
| `libdbm_kv_store.a` | `out/.../objs/` | 静态库 | 链接时链接 |
| `libkv_store.a` | `out/.../objs/` | 静态库 | 链接时链接 |

### 安装路径

| 产物 | 安装路径 |
|-----|---------|
| 动态库 | `/system/lib/` |
| 头文件 | `/usr/include/` |

### 运行时加载关系

```
应用进程
    │
    ├── libhilog.so (日志)
    ├── libnapi.so (N-API 运行时)
    │
    └── libdistributeddb.z.so (KV 核心库)
        │
        ├── libsqlite.z.so (SQLite)
        └── libcrypto.so (OpenSSL)
```

## 条件编译

### kv_store_cloud

**位置**：`BUILD.gn:69`

```gn
if (kv_store_cloud) {
  defines += [ "USE_DISTRIBUTEDDB_CLOUD" ]
}
```

### kv_store_device

**位置**：`BUILD.gn:72`

```gn
if (kv_store_device) {
  defines += [ "USE_DISTRIBUTEDDB_DEVICE" ]
}
```

## 相关文档

- [代码地图](03_CodeMap.md) - 代码文件导航
- [N-API 接口参考](04_NAPI_Reference.md)
- [Inner API 参考](05_Inner_API.md)
- [架构设计](02_Architecture.md)
