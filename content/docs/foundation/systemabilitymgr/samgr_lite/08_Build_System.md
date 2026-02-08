# GN 构建系统

## 概述

samgr_lite 使用 GN（Generate Ninja）作为构建系统，支持 M-core 和 A-core 两种平台的编译。

## 配置文件

### config.gni

根目录配置文件，定义全局构建参数。

**证据位置**: `config.gni:14-21`

```gn
declare_args() {
  # 共享任务栈大小配置
  config_ohos_systemabilitymgr_samgr_lite_shared_task_size = 2048

  # 启用 mini 系统 RPC
  enable_ohos_systemabilitymgr_samgr_lite_rpc_mini = false
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `config_ohos_systemabilitymgr_samgr_lite_shared_task_size` | 2048 | 共享任务栈大小，0 表示使用系统默认 |
| `enable_ohos_systemabilitymgr_samgr_lite_rpc_mini` | false | 是否为 mini 系统启用 RPC |

## 根 BUILD.gn

**证据位置**: `BUILD.gn:17-70`

### lite_component

```gn
lite_component("samgr") {
  features = [
    "samgr",                      # Samgr 核心
    "communication/broadcast",      # 广播服务
  ]

  # A-core 特有
  if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
    features += [
      "samgr_server:server",      # IPC 服务端
      "samgr_client:client",      # IPC 客户端
    ]
  }

  # Mini RPC 启用时
  if (enable_ohos_systemabilitymgr_samgr_lite_rpc_mini) {
    features += [
      "samgr_server:server",
      "samgr_client:client",
    ]
  }
}
```

### ndk_lib

```gn
ndk_lib("samgr_lite_ndk") {
  # M-core: 静态库
  if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
    lib_extension = ".a"
  } 
  # A-core: 动态库
  else if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
    lib_extension = ".so"
  }

  deps = [
    "communication/broadcast",
    "samgr",
  ]

  head_files = [
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/communication/broadcast",
  ]

  # A-core 特有头文件
  if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
    deps += [ "samgr_server:server" ]
    head_files += [ "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry" ]
  }
}
```

## samgr/BUILD.gn

**证据位置**: `samgr/BUILD.gn:37-94`

### M-core 构建（静态库）

```gn
if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
  static_library("samgr") {
    sources = [
      "registry/service_registry.c",
      "source/samgr_lite.c",
    ]

    public_configs = [ ":samgr_public" ]

    include_dirs = [
      "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite"
    ]

    public_deps = [
      "//foundation/systemabilitymgr/samgr_lite/samgr/adapter:samgr_adapter",
      "//foundation/systemabilitymgr/samgr_lite/samgr/source:samgr_source",
    ]

    # Mini RPC 依赖
    if (enable_ohos_systemabilitymgr_samgr_lite_rpc_mini) {
      defines += [ "MINI_SAMGR_LITE_RPC" ]
      public_deps += [
        "//foundation/communication/ipc/interfaces/innerkits/c/dbinder:dbinder",
      ]
    }
  }
}
```

### A-core 构建（动态库）

```gn
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  shared_library("samgr") {
    sources = [ "source/samgr_lite.c" ]

    cflags = [
      "-fPIC",     # 位置无关代码
      "-Wall",
    ]

    public_configs = [ ":samgr_public" ]

    include_dirs = [
      "//third_party/bounds_checking_function/include",
      "//third_party/cJSON",
      "//foundation/systemabilitymgr/samgr_lite/samgr_endpoint/source",
    ]

    public_deps = [
      "//build/lite/config/component/lite_component.gni",
      "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
      "//foundation/systemabilitymgr/samgr_lite/samgr/source:samgr_source",
      "//foundation/systemabilitymgr/samgr_lite/samgr_client:client",
    ]

    # Linux 特有依赖
    if (ohos_kernel_type == "linux") {
      deps = [ "//third_party/mksh" ]
      external_deps = [ "toybox:toybox" ]
    }
  }
}
```

## samgr/source/BUILD.gn

**证据位置**: `samgr/source/BUILD.gn:26-79`

### M-core 静态库

```gn
if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
  static_library("samgr_source") {
    sources = [
      "common.c",
      "feature.c",
      "iunknown.c",
      "message.c",
      "service.c",
      "task_manager.c",
    ]

    public_configs = [ ":samgr_source_public" ]

    public_deps = [
      "//foundation/systemabilitymgr/samgr_lite/samgr/adapter:samgr_adapter",
    ]
  }
}
```

### A-core 动态库

```gn
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  source_set("samgr_source") {
    sources = [
      "common.c",
      "feature.c",
      "iunknown.c",
      "message.c",
      "service.c",
      "task_manager.c",
    ]

    cflags = [
      "-fPIC",
      "-Wall",
    ]

    public_configs = [ ":samgr_source_public" ]

    public_deps = [
      "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
      "//foundation/systemabilitymgr/samgr_lite/samgr/adapter:samgr_adapter",
    ]
  }
}
```

## samgr_server/BUILD.gn

**证据位置**: `samgr_server/BUILD.gn:17-70`

```gn
if (!enable_ohos_systemabilitymgr_samgr_lite_rpc_mini) {
  # 标准 RPC
  shared_library("server") {
    sources = [ "source/samgr_server_rpc.c" ]

    cflags = [
      "-fPIC",
      "-Wall",
    ]

    include_dirs = [
      "../samgr_endpoint/source",
      "//base/security/permission_lite/interfaces/innerkits",
      "//base/security/permission_lite/services/pms_base/include",
      "//commonlibrary/utils_lite/include",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
      "//third_party/cJSON",
      "//third_party/bounds_checking_function/include",
    ]

    deps = [ "//foundation/systemabilitymgr/samgr_lite:ConfigFiles" ]

    public_deps = [
      "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
      "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
      "//foundation/systemabilitymgr/samgr_lite/samgr_endpoint:store_source",
      "//third_party/bounds_checking_function:libsec_shared",
    ]
  }
} else {
  # Mini RPC (liteos_m)
  if (ohos_kernel_type == "liteos_m") {
    static_library("server") {
      defines = [ "MINI_SAMGR_LITE_RPC" ]
      sources = [ "source/samgr_server_rpc.c" ]

      include_dirs = [
        "../samgr_endpoint/source",
        "//commonlibrary/utils_lite/include",
        "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
        "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
        "//third_party/bounds_checking_function/include",
      ]

      public_deps = [
        "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_static",
        "//foundation/communication/ipc/interfaces/innerkits/c/dbinder:dbinder",
        "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
        "//foundation/systemabilitymgr/samgr_lite/samgr_endpoint:store_source",
        "//third_party/bounds_checking_function:libsec_static",
      ]
    }
  }
}
```

## samgr_client/BUILD.gn

**证据位置**: `samgr_client/BUILD.gn:16-59`

```gn
if (!enable_ohos_systemabilitymgr_samgr_lite_rpc_mini) {
  source_set("client") {
    sources = [ "source/remote_register_rpc.c" ]

    cflags = [
      "-fPIC",
      "-Wall",
    ]

    include_dirs = [
      "../samgr_endpoint/source",
      "//commonlibrary/utils_lite/include",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
      "//third_party/bounds_checking_function/include",
    ]

    public_deps = [
      "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
      "//foundation/systemabilitymgr/samgr_lite/samgr_endpoint:endpoint_source",
      "//third_party/bounds_checking_function:libsec_shared",
    ]
  }
} else {
  source_set("client") {
    defines = [ "MINI_SAMGR_LITE_RPC" ]
    sources = [ "source/remote_register_rpc.c" ]

    include_dirs = [
      "../samgr_server/source",
      "../samgr_endpoint/source",
      "//commonlibrary/utils_lite/include",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
      "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
      "//third_party/bounds_checking_function/include",
    ]

    public_deps = [
      "//foundation/communication/ipc/interfaces/innerkits/c/dbinder:dbinder",
      "//foundation/systemabilitymgr/samgr_lite/samgr_endpoint:endpoint_source",
      "//third_party/bounds_checking_function:libsec_static",
    ]
  }
}
```

## samgr_endpoint/BUILD.gn

**证据位置**: `samgr_endpoint/BUILD.gn:16-145`

### 标准构建

```gn
source_set("endpoint_source") {
  sources = [
    "source/client_factory.c",
    "source/default_client_rpc.c",
    "source/default_client_small_adapter.c",
    "source/endpoint_rpc.c",
    "source/samgr_small_ipc_adapter.c",
    "source/token_bucket.c",
  ]

  cflags = [
    "-fPIC",
    "-Wall",
  ]
  cflags += [ "-Wno-int-conversion" ]

  # Linux 特有定义
  if (ohos_kernel_type == "linux") {
    defines = [
      "_GNU_SOURCE",
      "LITE_LINUX_BINDER_IPC",
    ]
  }

  public_deps = [
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}

source_set("store_source") {
  sources = [ "source/sa_store.c" ]
  # ...
}
```

## communication/broadcast/BUILD.gn

**证据位置**: `communication/broadcast/BUILD.gn:14-54`

```gn
if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
  static_library("broadcast") {
    sources = [
      "source/broadcast_service.c",
      "source/pub_sub_feature.c",
      "source/pub_sub_implement.c",
    ]
    public_configs = [ ":broadcast_public" ]
  }
}

if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  shared_library("broadcast") {
    sources = [
      "source/broadcast_service.c",
      "source/pub_sub_feature.c",
      "source/pub_sub_implement.c",
    ]
    public_configs = [ ":broadcast_public" ]
    configs -= [ "//build/lite/config:language_c" ]
    cflags_c = [
      "-std=c11",
      "-Wall",
    ]
    include_dirs = [ "//third_party/bounds_checking_function/include" ]
    public_deps = [
      "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
      "//third_party/bounds_checking_function:libsec_shared",
    ]
  }
}
```

## 产物清单

| 平台 | 类型 | 产物 |
|------|------|------|
| M-core | 静态库 | `libsamgr.a`, `libbroadcast.a` |
| A-core | 动态库 | `libsamgr.so`, `libbroadcast.so` |
| NDK | 头文件 | 见 head_files |

## 依赖关系图

```
lite_component("samgr")
├── samgr
│   ├── samgr_adapter (adapter/)
│   ├── samgr_source (samgr/source/)
│   └── [可选] dbinder
├── communication/broadcast
│   └── samgr
└── [A-core] samgr_server:server
│   ├── samgr
│   ├── endpoint store
│   └── permission_lite
└── [A-core] samgr_client:client
    ├── samgr_endpoint
    └── ipc_single
```

## 下一章

- [平台适配](./09_Platform_Adapter.md) - M-core 与 A-core 差异
- [安全风险评审](./12_Security_Review.md) - 安全考量
