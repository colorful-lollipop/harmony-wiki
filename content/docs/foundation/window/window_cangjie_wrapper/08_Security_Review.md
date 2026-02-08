# 安全风险评审

> 基于代码证据的安全风险分析、攻击面、可被利用点、修复建议

---

## 目的

本文档基于 `window_cangjie_wrapper` 的代码证据，进行安全风险评审，识别潜在的安全问题和提供修复建议。

---

## 威胁模型

```mermaid
graph TB
    subgraph外部输入[外部输入]
        I1[用户输入<br/>窗口名称/颜色/路径]
        I2[应用调用<br/>createWindow/setWindowProperties]
        I3[系统事件<br/>键盘高度/显示变化]
        I4[恶意应用<br/>权限提升]
    end

    subgraph输入校验层[输入校验层]
        V1[空值检查]
        V2[类型验证]
        V3[长度/范围校验]
    end

    subgraph Native层[Native 层]
        N1[window_manager<br/>窗口管理服务]
        N2[ability_runtime<br/>权限验证]
        N3[FFI Bridge<br/>指针操作]
    end

    subgraph敏感操作[敏感操作]
        O1[窗口创建<br/>TYPE_FLOAT]
        O2[隐私模式<br/>setWindowPrivacyMode]
        O3[内容加载<br/>loadContent(path)]
        O4[窗口控制<br/>showWindow/destroyWindow]
    end

    I1 --> V1
    I1 --> V2
    I1 --> V3
    I2 --> V1
    I3 --> V1
    I4 --> O1
    I4 --> O2

    V1 --> N1
    V2 --> N1
    V3 --> N1
    V1 --> N3
    V3 --> N3

    I2 --> N2
    I2 --> O3
    I2 --> O4
    I3 --> O1
    I3 --> O2
```

---

## 攻击面清单

### 数据输入攻击面

| 攻击类型 | 入口点 | 风险等级 | 证据 |
|----------|---------|----------|--------|
| **路径遍历** | `loadContent(path)` 参数 | 中 | `window_stage.cj:149-155` |
| **格式化字符串** | `setWindowBackgroundColor(color)` | 低 | `window.cj:456-463` |
| **数值溢出** | `resize(width, height)` | 低 | `window.cj:346-351` |
| **类型混淆** | 枚举参数类型错误 | 低 | `cj_window_enum.cj:319,512` |

### 权限攻击面

| 攻击类型 | 入口点 | 风险等级 | 证据 |
|----------|---------|----------|--------|
| **权限绕过** | 创建 TYPE_FLOAT 窗口 | 高 | `window.cj:187` |
| **权限提升** | 调用敏感 API 无权限 | 中 | Native 层权限检查 |
| **权限泄露** | 错误消息泄露权限信息 | 低 | 错误码信息 |

### 资源管理攻击面

| 攻击类型 | 入口点 | 风险等级 | 证据 |
|----------|---------|----------|--------|
| **内存泄露** | Window/Display 实例未正确释放 | 中 | `window.cj:290-297` |
| **悬垂指针** | FFI 调用后 Native 资源已释放 | 高 | `window.cj:32-146` |
| **双重释放** | Native 资源被多次释放 | 中 | RemoteDataLite 机制 |
| **UAF** | 使用已释放的 Native 资源 | 高 | FFI 指针检查不足 |

### 竞态条件攻击面

| 攻击类型 | 入口点 | 风险等级 | 证据 |
|----------|---------|----------|--------|
| **竞态注册** | 多线程注册同一回调 | 中 | `window.cj:777-801` |
| **竞态注销** | 回调注销期间触发回调 | 中 | `window.cj:846-862` |
| **Map 并发** | INSTANCE_MAP 并发访问不一致 | 中 | `window.cj:288` |

### 信息泄露攻击面

| 攻击类型 | 入口点 | 风险等级 | 证据 |
|----------|---------|----------|--------|
| **敏感信息日志** | 日志中打印窗口 ID、名称等 | 中 | `window.cj:292`、`display.cj:513` |
| **错误信息泄露** | 错误消息包含内部状态 | 低 | `cj_window_utils.cj:23-50` |

---

## 可被利用点

### 1. 路径遍历漏洞

**严重等级**：中

**证据**：`window_stage.cj:149-155`

```cangjie
public func loadContent(path: String): Unit {
    unsafe {
        try (cpath = LibC.mallocCString(path).asResource()) {
            FfiOHOSLoadContent(getID(), cpath.value)
        }
    }
}
```

**问题**：
- 未对 `path` 参数进行路径遍历检查
- 未规范化路径（如 `../file.ets`）
- 可能加载任意应用文件

**触发路径**：
1. 恶意应用调用 `windowStage.loadContent("../../../data/sensitive.ets")`
2. 尝试读取应用私有数据
3. 可能读取系统文件（如支持绝对路径）

**影响**：
- 信息泄露：读取应用或系统敏感文件
- 代码执行：配合其他漏洞可能执行任意代码

**修复建议**：
```cangjie
public func loadContent(path: String): Unit {
    unsafe {
        // 添加路径验证
        if (path.isEmpty() || path.contains("..") || path.startsWith("/")) {
            throw BusinessException(401, "Invalid path parameter")
        }
        // 限制路径长度
        if (path.size > 256) {
            throw BusinessException(401, "Path too long")
        }

        try (cpath = LibC.mallocCString(path).asResource()) {
            FfiOHOSLoadContent(getID(), cpath.value)
        }
    }
}
```

---

### 2. 权限验证不足

**严重等级**：高

**证据**：`window.cj:190-209`、`cj_window_enum.cj:52-59`

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.SYSTEM_FLOAT_WINDOW",  // 仅标注，未强制检查
    syscap: "SystemCapability.WindowManager.WindowManager.Core"
]
public func createWindow(config: Configuration): Window {
    // 权限检查仅依赖 Native 层
    // 仓颉层未进行二次验证
}
```

**问题**：
- 权限标注（`@!APILevel[permission: ..."]`）仅用于文档生成
- 实际权限检查完全依赖 Native 层（`window_manager` 子系统）
- 如果 Native 层存在漏洞，可能被绕过

**触发路径**：
1. 恶意应用调用 `createWindow(config { windowType: TypeFloat })`
2. 即使未获得权限，如果 Native 层检查失败，仍可能创建窗口
3. 可能创建浮动窗口覆盖系统 UI

**影响**：
- 权限提升：无权限应用创建浮动窗口
- UI 欺骗：创建恶意浮动窗口
- 隐私泄露：浮动窗口可截取敏感信息

**修复建议**：
```cangjie
// 添加仓颉层权限验证
public func createWindow(config: Configuration): Window {
    // 检查 TYPE_FLOAT 权限
    if (config.windowType == WindowType.TypeFloat) {
        // TODO: 调用仓颉权限检查 API（需 cangjie_ark_interop 支持）
        // throw BusinessException(201, "Permission SYSTEM_FLOAT_WINDOW required")
    }

    unsafe {
        let stageContext = unsafe {
            FFIGetContext(config.ctx.getID())
        }
        // ... 原有逻辑
    }
}
```

---

### 3. 内存泄露风险

**严重等级**：中

**证据**：`window.cj:269-289`、`window.cj:295-297`

```cangjie
public class Window <: RemoteDataLite {
    let callbackMaps = HashMap<String, ArrayList<(CallbackObject, Int64)>>()

    init(id: Int64) {
        super(id)  // 初始化 myDataId
    }

    ~init() {
        releaseFFIData(myDataId)  // 释放 Native 资源
        // 问题：callbackMaps 未清空，可能导致回调对象引用泄露
    }
}
```

**问题**：
- `~init()` 只释放 `myDataId`（Native 资源）
- `callbackMaps` 中存储的 `CallbackObject` 引用未清空
- 如果回调对象持有窗口引用，形成循环引用
- 多次创建/销毁窗口可能导致内存累积泄露

**触发路径**：
1. 应用频繁创建/销毁窗口
2. 每个窗口注册多个回调
3. 运行长时间后内存持续增长
4. 最终导致 OOM（内存溢出）

**影响**：
- 内存耗尽：系统可用内存减少
- 应用崩溃：内存不足导致应用崩溃
- 系统不稳定：影响其他应用

**修复建议**：
```cangjie
public class Window <: RemoteDataLite {
    let callbackMaps = HashMap<String, ArrayList<(CallbackObject, Int64)>>()

    init(id: Int64) {
        super(id)
    }

    ~init() {
        // 释放 Native 资源
        releaseFFIData(myDataId)

        // 清空所有回调
        synchronized(REGISTER_MUTEX) {
            for ((callbackType, list) in callbackMaps) {
                for ((callback, _) in list) {
                    // 移除回调引用，允许 GC 回收
                }
                list.clear()
            }
            callbackMaps.clear()
        }
    }
}
```

---

### 4. 竞态条件 - 回调注册

**严重等级**：中

**证据**：`window.cj:777-801`

```cangjie
func onKeyboardHeightChange(callbackType: String, callback: Callback1Argument<UInt32>): Unit {
    synchronized(REGISTER_MUTEX) {
        var value = callbackMaps.entryView(callbackType)
        if (value.value.isNone()) {
            WINDOW_LIB_LOG.error("[Window] Invalid param.")
            return  // 竞态窗口：多个线程同时进入
        }

        // 未使用 findCallbackObject 检查重复注册
        let registerCall = Callback1Param<CPointer<Unit>, Unit>(wrapper)
        // ... 注册逻辑
    }
}
```

**问题**：
- 检查 `isNone()` 后直接返回，未检查回调是否已注册
- 多个线程可以注册相同的回调对象
- 同一回调类型可以注册多个实例
- 注销时可能无法正确清理所有注册

**触发路径**：
1. 应用多线程同时注册回调
2. 线程 A 注册回调 `cb1`
3. 线程 B 也注册相同回调 `cb1`
4. 注销时只移除一次，导致 `cb1` 仍然活跃
5. Native 层仍然调用已释放的回调

**影响**：
- 悬垂回调：回调被销毁后仍被调用
- 内存泄露：回调对象引用无法释放
- 应用崩溃：回调访问已释放内存

**修复建议**：
```cangjie
func onKeyboardHeightChange(callbackType: String, callback: Callback1Argument<UInt32>): Unit {
    synchronized(REGISTER_MUTEX) {
        var value = callbackMaps.entryView(callbackType)
        if (value.value.isNone()) {
            WINDOW_LIB_LOG.error("[Window] Invalid param.")
            return
        }

        // 添加重复注册检查
        if (findCallbackObject(value.value.getOrThrow(), callback)) {
            WINDOW_LIB_LOG.info("[Window] The callback object already exists.")
            return
        }

        let wrapper = {
            data: CPointer<Unit> =>
            let val = CPointer<UInt32>(data).read()
            callback.invoke(None, val)
        }
        let registerCall = Callback1Param<CPointer<Unit>, Unit>(wrapper)
        // ... 注册逻辑
    }
}
```

---

### 5. 错误信息泄露

**严重等级**：低

**证据**：`window.cj:292`、`cj_window_utils.cj:23-50`

```cangjie
let WINDOW_LIB_LOG = HilogChannel(0, HILOG_DOMAIN_WINDOW, "CJ-Window-Manager")

func checkRet(errCode: Int32, message: String) {
    if (errCode != 0) {
        if (let Some(errMsg) <- ERR_CODE_MAP.get(errCode)) {
            // 泄露窗口 ID、状态等内部信息到日志
            let msg = message + errMsg
            WINDOW_LIB_LOG.error("[Window] ERROR: ${errCode} " + errMsg)  // 潜在信息泄露
            throw BusinessException(errCode, msg)
        }
        // ...
    }
}
```

**问题**：
- 错误日志包含详细的内部状态
- 错误消息直接拼接用户输入，可能包含敏感数据
- 日志文件可能被其他应用读取（如果权限允许）

**触发路径**：
1. 恶意应用传递包含敏感信息的参数（如窗口名称包含密码）
2. 触发错误（如无效的窗口类型）
3. 敏感信息被记录到系统日志
4. 其他应用读取日志获取敏感信息

**影响**：
- 信息泄露：敏感信息通过日志泄露
- 隐私风险：用户数据暴露
- 审计绕过：绕过审计机制

**修复建议**：
```cangjie
func checkRet(errCode: Int32, message: String) {
    if (errCode != 0) {
        if (let Some(errMsg) <- ERR_CODE_MAP.get(errCode)) {
            // 脱敏处理：不记录完整错误消息
            let sanitizedMsg = sanitizeErrorMessage(errMsg)
            // 仅记录错误码和脱敏消息
            WINDOW_LIB_LOG.error("[Window] ERROR: ${errCode} - ${sanitizedMsg}")
            throw BusinessException(errCode, sanitizedMsg)
        }
        // ...
    }
}

func sanitizeErrorMessage(msg: String): String {
    // 移除可能包含的敏感信息
    // 例如：窗口名称、路径、ID 等
    return "Internal error occurred"  // 统一的通用错误消息
}
```

---

### 6. 数值参数溢出风险

**严重等级**：低

**证据**：`window.cj:346-351`

```cangjie
public func resize(width: UInt32, height: UInt32): Unit {
    unsafe {
        let ret = FfiOHOSWindowResize(getID(), width, height)
        checkRet(ret, "[Window] resize: ")
    }
}
```

**问题**：
- 未检查 `width` 和 `height` 的合理性范围
- 虽然 `UInt32` 限制了最大值，但未设置上下限
- Native 层可能未进行额外验证
- 可能导致 Native 层整数溢出

**触发路径**：
1. 恶意应用传递超大数值（接近 UInt32.MAX）
2. 调用 `resize(4294967295, 4294967295)`
3. Native 层计算时可能溢出
4. 导致缓冲区溢出或内存损坏

**影响**：
- 拒绝服务：Native 层拒绝操作
- 内存损坏：整数溢出导致内存破坏
- 权限提升：利用溢出漏洞执行任意代码

**修复建议**：
```cangjie
const MAX_WINDOW_WIDTH: UInt32 = 7680
const MAX_WINDOW_HEIGHT: UInt32 = 4320

public func resize(width: UInt32, height: UInt32): Unit {
    unsafe {
        // 添加合理性检查
        if (width == 0 || height == 0) {
            throw BusinessException(401, "Window size cannot be zero")
        }
        if (width > MAX_WINDOW_WIDTH || height > MAX_WINDOW_HEIGHT) {
            throw BusinessException(401, "Window size exceeds maximum limit")
        }

        let ret = FfiOHOSWindowResize(getID(), width, height)
        checkRet(ret, "[Window] resize: ")
    }
}
```

---

### 7. 格式化字符串注入

**严重等级**：中

**证据**：`window.cj:456-463`

```cangjie
public func setWindowBackgroundColor(color: String): Unit {
    unsafe {
        try (rowColor = LibC.mallocCString(color).asResource()) {
            let ret = FfiOHOSSetWindowBackgroundColor(getID(), rowColor.value)
            checkRet(ret, "[Window] setWindowBackgroundColor: ")
        }
    }
}
```

**问题**：
- 未验证 `color` 参数的格式（如 `#RRGGBB`）
- 未检查颜色值的有效性
- Native 层可能将字符串直接传递给底层
- 可能导致解析错误或资源泄露

**触发路径**：
1. 恶意应用传递特殊格式颜色（如包含 ANSI 转义序列）
2. 导致 Native 层解析失败
3. 可能触发缓冲区溢出或命令注入

**影响**：
- 拒绝服务：Native 层拒绝处理
- 内存损坏：格式化字符串漏洞
- 信息泄露：错误信息泄露系统信息

**修复建议**：
```cangjie
// 添加颜色格式验证
func isValidColor(color: String): Bool {
    // 检查是否符合 #RRGGBB 格式
    if (color.size != 7 || !color.startsWith("#")) {
        return false
    }
    // 检查十六进制字符有效性
    for (i in 1..6) {
        let char = color[i]
        if (!((char >= '0' && char <= '9') ||
              (char >= 'a' && char <= 'f') ||
              (char >= 'A' && char <= 'F'))) {
            return false
        }
    }
    return true
}

public func setWindowBackgroundColor(color: String): Unit {
    // 验证颜色格式
    if (!isValidColor(color)) {
        throw BusinessException(401, "Invalid color format")
    }

    unsafe {
        try (rowColor = LibC.mallocCString(color).asResource()) {
            let ret = FfiOHOSSetWindowBackgroundColor(getID(), rowColor.value)
            checkRet(ret, "[Window] setWindowBackgroundColor: ")
        }
    }
}
```

---

## 安全检查范围与局限性

### 已检查范围

✅ **输入验证**：路径遍历、空值检查、类型验证
✅ **权限检查**：权限标注、Native 层依赖
✅ **内存安全**：资源生命周期、指针操作
✅ **竞态条件**：回调注册/注销、Map 并发
✅ **信息泄露**：日志脱敏、错误消息处理
✅ **数值溢出**：参数范围检查

### 未检查范围

⚠️ **Native 层实现**：不在当前仓库中，无法验证 Native 层的安全实现
⚠️ **权限系统**：未深入分析 OpenHarmony 权限系统的实现细节
⚠️ ** IPC 通信**：未检查 FFI 调用跨进程通信的安全性
⚠️ **系统调用**：未分析 Native 层系统调用的安全检查

---

## 修复优先级建议

| 风险 | 优先级 | 预计工作量 |
|--------|--------|----------|
| 路径遍历 | 高 | 2-3 小时 |
| 权限验证 | 高 | 4-6 小时（需 Native 层配合） |
| 内存泄露 | 高 | 3-5 小时 |
| 竞态条件 | 中 | 2-4 小时 |
| 信息泄露 | 低 | 1-2 小时 |
| 数值溢出 | 低 | 1-2 小时 |
| 格式化字符串 | 中 | 2-3 小时 |

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 识别 7 个可被利用点，覆盖输入验证、权限、内存、竞态、信息泄露等方面 | 代码分析 |
| 主要风险：路径遍历（中）、权限验证不足（高）、内存泄露（中） | 风险等级评估 |
| 所有风险都有具体的修复建议和代码示例 | 修复建议 |
| 仓颉封装层需要增强输入验证和安全检查 | 安全改进方向 |

---

**生成时间**: 2025-02-06
