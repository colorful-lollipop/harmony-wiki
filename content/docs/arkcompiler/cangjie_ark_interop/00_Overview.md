# 项目概览

## 项目定位

**cangjie_ark_interop** 是 OpenHarmony `arkcompiler` 子系统下的核心组件，提供 **仓颉 (Cangjie)** 编程语言与 **ArkTS** 之间的双向跨语言互操作能力。

## 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 跨语言函数调用 | Cangjie ↔ ArkTS 双向调用 | `ohos/ark_interop/` |
| 类型转换 | JS 类型 ↔ Cangjie 类型映射 | `ohos/ark_interop/js_interop_type.cj` |
| 异步互操作 | Promise/Callback 支持 | `ohos/ark_interop_helper/ark_api_call_async.cj` |
| 声明式互操作宏 | `@Interop[ArkTS]` 自动生成胶水代码 | `ohos/ark_interop_macro/` |
| JSON 序列化 | JSON ↔ Cangjie 对象转换 | `ohos/encoding/json/` |
| C 语言互操作 | FFI 绑定能力 | `ohos/ffi/` |

## 系统依赖

| 依赖组件 | 用途 | 来源 |
|----------|------|------|
| **napi** | 调用 ArkTS 虚拟机接口 | OpenHarmony 系统组件 |
| **ability_runtime** | 动态库加载与模块管理 | OpenHarmony 系统组件 |
| **hiviewdfx_cangjie_wrapper** | 日志输出 | OpenHarmony 系统组件 |
| **arkui_cangjie_wrapper** | UI 基础能力 | OpenHarmony 系统组件 |

## 运行环境

- **目标设备**: standard (标准设备)
- **系统能力**: `SystemCapability.ArkCompiler.CangjieInterop`
- **API Level**: 22+
- **ROM 限制**: 1024KB
- **RAM 限制**: 2046KB

## 目录结构

```
arkcompiler/cangjie_ark_interop/
├── ohos/                          # 核心实现
│   ├── ark_interop/              # 互操作库 (26个 .cj 文件)
│   ├── ark_interop_helper/        # 互操作工具 (7个 .cj 文件)
│   ├── ark_interop_macro/         # 互操作宏 (9个 .cj 文件)
│   ├── ffi/                      # C 互操作库
│   ├── encoding/                  # JSON 序列化
│   ├── business_exception/        # 异常类
│   ├── callback_invoke/          # 回调工具
│   ├── labels/                   # API 标签
│   └── utf16string/              # UTF16 字符串 (C++)
├── kit/CangjieKit/               # Kit 接口导出
├── tools/config_gen/             # 配置生成工具
├── figures/                       # 文档图片
├── BUILD.gn                      # 根构建配置
├── bundle.json                   # 组件配置
└── README_zh.md / README.md       # 项目说明
```

## 核心组件职责

### ohos.ark_interop (互操作库)

**职责**: 提供 ArkTS 运行时的访问能力

| 类/模块 | 职责 | 主要方法 |
|---------|------|----------|
| `JSRuntime` | 表示 ArkTS 运行时实例 | `init()`, `getNapiEnv()` |
| `JSContext` | 执行上下文管理 | 模块加载, JSValue 访问 |
| `JSCallInfo` | 函数调用信息 | `count`, `thisArg` |
| `JSValue` 系列 | 类型封装 | `asString()`, `asNumber()` 等 |

**证据来源**: `ohos/ark_interop/js_runtime.cj:68`, `jscontext.cj`, `js_func.cj`

### ohos.ark_interop_helper (互操作工具)

**职责**: 提供高级互操作 API 和工具函数

| 模块 | 职责 |
|------|------|
| `ark_api_call.cj` | ArkTS 函数调用 |
| `ark_api_call_async.cj` | 异步函数调用 (Promise) |
| `console.cj` | Console 对象支持 |
| `timer.cj` | 定时器支持 |

### ohos.ark_interop_macro (互操作宏)

**职责**: 编译时自动生成互操作代码

```cangjie
@Interop[ArkTS]
public func myFunction(): Int { ... }
```

自动生成:
- ArkTS 接口声明 (`.d.ts`)
- 胶水层代码 (C/C++)

## 使用场景

### 场景一: ArkTS 调用仓颉

```
ArkTS (JS) → @Interop[ArkTS] 宏 → 胶水代码 → 仓颉函数
```

### 场景二: 仓颉调用 ArkTS

```
仓颉函数 → ark_interop API → napi 调用 → ArkTS 模块
```

## 版本信息

- **当前状态**: Beta
- **许可协议**: Apache License 2.0
- **版本**: 6.1 (`bundle.json:4`)

## 相关文档

- [API 参考](./01_API_Reference.md)
- [架构文档](./02_Architecture.md)
- [构建文档](./03_Build_System.md)
