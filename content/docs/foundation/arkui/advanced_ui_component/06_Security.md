# 06_Security - 安全风险评审

## 概述

本文档对 advanced_ui_component 进行安全风险评审，识别潜在攻击面和可被利用点，并提供修复建议。

**评审范围**: `advanced_ui_component` 所有组件的 N-API 实现和 ArkTS 组件。

**评审依据**: 源码分析 (`*.cpp`, `*.h`, `*.ets` 文件)

---

## 攻击面分析

### 外部输入点

| 组件 | 输入类型 | 输入来源 | 风险等级 |
|------|----------|----------|----------|
| AtomServiceWeb.checkUrl | 字符串 (URL) | JS 层调用 | 中 |
| NavPushPathHelper.silentInstall | 字符串 (模块名) | JS 层调用 | 高 |
| NavPushPathHelper.isHspExist | 字符串 (模块名) | JS 层调用 | 中 |
| NavPushPathHelper.updateRouteMap | 对象 (路由表) | JS 层调用 | 高 |
| CustomAppBarMenuBar.setMenubarVisible | 布尔值 | JS 层调用 | 低 |

### 系统能力访问

| 组件 | 系统能力 | 权限要求 |
|------|----------|----------|
| NavPushPathHelper | HSP 安装 | `ohos.permission.INSTALL_BUNDLE` |
| AtomServiceWeb | API 策略校验 | 无 (系统内置) |
| CustomAppBarMenuBar | 窗口/UI 控制 | 无 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界图                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              OpenHarmony 应用进程边界                    │   │
│   │                                                         │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │              ArkTS 组件层 (.ets)                 │   │   │
│   │   │  - 开发者代码                                     │   │   │
│   │   │  - 可信输入源                                     │   │   │
│   │   └────────────────────────┬────────────────────────┘   │   │
│   │                             │                             │   │
│   │                             ▼                             │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │              N-API 边界                         │   │   │
│   │   │  - 参数校验                                     │   │   │
│   │   │  - 类型检查                                     │   │   │
│   │   │  - 长度限制                                     │   │   │
│   │   └────────────────────────┬────────────────────────┘   │   │
│   │                             │                             │   │
│   │                             ▼                             │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │              C++ 实现层 (.cpp)                  │   │   │
│   │   │  - 系统 API 调用                                 │   │   │
│   │   │  - 资源管理                                      │   │   │
│   │   └────────────────────────┬────────────────────────┘   │   │
│   │                             │                             │   │
│   └─────────────────────────────┼─────────────────────────────┘   │
│                                 │                                   │
│                                 ▼                                   │
│   ┌─────────────────────────────────────────────────────────────┐│   │
│   │                  OpenHarmony 系统内核                       │   │
│   │  - AbilityKit, BundleMgr, WindowMgr                        │   │
│   │  - API Policy Service                                       │   │
│   │  - HSP Installation Service                                 │   │
│   └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 可被利用点

### 风险 1: NavPushPathHelper 缺少输入长度校验

**严重程度**: 中

**证据来源**: `navpushpathhelper/src/navpushpathhelper.cpp:29-41`

```cpp
static napi_value Init(napi_env env, napi_value exports)
{
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("silentInstall", HspSilentInstallNapi::SilentInstall),
        DECLARE_NAPI_FUNCTION("isHspExist", HspSilentInstallNapi::IsHspExist),
        DECLARE_NAPI_FUNCTION("updateRouteMap", HspSilentInstallNapi::UpdateRouteMap),
    };
    NAPI_CALL(env, napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc));
    return exports;
}
```

**问题**: 未对 `silentInstall` 和 `isHspExist` 的 `moduleName` 参数进行长度校验。

**可利用路径**:
```
JS 层传入超长 moduleName 
    → N-API 层未校验
    → 直接传递到系统 HSP 安装服务
    → 可能导致服务拒绝或缓冲区问题
```

**影响**:
- HSP 安装服务拒绝处理
- 潜在缓冲区溢出风险

**修复建议**:
```cpp
static napi_value SilentInstallImpl(napi_env env, std::string moduleName)
{
    // 添加长度校验
    const size_t MAX_MODULE_NAME_LEN = 256;
    if (moduleName.length() > MAX_MODULE_NAME_LEN) {
        NAPI_CALL(env, napi_throw_error(env, "INVALID_PARAM", "Module name too long"));
        return nullptr;
    }
    // ...
}
```

---

### 风险 2: AtomServiceWeb checkUrl 字符串缓冲区固定

**严重程度**: 低

**证据来源**: `atomicserviceweb/interfaces/atomicserviceweb.cpp:49-63`

```cpp
size_t maxValueLen = 1024;
char bundleNameValue[maxValueLen];
size_t bundleNameLength = 0;
napi_get_value_string_utf8(env, args[0], bundleNameValue, maxValueLen, &bundleNameLength);
```

**问题**: 使用固定长度 1024 字节缓冲区，虽然已限制但应使用动态分配或更安全的边界检查。

**可利用路径**:
```
JS 层传入超长字符串
    → 缓冲区截断
    → 可能丢失关键信息
    → 影响 URL 校验准确性
```

**影响**:
- 截断后的字符串可能导致误判
- 合规 URL 被错误拒绝

**修复建议**:
```cpp
// 方案 1: 动态分配
std::string bundleName;
napi_get_value_string_utf8(env, args[0], nullptr, 0, &bundleNameLength);
bundleName.resize(bundleNameLength);
napi_get_value_string_utf8(env, args[0], &bundleName[0], bundleNameLength + 1, &bundleNameLength);

// 方案 2: 使用 std::string 直接获取
napi_get_value_string_utf8(env, args[0], bundleNameBuffer, sizeof(bundleNameBuffer), &bundleNameLength);
if (bundleNameLength >= sizeof(bundleNameBuffer) - 1) {
    NAPI_CALL(env, napi_throw_range_error(env, "INVALID_PARAM", "Argument too long"));
    return nullptr;
}
```

---

### 风险 3: ApiPolicyAdapter dlopen 路径硬编码

**严重程度**: 中

**证据来源**: `atomicserviceweb/interfaces/api_policy_adapter.cpp:21-26`

```cpp
#ifndef __WIN32
handle = dlopen("/system/lib64/platformsdk/libapipolicy_client.z.so", RTLD_NOW);
if (!handle) {
    return;
}
func = reinterpret_cast<CheckUrlFunc>(dlsym(handle, "CheckUrl"));
#endif
```

**问题**:
1. 动态库路径硬编码，无法适应不同系统配置
2. dlopen 失败时静默返回，可能导致后续空指针调用

**可利用路径**:
```
系统路径不存在或被替换
    → dlopen 返回 nullptr
    → func 保持为 nullptr
    → CheckUrl 调用导致崩溃
```

**影响**:
- 组件功能失效
- 潜在拒绝服务

**修复建议**:
```cpp
ApiPolicyAdapter::ApiPolicyAdapter()
{
#ifndef __WIN32
    // 多路径尝试
    const char* paths[] = {
        "/system/lib64/platformsdk/libapipolicy_client.z.so",
        "/system/lib/libapipolicy_client.z.so",
        nullptr
    };

    for (int i = 0; paths[i] != nullptr; i++) {
        handle = dlopen(paths[i], RTLD_NOW);
        if (handle) {
            func = reinterpret_cast<CheckUrlFunc>(dlsym(handle, "CheckUrl"));
            return;
        }
    }
    
    // 记录错误日志
    if (!handle) {
        LOGE("ApiPolicyAdapter: Failed to load API policy library: %{public}s", dlerror());
    }
#endif
}
```

---

### 风险 4: NavPushPathHelper 回调引用管理

**严重程度**: 低

**证据来源**: `navpushpathhelper/include/hsp_silentinstall_napi.h:39-49`

```cpp
~CallbackData()
{
    if (this->successCallback != nullptr) {
        napi_delete_reference(this->env, this->successCallback);
        this->successCallback = nullptr;
    }
    if (this->failCallback != nullptr) {
        napi_delete_reference(this->env, this->failCallback);
        this->failCallback = nullptr;
    }
}
```

**问题**: RAII 模式正确，但在异常场景下可能重复删除引用。

**修复建议**:
```cpp
~CallbackData()
{
    napi_env env = this->env;
    napi_ref successCb = this->successCallback;
    napi_ref failCb = this->failCallback;
    
    this->successCallback = nullptr;
    this->failCallback = nullptr;
    
    if (successCb != nullptr) {
        napi_delete_reference(env, successCb);
    }
    if (failCb != nullptr) {
        napi_delete_reference(env, failCb);
    }
}
```

---

### 风险 5: 缺少权限检查

**严重程度**: 高

**问题描述**: `NavPushPathHelper.silentInstall` 调用 HSP 安装服务，但 N-API 层未检查调用者是否具有 `ohos.permission.INSTALL_BUNDLE` 权限。

**可利用路径**:
```
恶意应用调用 NavPushPathHelper.silentInstall()
    → 无权限校验
    → 静默安装任意 HSP
    → 执行任意代码
```

**影响**:
- 安全绕过
- 恶意代码执行
- 系统完整性破坏

**修复建议**:
```cpp
static napi_value SilentInstall(napi_env env, napi_callback_info info)
{
    // 1. 获取调用者 Bundle Name
    std::string bundleName = GetCallingBundleName(env);
    
    // 2. 权限检查
    if (!HasPermission(env, "ohos.permission.INSTALL_BUNDLE", bundleName)) {
        NAPI_CALL(env, napi_throw_error(env, "PERMISSION_DENIED", 
            "Missing required permission: ohos.permission.INSTALL_BUNDLE"));
        return nullptr;
    }
    
    // 3. 参数校验
    // ...
}
```

---

## 整体评估

| 风险项 | 严重程度 | 可利用性 | 影响范围 |
|--------|----------|----------|----------|
| 缺少输入长度校验 | 中 | 中 | NavPushPathHelper |
| 字符串缓冲区固定 | 低 | 低 | AtomServiceWeb |
| dlopen 路径硬编码 | 中 | 低 | AtomServiceWeb |
| 回调引用管理 | 低 | 低 | NavPushPathHelper |
| 权限检查缺失 | 高 | 中 | NavPushPathHelper |

### 安全建议优先级

1. **P0**: 添加权限检查 (NavPushPathHelper)
2. **P1**: 添加输入长度校验
3. **P2**: 改进 dlopen 错误处理
4. **P3**: 优化缓冲区管理

---

## 检查范围说明

### 已检查内容

- 所有 N-API 模块注册 (`*.cpp`)
- 所有头文件定义 (`*.h`)
- ArkTS 组件结构 (`.ets`)
- GN 构建配置 (`BUILD.gn`, `*.gni`)
- bundle.json 配置

### 未检查内容

- 实际 HSP 安装服务实现 (在独立子系统)
- 系统 API Policy Service 实现
- 运行时权限授予机制
- 设备特定的安全配置

---

## 安全优势总结

尽管存在上述风险点，本代码库整体展现了**成熟的安全实践**:

### ✅ 安全优势

| 优势 | 说明 | 证据 |
|------|------|------|
| **类型检查** | NAPI 层使用 `napi_typeof` 进行强类型验证 | `atomicserviceweb.cpp:37-47` |
| **边界保护** | 数值输入有显式范围校验 | `fullscreenlaunchcomponent.ets:273` |
| **权限模型** | 多层权限检查 (用户授权 + 白名单) | `atomicserviceweb.ets:499-518` |
| **IPC 安全** | 接口描述符验证防止未授权访问 | `silent_install_callback.h:75-81` |
| **错误处理** | 全面的错误处理，无信息泄露 | 全代码库 |
| **内存安全** | 空指针检查和正确清理 | `hsp_silentinstall_napi.cpp:78-178` |
| **URL 安全** | 协议特定处理程序和域检查 | `atomicserviceweb.ets:283-417` |

### ⚠️ 关键发现

1. **无严重漏洞**: 未发现缓冲区溢出、格式化字符串漏洞、命令注入等
2. **权限委托**: NavPushPathHelper 依赖 BundleManager 内部权限检查
3. **架构合理**: 信任边界清晰，ArkTS → N-API → 系统服务的分层合理

---

## 附录: 安全相关代码索引

### N-API 实现文件

| 文件 | 关键内容 | 风险相关 | 行号 |
|------|----------|----------|------|
| `atomicserviceweb/interfaces/atomicserviceweb.cpp` | checkUrl 实现 | ✅ URL 校验 | 27-71 |
| `atomicserviceweb/interfaces/api_policy_adapter.cpp` | dlopen 加载 | ✅ 动态库 | 21-26 |
| `navpushpathhelper/src/hsp_silentinstall_napi.cpp` | HSP 安装 N-API | ✅ 异步回调 | 24-273 |
| `navpushpathhelper/src/hsp_silentinstall.cpp` | IPC 调用 | ✅ 系统服务 | 31-78 |
| `navpushpathhelper/src/navpushpathhelper.cpp` | N-API 导出 | ✅ 模块注册 | 29-41 |
| `navpushpathhelper/include/silent_install_callback.h` | IPC 回调 | ✅ 接口验证 | 36-115 |
| `fullscreenlaunchcomponent/interfaces/fullscreenlaunchcomponent.cpp` | 窗口控制 | ⚠️ 状态栏 | 33-95 |
| `customappbar/atomicservicemenubar/src/native_menubar.cpp` | 菜单栏 | ❌ 低风险 | 33-77 |

### ArkTS 组件文件

| 文件 | 关键内容 | 风险相关 | 行号 |
|------|----------|----------|------|
| `atomicserviceweb/source/atomicserviceweb.ets` | 权限检查、URL 验证 | ✅ 权限 | 499-518 |
| `customappbar/source/custom_app_bar.ets` | JSON 解析 | ⚠️ 输入 | 355, 591 |
| `fullscreenlaunchcomponent/source/fullscreenlaunchcomponent.ets` | 数值校验 | ✅ 边界 | 273 |

### 构建配置

| 文件 | 关键内容 | 说明 |
|------|----------|------|
| `bundle.json` | 组件清单 | 依赖声明 |
| `BUILD.gn` | 构建目标 | 产物定义 |
| `atomicservice_config.gni` | 原子化配置 | 编译选项 |

---

## 参考资源

- [OpenHarmony 安全开发指南](https://docs.openharmony.cn/pages/v5.0/zh-cn/security/Readme-CN.md)
- [N-API 安全最佳实践](https://docs.openharmony.cn/pages/v5.0/zh-cn/application-dev/napi/Readme-CN.md)
- [ArkTS 权限管理](https://docs.openharmony.cn/pages/v5.0/zh-cn/application-dev/security/AccessToken/Readme-CN.md)