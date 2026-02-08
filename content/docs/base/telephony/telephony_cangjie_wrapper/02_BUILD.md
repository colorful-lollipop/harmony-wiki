# 构建系统

## 概述

 telephony_cangjie_wrapper 使用 OpenHarmony 的 GN (Generate Ninja) 构建系统，通过 `ohos_cangjie_shared_library` 模板编译仓颉源码。

**构建入口**：`//base/telephony/telephony_cangjie_wrapper/BUILD.gn`

## GN Targets 清单

| Target | 类型 | 产物 | 位置 |
|--------|------|------|------|
| `copy_sdk_telephony_cangjie_libs` | copy_ohos_cangjie_sdk_api_lib | SDK 库拷贝 | 根 BUILD.gn |
| `ohos.telephony` | ohos_cangjie_shared_library | libohos.telephony.so | ohos/telephony/BUILD.gn |
| `ohos.telephony.call` | ohos_cangjie_shared_library | libohos.telephony.call.so | ohos/telephony/call/BUILD.gn |
| `kit.TelephonyKit` | ohos_cangjie_shared_library | libkit.TelephonyKit.so | kit/TelephonyKit/BUILD.gn |

---

## Target 详细配置

### 1. ohos.telephony.call

**路径**：`ohos/telephony/call/BUILD.gn:18`

```gn
ohos_cangjie_shared_library("ohos.telephony.call") {
  if (is_mingw || is_mac) {
    sources = [ "../../../mock/ohos.telephony.call.cj" ]
  } else {
    sources = [
      "number_format_options.cj",
      "call.cj",
      "telephony_call_ffi.cj",
    ]
  }

  cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",
    "ability_cangjie_wrapper:ohos.app.ability.want",
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
    "cangjie_ark_interop:ohos.encoding.json",
    "cangjie_ark_interop:ohos.labels",
  ]

  external_deps = [ "call_manager:cj_telephony_call_ffi" ]

  subsystem_name = "telephony"
  part_name = "telephony_cangjie_wrapper"
}
```

**Source 文件**：
| 文件 | 说明 |
|------|------|
| `call.cj` | Call 类 API 实现 |
| `number_format_options.cj` | CallState, 选项类, 错误码 |
| `telephony_call_ffi.cj` | FFI 函数声明 |

**Cangjie 依赖**：
- `ability_cangjie_wrapper` - UI 能力上下文
- `cangjie_ark_interop` - 异常、FFI 工具、API 标签
- `hiviewdfx_cangjie_wrapper` - 日志

**外部依赖**：
- `call_manager:cj_telephony_call_ffi` - 原生通话管理 FFI

---

### 2. kit.TelephonyKit

**路径**：`kit/TelephonyKit/BUILD.gn:19`

```gn
ohos_cangjie_shared_library("kit.TelephonyKit") {
  sources = [ "index.cj" ]

  cj_deps = [
    "../../ohos/telephony/call:ohos.telephony.call",
  ]

  subsystem_name = "telephony"
  part_name = "telephony_cangjie_wrapper"
}
```

**Source 文件**：
| 文件 | 说明 |
|------|------|
| `index.cj` | 重新导出 `ohos.telephony.call.*` |

---

### 3. ohos.telephony

**路径**：`ohos/telephony/BUILD.gn:18`

```gn
ohos_cangjie_shared_library("ohos.telephony") {
  if (is_mingw || is_mac){
    sources = [ "../../mock/ohos.telephony.cj" ]
  } else {
    sources = [ "telephony.cj" ]
  }

  subsystem_name = "telephony"
  part_name = "telephony_cangjie_wrapper"
}
```

**说明**：此为命名空间包，仅包含空包定义。

---

### 4. copy_sdk_telephony_cangjie_libs

**路径**：`BUILD.gn:21`

```gn
copy_ohos_cangjie_sdk_api_lib("copy_sdk_telephony_cangjie_libs") {
  ohos_inputs = [
    "//base/telephony/telephony_cangjie_wrapper/ohos/telephony/call:ohos.telephony.call",
    "//base/telephony/telephony_cangjie_wrapper/ohos/telephony:ohos.telephony",
  ]
  kit_inputs = [
    "//base/telephony/telephony_cangjie_wrapper/kit/TelephonyKit:kit.TelephonyKit"
  ]
}
```

**用途**：将编译产物复制到 SDK 输出目录

---

## 依赖关系图

```
kit.TelephonyKit
    │
    ▼ cj_deps
ohos.telephony.call
    │
    ├── cj_external_deps ──────────────────────────┐
    │   ability_cangjie_wrapper                   │
    │   hiviewdfx_cangjie_wrapper                 │
    │   cangjie_ark_interop                       │
    │                                              │
    ▼ external_deps ─────────────────────────────┤
call_manager:cj_telephony_call_ffi (C++ 原生库)    │
                                                  │
ohos.telephony (独立，无依赖) ◄────────────────────┘
```

---

## 编译产物

| 产物 | 类型 | 说明 |
|------|------|------|
| `libohos.telephony.so` | .cj.o | 命名空间包（存根） |
| `libohos.telephony.call.so` | .cj.o | 核心呼叫管理 |
| `libkit.TelephonyKit.so` | .cj.o | Kit 层封装 |

**安装路径**：SDK sysroot 对应目录

**运行时加载**：
1. 应用导入 `kit.TelephonyKit`
2. 自动加载 `libkit.TelephonyKit.so`
3. 依赖链触发 `libohos.telephony.call.so` 加载
4. FFI 调用 `call_manager:cj_telephony_call_ffi` 原生库

---

## 平台条件编译

```gn
if (is_mingw || is_mac) {
  # Windows/Mac 构建使用 mock 存根
  sources = [ "../../mock/ohos.telephony.call.cj" ]
} else {
  # Linux (standard 设备) 使用真实实现
  sources = [ ...真实源码... ]
}
```

**说明**：仓颉工具链在非 Linux 主机上使用时，使用 mock 实现进行编译验证。

---

## 构建配置 (bundle.json)

**路径**：`bundle.json`

```json
{
  "component": {
    "name": "telephony_cangjie_wrapper",
    "subsystem": "telephony",
    "part_name": "telephony_cangjie_wrapper",
    "adapted_system_type": ["standard"],
    "rom": "180KB",
    "ram": "160KB",
    "deps": {
      "components": [
        "cangjie_ark_interop",
        "hiviewdfx_cangjie_wrapper",
        "ability_cangjie_wrapper",
        "call_manager"
      ]
    }
  }
}
```

**资源占用**：
- ROM: 180KB
- RAM: 160KB

---

## 常见构建问题

### 1. 依赖缺失

**症状**：GN gen 报错找不到 `call_manager:cj_telephony_call_ffi`

**解决**：确保 telephony_call_manager 子系统已添加到构建中

### 2. 平台不匹配

**症状**：在非 standard 设备上构建失败

**解决**：此封装仅支持 standard 设备，使用条件编译的 mock 模式

### 3. API Level 不匹配

**症状**：API 注解警告

**解决**：确保编译配置使用 API Level 22+
