# 安全风险评审

> global_cangjie_wrapper 攻击面、信任边界、安全风险与修复建议

## 评审范围

| 区域 | 覆盖 | 说明 |
|------|------|------|
| N-API / FFI 接口 | ✅ | `ohos/i18n/*.cj`, `ohos/resource_manager/*.cj`, `ohos/resource/*.cj` |
| 输入校验 | ✅ | 参数解析、资源 ID/名称处理 |
| 资源访问 | ✅ | 文件描述符、原始文件、媒体内容 |
| 原生服务交互 | ✅ | FFI 调用到 native 服务 |
| 跨进程通信 | ⚠️ | 依赖 global_i18n/global_resource_management (代码不在本仓库) |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      Cangjie 应用层                               │
│                                                                 │
│  信任边界: 应用沙箱内                                             │
│  - 用户代码可自由调用 API                                         │
│  - 资源操作受权限控制                                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ FFI 调用
┌─────────────────────────────────────────────────────────────────┐
│                      FFI 绑定层                                   │
│                                                                 │
│  信任边界: 系统服务边界                                            │
│  - 传入参数需校验                                                 │
│  - 原生句柄 (Int64) 需验证有效性                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC / Native Call
┌─────────────────────────────────────────────────────────────────┐
│                    原生服务层 (global_i18n,                        │
│                              global_resource_management)          │
│                                                                 │
│  信任边界: 内核/系统服务边界                                       │
│  - 文件系统访问                                                   │
│  - 系统资源配置                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## 攻击面清单

| 攻击面 | 类型 | 模块 | 风险等级 |
|--------|------|------|----------|
| 资源 ID 注入 | 输入验证 | resource_manager | **高** |
| 资源名称路径遍历 | 文件系统 | resource_manager | **高** |
| 字符串格式化漏洞 | 注入 | resource_manager | **中** |
| 句柄泄漏/重用 | 资源管理 | 全部 | **中** |
| 原始文件描述符泄漏 | 资源管理 | raw_file_descriptor | **中** |
| 媒体内容解析 | 内存安全 | resource_manager | **低** |
| 时区/语言配置注入 | 配置注入 | i18n | **低** |
| 密集资源耗尽 | DoS | resource_manager | **低** |

## 可被利用点分析

### 1. 资源 ID 注入风险

**证据**: `resource_manager.cj:565-590`

```cangjie
public func getString(resId: UInt32, args: Array<ArgsValueType>): String {
    // resId 直接传入 FFI，无校验
    let res = CJ_GetString(this.mDataId, resId, argsObj)
    throwIfNotSuccess(res.code)
    return res.data
}
```

**触发条件**:
- 攻击者控制 `resId` 参数
- 传入非本应用资源 ID

**影响**:
- 越权访问其他应用的资源
- 潜在信息泄露

**修复建议**:
```cangjie
public func getString(resId: UInt32, args: Array<ArgsValueType>): String {
    // 添加资源归属校验
    if (!isValidResourceId(resId)) {
        throw BusinessException("Invalid resource ID")
    }
    let res = CJ_GetString(this.mDataId, resId, argsObj)
    throwIfNotSuccess(res.code)
    return res.data
}
```

---

### 2. 资源名称路径遍历风险

**证据**: `resource_manager.cj:186-210`

```cangjie
public func getColorByName(resName: String): UInt32 {
    // resName 直接传入 FFI
    let res = CJ_GetColorByName(this.mDataId, resName)
    throwIfNotSuccess(res.code)
    return res.data
}
```

**触发条件**:
- 传入 `../` 等路径遍历字符
- 如 `resName = "../../../etc/passwd"`

**影响**:
- 访问应用沙箱外文件
- 信息泄露或配置篡改

**修复建议**:
```cangjie
public func getColorByName(resName: String): UInt32 {
    // 校验资源名称不包含路径分隔符
    if (resName.contains("/") || resName.contains("..")) {
        throw BusinessException("Invalid resource name: path traversal not allowed")
    }
    let res = CJ_GetColorByName(this.mDataId, resName)
    throwIfNotSuccess(res.code)
    return res.data
}
```

---

### 3. 字符串格式化风险

**证据**: `resource_manager_common.cj:496-540`

```cangjie
public func formatString(template: String, args: Array<ArgsValueType>): String {
    // args 直接格式化，无类型校验
    for (arg in args) {
        if (arg.type == NumberValueType.NUMBER_FLOAT) {
            result = result.replace("%f", arg.value)
        }
        // ...
    }
}
```

**触发条件**:
- 传入不匹配的格式符
- 格式符数量不匹配

**影响**:
- 格式化错误导致崩溃
- 潜在信息泄露（格式化溢出）

**修复建议**:
```cangjie
public func formatString(template: String, args: Array<ArgsValueType>): String {
    // 预校验格式符数量
    let expectedCount = countFormatPlaceholders(template)
    if (args.size != expectedCount) {
        throw BusinessException("Format argument mismatch")
    }
    // ... 继续格式化
}
```

---

### 4. 原始文件描述符泄漏

**证据**: `resource_manager.cj:78-100`

```cangjie
public func getRawFd(rawFileName: CString): RawFileDescriptor {
    let res = CJ_GetRawFd(this.mDataId, rawFileName, fdPtr)
    throwIfNotSuccess(res.code)
    return RawFileDescriptor(fdPtr.data.fd, fdPtr.data.offset, fdPtr.data.length)
}
```

**触发条件**:
- 应用未调用 `closeRawFd()` 关闭描述符
- 异常路径未清理描述符

**影响**:
- 文件描述符泄漏
- 文件句柄耗尽 (DoS)

**修复建议**:
```cangjie
public func getRawFd(rawFileName: CString): RawFileDescriptor {
    let res = CJ_GetRawFd(this.mDataId, rawFileName, fdPtr)
    throwIfNotSuccess(res.code)
    let fd = RawFileDescriptor(...)
    // 使用 RAII 模式确保资源释放
    return ManagedRawFileDescriptor(fd, this)
}
```

---

### 5. 越权资源访问

**证据**: `resource_manager.cj:650-675`

```cangjie
public func addResource(path: CString): Bool {
    // path 无校验
    let res = CJ_AddResource(this.mDataId, path)
    throwIfNotSuccess(res.code)
    return true
}
```

**触发条件**:
- 传入应用沙箱外路径
- 传入符号链接

**影响**:
- 加载恶意资源
- 绕过沙箱限制

**修复建议**:
```cangjie
public func addResource(path: CString): Bool {
    // 校验路径在应用沙箱内
    if (!isPathInSandbox(path)) {
        throw BusinessException("Path outside sandbox not allowed")
    }
    let res = CJ_AddResource(this.mDataId, path)
    throwIfNotSuccess(res.code)
    return true
}
```

---

### 6. 密集资源耗尽 (DoS)

**证据**: `resource_manager.cj:36-63` (ResourceManager 缓存)

```cangjie
private static let RES_MGR_MAP = HashMap<UIntNative, ResourceManager>()
private static let APP_MUTEX = Mutex()

protected static func getResourceManager(context: StageContext): ResourceManager {
    synchronized(APP_MUTEX) {
        // 缓存管理，无数量限制
        if (!RES_MGR_MAP.contains(context.nativePtr)) {
            RES_MGR_MAP.put(context.nativePtr, ResourceManager(...))
        }
        return RES_MGR_MAP.get(context.nativePtr)
    }
}
```

**触发条件**:
- 大量不同 context 创建 ResourceManager
- 无缓存淘汰机制

**影响**:
- 内存耗尽 (DoS)
- 性能下降

**修复建议**:
```cangjie
private static const MAX_CACHE_SIZE = 16
private static let LRU_MAP = LRUMap<UIntNative, ResourceManager>(MAX_CACHE_SIZE)
```

---

### 7. 恶意媒体内容解析

**证据**: `resource_manager.cj:366-400`

```cangjie
public func getMediaContent(resId: UInt32, density!: ?Int32): Array<UInt8> {
    // 媒体内容直接返回原始字节
    let res = CJ_GetMediaContent(this.mDataId, resId, density, contentPtr)
    throwIfNotSuccess(res.code)
    return Array<UInt8>(...)
}
```

**触发条件**:
- 解析特制的恶意图片/媒体文件
- 资源服务漏洞

**影响**:
- 依赖原生服务的解析安全性
- 本层难以防护

**修复建议**:
- 依赖 native 层的安全解析
- 添加资源完整性校验
- 限制可解析资源类型

---

### 8. 句柄重用/释放后使用 (UAF)

**证据**: `calendar.cj:99-117`

```cangjie
public class Calendar <: RemoteDataLite {
    protected func releaseFFIData(myDataId: Int64): Unit {
        // 释放原生句柄
        FfiOHOSCalendarRelease(myDataId)
    }
}
```

**触发条件**:
- Cangjie 对象释放后，原生端句柄仍可能被引用
- 多线程竞争

**影响**:
- 释放后使用 (UAF)
- 潜在崩溃或代码执行

**修复建议**:
```cangjie
protected func releaseFFIData(myDataId: Int64): Unit {
    synchronized(RELEASE_MUTEX) {
        if (!isReleased) {
            FfiOHOSCalendarRelease(myDataId)
            isReleased = true
        }
    }
}
```

## 安全最佳实践

### 输入验证

| 场景 | 验证项 |
|------|--------|
| 资源 ID | 范围检查、归属检查 |
| 资源名称 | 路径遍历检查、特殊字符过滤 |
| 格式字符串 | 参数数量匹配、类型匹配 |
| 文件路径 | 沙箱边界检查、符号链接解析 |

### 资源管理

| 场景 | 实践 |
|------|------|
| 文件描述符 | RAII 模式、自动关闭 |
| 原生句柄 | 引用计数、状态标记 |
| 内存 | 避免大对象直接传递、使用智能指针 |
| 缓存 | LRU 淘汰、大小限制 |

### 错误处理

| 场景 | 处理 |
|------|------|
| 资源不存在 | 抛出明确异常、记录日志 |
| 权限不足 | 降级处理、用户通知 |
| 格式错误 | 优雅失败、详细错误码 |

## 依赖组件安全

| 组件 | 用途 | 安全考量 |
|------|------|----------|
| `i18n:cj_i18n_ffi` | 日历 FFI | 依赖 native 服务正确性 |
| `resource_management:cj_resource_manager_ffi` | 资源管理 FFI | 依赖 native 服务正确性 |
| `ace_engine:cj_frontend_ohos` | UI 集成 | 依赖 ArkUI 安全性 |
| `hiviewdfx_cangjie_wrapper` | 日志 | 避免敏感信息泄露 |

## 未覆盖区域

| 区域 | 说明 |
|------|------|
| 原生服务层 (global_i18n, global_resource_management) | 代码在独立仓库，需单独评审 |
| 系统内核交互 | 超出本仓库范围 |
| 加密/签名验证 | 当前代码未涉及 |

## 相关文档

- [API 参考](./03_NAPI_Reference.md)
- [架构设计](./02_Architecture.md)
- [常见问题](./08_Troubleshooting.md)
