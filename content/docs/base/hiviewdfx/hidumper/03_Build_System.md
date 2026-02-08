# 构建系统

> 目的：理解 HiDumper 的 GN 构建配置、Targets 列表、依赖关系

## 1. 构建文件结构

```
hidumper/
├── BUILD.gn              # 根构建入口
├── hidumper.gni         # 全局配置 (特性开关、路径变量)
├── frameworks/native/
│   └── BUILD.gn         # 框架构建
├── services/
│   └── BUILD.gn         # 服务构建
├── interfaces/innerkits/
│   └── BUILD.gn         # 接口库构建
├── utils/
│   └── BUILD.gn         # 工具库构建
└── sa_profile/
    └── BUILD.gn         # SA 配置构建
```

## 2. 全局配置 (hidumper.gni)

**文件**: `hidumper.gni`

### 2.1 路径变量

```gn
hidumper_subsystem_name = "hiviewdfx"
hidumper_part_name = "hidumper"
hidumper_root_path = "//base/hiviewdfx/hidumper"
hidumper_frameworks_path = "${hidumper_root_path}/frameworks/native"
hidumper_client_path = "${hidumper_root_path}/client"
hidumper_service_path = "${hidumper_root_path}/services"
hidumper_interface = "${hidumper_root_path}/interfaces/native"
```

### 2.2 特性开关

| 宏定义 | 默认值 | 说明 |
|-------|-------|------|
| `hidumper_ability_runtime_enable` | true | 启用 Ability Runtime |
| `hidumper_ablility_base_enable` | false | 启用 Ability Base |
| `hidumper_netmanager_base_enable` | true | 启用网络管理 |
| `hidumper_bundlemanager_framework_enable` | true | 启用 Bundle 管理 |
| `hidumper_hiviewdfx_hisysevent_enable` | true | 启用 HiSysEvent |
| `hidumper_hiviewdfx_hiview_enable` | false | 启用 HiView |
| `hidumper_report_memmgr` | false | 启用内存管理器 |

## 3. 根构建 (BUILD.gn)

**文件**: `BUILD.gn`

```gn
import("//build/ohos.gni")

# bin 分组 - 可执行文件
group("bin") {
  deps = [ "frameworks/native:hidumper" ]
}

# service 分组 - 所有服务组件
group("service") {
  deps = [
    "frameworks/native:hidumperclient",
    "interfaces/innerkits:lib_dump_usage",
    "sa_profile:hidumper_service_sa_profile",
    "services:event_reason_config",
    "services:hidumper_service.rc",
    "services:hidumpermemory",
    "services:hidumperservice",
  ]
  if (build_variant == "root") {
    deps += [ "services:infos_config" ]
  }
}
```

## 4. 框架构建 (frameworks/native/BUILD.gn)

### 4.1 Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|-----|------|
| `hidumper_include` | config | - | 头文件包含路径 |
| `dump_main` | source_set | - | 核心功能源码 |
| `dump_framework` | source_set | - | 框架层源码 |
| `hidumperclient_source` | source_set | - | 客户端源码 |
| `hidumperclient` | shared_library | libhidumperclient.so | 客户端库 |
| `hidumper` | executable | hidumper | 主可执行文件 |

### 4.2 关键依赖

```gn
hidumperclient_shared_library {
  deps = [
    ":hidumperclient_source",
    "//base/hiviewdfx/hidumper/utils:utils",
    "//foundation/ability/ability_runtime:ability_runtime_lite",
  ]
}

hidumper_executable {
  deps = [
    ":hidumperclient",
    "//third_party/bzip2:bz2",
    "//third_party/zlib:zlib",
  ]
}
```

## 5. 服务构建 (services/BUILD.gn)

### 5.1 Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|-----|------|
| `hidumper_client` | shared_library | libhidumper_client.so | 服务客户端库 |
| `hidumperservice` | shared_library | libhidumperservice.so | 主服务 (SA) |
| `hidumpermemory` | shared_library | libhidumpermemory.so | 内存 dump 库 |
| `hidumpercpuservice` | shared_library | libhidumpercpuservice.so | CPU 服务 (条件) |
| `hidumper_service.rc` | prebuilt_etc | hidumper_service.cfg | 启动配置 |

### 5.2 ZIDL 配置

```gn
hidumpercpuservice_interface = zidl_interface("hidumpercpuservice") {
  sources = [ "IHidumperCpuService.idl" ]
  output_dir = "include"
}
```

### 5.3 关键依赖

```gn
hidumperservice_shared_library {
  deps = [
    ":hidumperservice_source",
    ":zidl_service",
    "//base/hiviewdfx/hidumper/utils:utils",
    "//base/startup/init:libinit",
  ]
  shlib_type = "sa"  # 标记为 System Ability
}
```

## 6. 接口构建 (interfaces/innerkits/BUILD.gn)

### 6.1 Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|-----|------|
| `lib_dump_usage` | shared_library | lib_dump_usage.so | 内存/CPU 统计库 |

### 6.2 关键配置

```gn
lib_dump_usage_shared_library {
  public_deps = [
    "//base/hiviewdfx/hidumper/services:interface_include",
  ]
}
```

## 7. 工具库构建 (utils/BUILD.gn)

### 7.1 Targets 列表

| Target | 类型 | 输出 | 说明 |
|--------|------|-----|------|
| `utils` | source_set | - | 工具源码 |

### 7.2 包含内容

- `permission.h` - 权限校验
- `dump_errors.h` - 错误码定义

## 8. SA 配置构建 (sa_profile/BUILD.gn)

### 8.1 Target

```gn
ohos_sa_profile("hidumper_service_sa_profile") {
  sources = [ "1212.json" ]
}
```

## 9. 依赖关系图

```
hidumper (executable)
├── hidumperclient (shared_library)
│   ├── dump_main (source_set)
│   ├── utils (source_set)
│   └── ability_runtime
│
hidumperservice (shared_library, sa)
├── zidl_service (source_set)
│   └── hidumpercpuservice_interface (idl)
├── dump_main (source_set)
├── utils (source_set)
└── libhidumpermemory (shared_library)

lib_dump_usage (shared_library)
└── services:interface_include
```

## 10. 编译命令

### 10.1 使用 hb 构建

```bash
# 构建整个组件
hb build hidumper

# 仅构建可执行文件
hb build hidumper -T frameworks/native:hidumper

# 构建所有服务
hb build hidumper -T :service
```

### 10.2 使用 gn + ninja

```bash
# 生成构建文件
gn gen out/hidumper --root=/path/to/openharmony

# 构建所有
ninja -C out/hidumper //base/hiviewdfx/hidumper/...

# 仅构建可执行文件
ninja -C out/hidumper //base/hiviewdfx/hidumper/frameworks/native:hidumper
```

## 相关文档

- [编译产物](./04_Outputs.md)
- [系统架构](./01_Architecture.md)
- [API 参考](./02_API_Reference.md)
