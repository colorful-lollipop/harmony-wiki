# 编译产物

本文档描述 cangjie_ark_interop 的编译产物清单、安装路径和运行时加载关系。

## 产物清单

### 共享库 (.so)

| 产物名称 | 源模块 | 描述 | 大小限制 |
|----------|--------|------|----------|
| `libark_interop.z.so` | ohos.ark_interop | 核心互操作库 | 1024KB ROM |
| `libark_interop_helper.z.so` | ohos.ark_interop_helper | 互操作工具库 | - |
| `libffi.z.so` | ohos.ffi | C 互操作库 | - |
| `libencoding.json.z.so` | ohos.encoding.json | JSON 序列化库 | - |
| `libutf16string.z.so` | utf16string | UTF16 字符串处理 | - |
| `libCangjieKit.z.so` | kit.CangjieKit | Kit 接口库 | - |

### 静态库 (.a)

| 产物名称 | 源模块 | 描述 |
|----------|--------|------|
| `libjson.static.a` | ohos.json.static | JSON 静态库 |
| `libbusiness_exception.static.a` | ohos.business_exception | 异常静态库 |
| `libcallback_invoke.static.a` | ohos.callback_invoke | 回调静态库 |
| `liblabels.static.a` | ohos.labels | 标签静态库 |

### 宏库

| 产物名称 | 源模块 | 描述 |
|----------|--------|------|
| `libark_interop_macro.so` | ohos.ark_interop_macro | 编译时互操作宏 |

---

## 安装路径

### 系统库路径

```
# 共享库
/system/lib/module/arkcompiler/
├── libark_interop.z.so
├── libark_interop_helper.z.so
├── libffi.z.so
├── libencoding.json.z.so
└── libutf16string.z.so

# Kit 库
/system/lib/module/arkcompiler/
└── libCangjieKit.z.so
```

### SDK 路径

```
# SDK 产物
prebuilts/sdk/
└── ohos/
    └── cangjie/
        └── api/
            ├── ohos.ffi/
            ├── ohos.encoding.json/
            ├── ohos.ark_interop/
            ├── ohos.ark_interop_helper/
            ├── ohos.labels/
            ├── ohos.business_exception/
            ├── ohos.callback_invoke/
            └── kit.CangjieKit/
```

---

## 运行时加载关系

```
应用程序 (HAP)
     │
     │ 加载
     ▼
┌─────────────────────────────────────────────────────────┐
│  ability_runtime                                          │
│  ─────────────────                                       │
│  动态库加载 (dlopen)                                      │
│       │                                                  │
│       ▼                                                  │
│  libark_interop_helper.z.so                              │
│       │                                                  │
│       ├── 依赖 ─────────────────────────────────────────┐ │
│       │                                                   │ │
│       ▼                                                   │ │
│  libark_interop.z.so                                     │ │
│       │                                                   │ │
│       ├── 依赖 ─────────────────────────────────────────┐ │ │
│       │                                                    │ │ │
│       ▼                                                    │ │ │
│  libutf16string.z.so                                     │ │ │
│       │                                                    │ │ │
│       ▼                                                    │ │ │
│  napi (系统组件)                                          │ │ │
│       │                                                    │ │ │
│       └──────────────────────────────────────────────────┘ │ │
│       │                                                   │ │
│       ▼                                                   │ │
│  libffi.z.so                                             │ │
│       │                                                    │ │
│       └──────────────────────────────────────────────────┘ │
│       │                                                   │
│       ▼                                                   │
│  libencoding.json.z.so                                   │
│       │                                                   │
│       ▼                                                   │
│  libCangjieKit.z.so (Kit 接口)                            │
│       │                                                   │
│       ▼                                                   │
│  应用代码 (调用 ohos.ark_interop API)                      │
└─────────────────────────────────────────────────────────┘
```

### 加载顺序

```
1. libark_interop_helper.z.so (入口库)
   │
   ├── 加载 libark_interop.z.so
   │   ├── 加载 libutf16string.z.so
   │   │   └── 链接 napi (运行时)
   │   │
   │   └── 加载 libffi.z.so
   │       └── 链接 napi:cj_bind_ffi (运行时)
   │
   ├── 加载 libencoding.json.z.so
   │
   └── 加载 libCangjieKit.z.so (Kit)
```

---

## 资源消耗

| 资源 | 限制 | 说明 |
|------|------|------|
| ROM | 1024KB | 代码+静态资源 |
| RAM | 2046KB | 运行时内存 |

---

## 符号导出

### ark_interop 核心导出

| 符号 | 类型 | 说明 |
|------|------|------|
| `JSRuntime` | 类 | ArkTS 运行时 |
| `JSContext` | 类 | 执行上下文 |
| `JSCallInfo` | 类 | 调用信息 |
| `JSValue` | 类 | JS 值封装 |
| `JSObject` | 类 | JS 对象 |
| `JSArray` | 类 | JS 数组 |

### ffi 导出

| 符号 | 类型 | 说明 |
|------|------|------|
| `FFIData` | 类 | FFI 数据基类 |
| `Callback*Param` | 类 | 回调函数模板 |

### encoding 导出

| 符号 | 类型 | 说明 |
|------|------|------|
| `JsonValue` | 类 | JSON 值 |
| `JsonObject` | 类 | JSON 对象 |
| `JsonArray` | 类 | JSON 数组 |
| `JsonParser` | 类 | JSON 解析器 |
