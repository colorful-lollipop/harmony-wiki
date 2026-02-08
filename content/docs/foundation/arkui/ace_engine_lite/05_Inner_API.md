# 内部 API 参考

## 目的

本文档列出 ace_engine_lite 的内部 API 接口，包括核心类、稳定性评估和依赖关系。这些 API 主要供框架内部模块使用。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 内部模块间的接口（不对外暴露给 JS 应用）

---

## 核心类接口

### 1. Component 类

**位置**：`frameworks/src/core/components/component.h`  
**稳定性**：✅ **稳定**（核心基类，所有组件继承）

**接口定义**：
```cpp
class Component {
public:
    // 构造函数
    Component(JSIValue options, JSIValue children, AppStyleManager *styleManager);
    
    // 虚析构函数（子类实现）
    virtual ~Component();
    
    // 属性设置（子类实现）
    virtual bool SetPrivateAttribute(uint16_t attrKeyId, JSIValue attrValue);
    
    // 事件注册（子类实现）
    virtual bool RegisterPrivateEventListener(uint16_t eventTypeId, 
                                         JSIValue funcValue, bool isStopPropagation);
    
    // 渲染（子类实现）
    virtual UIView* RenderComponent(bool isRenderTree);
    
    // 样式应用
    virtual void ApplyStyles(AppStyleManager *styleManager);
    
    // 生命周期
    virtual void Release();
};
```

**证据**：`frameworks/src/core/components/component.h`

---

### 2. Router 类

**位置**：`frameworks/src/core/router/js_router.h`  
**稳定性**：✅ **稳定**（核心路由机制）

**接口定义**：
```cpp
class Router {
public:
    // 页面替换（异步支持）
    JSIValue Replace(JSIValue object, bool async = true);
};
```

**证据**：`frameworks/src/core/router/js_router.h:26-54`

---

### 3. AppStyleManager 类

**位置**：`frameworks/src/core/stylemgr/app_style_manager.h`  
**稳定性**：✅ **稳定**（样式管理核心）

**接口定义**：
```cpp
class AppStyleManager {
public:
    // 初始化样式表
    void InitStyleSheet(JSIValue jsStyleSheetObj);
    
    // 应用样式到组件
    void ApplyComponentStyles(const JSIValue options, Component& curr);
    
    // 处理静态样式
    void HandleStaticStyle(const JSIValue options, Component& curr);
    
    // 处理动态样式
    void HandleDynamicStyle(const JSIValue options, Component& curr);
    
    // 处理 ID 选择器
    void HandleIDSelectors(const JSIValue options, Component& curr);
    
    // 处理 Class 选择器
    void HandleClassSelectors(const JSIValue options, Component& curr);
};
```

**证据**：`frameworks/src/core/stylemgr/app_style_manager.h:21-47`

---

### 4. JsAppContext 类

**位置**：`frameworks/src/core/context/js_app_context.h`  
**稳定性**：✅ **稳定**（应用上下文管理）

**接口定义**：
```cpp
class JsAppContext {
public:
    // 单例访问
    static JsAppContext* GetInstance();
    
    // JS 代码执行
    JSIValue Eval(char *fullPath, size_t fullPathLength, bool isAppEval) const;
    
    // 渲染 JS 视图模型
    JSIValue Render(JSIValue viewModel) const;
    
    // 应用终止
    void TerminateAbility();
    
    // Ability 信息获取
    const char* GetCurrentBundleName();
    const char* GetCurrentAbilityPath();
    
    // 状态管理
    void SetCurrentAbilityInfo(const char *abilityPath, const char *bundleName, uint16_t token);
};
```

**证据**：`frameworks/src/core/context/js_app_context.h:35-217`

---

### 5. ModuleManager 类

**位置**：`frameworks/module_manager/module_manager.h`  
**稳定性**：✅ **稳定**（模块加载核心）

**接口定义**：
```cpp
class ModuleManager {
public:
    // 单例访问
    static ModuleManager* GetInstance();
    
    // 模块请求
    JSIValue RequireModule(const char * const moduleName);
    
    // 模块清理
    void CleanUpModule();
    
    // 终止回调
    void OnTerminate();
    
    // 产品模块设置
    void SetProductModulesGetter(ProductModulesGetter getter);
    void SetPrivateModulesGetter(PrivateModulesGetter getter);
    void SetBundleNameGetter(BundleNameGetter getter);
};
```

**证据**：`frameworks/module_manager/module_manager.h:33-88`

---

### 6. JSI 类

**位置**：`interfaces/inner_api/builtin/jsi/jsi.h`  
**稳定性**：✅ **稳定**（JerryScript 封装层）

**接口定义**：
```cpp
class JSI {
public:
    // 值创建
    static JSIValue CreateObject();
    static JSIValue CreateNumber(double value);
    static JSIValue CreateString(const char *value);
    static JSIValue CreateBoolean(bool value);
    static JSIValue CreateUndefined();
    static JSIValue CreateNull();
    static JSIValue CreateArray();
    static JSIValue CreateErrorWithCode(uint16_t code, const char *msg);
    static JSIValue CreateArrayBuffer(const void *buffer, size_t length);
    
    // 属性操作
    static void SetProperty(JSIValue object, JSIValue key, JSIValue value);
    static void SetNamedProperty(JSIValue object, const char * const propName, JSIValue value);
    static void SetNumberProperty(JSIValue object, const char * const propName, double value);
    static void SetStringProperty(JSIValue object, const char * const propName, const char *value);
    static void SetBooleanProperty(JSIValue object, const char * const propName, bool value);
    static void SetFuncProperty(JSIValue object, const char * const propName, JSIFunctionHandler handler);
    
    // 属性获取
    static JSIValue GetNamedProperty(JSIValue object, const char * const propName);
    static JSIValue GetProperty(JSIValue object, JSIValue key);
    static JSIValue GetPropertyByIndex(JSIValue object, uint32_t index);
    
    // 类型检查
    static bool ValueIsUndefined(JSIValue value);
    static bool ValueIsNull(JSIValue value);
    static bool ValueIsBoolean(JSIValue value);
    static bool ValueIsNumber(JSIValue value);
    static bool ValueIsString(JSIValue value);
    static bool ValueIsObject(JSIValue value);
    static bool ValueIsFunction(JSIValue value);
    static bool ValueIsArray(JSIValue value);
    static bool ValueIsArrayBuffer(JSIValue value);
    static bool ValueIsSymbol(JSIValue value);
    
    // 值转换
    static double ValueToNumber(JSIValue value);
    static int16_t ValueToInteger(JSIValue value);
    static bool ValueToBoolean(JSIValue value);
    static char* MallocStringOf(JSIValue source);
    
    // 函数调用
    static JSIValue CallJSFunction(const JSIValue func, const JSIValue context, 
                                 const JSIValue args[], uint8_t argsCount);
    static JSIValue CallJSFunctionOnRoot(const JSIValue func, const JSIValue args[], uint8_t argsCount);
    
    // 内存管理
    static void ReleaseValue(JSIValue value, ...);
};
```

**证据**：`interfaces/inner_api/builtin/jsi/jsi.h:82-325`

**稳定性评估**：✅ **稳定** - 这是对外暴露的 JSI 接口，所有 JS-C++ 绑定都依赖此接口

---

## 模块间依赖

### 依赖层次结构

```
┌─────────────────────────────────────────────────────────┐
│           Components (UI 组件层)                 │
│  ┌─────────┐                                      │
│  │ 依赖     │                                      │
│  ▼         │                                      │
│  ┌─────────────────────────────────────┐        │
│  │         │                         │        │
│  ▼         │                         ▼        │
│  AppStyleManager               JSI (Jerry)  │
│  (样式管理)                      (JS 引擎封装) │
│  └─────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│           Context (上下文层)                      │
│  ┌─────────┐  ┌─────────┐                      │
│  │ 依赖     │  │ 依赖     │                      │
│  ▼         │  ▼         │                      │
│  ┌─────────────┐  ┌─────────────────────────┐       │
│  │Router       │  │ JsAppContext               │       │
│  │             │  │ ModuleManager               │       │
│  └─────────────┘  └─────────────────────────┘       │
│  └─────────────────┐                                │
│  ┌─────────────┐                                │
│  ▼             │                                │
│  ┌─────────────────────────────────┐               │
│  │ JSI & JerryScript        │               │
│  └─────────────────────────────────┘               │
└─────────────────────────────────────────────────────────┘
```

**依赖方向**：
- **Components** → **AppStyleManager**（单向，不反向依赖）
- **Components** → **JSI**（单向，值操作）
- **Context** → **Router**（单向）
- **Context** → **ModuleManager**（单向）
- **Context** → **JSI**（单向）

**证据**：各模块的头文件引用

---

## 接口稳定性评估

### 稳定接口（Stable API）

以下接口标记为 **稳定**，可在框架内安全使用：

| 接口 | 位置 | 稳定性 | 说明 |
|------|--------|--------|------|
| `JSI` | `interfaces/inner_api/builtin/jsi/jsi.h` | ✅ 稳定 | 对外暴露的 JSI 封装层 |
| `Component` | `frameworks/src/core/components/component.h` | ✅ 稳定 | 所有 UI 组件的基类 |
| `Router` | `frameworks/src/core/router/js_router.h` | ✅ 稳定 | 页面路由核心 |
| `AppStyleManager` | `frameworks/src/core/stylemgr/app_style_manager.h` | ✅ 稳定 | 样式管理核心 |
| `JsAppContext` | `frameworks/src/core/context/js_app_context.h` | ✅ 稳定 | 应用上下文管理 |
| `ModuleManager` | `frameworks/module_manager/module_manager.h` | ✅ 稳定 | 模块加载管理 |

### 内部接口（Internal API）

以下接口标记为 **内部**，仅在特定模块内部使用，不建议跨模块调用：

| 接口 | 位置 | 稳定性 | 说明 |
|------|--------|--------|------|
| `ComponentFactory` | `frameworks/src/core/components/component_factory.h` | ⚠️ 内部 | 组件工厂，内部使用 |
| `AsyncTaskManager` | `frameworks/src/core/base/async_task_manager.h` | ⚠️ 内部 | 异步任务管理 |
| `ProductAdapter` | `frameworks/src/core/base/product_adapter.h` | ⚠️ 内部 | 产品适配层，平台特定 |
| `AceAbility` | `frameworks/src/core/context/ace_ability.h` | ⚠️ 内部 | Ability 实现，平台特定 |
| `SliteAceAbility` | `frameworks/src/core/context/slite_ace_ability.h` | ⚠️ 内部 | LiteOS-M 专用 Ability |

---

## 数据结构

### JSIValue 类型

**定义**：`typedef jerry_value_t JSIValue;`

**说明**：JSIValue 代表任意 JavaScript 值，包括：
- undefined
- null
- boolean
- number
- string
- object
- array
- function
- symbol（可选，通过宏控制）

**证据**：`interfaces/inner_api/builtin/jsi/jsi_types.h`

### Component 生命周期

```
创建 (Constructor)
  ↓
解析属性 (Parse)
  ↓
应用样式 (ApplyStyles)
  ↓
渲染 (Render)
  ↓
事件注册 (RegisterEventListener)
  ↓
销毁 (Destructor)
```

**证据**：`frameworks/src/core/components/component.h`

### Module 生命周期

```
初始化 (InitXXXModule)
  ↓
注册 API (JSI::SetModuleAPI)
  ↓
清理 (CleanUpModule)
  ↓
终止 (OnTerminate)
```

**证据**：`frameworks/module_manager/module_manager.cpp`

---

## 可替换点（Extension Points）

### 产品模块扩展

**位置**：`frameworks/module_manager/module_manager.h:74-82`

```cpp
void SetProductModulesGetter(ProductModulesGetter getter);
```

**用途**：允许产品厂商注册自定义 JS 模块

**证据**：`frameworks/module_manager/module_manager.h:74-82`

### 私有模块扩展

**位置**：`frameworks/module_manager/module_manager.h:81-82`

```cpp
void SetPrivateModulesGetter(PrivateModulesGetter getter);
```

**用途**：允许应用按 bundleName 隔离私有模块

**证据**：`frameworks/module_manager/module_manager.h:81-82`

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - JSI 详解
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [04_JS_API.md](04_JS_API.md) - 对外 JS API
- [06_GN_Targets.md](06_GN_Targets.md) - 构建目标
