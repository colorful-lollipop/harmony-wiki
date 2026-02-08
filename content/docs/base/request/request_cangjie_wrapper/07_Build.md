# 构建配置

本文档描述 `request_cangjie_wrapper` 的构建系统、编译产物和配置选项。

---

## 7.1 GN 构建系统

### 构建文件位置

```
base/request/request_cangjie_wrapper/
├── BUILD.gn                    # 根构建入口
└── ohos/request/
    └── BUILD.gn               # 子组件构建配置
```

### 构建目标

| Target 名称 | 类型 | 产物 | 说明 |
|-------------|------|------|------|
| `ohos.request` | ohos_cangjie_shared_library | .so | 核心库 |
| `copy_sdk_request_cangjie_libs` | copy_ohos_cangjie_sdk_api_lib | SDK 产物 | SDK 复制 |

---

## 7.2 核心构建配置

### ohos.request 目标

**文件**: `ohos/request/BUILD.gn:18-44`

```gn
ohos_cangjie_shared_library("ohos.request") {

  # 源文件配置
  if (is_mingw || is_mac) {
    # Windows/Mac 平台使用 mock 实现
    sources = [ "../../mock/ohos.request.cj" ]
  } else {
    # Linux 平台使用完整实现
    sources = [
      "agent.cj",
      "error.cj",
      "ffi.cj",
    ]
  }

  # Cangjie 外部依赖
  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",  # 业务异常
    "cangjie_ark_interop:ohos.callback_invoke",     # 回调调用
    "cangjie_ark_interop:ohos.ffi",                 # FFI 支持
    "cangjie_ark_interop:ohos.labels",               # 注解标签
    "arkui_cangjie_wrapper:ohos.base",              # 基础类型
    "hiviewdfx_cangjie_wrapper:ohos.hilog",         # 日志
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",  # Ability 上下文
  ]

  # Native 外部依赖
  external_deps = [ "request:cj_request_ffi" ]

  # 子系统信息
  subsystem_name = "request"
  part_name = "request_cangjie_wrapper"
}
```

---

## 7.3 依赖关系

### Cangjie 依赖

| 依赖组件 | 子目标 | 用途 |
|----------|--------|------|
| cangjie_ark_interop | ohos.business_exception | 业务异常类 |
| cangjie_ark_interop | ohos.callback_invoke | 回调处理 |
| cangjie_ark_interop | ohos.ffi | FFI 接口 |
| cangjie_ark_interop | ohos.labels | API 注解 |
| arkui_cangjie_wrapper | ohos.base | 基础类型 |
| hiviewdfx_cangjie_wrapper | ohos.hilog | 日志 |
| ability_cangjie_wrapper | ohos.app.ability.ui_ability | 上下文 |

### Native 依赖

| 依赖组件 | 子目标 | 用途 |
|----------|--------|------|
| request | cj_request_ffi | 底层 FFI 接口 |

---

## 7.4 SDK 产物

### SDK 复制目标

**文件**: `BUILD.gn:19-21`

```gn
copy_ohos_cangjie_sdk_api_lib("copy_sdk_request_cangjie_libs") {
  ohos_inputs = request_cangjie_wrapper_packages_ohos
}
```

### SDK 产物清单

| 产物类型 | 说明 |
|----------|------|
| .so 库文件 | Cangjie 共享库 |
| .cji 文件 | Cangjie 接口定义 |
| .d.ts 文件 | TypeScript 类型声明 (如适用) |

---

## 7.5 平台差异

### Windows/Mac 平台

```gn
if (is_mingw || is_mac) {
  sources = [ "../../mock/ohos.request.cj" ]
}
```

**说明**:
- 使用 mock 实现作为占位符
- 功能受限，不支持完整功能
- 用于开发阶段编译通过

### Linux 平台

```gn
sources = [
  "agent.cj",
  "error.cj",
  "ffi.cj",
]
```

**说明**:
- 使用完整实现
- 支持全部功能
- 用于目标设备部署

---

## 7.6 组件配置

### bundle.json 定义

**文件**: `bundle.json`

```json
{
  "name": "@ohos/request_cangjie_wrapper",
  "description": "The request_cangjie_wrapper is a Cangjie API encapsulated on OpenHarmony...",
  "version": "6.1",
  "component": {
    "name": "request_cangjie_wrapper",
    "subsystem": "request",
    "adapted_system_type": [
      "standard"
    ],
    "rom": "400KB",
    "ram": "332KB",
    "build": {
      "sub_component": [
        "//base/request/request_cangjie_wrapper/ohos/request:ohos.request"
      ],
      "inner_kit": [
        {
          "name": "//base/request/request_cangjie_wrapper/ohos/request:ohos.request"
        },
        {
          "name": "//base/request/request_cangjie_wrapper:copy_sdk_request_cangjie_libs"
        }
      ]
    }
  }
}
```

### 资源占用

| 指标 | 大小 | 说明 |
|------|------|------|
| ROM | 400KB | 代码存储空间 |
| RAM | 332KB | 运行时内存 |

---

## 7.7 Feature 开关

### API Level 支持

| 特性 | 最小 API Level | 说明 |
|------|----------------|------|
| 基础功能 | 22 | Task/Config/Progress 等 |
| 高级配置 | 22 | token/gauge/precise 等 |
| 断点续传 | 22 | begins/ends/index |

### Syscap 要求

```cj
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Request.FileTransferAgent"
]
```

**要求**:
- 系统能力: `SystemCapability.Request.FileTransferAgent`
- 仅支持 standard 设备类型

---

## 7.8 构建命令

### 编译模块

```bash
# 编译整个子系统
hb build -f

# 编译单个模块
hb build -p request_cangjie_wrapper

# 仅编译 Cangjie 部分
./build.sh --parts request_cangjie_wrapper
```

### 编译产物位置

```
out/ohos-arm/release/
├── libs/
│   └── libohos_request_cangjie_wrapper.z.so
├──司
│   └── request_cangjie_wrapper/
│       └── ...
└── generated/
    └── ...
```

---

## 7.9 调试构建

### 启用日志

```bash
# 启用 hilog 日志
hilog -v &
```

### 查看依赖

```bash
# 查看构建依赖树
gn deps //base/request/request_cangjie_wrapper/ohos/request:ohos.request
```

---

## 7.10 常见构建问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 找不到 cj_request_ffi | request 子系统未构建 | 先构建 request 子系统 |
| mock 实现被使用 | 平台判断错误 | 确认 is_mingw/is_mac 标志 |
| API Level 不匹配 | 系统版本过低 | 升级 OpenHarmony 到 API 22+ |
| 依赖循环 | 依赖配置错误 | 检查 cj_external_deps 配置 |
