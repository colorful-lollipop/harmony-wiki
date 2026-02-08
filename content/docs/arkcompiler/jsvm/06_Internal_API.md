# 内部 API

> 模块接口、依赖方向、稳定性、可替换点

## 目的与适用范围

### 目的
本文档描述 JSVM 的内部 API，包括模块接口、依赖方向、稳定性等。

### 适用范围
- 需要理解模块间交互的开发者
- 需要进行模块重构的开发者
- 需要进行性能优化的开发者

---

## 模块划分

### 1. API 层（API Layer）

**职责**: 提供稳定的 C 语言接口。

**接口**: `interface/kits/jsvm.h`

**实现**: `src/js_native_api_v8.cpp`

**依赖**: Core 层

**稳定性**: 稳定（Public API）

**证据位置**:
- `interface/kits/jsvm.h` - API 声明
- `src/js_native_api_v8.cpp` - API 实现

### 2. Core 层（Core Layer）

**职责**: 实现 JSVM API 到 V8 的桥接。

**接口**:
- Environment 管理
- Reference 管理
- Scope 管理

**实现**:
- `src/js_native_api_v8.cpp`
- `src/jsvm_env.cpp`
- `src/jsvm_reference.cpp`

**依赖**: V8 Engine, Platform Layer

**稳定性**: 较稳定（内部 API）

**证据位置**:
- `src/js_native_api_v8.cpp` - Core 实现
- `jsvm.gni:16-20` - 源文件列表

### 3. Inspector 模块

**职责**: 提供调试支持。

**接口**: JSVM Inspector API

**实现**: `src/inspector/*.cpp`

**依赖**: Core 层, V8 Inspector

**稳定性**: 较稳定

**证据位置**:
- `src/inspector/js_native_api_v8_inspector.cpp` - Inspector API
- `jsvm.gni:22-27` - Inspector 源文件列表

### 4. Platform 层

**职责**: 提供平台抽象。

**接口**: Platform 抽象接口

**实现**:
- `src/platform/platform.cpp`
- `src/platform/platform_ohos.cpp`

**依赖**: 操作系统

**稳定性**: 可替换（多平台支持）

**证据位置**:
- `src/platform/platform.h` - Platform 抽象
- `src/platform/platform_ohos.cpp` - OpenHarmony 实现

---

## 依赖方向

### 依赖图

```
应用层（Application）
    ↓ 依赖
API 层（JSVM-API）
    ↓ 依赖
Core 层（JSVM Core）
    ↓ 依赖
├── Inspector 模块
├── Platform 层
└── V8 Engine
```

**证据位置**:
- `BUILD.gn` - 依赖关系
- `jsvm.gni` - 源文件列表

---

## 内部接口

### Environment 管理

**接口**: JSVM_Env

**实现位置**: `src/jsvm_env.cpp`

**主要函数**:
```cpp
JSVM_Env CreateJSVMEnv(v8::Local<v8::Context> context);
void DestroyJSVMEnv(JSVM_Env env);
```

**稳定性**: 内部 API，可能变更

**证据位置**: `src/jsvm_env.cpp:1-137`

### Reference 管理

**接口**: JSVM_Ref

**实现位置**: `src/jsvm_reference.cpp`

**主要函数**:
```cpp
JSVM_Ref CreateJSVMRef(v8::Local<v8::Value> value, uint32_t refcount);
void DeleteJSVMRef(JSVM_Ref ref);
```

**稳定性**: 内部 API，可能变更

**证据位置**: `src/jsvm_reference.cpp:1-195`

### Platform 抽象

**接口**: Platform 类

**实现位置**: `src/platform/platform.h`

**主要接口**:
```cpp
class Platform {
public:
    virtual void RunTask(std::function<void()> task) = 0;
    virtual void* CreateTimer(uint32_t delay, std::function<void()> callback) = 0;
    virtual void DeleteTimer(void* timer) = 0;
    // ...
};
```

**稳定性**: 稳定（平台抽象层）

**证据位置**: `src/platform/platform.h` - Platform 类定义

---

## 稳定性说明

### 稳定接口

以下接口是稳定的，外部可以依赖：

1. **JSVM-API 层**: `interface/kits/jsvm.h`
   - 所有 `OH_JSVM_*` 函数
   - 版本化 API

**证据位置**: `interface/kits/jsvm.h:16-2500+`

### 较稳定接口

以下接口较稳定，但可能在版本间变更：

1. **Platform 抽象层**: `src/platform/platform.h`
   - Platform 接口
   - 平台实现

**证据位置**: `src/platform/platform.h`

### 不稳定接口

以下接口不稳定，可能在版本间重大变更：

1. **Core 层内部实现**: `src/js_native_api_v8.cpp`
2. **Inspector 内部实现**: `src/inspector/*.cpp`

**证据位置**:
- `src/js_native_api_v8.cpp` - Core 实现
- `src/inspector/` - Inspector 实现

---

## 可替换点

### Platform 层

**可替换性**: 高

**替换方式**: 实现新的 Platform 子类

**示例**: 为新操作系统实现 `NewOSPlatform`

**证据位置**: `src/platform/platform_ohos.cpp` - OpenHarmony 实现

### Inspector 协议

**可替换性**: 中

**替换方式**: 修改 Inspector 协议实现

**示例**: 支持新的 Inspector 协议版本

**证据位置**: `src/inspector/v8_inspector_protocol_json.h` - Inspector 协议

---

## 相关链接

- [架构说明](./04_Architecture.md) - 组件关系
- [目录结构与模块职责](./02_Directory_Structure.md) - 模块职责
