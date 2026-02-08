# 构建与编译

## 概述

输入法框架使用 GN (Generate Ninja) 作为构建系统，通过 `BUILD.gn` 和 `.gni` 文件定义构建规则。

## 构建配置

### 根构建文件

**路径**: `BUILD.gn`

```gn
import("//build/ohos.gni")

group("imf_packages") {
  if (is_standard_system) {
    deps = [
      "etc/init:inputmethodservice.cfg",
      "etc/para:inputmethod.para",
      "etc/para:inputmethod_para",
      "frameworks/cj:cj_inputmethod_ffi",
      "frameworks/js/napi/inputmethodability:inputmethodengine",
      "frameworks/js/napi/inputmethodclient:inputmethod",
      "frameworks/js/napi/inputmethodlist:inputmethodlist",
      "frameworks/js/napi/keyboardpanelmanager:keyboardpanelmanager",
      "interfaces/inner_api/inputmethod_ability:inputmethod_ability",
      "interfaces/inner_api/inputmethod_controller:inputmethod_client",
      "profile:inputmethod_inputmethod_sa_profiles",
      "services:inputmethod_service",
      "test/unitest/src:unittest",
    ]
  }
}
```

### bundle.json 配置

**路径**: `bundle.json`

```json
{
  "name": "@ohos/imf",
  "version": "3.1",
  "component": {
    "name": "imf",
    "subsystem": "inputmethod",
    "syscap": [
      "SystemCapability.MiscServices.InputMethodFramework"
    ],
    "features": [
      "imf_screenlock_mgr_enable",
      "imf_on_demand_start_stop_sa_enable",
      "imf_restore_in_high_cpu_usage"
    ]
  }
}
```

## Targets 清单

### 基础组件 (base_group)

| Target | 类型 | 输出 | 描述 |
|--------|------|------|------|
| `//base/inputmethod/imf/common:inputmethod_common` | static_library | libinputmethod_common.a | 公共代码 |

### 框架组件 (fwk_group)

| Target | 类型 | 输出 | 描述 |
|--------|------|------|------|
| `//base/inputmethod/imf/interfaces/inner_api/inputmethod_controller:inputmethod_client` | shared_library | libinputmethod_client.z.so | 应用客户端 |
| `//base/inputmethod/imf/frameworks/js/napi/inputmethodclient:inputmethod` | shared_library | libinputmethod.z.so | JS API |
| `//base/inputmethod/imf/frameworks/js/napi/inputmethodlist:inputmethodlist` | shared_library | libinputmethodlist.z.so | 输入法列表 |
| `//base/inputmethod/imf/frameworks/js/napi/inputmethodpanel:panel` | shared_library | libpanel.z.so | 面板控制 |
| `//base/inputmethod/imf/frameworks/ndk:ohinputmethod` | shared_library | libnative_inputmethod.z.so | NDK 接口 |
| `//base/inputmethod/imf/frameworks/ets/taihe/inputMethod:inputmethod_taihe` | ets | inputmethod_taihe | Taihe 引擎 |
| `//base/inputmethod/imf/frameworks/ets/taihe/inputMethodEngine:inputmethod_engine_taihe` | ets | inputmethod_engine_taihe | Taihe 引擎 |
| `//base/inputmethod/imf/frameworks/ets/inputmethodlist:inputmethod_list_dialog` | ets | inputmethod_list_dialog | 列表对话框 |

### 服务组件 (service_group)

| Target | 类型 | 输出 | 描述 |
|--------|------|------|------|
| `//base/inputmethod/imf/etc/init:inputmethodservice.cfg` | config | inputmethodservice.cfg | 服务配置 |
| `//base/inputmethod/imf/etc/para:inputmethod.para.dac` | config | inputmethod.para.dac | DAC 配置 |
| `//base/inputmethod/imf/etc/para:inputmethod.para` | config | inputmethod.para | 参数配置 |
| `//base/inputmethod/imf/interfaces/inner_api/inputmethod_ability:inputmethod_ability` | shared_library | libinputmethod_ability.z.so | 输入法客户端 |
| `//base/inputmethod/imf/profile:inputmethod_inputmethod_sa_profiles` | config | sa_profiles | SA 配置 |
| `//base/inputmethod/imf/services:inputmethod_service` | shared_library | libinputmethod_service.z.so | 输入法服务 |
| `//base/inputmethod/imf/frameworks/ets/extension/ani:inputmethod_extension_ani` | ani | inputmethod_extension_ani | ANI 扩展 |
| `//base/inputmethod/imf/frameworks/ets/extension/ets:inputmethod_extension_ability_etc` | config | inputmethod_extension_ability_etc | 扩展配置 |
| `//base/inputmethod/imf/frameworks/kits/extension:inputmethod_extension` | ets | inputmethod_extension | 扩展套件 |
| `//base/inputmethod/imf/frameworks/kits/extension:inputmethod_extension_module` | ets | inputmethod_extension_module | 扩展模块 |
| `//base/inputmethod/imf/frameworks/js/napi/inputmethodability:inputmethodengine` | shared_library | libinputmethodengine.z.so | 引擎接口 |
| `//base/inputmethod/imf/frameworks/js/napi/inputmethod_extension_ability:inputmethodextensionability_napi` | shared_library | libinputmethodextensionability_napi.z.so | 扩展能力 NAPI |
| `//base/inputmethod/imf/frameworks/js/napi/inputmethod_extension_context:inputmethodextensioncontext_napi` | shared_library | libinputmethodextensioncontext_napi.z.so | 扩展上下文 NAPI |
| `//base/inputmethod/imf/frameworks/js/napi/keyboardpanelmanager:keyboardpanelmanager` | shared_library | libkeyboardpanelmanager.z.so | 键盘面板管理 |
| `//base/inputmethod/imf/frameworks/cj:cj_inputmethod_ffi` | shared_library | libcj_inputmethod_ffi.z.so | CJ FFI |
| `//base/inputmethod/imf/frameworks/kits/extension_cj:cj_inputmethod_extension_ffi` | shared_library | libcj_inputmethod_extension_ffi.z.so | CJ 扩展 FFI |
| `//base/inputmethod/imf/seccomp_policy:imf_ext_secure_filter` | config | imf_ext_secure_filter | Seccomp 策略 |
| `//base/inputmethod/imf/services/dialog:input_method_choose_dialog` | ets | input_method_choose_dialog | 选择对话框 |
| `//base/inputmethod/imf/tools/ime:ime` | executable | ime | IME 工具 |

## 编译产物

### 动态库 (.so)

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `libinputmethod_client.z.so` | system/lib | 应用客户端库 |
| `libinputmethod_ability.z.so` | system/lib | 输入法客户端库 |
| `libinputmethod_service.z.so` | system/lib | 输入法服务库 |
| `libinputmethod_para.z.so` | system/lib | 参数库 |
| `libinputmethod.z.so` | system/lib/module | JS API 库 |
| `libinputmethodengine.z.so` | system/lib/module | 引擎库 |
| `libinputmethodlist.z.so` | system/lib/module | 列表库 |
| `libpanel.z.so` | system/lib/module | 面板库 |

### 配置文件

| 产物 | 路径 |
|------|------|
| `inputmethodservice.cfg` | system/etc/init/ |
| `inputmethod.para` | system/etc/parameter/ |
| `inputmethod.para.dac` | system/etc/ |

### Seccomp 策略

| 产物 | 路径 |
|------|------|
| `imf_ext_secure_filter` | system/etc/sec_policy/ |

## 编译命令

### 完整编译

```bash
./build.sh --product-name <product_name> --build-target imf
```

### 单独编译

```bash
# 编译指定 target
./build.sh --product-name <product_name> --build-target inputmethod

# 编译服务
./build.sh --product-name <product_name> --build-target inputmethod_service
```

## 依赖关系

### 系统依赖

| 组件 | 版本 |
|------|------|
| napi | - |
| samgr | - |
| ipc | - |
| ability_runtime | - |
| hilog | - |
| access_token | - |

### 内部依赖

```
inputmethod_service
    │
    ├── inputmethod_ability
    │       │
    │       └── inputmethod_common
    │
    └── inputmethod_client
            │
            └── inputmethod_common
```

## 产物安装验证

编译完成后，产物位于：

```
out/<product>/inputmethod/imf/
├── libinputmethod_client.z.so
├── libinputmethod_ability.z.so
├── libinputmethod_service.z.so
├── libinputmethod_para.z.so
├── libinputmethod.z.so
├── libinputmethodengine.z.so
├── libinputmethodlist.z.so
├── libpanel.z.so
└── ...
```

## 相关文档

- [架构说明](./01_Architecture.md)
- [N-API 接口](./02_N-API.md)
- [安全评审](./05_Security.md)
