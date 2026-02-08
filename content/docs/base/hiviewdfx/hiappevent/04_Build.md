# 构建配置与编译产物

## 4.1 GN 构建系统概述

HiAppEvent 组件使用 OpenHarmony 标准的 GN（Generate Ninja）构建系统进行编译管理。GN 构建系统通过 .gni 和 .gn 文件定义构建配置，生成 Ninja 构建文件后执行实际编译。这种构建方式具有高度的可扩展性和良好的增量编译性能，是 OpenHarmony 项目推荐的标准构建方式。

### 4.1.1 构建配置入口

HiAppEvent 组件的根目录包含两个关键的 GN 配置文件：

**hiappevent.gni**：定义组件的路径别名和导入配置，供其他模块引用时使用。

```gn
# 文件位置：/base/hiviewdfx/hiappevent/hiappevent.gni

hiappevent_path = "//base/hiviewdfx/hiappevent"
hiappevent_interfaces = "//base/hiviewdfx/hiappevent/interfaces"
hiappevent_framework = "//base/hiviewdfx/hiappevent/frameworks"
```

**hiappevent_aafwk.gni**：定义与 ability_runtime 相关的路径配置。

```gn
# 文件位置：/base/hiviewdfx/hiappevent/hiappevent_aafwk.gni

ability_runtime_path = "//foundation/ability/ability_runtime"
ability_runtime_kits_path = "${ability_runtime_path}/frameworks/kits"
```

### 4.1.2 构建依赖关系

根据 bundle.json 文件中的配置，HiAppEvent 组件在构建时依赖以下系统组件：

| 依赖组件 | 用途说明 | 关键依赖项 |
|---------|---------|-----------|
| ability_runtime | 应用运行框架 | app_context |
| bundle_framework | 包管理框架 | appexecfwk_base, appexecfwk_core |
| c_utils | C 工具库 | utils |
| eventhandler | 事件处理 | - |
| ffrt | 任务调度库 | libffrt |
| hitrace | 追踪机制 | libhitracechain |
| hilog | 日志系统 | libhilog |
| hicollie | 性能监控 | libhicollie |
| hisysevent | 系统事件 | libhisysevent |
| init | 初始化系统 | libbegetutil |
| ipc | 进程间通信 | ipc_core |
| napi | Native API 框架 | ace_napi |
| relational_store | 关系型存储 | native_rdb |
| samgr | 服务管理 | samgr_proxy |
| storage_service | 存储服务 | storage_manager_acl, storage_manager_sa_proxy |
| jsoncpp | JSON 解析库 | - |
| runtime_core | 运行时核心 | - |

## 4.2 核心 Targets 清单

### 4.2.1 Native 核心库（libhiappevent_base）

**构建目标**：`//base/hiviewdfx/hiappevent/frameworks/native/libhiappevent:libhiappevent_base`

**目标类型**：shared_library（共享库）

**构建配置**（BUILD.gn:28-80）：

```gn
ohos_shared_library("libhiappevent_base") {
  # 分支保护
  branch_protector_ret = "pac_ret"
  
  # 公共配置
  public_configs = [
    ":libhiappevent_source_config",
    "cache:hiappevent_cache_config",
    "observer:hiappevent_watcher_config",
  ]
  
  # 源文件列表
  sources = [
    "app_event_util.cpp",
    "dfr/event_config_mgr.cpp",
    "dfr/resource_overlimit_mgr.cpp",
    "hiappevent_base.cpp",
    "hiappevent_c.cpp",
    "hiappevent_clean.cpp",
    "hiappevent_config.cpp",
    "hiappevent_userinfo.cpp",
    "hiappevent_verify.cpp",
    "hiappevent_write.cpp",
    "load/module_loader.cpp",
    "load/processor_config_loader.cpp",
  ]
  
  # 内部依赖
  deps = [
    "cache:hiappevent_cache",
    "cleaner:hiappevent_cleaner",
    "observer:hiappevent_observer",
    "utility:hiappevent_utility",
  ]
  
  # 外部依赖
  external_deps = [
    "ability_runtime:app_context",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "c_utils:utils",
    "ffrt:libffrt",
    "hicollie:libhicollie",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:libhitracechain",
    "init:libbegetutil",
    "ipc:ipc_core",
    "jsoncpp:jsoncpp",
    "relational_store:native_rdb",
    "samgr:samgr_proxy",
    "storage_service:storage_manager_acl",
    "storage_service:storage_manager_sa_proxy",
  ]
  
  # 元数据
  part_name = "hiappevent"
  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "hiviewdfx"
}
```

**包含路径**：

| 路径 | 说明 |
|-----|------|
| frameworks/native/libhiappevent/include | 核心头文件 |
| frameworks/native/libhiappevent/dfr/include | DFR 配置头文件 |
| frameworks/native/libhiappevent/load/include | 模块加载头文件 |
| interfaces/native/inner_api/include | Inner API 头文件 |
| interfaces/native/kits/include | NDK 头文件 |

### 4.2.2 JS N-API 模块（hiappevent）

**构建目标**：`//base/hiviewdfx/hiappevent/frameworks/js/napi:hiappevent`

**目标类型**：shared_library（共享库）

**构建配置**（BUILD.gn:16-42）：

```gn
ohos_shared_library("hiappevent") {
  include_dirs = [ "include/" ]
  
  sources = [
    "./src/napi_error.cpp",
    "./src/napi_hiappevent_builder.cpp",
    "./src/napi_hiappevent_config.cpp",
    "./src/napi_hiappevent_init.cpp",
    "./src/napi_hiappevent_js.cpp",
    "./src/napi_hiappevent_write.cpp",
    "./src/napi_util.cpp",
  ]
  
  deps = [ "../../native/libhiappevent:libhiappevent_base" ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "jsoncpp:jsoncpp",
    "napi:ace_napi",
  ]
  
  relative_install_dir = "module"
  part_name = "hiappevent"
  subsystem_name = "hiviewdfx"
}
```

### 4.2.3 JS N-API V9 模块（hiappevent_v9）

**构建目标**：`//base/hiviewdfx/hiappevent/frameworks/js/napi:hiappevent_v9`

**目标类型**：shared_library（共享库）

**构建配置**（BUILD.gn:44-84）：

```gn
ohos_shared_library("hiappevent_v9") {
  include_dirs = [ "include/" ]
  
  sources = [
    "./src/napi_app_event_holder.cpp",
    "./src/napi_app_event_watcher.cpp",
    "./src/napi_config_builder.cpp",
    "./src/napi_env_watcher_manager.cpp",
    "./src/napi_error.cpp",
    "./src/napi_hiappevent_builder.cpp",
    "./src/napi_hiappevent_config.cpp",
    "./src/napi_hiappevent_init.cpp",
    "./src/napi_hiappevent_js_v9.cpp",
    "./src/napi_hiappevent_processor.cpp",
    "./src/napi_hiappevent_userinfo.cpp",
    "./src/napi_hiappevent_watch.cpp",
    "./src/napi_hiappevent_write.cpp",
    "./src/napi_param_builder.cpp",
    "./src/napi_util.cpp",
  ]
  
  deps = [
    "../../native/libhiappevent:libhiappevent_base",
    "../../native/libhiappevent/utility:hiappevent_utility",
  ]
  
  external_deps = [
    "c_utils:utils",
    "ffrt:libffrt",
    "hilog:libhilog",
    "jsoncpp:jsoncpp",
    "napi:ace_napi",
    "relational_store:native_rdb",
  ]
  
  output_name = "hiappevent_napi"
  relative_install_dir = "module/hiviewdfx"
  
  part_name = "hiappevent"
  subsystem_name = "hiviewdfx"
}
```

### 4.2.4 Native NDK 模块（hiappevent_ndk）

**构建目标**：`//base/hiviewdfx/hiappevent/frameworks/native/ndk:hiappevent_ndk`

**目标类型**：shared_library（共享库）

**构建配置**（ndk/BUILD.gn:18-45）：

```gn
ohos_shared_library("hiappevent_ndk") {
  include_dirs = [
    "$hiappevent_native_path/libhiappevent/include",
    "$hiappevent_native_path/ndk/include",
    "//base/hiviewdfx/hiappevent/interfaces/native/kits/include",
  ]
  
  sources = [
    "hiappevent_ndk.c",
    "src/ndk_app_event_processor.cpp",
    "src/ndk_app_event_processor_service.cpp",
    "src/ndk_app_event_watcher.cpp",
    "src/ndk_app_event_watcher_proxy.cpp",
    "src/ndk_app_event_watcher_service.cpp",
  ]
  
  deps = [ "$hiappevent_native_path/libhiappevent:libhiappevent_base" ]
  
  external_deps = [
    "c_utils:utils",
    "ffrt:libffrt",
    "hilog:libhilog",
    "relational_store:native_rdb",
  ]
  
  part_name = "hiappevent"
  subsystem_name = "hiviewdfx"
}
```

### 4.2.5 子模块 Targets

HiAppEvent 组件还包含以下子模块 Targets：

| 目标名称 | 构建路径 | 类型 | 说明 |
|---------|---------|------|------|
| hiappevent_cache | frameworks/native/libhiappevent/cache:BUILD.gn | static_library | 事件缓存模块 |
| hiappevent_cleaner | frameworks/native/libhiappevent/cleaner:BUILD.gn | static_library | 数据清理模块 |
| hiappevent_observer | frameworks/native/libhiappevent/observer:BUILD.gn | static_library | 事件观察者模块 |
| hiappevent_utility | frameworks/native/libhiappevent/utility:BUILD.gn | static_library | 工具函数模块 |

## 4.3 产物清单与安装路径

### 4.3.1 主要编译产物

| 产物名称 | 源 Build Target | 输出文件名 | 产物类型 |
|---------|---------------|-----------|---------|
| libhiappevent_base | libhiappevent:libhiappevent_base | libhiappevent_base.z.so | 共享库 |
| hiappevent | napi:hiappevent | libhiappevent.z.so | 共享库（JS API 7） |
| hiappevent_v9 | napi:hiappevent_v9 | hiappevent_napi.so | 共享库（JS API 9+） |
| hiappevent_ndk | ndk:hiappevent_ndk | libhiappevent_ndk.z.so | 共享库（NDK） |

### 4.3.2 运行时加载关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                        应用进程内存空间                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   JavaScript 应用 (@ohos.hiAppEvent)                                │
│           │                                                        │
│           ▼                                                        │
│   libhiappevent.z.so / hiappevent_napi.so                          │
│           │                                                        │
│           ▼                                                        │
│   libhiappevent_base.z.so ◄─── 依赖关系                            │
│           │                                                        │
│   ├── libffrt.so                                                   │
│   ├── libhilog.so                                                   │
│   ├── libhisysevent.so                                              │
│   ├── libnative_rdb.so                                             │
│   ├── libipc_core.so                                               │
│   └── ...                                                          │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                        系统服务进程空间                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   事件存储服务                                                      │
│           │                                                        │
│           ▼                                                        │
│   /data/service/el2/100/hiappevent/                                │
│           ├── hiappevent.cfg          （配置文件）                 │
│   /data/log/hiappevent/                                            │
│           ├── events/                  （事件日志目录）             │
│   └── ...                                                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.3.3 安装路径映射

| 产物 | 预安装路径 | 说明 |
|-----|-----------|------|
| libhiappevent.z.so | /system/lib/module/libhiappevent.z.so | JS API 7 共享库 |
| hiappevent_napi.so | /system/lib/module/hiviewdfx/hiappevent_napi.so | JS API 9+ 共享库 |
| libhiappevent_ndk.z.so | /system/lib64/libhiappevent_ndk.z.so | NDK 共享库 |
| libhiappevent_base.z.so | /system/lib/libhiappevent_base.z.so | 核心库 |

### 4.3.4 配置文件路径

| 配置类型 | 路径 | 说明 |
|---------|------|------|
| 系统配置 | /system/etc/hiappevent/ | 系统级配置 |
| 应用配置 | /data/service/el2/100/hiappevent/ | 应用私有配置 |
| 事件日志 | /data/log/hiappevent/ | 事件日志目录 |
| 缓存数据 | /data/storage/hiappevent/ | 缓存数据目录 |

## 4.4 编译配置项

### 4.4.1 编译器标志

HiAppEvent 组件在编译时使用以下关键编译器标志：

| 标志名称 | 值 | 用途 |
|---------|-----|------|
| -std=c++14 | C++14 | C++ 标准版本 |
| -fPIC | 启用 | 位置无关代码 |
| -Wall | 启用 | 警告信息全开 |
| -Wextra | 启用 | 额外警告检查 |
| -Werror | 禁用 | 警告不作为错误 |
| -O2 | 优化级别 | 编译器优化 |
| -g | 调试信息 | 生成调试符号 |

### 4.4.2 链接配置

**静态链接依赖**：
- libc++_shared.so（C++ 标准库）
- libclang_rt.builtins（编译器运行时库）

**动态链接依赖**：
- libhilog.so（HiLog 日志库）
- libhisysevent.so（系统事件库）
- libffrt.so（FFRT 任务调度）
- libipc_core.so（IPC 核心库）
- libnative_rdb.so（关系型数据库）
- libutils.so（工具库）

## 4.5 构建命令

### 4.5.1 完整构建

```bash
# 设置构建环境
source build.sh set Judao

# 执行构建
hb build -f -p hiappevent
```

### 4.5.2 单独模块构建

```bash
# 构建 JS N-API 模块
hb build -f -p hiappevent -T //base/hiviewdfx/hiappevent/frameworks/js/napi:hiappevent_v9

# 构建 NDK 模块
hb build -f -p hiappevent -T //base/hiviewdfx/hiappevent/frameworks/native/ndk:hiappevent_ndk

# 构建核心库
hb build -f -p hiappevent -T //base/hiviewdfx/hiappevent/frameworks/native/libhiappevent:libhiappevent_base
```

### 4.5.3 产物验证

构建完成后，可通过以下命令验证产物：

```bash
# 列出构建产物
ls -la out/hiappevent/.../lib/*.so

# 检查依赖关系
readelf -d libhiappevent_base.z.so

# 检查导出符号
nm -D libhiappevent_base.z.so | grep -i "OH_HiAppEvent"
```

## 4.6 构建问题排查

### 4.6.1 常见构建错误

**错误类型一：依赖缺失**

```
error: dependency 'xxx' not found
```

**解决方案**：确保对应的子系统已包含在产品配置中。检查 build/lite/product 目录下的产品配置文件。

**错误类型二：头文件缺失**

```
fatal error: 'xxx.h' file not found
```

**解决方案**：检查 include_dirs 配置是否包含所需头文件路径。确认依赖组件的头文件已正确安装。

**错误类型三：符号未定义**

```
undefined reference to 'xxx'
```

**解决方案**：检查 external_deps 配置是否包含正确的依赖项。确认依赖库的链接顺序正确。

### 4.6.2 构建性能优化

**启用分布式构建**：

```bash
# 使用分布式编译
hb build -f -p hiappevent --build-type release --enable-distributed-build
```

**启用缓存**：

```bash
# 使用编译缓存
hb build -f -p hiappevent --ccache
```
