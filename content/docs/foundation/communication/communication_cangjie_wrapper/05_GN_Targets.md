# GN Targets 与编译产物

## 目的

本文档描述 `communication_cangjie_wrapper` 的 GN 构建目标、依赖关系和产物输出。

## 适用范围

- 构建工程师
- 系统集成人员

## GN 文件概览

| 文件 | 路径 | 职责 |
|------|------|------|
| BUILD.gn | 根目录 | 定义 SDK 库复制目标 |
| BUILD.gn | ohos/rpc/ | 定义核心共享库 |
| BUILD.gn | kit/IPCKit/ | 定义 Kit 层共享库 |

## 构建目标详解

### 1. 根 BUILD.gn

**文件路径**: `//foundation/communication/communication_cangjie_wrapper/BUILD.gn`

#### 目标: copy_sdk_communication_cangjie_libs

```gn
copy_ohos_cangjie_sdk_api_lib("copy_sdk_communication_cangjie_libs") {
  ohos_inputs = [
    "//foundation/communication/communication_cangjie_wrapper/ohos/rpc:ohos.rpc"
  ]
  kit_inputs = [
    "//foundation/communication/communication_cangjie_wrapper/kit/IPCKit:kit.IPCKit"
  ]
}
```

**目标类型**: `copy_ohos_cangjie_sdk_api_lib`

**输入**:
- `ohos_inputs`: ohos.rpc 目标
- `kit_inputs`: kit.IPCKit 目标

**输出**: SDK 库文件（复制到 SDK 目录）

**在 bundle.json 中的引用**:
```json
"inner_kits": [
  {
    "name": "//foundation/communication/communication_cangjie_wrapper:copy_sdk_communication_cangjie_libs"
  },
  {
    "name": "//foundation/communication/communication_cangjie_wrapper:copy_sdk_communication_cangjie_libs_kit"
  }
]
```

---

### 2. ohos/rpc/BUILD.gn

**文件路径**: `//foundation/communication/communication_cangjie_wrapper/ohos/rpc/BUILD.gn`

#### 目标: ohos.rpc

```gn
ohos_cangjie_shared_library("ohos.rpc") {
  # 条件编译
  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.rpc.cj" ]
  } else {
    sources = [
      "ashmem.cj",
      "cj_rpc_ffi.cj",
      "parcelable.cj",
      "cj_rpc_utils.cj",
      "iremote_object.cj",
      "message_sequence.cj",
      "remote_object.cj",
      "remote_proxy.cj",
      "request_result.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.business_exception",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "ipc:cj_ipc_ffi" ]

  subsystem_name = "communication"
  part_name = "communication_cangjie_wrapper"
}
```

**目标类型**: `ohos_cangjie_shared_library`

**目标名称**: `ohos.rpc`

**源文件列表**:

| 文件 | 行号 | 说明 |
|------|------|------|
| ashmem.cj | 24 | 匿名共享内存实现 |
| cj_rpc_ffi.cj | 26 | FFI 外部函数声明 |
| parcelable.cj | 27 | Parcelable 接口 |
| cj_rpc_utils.cj | 28 | 错误码与工具 |
| iremote_object.cj | 29 | IRemoteObject 接口 |
| message_sequence.cj | 30 | MessageSequence 实现 |
| remote_object.cj | 31 | RemoteObject 实现 |
| remote_proxy.cj | 32 | RemoteProxy 实现 |
| request_result.cj | 33 | 数组类型定义 |

**条件编译**:
- Windows/Mac (`is_mingw || is_mac`): 使用 `mock/ohos.rpc.cj`
- 其他平台: 使用完整实现

**Cangjie 外部依赖** (`cj_external_deps`):

| 依赖 | 目标 | 用途 |
|------|------|------|
| cangjie_ark_interop | ohos.ffi | FFI 基础功能 |
| cangjie_ark_interop | ohos.labels | APILevel 注解 |
| cangjie_ark_interop | ohos.business_exception | 异常定义 |
| hiviewdfx_cangjie_wrapper | ohos.hilog | 日志接口 |

**外部依赖** (`external_deps`):

| 依赖 | 用途 |
|------|------|
| ipc:cj_ipc_ffi | 底层 IPC C/C++ 实现 |

**子系统/部件**: `communication` / `communication_cangjie_wrapper`

**在 bundle.json 中的引用**:
```json
"sub_component": [
  "//foundation/communication/communication_cangjie_wrapper/ohos/rpc:ohos.rpc"
]
```

---

### 3. kit/IPCKit/BUILD.gn

**文件路径**: `//foundation/communication/communication_cangjie_wrapper/kit/IPCKit/BUILD.gn`

#### 目标: kit.IPCKit

```gn
ohos_cangjie_shared_library("kit.IPCKit") {
  sources = ["index.cj"]

  cj_deps = ["../../ohos/rpc:ohos.rpc"]

  subsystem_name = "communication"
  part_name = "communication_cangjie_wrapper"
}
```

**目标类型**: `ohos_cangjie_shared_library`

**目标名称**: `kit.IPCKit`

**源文件**:
- `index.cj` - 统一导出 `ohos.rpc.*`

**Cangjie 依赖** (`cj_deps`):
- `../../ohos/rpc:ohos.rpc`

**依赖关系**:
```
kit.IPCKit
    └── ohos.rpc (cj_deps)
```

**在 bundle.json 中的引用**:
```json
"sub_component": [
  "//foundation/communication/communication_cangjie_wrapper/kit/IPCKit:kit.IPCKit"
]
```

---

## 构建依赖图

```
┌─────────────────────────────────────────────────────────────┐
│  copy_sdk_communication_cangjie_libs                        │
│  (根 BUILD.gn)                                              │
└──────────────┬──────────────────────────────┬───────────────┘
               │                              │
       ohos_inputs                   kit_inputs
               │                              │
               ▼                              ▼
┌──────────────────────────┐      ┌──────────────────────────┐
│  ohos.rpc                │      │  kit.IPCKit              │
│  (ohos/rpc/BUILD.gn)     │      │  (kit/IPCKit/BUILD.gn)   │
│                          │      │                          │
│  cj_external_deps:       │      │  cj_deps:                │
│  - cangjie_ark_interop   │◄─────│  - ohos.rpc              │
│  - hiviewdfx_cangjie_... │      │                          │
│                          │      │                          │
│  external_deps:          │      │                          │
│  - ipc:cj_ipc_ffi        │      │                          │
└──────────────────────────┘      └──────────────────────────┘
```

---

## 产物输出

### 预期编译产物

基于 GN 目标和 OpenHarmony 构建规则，预期产物：

| 目标 | 产物类型 | 预期文件名 | 安装路径 |
|------|----------|------------|----------|
| ohos.rpc | 共享库 | `libohos.rpc.so` | `/system/lib/` 或 `/vendor/lib/` |
| kit.IPCKit | 共享库 | `libkit.IPCKit.so` | SDK 目录 |
| copy_sdk_communication_cangjie_libs | 文件集合 | 上述库文件 | SDK 输出目录 |

### 运行时加载关系

```
Cangjie 应用
    └── 链接 libkit.IPCKit.so (开发时)
            └── 运行时依赖 libohos.rpc.so
                    └── 运行时依赖 libipc_cj_ffi.so (外部组件)
                            └── Binder / SoftBus 驱动
```

### bundle.json 产物信息

```json
{
  "component": {
    "rom": "300KB",
    "ram": "228KB"
  }
}
```

**ROM 占用**: 约 300KB（包含两个共享库）
**RAM 占用**: 约 228KB（运行时内存）

---

## 构建命令示例

### 完整构建

```bash
# 在 OpenHarmony 源码根目录
./build.sh --product {product_name} \
    --target \"//foundation/communication/communication_cangjie_wrapper:copy_sdk_communication_cangjie_libs\"
```

### 仅构建核心库

```bash
./build.sh --product {product_name} \
    --target \"//foundation/communication/communication_cangjie_wrapper/ohos/rpc:ohos.rpc\"
```

### 仅构建 Kit 层

```bash
./build.sh --product {product_name} \
    --target \"//foundation/communication/communication_cangjie_wrapper/kit/IPCKit:kit.IPCKit\"
```

---

## 关键配置项

### 条件编译变量

| 变量 | 用途 | 影响 |
|------|------|------|
| `is_mingw` | Windows 平台检测 | 使用 Mock 实现 |
| `is_mac` | macOS 平台检测 | 使用 Mock 实现 |

### 外部依赖检查

构建前需确保以下组件已编译：

```bash
# 检查 cangjie_ark_interop
ls out/{product}/gen/cangjie_ark_interop/

# 检查 hiviewdfx_cangjie_wrapper  
ls out/{product}/gen/hiviewdfx_cangjie_wrapper/

# 检查 ipc
ls out/{product}/obj/foundation/communication/ipc/
```

---

## 参考文档

- [目录结构](02_Directory_Structure.md) - 源代码组织
- [内部 API](04_Internal_API.md) - 模块接口
- [编译产物](07_Build_Artifacts.md) - 运行时产物详情
