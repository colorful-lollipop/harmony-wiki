# 安全风险评审

## 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不可信区域                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    JS 应用层                              │   │
│  │  • 用户编写的 JS/ArkTS 代码                               │   │
│  │  • 第三方 NAPI 模块（未经签名验证）                        │   │
│  │  • 网络输入（fetch, WebSocket）                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                  │
│                              │                                  │
│                    N-API 接口边界                                │
│  • 参数类型转换                                               │
│  • 内存安全转换                                               │
│  • 权限校验点                                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                       可信区域                                   │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                   Native Engine                         │     │
│  │  • NativeEngine 运行时                                │     │
│  │  • 模块管理器（ModuleManager）                         │     │
│  │  • 引用管理器（ReferenceManager）                       │     │
│  └─────────────────────────────────────────────────────────┘     │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                   系统服务                               │     │
│  │  • 文件系统（受限）                                     │     │
│  │  • 网络套接字（权限管控）                               │     │
│  │  • 系统能力（IPC）                                      │     │
│  └─────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

### 攻击面清单

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| **N-API 参数解析** | JS 到 native 的类型转换 | 高 |
| **模块加载** | `.so` 动态加载和执行 | 高 |
| **外部数据绑定** | `napi_wrap` / `napi_external` | 中 |
| **Async Work** | 线程池任务队列 | 中 |
| **TSFN** | 线程安全函数回调 | 中 |
| **序列化/反序列化** | `napi_serialize` / `napi_deserialize` | 高 |
| **模块路径遍历** | 模块查找路径注入 | 中 |
| **Native 回调注入** | JS 函数作为回调传入 native | 中 |

## 安全风险分析

### 风险 1：路径遍历漏洞

**风险等级**：中

**证据位置**：`native_module_manager.h:144-148`

```cpp
bool GetNativeModulePath(const char* moduleName, const char* path, const char* relativePath,
    bool isAppModule, char nativeModulePath[][NAPI_PATH_MAX], int32_t pathLength);
```

**问题描述**：
模块路径可能包含用户控制的数据，如果路径验证不严格，可能导致路径遍历攻击。

**利用路径**：
```
攻击者输入 → moduleName 包含 "../" → 加载恶意模块
```

**修复建议**：
1. 对模块名进行严格的白名单校验
2. 拒绝路径分隔符（`/`, `\`, `..`）
3. 使用 realpath 规范化路径后验证

---

### 风险 2：类型转换导致的内存安全问题

**风险等级**：高

**证据位置**：`native_api.h:46-68`

```cpp
NAPI_EXTERN napi_status napi_create_string_utf16(napi_env env,
                                                  const char16_t* str,
                                                  size_t length,
                                                  napi_value* result);
```

**问题描述**：
字符串和缓冲区创建时，长度参数由用户提供，可能导致：
- 整数溢出导致缓冲区过小
- 长度验证缺失导致越界读写

**利用路径**：
```
JS: 创建超大字符串 → length 溢出 → 缓冲区分配过小 → 堆溢出
```

**修复建议**：
1. 在长度参数使用前进行严格范围检查
2. 对 size_t 进行溢出检测
3. 使用安全的内存分配器

---

### 风险 3：序列化安全

**风险等级**：高

**证据位置**：`native_api.h:116-122`

```cpp
NAPI_EXTERN napi_status napi_serialize(napi_env env,
                                        napi_value object,
                                        napi_value transfer_list,
                                        napi_value clone_list,
                                        void** result);
NAPI_EXTERN napi_status napi_deserialize(napi_env env, void* buffer, napi_value* object);
```

**问题描述**：
序列化/反序列化过程可能受到恶意数据攻击：
- 反序列化漏洞（类似 JSON 解析攻击）
- 序列化数据被篡改

**利用路径**：
```
攻击者 → 修改序列化 buffer → napi_deserialize → 代码执行
```

**修复建议**：
1. 对序列化数据进行签名/加密验证
2. 在反序列化前检查魔法数（magic number）
3. 对反序列化类型进行白名单限制

---

### 风险 4：外部数据引用泄漏

**风险等级**：中

**证据位置**：`native_node_api.h:95-100`

```cpp
NAPI_EXTERN napi_status napi_create_external_with_size(napi_env env,
                                                        void* data,
                                                        napi_finalize finalize_cb,
                                                        void* finalize_hint,
                                                        napi_value* result,
                                                        size_t native_binding_size);
```

**问题描述**：
`napi_create_external` 创建的外部数据，如果 finalize 回调缺失或错误：
- Native 数据泄漏（悬空指针）
- 资源未释放（内存泄漏）

**利用路径**：
```
Native 模块 → 创建 external → 未设置 finalize → JS 释放对象 → native 数据悬空
```

**修复建议**：
1. 始终为 external 数据设置 finalize 回调
2. 使用引用计数管理生命周期
3. 避免在 finalize 中执行复杂逻辑

---

### 风险 5：异步任务队列阻塞

**风险等级**：中

**证据位置**：`native_async_work.h:49-98`

```cpp
class NativeAsyncWork {
    uv_work_t work_;
    NativeAsyncExecuteCallback execute_;
    NativeAsyncCompleteCallback complete_;
    // ...
};
```

**问题描述**：
Async Work 使用 libuv 线程池，任务过多可能导致：
- 线程池耗尽
- 拒绝服务

**利用路径**：
```
恶意 JS → 大量创建 async_work → 线程池满载 → 系统无响应
```

**修复建议**：
1. 对异步任务数量进行限制
2. 实现任务超时机制
3. 使用信号量控制并发数

---

### 风险 6：回调注入

**风险等级**：中

**证据位置**：`native_engine.h:224-227`

```cpp
virtual napi_value CallFunction(napi_value thisVar,
                                napi_value function,
                                napi_value const *argv,
                                size_t argc) = 0;
```

**问题描述**：
JS 函数作为回调传入 native，如果验证不严格：
- 跨站脚本攻击（通过回调执行恶意代码）
- 权限提升（回调访问受限资源）

**利用路径**：
```
攻击者 JS → 注入恶意回调 → native 代码调用 → 执行任意 JS 代码
```

**修复建议**：
1. 对传入的回调进行类型检查
2. 在敏感操作前验证调用上下文
3. 限制回调可以访问的 API 范围

---

### 风险 7：未验证的 API 白名单绕过

**风险等级**：低

**证据位置**：`native_module_manager.h:120-138`

```cpp
inline bool CheckModuleRestricted(const std::string& moduleName)
{
    const std::string whiteList[] = {
        "worker",
        "arkui.uicontext",
        "arkui.node",
        "arkui.modifier",
        "measure",
    };
    // ...
}
```

**问题描述**：
白名单检查可能存在遗漏：
- 新增受限模块未加入白名单
- 白名单本身可能过时

**修复建议**：
1. 定期审查和更新白名单
2. 对非白名单模块记录审计日志
3. 实现动态白名单更新机制

---

## 权限模型

### 权限检查点

**证据位置**：`native_engine.h:475-476`

```cpp
virtual void RegisterPermissionCheck(PermissionCheckCallback callback) = 0;
virtual bool ExecutePermissionCheck() = 0;
```

**证据位置**：`ark_native_engine.cpp:3002-3012`

```cpp
if (permissionCheckCallback_ == nullptr) {
    permissionCheckCallback_ = callback;
}
// ...
if (permissionCheckCallback_ != nullptr) {
    return permissionCheckCallback_();
}
```

### 模块签名验证

**证据位置**：`module_manager/module_load_checker.cpp`

模块加载时进行签名检查，确保：
- 模块来自可信源
- 模块签名有效
- 模块未被篡改

## 安全最佳实践

### 开发者指南

| 实践 | 描述 |
|------|------|
| **输入验证** | 验证所有来自 JS 的输入参数 |
| **最小权限** | 只请求必要的系统能力 |
| **资源清理** | 确保所有资源在合适时机释放 |
| **错误处理** | 不要吞掉异常，返回适当错误码 |
| **敏感数据** | 避免在日志中输出敏感信息 |

### 代码示例：安全参数处理

```cpp
// 不安全示例
napi_value UnsafeExample(napi_env env, napi_callback_info info) {
    size_t argc = 1;
    napi_value argv[1];
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
    
    // 直接使用用户提供的 length，可能导致溢出
    size_t length;
    napi_get_value_uint32(env, argv[0], &length);
    char* buffer = new char[length];  // 危险！
    // ...
}

// 安全示例
napi_value SafeExample(napi_env env, napi_callback_info info) {
    size_t argc = 1;
    napi_value argv[1];
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
    
    uint32_t inputLength;
    napi_get_value_uint32(env, argv[0], &inputLength);
    
    // 验证长度范围
    constexpr size_t MAX_LENGTH = 1024 * 1024;  // 1MB 限制
    if (inputLength > MAX_LENGTH) {
        napi_throw_range_error(env, "Length exceeds maximum allowed");
        return nullptr;
    }
    
    size_t length = static_cast<size_t>(inputLength);
    char* buffer = new (std::nothrow) char[length];  // 使用 nothrow
    if (buffer == nullptr) {
        napi_throw_error(env, "Memory allocation failed");
        return nullptr;
    }
    
    std::unique_ptr<char[]> guard(buffer);  // RAII 保护
    // ...
}
```

---

## 检查范围与局限性

### 已检查范围

- N-API 接口参数处理（`native_api.h`, `native_node_api.h`）
- 模块加载流程（`module_manager/`）
- 异步任务处理（`native_async_work.h/cpp`）
- 序列化/反序列化（`native_api.h`）
- 引用生命周期管理（`reference_manager/`）

### 未检查范围

- 第三方依赖的内部实现
- Ark 运行时（ecmascript）的 GC 安全
- 系统调用层的权限检查
- 签名验证的具体实现细节

---

**相关文档**：
- [架构说明](./03_Architecture.md)
- [N-API 接口参考](./04_NAPI_Reference.md)
