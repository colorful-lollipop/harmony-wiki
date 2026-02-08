# GN 构建系统

## 构建入口

| 文件 | 说明 |
|------|------|
| [BUILD.gn](../BUILD.gn) | 项目根构建入口 |
| [hichecker.gni](../hichecker.gni) | 模块级配置 |

## hichecker.gni 配置

**文件**: `hichecker.gni`

```gni
declare_args() {
  hichecker_support_asan = true  # ASan 支持开关
}
```

## 子组件列表

bundle.json 中定义了以下子组件：

| # | 子组件路径 | 类型 | 产出 |
|---|-----------|------|------|
| 1 | `//base/hiviewdfx/hichecker/interfaces/native/innerkits:libhichecker` | ohos_shared_library | libhichecker.so |
| 2 | `//base/hiviewdfx/hichecker/interfaces/js/kits/napi:hiccheckter` | ohos_shared_library | libhichecker.so (JS) |
| 3 | `//base/hiviewdfx/hichecker/interfaces/js/kits/napi/js_leak_watcher:jsleakwatcher` | ohos_shared_library | libjsleakwatcher.so |
| 4 | `//base/hiviewdfx/hichecker/interfaces/js/kits/napi/js_leak_watcher:jsleakwatchernative` | ohos_shared_library | libjsleakwatchernative.so |
| 5 | `//base/hiviewdfx/hichecker/frameworks/native:libhichecker_source` | ohos_source_set | libhichecker.a (中间产物) |
| 6 | `//base/hiviewdfx/hichecker/interfaces/ets/ani:ani_hichecker_package` | group | ani 产物包 |

## Native InnerKit (libhichecker)

**文件**: `interfaces/native/innerkits/BUILD.gn`

```gni
ohos_shared_library("libhichecker") {
  branch_protector_ret = "pac_ret"
  public_configs = [ ":hiccheckER_native_config" ]
  
  deps = [
    ":hichecker_etc",  # 参数文件
    "../../../frameworks/native:libhichecker_source",
  ]
  external_deps = [ "hilog:libhilog" ]
  
  output_extension = "so"
  innerapi_tags = [ "platformsdk" ]
  part_name = "hichecker"
  subsystem_name = "hiviewdfx"
}
```

**配置**:

```gni
config("hichecker_native_config") {
  visibility = [ ":*" ]
  include_dirs = [ "include" ]
}
```

**外部依赖**:
- `hilog:libhilog` - 日志库

## 参数文件

```gni
ohos_prebuilt_etc("hichecker.para") {
  source = "hichecker.para"
  install_images = [ "system", "updater" ]
  module_install_dir = "etc/param"
  part_name = "hichecker"
}

ohos_prebuilt_etc("hichecker.para.dac") {
  source = "hichecker.para.dac"
  install_images = [ "system", "updater" ]
  module_install_dir = "etc/param"
  part_name = "hichecker"
}
```

## N-API 模块 (hichecker)

**文件**: `interfaces/js/kits/napi/BUILD.gn`

```gni
ohos_shared_library("hicchecker") {
  if (support_jsapi) {
    include_dirs = [
      "./",
      "include/",
    ]
    configs = [ ":hichecker_js_source_config" ]
    
    sources = [ "./src/napi_hichecker.cpp" ]
    
    deps = [ "../../../native/innerkits:libhichecker" ]
    
    external_deps = [
      "hilog:libhilog",
      "napi:ace_napi",
    ]
    
    relative_install_dir = "module"
    
    subsystem_name = "hiviewdfx"
    part_name = "hichecker"
  }
}
```

**包含子模块**:

```gni
ohos_shared_library("jsleakwatcher") {
  # ... js leak watcher 配置
}

ohos_shared_library("jsleakwatchernative") {
  # ... native part
}
```

## Native 框架 (libhichecker_source)

**文件**: `frameworks/native/BUILD.gn`

```gni
ohos_source_set("libhichecker_source") {
  branch_protector_ret = "pac_ret"
  include_dirs = [ "../../interfaces/native/innerkits/include" ]
  
  sources = [
    "caution.cpp",
    "hichecker.cpp",
    "hichecker_wrapper.cpp",
  ]
  
  external_deps = [
    "c_utils:utils",
    "faultloggerd:libbacktrace_local",
    "hilog:libhilog",
    "init:libbeget_proxy",
    "init:libbegetutil",
  ]
  
  part_name = "hichecker"
  subsystem_name = "hiviewdfx"
}
```

**外部依赖**:
- `c_utils:utils` - C 工具库
- `faultloggerd:libbacktrace_local` - 本地回溯
- `hilog:libhilog` - 日志
- `init:libbeget_proxy` - BeGet 代理
- `init:libbegetutil` - BeGet 工具

## ANI 模块 (ani_hichecker_package)

**文件**: `interfaces/ets/ani/hichecker/BUILD.gn`

```gni
# ANI 模块构建配置
# 产出 ani_hichecker_package
```

## 编译产物

### 动态库 (.so)

| 产物 | 路径 (预计) | 用途 |
|------|------------|------|
| libhichecker.so | system/lib64/ | Native InnerKit |
| libhichecker.so (JS) | system/lib64/module/ | N-API 模块 |
| libjsleakwatcher.so | system/lib64/module/ | JS LeakWatcher |
| libjsleakwatchernative.so | system/lib64/ | Native LeakWatcher |
| libani_hichecker.so | system/lib64/ | ANI 模块 |

### 参数文件

| 产物 | 路径 | 用途 |
|------|------|------|
| hichecker.para | system/etc/param/ | 参数配置 |
| hichecker.para.dac | system/etc/param/ | DAC 配置 |

### 中间产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libhichecker.a | out/.../obj/ | 静态库 (libhichecker_source) |

## 运行时加载关系

```
应用进程
    │
    ├── load libhichecker.so (Native API)
    │       │
    │       └── 依赖: libhilog.so
    │
    ├── load libhichecker.so (JS Module)
    │       │
    │       └── 依赖: libhichecker.so, libace_napi.so
    │
    └── load libjsleakwatchernative.so
            │
            └── 依赖: libhilog.so, libeventhandler.so
```

## 构建命令示例

```bash
# 完整构建
hb set
hb build -f

# 只构建 hichecker
hb build -f --parts hiviewdfx --parts hichecker

# 查看构建产物
ls out/{product}/system/lib64/
ls out/{product}/system/lib64/module/
ls out/{product}/system/etc/param/
```
