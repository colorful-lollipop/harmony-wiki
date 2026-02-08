# 附录：关键调用链

**文档目的**: 提供关键功能的详细调用链，便于源码阅读和调试  
**目标读者**: 核心模块开发者、调试工程师

---

## 1. 框架初始化调用链

### 1.1 UITest.setup() 完整调用链

```
应用启动 / 测试用例开始
    │
    ▼
UITest.setup() [ui_test_api.cj:51]
    │
    ├─► AtomicBool.compareAndSwap(false, true) [检查是否已初始化]
    │       │
    │       └─► 如果已初始化，直接返回
    │
    ├─► Systemparameter.get(TESTMODE_ENABLE, def: "0") [ui_test_api.cj:55]
    │       │
    │       ├─► FfiOHOSSysTemParameterGet(key, def) [systemparameter.cj:62]
    │       │       │
    │       │       └─► [FFI] 调用原生系统参数服务
    │       │
    │       └─► 检查返回值是否为 "1"
    │               │
    │               ├─► 是：继续
    │               └─► 否：记录警告日志
    │
    ├─► AbilityDelegatorRegistry.getAbilityDelegator() [ui_test_api.cj:59]
    │       │
    │       └─► 返回 AbilityDelegator 实例
    │
    ├─► delegator.getAppContext() [ui_test_api.cj:60]
    │       │
    │       └─► 返回 ApplicationContext
    │
    ├─► 构建 Token [ui_test_api.cj:62]
    │       │
    │       └─► "${appCtx.applicationInfo.name}@${Process.pid}@${Process.uid}@${appCtx.area.getValue()}"
    │
    ├─► LibC.mallocCString(token) [ui_test_api.cj:64]
    │       │
    │       └─► 分配 C 字符串内存
    │
    ├─► CJ_InitConnection(ctoken) [ui_test_api.cj:65]
    │       │
    │       └─► [FFI] 初始化与 uitest 服务的连接
    │
    ├─► delegator.executeShellCommand("uitest start-daemon ${token}", timeoutSecs: 3) [ui_test_api.cj:68]
    │       │
    │       ├─► [IPC] 向系统发送 Shell 命令
    │       │
    │       └─► 启动 uitest 守护进程
    │
    └─► 检查结果
            │
            ├─► result.exitCode == 0：成功
            └─► result.exitCode != 0：记录错误日志
```

**关键文件**: `ohos/ui_test/ui_test_api.cj:42-73`

---

## 2. 组件查找调用链

### 2.1 Driver.findComponent() 调用链

```
测试脚本
    │
    ▼
Driver.findComponent(on: On) [ui_test_api.cj:172]
    │
    ├─► 构建 JSON 参数: "[\"${on.ref}\"]"
    │       │
    │       └─► 引用格式如: "On#123"
    │
    ├─► ApiCallParams(DRIVER_FINDCOMPONENT, ref, params) [ui_test_api.cj:173]
    │       │
    │       ├─► mallocCString(DRIVER_FINDCOMPONENT) [ui_test_ffi.cj:48]
    │       │       └─► "Driver.findComponent"
    │       │
    │       ├─► mallocCString(ref) [ui_test_ffi.cj:50]
    │       │       └─► "Driver#xxx"
    │       │
    │       ├─► mallocCString(params) [ui_test_ffi.cj:56]
    │       │       └─► '["On#xxx"]'
    │       │
    │       └─► 返回 ApiCallParams 实例
    │
    ├─► getData(cjCallParams, DRIVER_FINDCOMPONENT) [ui_test_api.cj:174]
    │       │
    │       ├─► CJ_ApiCall(params) [ui_test_api.cj:76]
    │       │       │
    │       │       └─► [FFI] 调用 arkxtest
    │       │               │
    │       │               ├─► 查找 UI 树
    │       │               │
    │       │               ├─► 匹配组件
    │       │               │
    │       │               └─► 返回引用或 "null"
    │       │
    │       ├─► params.free() [释放内存]
    │       │
    │       ├─► 解析返回码 ret.code
    │       │       │
    │       │       ├─► code != 0: 抛出 BusinessException
    │       │       └─► code == 0: 继续
    │       │
    │       └─► 返回数据字符串
    │
    ├─► checkNull(data) [ui_test_api.cj:175]
    │       │
    │       ├─► data == "null": 返回 None
    │       └─► data != "null": 继续
    │
    └─► Component(data[1..data.size - 1]) [ui_test_api.cj:178]
            │
            ├─► checkRef(ref) [ui_test_api.cj:97]
            │       │
            │       ├─► ref.contains("#")：有效
            │       └─► !ref.contains("#")：抛出 BusinessException(17000004)
            │
            └─► 返回 Component 实例
```

**关键文件**: 
- `ohos/ui_test/ui_test_api.cj:172-179`
- `ohos/ui_test/ui_test_ffi.cj:23-82`

---

### 2.2 On 选择器构建调用链

```
测试脚本
    │
    ▼
On().text("确定") [ui_test_api.cj:1229]
    │
    ├─► 构建参数: "\"${eatEscape(txt)}\",${pattern.getValue()}"
    │       │
    │       └─► 如: '"确定",0'
    │
    ├─► ApiCallParams(ON_TEXT, ON_SEED_REF, params)
    │       │
    │       └─► ON_SEED_REF = "On#seed"
    │
    ├─► CJ_ApiCall(params) [FFI]
    │       │
    │       └─► 创建 On 对象
    │
    └─► 返回 On 实例 (引用如 "On#123")

链式调用示例:
On().text("确定").id("btn_ok").enabled(true)
    │
    ├─► 第一个调用创建基础 On 对象
    │
    ├─► 后续调用在现有 On 基础上添加条件
    │       │
    │       ├─► ApiCallParams(ON_ID, "On#xxx", '"btn_ok"')
    │       ├─► CJ_ApiCall() [FFI]
    │       └─► 返回新的 On 引用
    │
    └─► 最终返回组合条件的 On 对象
```

---

## 3. 组件操作调用链

### 3.1 Component.click() 调用链

```
测试脚本
    │
    ▼
Component.click() [ui_test_api.cj:1620]
    │
    ├─► ApiCallParams(COMPONENT_CLICK, ref, "[]")
    │       │
    │       └─► ref 如: "Component#xxx"
    │
    ├─► getData(params, COMPONENT_CLICK)
    │       │
    │       ├─► CJ_ApiCall(params) [FFI]
    │       │       │
    │       │       └─► [arkxtest] 执行点击操作
    │       │               │
    │       │               ├─► 向系统注入点击事件
    │       │               ├─► 等待操作完成
    │       │               └─► 返回结果
    │       │
    │       └─► 返回结果数据
    │
    └─► 返回 Unit
```

---

## 4. 窗口操作调用链

### 4.1 Driver.findWindow() 调用链

```
测试脚本
    │
    ▼
Driver.findWindow(filter: WindowFilter) [ui_test_api.cj:192]
    │
    ├─► 构建 JSON 参数 [ui_test_api.cj:193-225]
    │       │
    │       ├─► filter.bundleName? → "bundleName":"xxx"
    │       ├─► filter.title? → "title":"xxx"
    │       ├─► filter.focused? → "focused":true/false
    │       ├─► filter.active? → "active":true/false
    │       └─► filter.displayId? → "displayId":0
    │
    │       示例: '{"bundleName":"com.example","focused":true}'
    │
    ├─► ApiCallParams(DRIVER_FINDWINDOW, ref, params)
    │
    ├─► getData(params, DRIVER_FINDWINDOW)
    │       │
    │       ├─► CJ_ApiCall(params) [FFI]
    │       │       │
    │       │       └─► [arkxtest] 查找窗口
    │       │               │
    │       │               ├─► 查询窗口管理器
    │       │               ├─► 匹配过滤条件
    │       │               └─► 返回窗口引用
    │       │
    │       └─► 返回窗口引用或 "null"
    │
    ├─► checkNull(data)
    │       │
    │       ├─► "null" → 返回 None
    │       └─► 其他 → 继续
    │
    └─► UiWindow(data[1..data.size - 1])
            │
            ├─► checkRef(ref)
            └─► 返回 UiWindow 实例
```

**关键文件**: `ohos/ui_test/ui_test_api.cj:192-232`

---

## 5. 系统参数调用链

### 5.1 Systemparameter.get() 调用链

```
调用方
    │
    ▼
Systemparameter.get(key: String, def: String) [systemparameter.cj:52]
    │
    ├─► 长度检查 [systemparameter.cj:55]
    │       │
    │       ├─► key.size >= 128 → 抛出异常
    │       ├─► def.size >= 4096 → 抛出异常
    │       └─► 通过 → 继续
    │
    ├─► LibC.mallocCString(key) [systemparameter.cj:60]
    │
    ├─► LibC.mallocCString(def) [systemparameter.cj:61]
    │
    ├─► FfiOHOSSysTemParameterGet(cKey, cDef) [systemparameter.cj:62]
    │       │
    │       └─► [FFI] 调用系统参数服务
    │               │
    │               ├─► 检查权限
    │               ├─► 读取参数值
    │               └─► 返回 RetDataCString
    │
    ├─► LibC.free(cKey) [释放内存]
    │
    ├─► LibC.free(cDef) [释放内存]
    │
    ├─► 检查返回码 [systemparameter.cj:65]
    │       │
    │       ├─► code != 0 → 抛出 BusinessException
    │       └─► code == 0 → 继续
    │
    ├─► getStringAndFree(ret.data) [systemparameter.cj:69]
    │       │
    │       ├─► value.toString()
    │       ├─► LibC.free(value)
    │       └─► 返回结果字符串
    │
    └─► 返回参数值
```

**关键文件**: `ohos/ui_test/systemparameter.cj:48-71`

---

## 6. FFI 调用通用模式

### 6.1 ApiCall 通用调用模式

所有 UI 测试 API 遵循以下模式：

```cangjie
// 1. 定义 API 常量
const API_NAME = "ClassName.methodName"

// 2. 构建参数
let params = "[\"arg1\",arg2]"  // JSON 格式

// 3. 创建调用参数
let callParams = ApiCallParams(API_NAME, callerRef, params)

// 4. 调用 FFI
let result = unsafe { CJ_ApiCall(callParams) }

// 5. 释放参数内存
callParams.free()

// 6. 检查结果
if (result.code != 0) {
    // 错误处理
    throw BusinessException(result.code, errorMessage)
}

// 7. 解析返回数据
let data = result.data.toString()
unsafe { LibC.free(result.data) }

// 8. 返回处理后的结果
return parseResult(data)
```

**关键文件**: 
- `ohos/ui_test/ui_test_api.cj:75-84` (getData 函数)
- `ohos/ui_test/ui_test_ffi.cj:39-73` (ApiCallParams)

---

## 7. 对象生命周期调用链

### 7.1 Driver 对象生命周期

```
创建
    │
    ▼
Driver.create() [ui_test_api.cj:139]
    │
    ├─► ApiCallParams(DRIVER_CREATE, "", "[]")
    ├─► CJ_ApiCall() [FFI] → 原生创建 Driver 对象
    └─► 返回 Driver("Driver#xxx")
            │
            └─► checkRef("Driver#xxx") [验证包含 "#"]

使用
    │
    ▼
driver.findComponent(...)
driver.click(...)
... [多次调用]

销毁
    │
    ▼
~init() [ui_test_api.cj:120-122]
    │
    ├─► releaseRef(ref) [ui_test_api.cj:121]
    │       │
    │       ├─► LibC.mallocCString(ref) [ui_test_api.cj:88]
    │       ├─► CJ_UITestObjDelete(cref.value) [ui_test_api.cj:89]
    │       │       │
    │       │       └─► [FFI] 通知原生层释放对象
    │       │
    │       └─► [自动释放 CString]
    │
    └─► 对象销毁完成
```

**关键文件**: `ohos/ui_test/ui_test_api.cj:115-123`

---

## 8. 错误处理调用链

### 8.1 BusinessException 抛出流程

```
错误发生点
    │
    ▼
抛出 BusinessException [ui_test_api.cj:81]
    │
    ├─► BusinessException(code, message)
    │       │
    │       └─► 继承自 Exception
    │
    └─► 向上传播
            │
            ▼
    测试脚本捕获
            │
            ▼
try {
    driver.assertComponentExist(on)
} catch (e: BusinessException) {
    // e.code: 17000003
    // e.message: "assertComponentExist failed: ..."
}
```

---

## 9. 日志记录调用链

### 9.1 日志输出流程

```
代码中记录日志
    │
    ▼
TEST_LOG.warn("message") [ui_test_api.cj:57]
    │
    ├─► TEST_LOG = HilogChannel(3, 0xD003100, "CJ-UITEST") [ui_test_api.cj:32]
    │
    ├─► 调用 HiLog 接口 [通过 hiviewdfx_cangjie_wrapper]
    │
    └─► 输出到系统日志缓冲区
            │
            ▼
    通过 hdc 查看
            │
            ▼
hdc shell hilog | grep CJ-UITEST
```

**关键文件**: 
- `ohos/ui_test/ui_test_api.cj:31-32` (日志初始化)
- `ohos/ui_test/ui_test_api.cj:57,70` (日志使用)

---

## 10. 调用链图索引

| 调用链 | 场景 | 关键文件 |
|--------|------|----------|
| 框架初始化 | 应用启动时 | `ui_test_api.cj:42-73` |
| 组件查找 | findComponent | `ui_test_api.cj:172-179` |
| 组件操作 | click, inputText | `ui_test_api.cj:1620-1680` |
| 窗口查找 | findWindow | `ui_test_api.cj:192-232` |
| 系统参数 | get/set | `systemparameter.cj:48-91` |
| FFI 调用 | 通用模式 | `ui_test_api.cj:75-84` |
| 对象生命周期 | 创建/销毁 | `ui_test_api.cj:115-123` |
| 错误处理 | 异常抛出 | `ui_test_api.cj:81` |

---

*本文档基于代码仓库静态分析生成*  
*证据位置: ohos/ui_test/*.cj*
