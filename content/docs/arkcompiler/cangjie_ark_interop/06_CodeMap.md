# 代码地图

本文档提供 cangjie_ark_interop 项目代码导航，帮助快速定位关键功能对应的文件路径。

> **文档版本**: v1.0
> **最后更新**: 2025-02-07
> **用途**: 代码导航、功能定位

---

## 目录结构总览

```
arkcompiler/cangjie_ark_interop/
├── ohos/                          # 核心实现目录
│   ├── ark_interop/              # 互操作库 (26个 .cj 文件)
│   ├── ark_interop_helper/        # 互操作工具 (7个 .cj 文件)
│   ├── ark_interop_macro/         # 互操作宏 (9个 .cj 文件)
│   ├── ffi/                      # C 互操作库 (9个 .cj 文件)
│   ├── encoding/                  # JSON 序列化 (6个 .cj 文件)
│   ├── business_exception/        # 异常类
│   ├── callback_invoke/          # 回调工具
│   ├── labels/                   # API 标签
│   └── utf16string/              # UTF16 字符串 (C++)
├── kit/CangjieKit/               # Kit 接口
├── tools/config_gen/             # 配置生成工具
├── figures/                       # 文档图片
├── BUILD.gn                      # 根构建配置
├── bundle.json                   # 组件配置
└── test/                         # 测试用例 (不作为证据)
```

---

## 核心功能代码定位

### 跨语言互操作

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| ArkTS 运行时管理 | `ohos/ark_interop/js_runtime.cj:68` | `JSRuntime` |
| 执行上下文 | `ohos/ark_interop/jscontext.cj` | `JSContext` |
| 函数调用信息 | `ohos/ark_interop/js_func.cj:54` | `JSCallInfo` |
| 模块加载 | `ohos/ark_interop/js_module.cj` | `JSModule.import()` |
| JS 值封装 | `ohos/ark_interop/jsvalue.cj` | `JSValue` 系列 |
| JS 对象操作 | `ohos/ark_interop/jsobject.cj:89` | `JSObject` |
| JS 数组操作 | `ohos/ark_interop/jsarray.cj:42` | `JSArray` |
| 异常处理 | `ohos/ark_interop/js_exception.cj` | `jsTypeMisMatch()` |

### 互操作工具

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| ArkTS 函数调用 | `ohos/ark_interop_helper/ark_api_call.cj` | `callArkTS()` |
| 异步函数调用 | `ohos/ark_interop_helper/ark_api_call_async.cj` | `evalAsync()` |
| Console 支持 | `ohos/ark_interop_helper/console.cj` | `Console` |
| 定时器支持 | `ohos/ark_interop_helper/timer.cj` | `setTimeout()` |

### 互操作宏

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| 宏注解处理 | `ohos/ark_interop_macro/` | `@Interop[ArkTS]` |
| IDL 解析 | `ohos/ark_interop_macro/ark_idl_*.cj` | `IDL Parser` |

### C 互操作

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| FFI 回调 | `ohos/ffi/ffi_callback.cj` | `FFICallback` |
| FFI 数据 | `ohos/ffi/ffi_data.cj` | `FFIData` |
| 远程数据 | `ohos/ffi/remote_data_lite.cj` | `RemoteData` |

### 公共能力

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| 业务异常 | `ohos/business_exception/business_exception.cj:47` | `BusinessException` |
| 回调工具 | `ohos/callback_invoke/callback_object.cj` | `AsyncCallback` |
| JSON 序列化 | `ohos/encoding/json/` | `JsonEncoder/Decoder` |
| API 标签 | `ohos/labels/api_level.cj` | `@since`, `@until` |

### 原生模块

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| UTF16 字符串 | `ohos/utf16string/utf16string.cpp` | `Utf16String` |
| CFFI 接口 | `ohos/utf16string/utf16string_cffi.cpp` | `CFFI Wrapper` |
| DFX 统计 | `ohos/utf16string/utf16string_dfx.cpp` | `Statistics` |

---

## 构建配置代码定位

| 配置项 | 文件路径 | 说明 |
|--------|----------|------|
| 根构建配置 | `BUILD.gn:14-31` | 导入模板、SDK 模块定义 |
| 组件配置 | `bundle.json:12-82` | 子系统、特性、依赖配置 |
| 模块配置 | `ohos/BUILD.gn` | 各模块 targets 定义 |

---

## API 快速跳转

### JSRuntime API

```
初始化:     js_runtime.cj:68      → JSRuntime.init()
主上下文:    js_runtime.cj:72      → runtime.mainContext
NAPI 环境:  js_runtime.cj:80      → runtime.getNapiEnv()
```

### JSContext API

```
获取/创建:   jscontext.cj        → JSContext.getOrCreate()
创建函数:    jscontext.cj        → context.function()
提交任务:    jscontext.cj        → context.postJSTask()
线程检查:    jscontext.cj        → context.checkLifecycleAndThread()
```

### JSCallInfo API

```
参数数量:   js_func.cj:97        → callInfo.count
获取参数:   js_func.cj:104       → callInfo.get(index)
This 指针:  js_func.cj:98        → callInfo.thisArg
```

### JSObject API

```
创建对象:   jsobject.cj:127     → JSObject.create()
获取属性:   jsobject.cj:134     → obj.getProperty(key)
设置属性:   jsobject.cj:135     → obj.setProperty(key, value)
调用方法:   jsobject.cj:138     → obj.callMethod(name, args)
```

### JSArray API

```
构造函数:   jsarray.cj:150      → JSArray.init()
获取长度:   jsarray.cj:161      → array.length
获取元素:   jsarray.cj:167      → array.get(index)
设置元素:   jsarray.cj:168      → array.set(index, value)
```

---

## 错误码定位

| 错误码 | 常量名 | 位置 |
|--------|--------|------|
| 34300001 | 数组索引越界 | `business_exception.cj` |
| 34300002 | 外部错误 | `business_exception.cj` |
| 34300003 | 引用访问越界 | `js_func.cj:140` |
| 34300004 | 线程不匹配 | `js_func.cj:140` |
| 34300005 | 类型不匹配 | `js_exception.cj:169` |
| 34300014 | 创建引擎失败 | `js_runtime.cj` |
| 201 | ERR_NO_PERMISSION | `business_exception.cj:145` |
| 202 | ERR_NOT_SYSTEM_APP | `business_exception.cj:146` |
| 401 | ERR_PARAMETER_ERROR | `business_exception.cj:147` |
| 801 | ERR_NOT_SUPPOERTED | `business_exception.cj:148` |

---

## 代码导航图

### ArkTS → Cangjie 调用路径

```
ArkTS 函数调用
    ↓
@Interop[ArkTS] 宏 (编译时)
    ↓
胶水层代码 (C/C++)
    ↓
js_func.cj (JSCallInfo 参数解析)
    ↓
js_*.cj (类型转换)
    ↓
仓颉函数 (目标)
```

### Cangjie → ArkTS 调用路径

```
仓颉函数
    ↓
ark_interop_helper/ark_api_call.cj
    ↓
jscontext.cj (JSContext API)
    ↓
napi 调用
    ↓
ArkTS 函数执行
```

---

## 快速搜索关键词

| 搜索内容 | 推荐搜索位置 |
|----------|--------------|
| `JSRuntime` | `ohos/ark_interop/js_runtime.cj` |
| `JSContext` | `ohos/ark_interop/jscontext.cj` |
| `JSCallInfo` | `ohos/ark_interop/js_func.cj` |
| `JSValue` | `ohos/ark_interop/jsvalue.cj` |
| `JSObject` | `ohos/ark_interop/jsobject.cj` |
| `JSArray` | `ohos/ark_interop/jsarray.cj` |
| `BusinessException` | `ohos/business_exception/business_exception.cj` |
| `AsyncCallback` | `ohos/callback_invoke/callback_object.cj` |
| `@Interop[ArkTS]` | `ohos/ark_interop_macro/` |
| `checkLifecycleAndThread` | `ohos/ark_interop/jscontext.cj` |

---

## 相关文档

- [项目概览](./00_Overview.md)
- [API 参考](./01_API_Reference.md)
- [内部架构](./02_Architecture.md)
- [安全评审](./05_Security_Review.md)
