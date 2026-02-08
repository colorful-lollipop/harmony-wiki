# 安全风险评审

本文档对 cangjie_ark_interop 进行安全风险评审，识别攻击面、信任边界和潜在风险。

## 评审范围

| 范围 | 说明 |
|------|------|
| 代码位置 | `ohos/ark_interop/` |
| 代码位置 | `ohos/ark_interop_helper/` |
| 代码位置 | `ohos/ffi/` |
| 代码位置 | `ohos/utf16string/` |
| 排除 | `test/` (测试代码不作为证据) |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                  │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  内部 (可信)                                                 │ │
│  │  ───────────                                                │ │
│  │  • ark_interop 库代码                                        │ │
│  │  • napi 接口 (系统组件)                                       │ │
│  │  • ArkTS Runtime                                            │ │
│  │  • Cangjie Runtime                                          │ │
│  │                                                              │ │
│  │  边界                                                        │ │
│  │  ─────                                                        │ │
│  │  • N-API 接口 (napi_*)                                       │ │
│  │  • FFI 接口 (ffi_*)                                          │ │
│  │  • 文件系统 (abc 模块文件)                                     │ │
│  │  • 外部输入 (JSValue)                                         │ │
│  │                                                              │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                          │                                        │
│                          ▼                                        │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  外部 (不可信)                                               │ │
│  │  ───────────                                                │ │
│  │  • 应用代码 (HAP)                                            │ │
│  │  • ArkTS 模块 (.abc 文件)                                    │ │
│  │  • 用户输入 (JSValue)                                        │ │
│  │  • 网络数据 (通过 fetch 等)                                   │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 攻击面分析

### 1. N-API 接口 (外部输入)

**描述**: 通过 napi 调用传入的 JSValue 可能包含恶意数据

**代码位置**: `ohos/ark_interop/jscontext.cj`

**风险等级**: 中

**检查**:
```cangjie
// 参数校验示例
context.checkLifecycleAndThread()  // 检查线程
```

**需确认**: 参数边界检查是否充分

---

### 2. 模块加载 (文件系统)

**描述**: 加载 ArkTS 模块 (.abc 文件) 可能存在路径遍历风险

**代码位置**: `ohos/ark_interop/js_module.cj`

**调用链**:
```
JSModule.import() → ARKTS_LoadEntryFromAbc() → dlopen()
```

**风险等级**: 低

**检查**: 依赖 ability_runtime 的模块加载机制

**需确认**: abc 文件完整性校验机制

---

### 3. 字符串处理 (缓冲区溢出)

**描述**: UTF-16 字符串操作可能存在缓冲区溢出

**代码位置**: `ohos/utf16string/utf16string.cpp`

**证据**:
```cpp
// utf16string.h:167
Utf16StringHandle SubString(uint32_t start, uint32_t end) const;

// utf16string.h:155-165
char16_t RawCharAt(uint32_t index) const {
    if (index >= length_) {  // 边界检查
        return 0;
    }
    // ...
}
```

**风险等级**: 低

**评估**: 代码已有边界检查 (index >= length_)

---

### 4. 引用管理 (Use-After-Free)

**描述**: JSValue 引用可能在生命周期外被访问

**代码位置**: `ohos/ark_interop/js_func.cj:60-68`

**证据**:
```cangjie
// JSCallInfo 使用后释放检测
private var isSafe = true

prop callInfo: JSCallInfoPrivate {
    get() {
        if (!isSafe) {
            printExp(jsObjUseAfterFree("JSCallInfo"))
        }
        callInfo_
    }
}
```

**风险等级**: 中

**评估**: 已有 Use-After-Free 检测机制

---

### 5. 线程安全 (竞态条件)

**描述**: 跨线程访问 JSValue 可能导致竞态

**代码位置**: `ohos/ark_interop/jscontext.cj`

**证据**:
```cangjie
// 线程检查
context.checkLifecycleAndThread()  // 抛出 34300004
```

**风险等级**: 中

**评估**: 已有线程检查机制

---

### 6. 类型转换 (Type Confusion)

**描述**: JSValue 类型转换不匹配可能导致安全风险

**代码位置**: `ohos/ark_interop/js_exception.cj:54-60`

**证据**:
```cangjie
internal func jsTypeMisMatch(acquireType: String, givenType: JSType): BusinessException {
    BusinessException(34300005, "The ArkTS data types do not match...")
}
```

**风险等级**: 低

**评估**: 已有类型检查机制

---

### 7. 异常信息泄露

**描述**: 异常消息可能泄露内部实现细节

**代码位置**: `ohos/business_exception/business_exception.cj:92-99`

**证据**:
```cangjie
public func toString(): String {
    let className: String = getClassName()
    let message: String = this.message
    if (message.isEmpty()) {
        return "${className}:  errorcode: ${code}"
    }
    return "${className}: errorcode: ${code}, message: ${message}"
}
```

**风险等级**: 低

**评估**: 异常信息可控

---

## 风险汇总表

| ID | 攻击面 | 风险类型 | 等级 | 当前缓解措施 |
|----|--------|----------|------|--------------|
| R1 | N-API 参数 | 输入验证 | 中 | 参数校验 |
| R2 | 模块加载 | 路径遍历 | 低 | 依赖系统机制 |
| R3 | 字符串处理 | 缓冲区溢出 | 低 | 边界检查 |
| R4 | 引用管理 | Use-After-Free | 中 | 检测机制 |
| R5 | 线程安全 | 竞态条件 | 中 | 线程检查 |
| R6 | 类型转换 | Type Confusion | 低 | 类型检查 |
| R7 | 异常信息 | 信息泄露 | 低 | 受控输出 |

---

## 修复建议

### R1: N-API 参数校验

**建议**: 增强参数校验范围

**位置**: `ohos/ark_interop/` 各模块

**具体措施**:
- 添加参数范围校验
- 添加空值检查
- 添加编码校验

---

### R4: 引用生命周期管理

**建议**: 增强引用跟踪

**位置**: `ohos/ark_interop/js_func.cj`

**具体措施**:
- 细化引用计数机制
- 添加引用泄漏检测

---

### R5: 线程安全加固

**建议**: 细粒度线程锁

**位置**: `ohos/ark_interop/jscontext.cj`

**具体措施**:
- 添加写时复制机制
- 优化锁粒度

---

## 安全机制总结

### 已实现的安全机制

| 机制 | 位置 | 说明 |
|------|------|------|
| 线程检查 | `jscontext.cj`, `js_func.cj` | `checkLifecycleAndThread()` |
| 引用释放检测 | `js_func.cj` | `isSafe` 标记 |
| 类型检查 | `js_exception.cj` | `jsTypeMisMatch()` |
| 边界检查 | `utf16string.cpp` | `index >= length_` |
| 异常隔离 | `business_exception.cj` | 受控错误码 |

### 依赖的安全机制

| 依赖组件 | 提供的能力 |
|----------|------------|
| napi | ArkTS 运行时安全沙箱 |
| ability_runtime | 动态库加载隔离 |
| ArkTS Runtime | JS 执行隔离 |

---

## 局限性声明

1. **代码范围**: 本评审仅覆盖 `ohos/` 目录下的代码
2. **外部依赖**: napi, ability_runtime, ArkTS Runtime 的安全机制未深入评审
3. **运行时**: 运行时内存安全依赖 ArkTS Runtime 实现
4. **配置**: 构建配置安全 (编译选项) 未深入分析

---

## 后续工作

- [ ] 深入评审 napi 接口安全性
- [ ] 评审 FFI 接口安全性
- [ ] 添加模糊测试用例
- [ ] 完善安全测试覆盖
