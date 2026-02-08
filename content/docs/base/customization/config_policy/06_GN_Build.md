# GN 构建配置

## 配置文件清单

| 文件 | 用途 |
|------|------|
| `config_policy.gni` | 全局配置参数 |
| `BUILD.gn` | 根构建入口 |
| `frameworks/config_policy/BUILD.gn` | 核心模块构建 |
| `frameworks/config_policy/etc/BUILD.gn` | ETC 文件构建 |
| `interfaces/kits/js/BUILD.gn` | JS N-API 构建 |
| `interfaces/kits/cj/BUILD.gn` | CJ FFI 构建 |
| `interfaces/ets/ani/BUILD.gn` | ETS ANI 构建 |

---

## 全局配置参数

### config_policy.gni

```gni
declare_args() {
  # if fs has prefix before OH path, set it here
  config_policy_fs_prefix = ""

  # Whether support api so
  config_policy_api_support = true
}
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `config_policy_fs_prefix` | string | `""` | 文件系统路径前缀 |
| `config_policy_api_support` | bool | `true` | 是否支持 API 动态库 |

**代码证据**: `config_policy.gni:14-20`

---

## 根构建入口

### BUILD.gn

```gni
import("./config_policy.gni")

group("config_policy_components") {
  if (os_level == "standard" && config_policy_api_support && support_jsapi) {
    deps = [
      "./frameworks/config_policy:configpolicy_util",
      "./interfaces/kits/cj:cj_config_policy_ffi",
      "./interfaces/kits/js:configpolicy",
      "./interfaces/kits/js:customconfig",
    ]
  } else {
    deps = [ "./frameworks/config_policy:configpolicy_util" ]
  }
}

group("ani_config_policy_components") {
  if (os_level == "standard" && config_policy_api_support && support_jsapi) {
    deps = [
      "./interfaces/ets/ani:configPolicy_etc",
      "./interfaces/ets/ani:configpolicy_ani",
      "./interfaces/ets/ani:custom_config_etc",
      "./interfaces/ets/ani:customconfig_ani",
    ]
  }
}
```

**代码证据**: `BUILD.gn:14-38`

---

## 核心模块构建

### frameworks/config_policy/BUILD.gn

```gni
import("../../config_policy.gni")
if (defined(ohos_lite)) {
  import("//build/lite/config/component/lite_component.gni")
} else {
  import("//build/ohos.gni")
}

config_policy_sources = [ "src/config_policy_utils.c" ]
config("config_policy_config") {
  include_dirs = [ "../../interfaces/inner_api/include" ]
}
```

#### 目标列表

| 目标名称 | 类型 | 平台 | 输出 |
|----------|------|------|------|
| `configpolicy_util` | static_library | LiteOS-M | `libconfigpolicy_util.a` |
| `configpolicy_util` | shared_library | LiteOS（非 M） | `libconfigpolicy_util.so` |
| `configpolicy_util` | ohos_shared_library | 标准系统 | `libconfigpolicy_util.z.so` |

**代码证据**: `frameworks/config_policy/BUILD.gn:21-65`

#### 依赖配置

```gni
external_deps = [
  "bounds_checking_function:libsec_shared",  # 安全函数
  "init:libsystemparam",                        # 系统参数
]
```

**代码证据**: `frameworks/config_policy/BUILD.gn:49-52`

#### 安装配置

```gni
install_images = [
  "system",
  "updater",
]
```

**代码证据**: `frameworks/config_policy/BUILD.gn:53-56`

---

## JS N-API 构建

### interfaces/kits/js/BUILD.gn

#### 目标 1: configpolicy

```gni
ohos_shared_library("configpolicy") {
  include_dirs = [
    "include",
    "../../../interfaces/inner_api/include",
    "../../../frameworks/dfx/hisysevent_adapter",
  ]

  sources = [
    "../../../frameworks/dfx/hisysevent_adapter/hisysevent_adapter.cpp",
    "src/config_policy_napi.cpp",
  ]

  deps = [ "../../../frameworks/config_policy:configpolicy_util" ]
  external_deps = [
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "napi:ace_napi",
  ]
  relative_install_dir = "module"
  subsystem_name = "customization"
  part_name = "config_policy"
}
```

| 属性 | 值 |
|------|-----|
| 输出 | `libconfigpolicy.z.so` |
| 安装路径 | `system/module/` |

**代码证据**: `interfaces/kits/js/BUILD.gn:16-37`

#### 目标 2: customconfig

```gni
ohos_shared_library("customconfig") {
  sources = [ "src/custom_config_napi.cpp" ]
  external_deps = [
    "ability_runtime:abilitykit_native",
    "ability_runtime:app_context",
    "c_utils:utils",
    "hilog:libhilog",
    "init:libbegetutil",
    "ipc:ipc_single",
    "napi:ace_napi",
  ]
  relative_install_dir = "module/customization"
  subsystem_name = "customization"
  part_name = "config_policy"
}
```

| 属性 | 值 |
|------|-----|
| 输出 | `libcustomconfig.z.so` |
| 安装路径 | `system/module/customization/` |

**代码证据**: `interfaces/kits/js/BUILD.gn:39-61`

---

## 编译产物清单

| 产物 | 类型 | 路径 | 说明 |
|------|------|------|------|
| `libconfigpolicy_util.z.so` | 动态库 | `system/lib/` | 核心配置策略库 |
| `libconfigpolicy.z.so` | 动态库 | `system/module/` | JS N-API 绑定 |
| `libcustomconfig.z.so` | 动态库 | `system/module/customization/` | Custom Config API |
| `customization.para.dac` | 预置文件 | `etc/param/` | 参数配置文件 |

---

## 内部接口标签

```gni
innerapi_tags = [
  "chipsetsdk_sp",
  "platformsdk",
  "sasdk",
]
```

**代码证据**: `frameworks/config_policy/BUILD.gn:57-61`

---

## 条件编译

### 标准系统完整构建

当满足以下条件时，构建完整功能：
- `os_level == "standard"`
- `config_policy_api_support == true`
- `support_jsapi == true`

### 小型系统基础构建

小型系统仅构建核心静态库：
- `libconfigpolicy_util.a`

**代码证据**: `BUILD.gn:16-27`

---

## 相关文档

- [目录结构与模块职责](./02_Directory_Structure.md)
- [N-API 接口文档](./04_N-API_Reference.md)
