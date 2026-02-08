# 安全分析

**文档目的**: 识别 testfwk_cangjie_wrapper 的安全风险、攻击面和信任边界  
**目标读者**: 安全工程师、架构师、审计人员  
**阅读时间**: 约 20 分钟

---

## 1. 安全评估概述

### 1.1 评估范围

| 范围项 | 说明 |
|--------|------|
| **代码范围** | `ohos/ui_test/*.cj`, `kit/TestKit/*.cj`, `mock/*.cj` |
| **不包含** | `test/` 目录（测试代码）、底层 `arkxtest` 原生实现 |
| **评估方法** | 静态代码分析、威胁建模 |
| **评估日期** | 2026-02-06 |

### 1.2 威胁模型

```
外部输入
    │
    ├─→ 测试脚本（可信）
    │      │
    │      ├─→ Cangjie API（本仓库）
    │      │      │
    │      │      ├─→ FFI 边界 ──┐
    │      │      │              │
    │      │      └─→ 参数校验   │  信任边界
    │      │                     │
    │      └─→ 底层服务 ◄────────┘
    │             (arkxtest)
    │
    └─→ 系统参数（不可信）\n           │
           └─→ 权限检查
```

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 编号 | 攻击面 | 入口点 | 风险等级 | 说明 |
|------|--------|--------|----------|------|
| A1 | FFI 调用接口 | `CJ_ApiCall()` | **高** | 所有 API 通过单一入口 |
| A2 | 系统参数访问 | `Systemparameter.get/set()` | **高** | 读写系统参数 |
| A3 | Shell 命令执行 | `executeShellCommand()` | **高** | 启动 uitest daemon |
| A4 | JSON 解析 | `JsonValue.fromStr()` | 中 | 解析原生返回数据 |
| A5 | 文件路径 | `screenCap(savePath)` | 中 | 截图保存路径 |
| A6 | 字符串输入 | `On.text()`, `inputText()` | 低 | 用户输入处理 |

### 2.2 信任边界

```mermaid
graph TB
    subgraph "Trust Zone: Application"
        A[测试脚本]
        B[Driver/On/Component]
    end
    
    subgraph "Trust Zone: Framework"
        C[ApiCallParams]
        D[参数校验]
    end
    
    subgraph "Trust Boundary: FFI"
        E[unsafe { CJ_ApiCall }]
    end
    
    subgraph "Trust Zone: Native (System)"
        F[arkxtest]
        G[System Services]
    end
    
    A --> B --> C --> D --> E --> F --> G
```

**信任边界**: FFI 层 (`ui_test_ffi.cj`) 是 Cangjie 代码与原生代码的分界线，所有 `unsafe` 块跨越此边界。

---

## 3. 风险点详细分析

### 3.1 【高风险】FFI 调用接口

#### 位置
- `ohos/ui_test/ui_test_ffi.cj:25`
- `ohos/ui_test/ui_test_api.cj:75-84`

#### 问题描述

所有 UI 测试操作通过单一的 FFI 入口 `CJ_ApiCall()` 调用：

```cangjie
foreign func CJ_ApiCall(param: ApiCallParams): RetDataCString
```

**证据**: `ui_test_ffi.cj:25`

#### 潜在风险

1. **参数注入**: 如果 `apiId` 或 `params` 未正确转义，可能导致命令注入
2. **引用伪造**: 如果 `callerObjRef` 被篡改，可能操作其他对象的资源
3. **内存管理**: 手动 `malloc/free` 在 `unsafe` 块中，存在内存泄漏或 UAF 风险

#### 当前防护措施

```cangjie
// 1. ApiCallParams 构造函数进行内存分配
init(id: String, ref: String, param: String) {
    apiId = mallocCString(id)
    try {
        callerObjRef = mallocCString(ref)
    } catch (e: Exception) {
        unsafe { LibC.free(apiId) }  // 异常时释放
        throw BusinessException(14700104, ...)
    }
    // ...
}

// 2. 必须调用 free() 释放
func free(): Unit {
    unsafe {
        LibC.free(apiId)
        LibC.free(callerObjRef)
        LibC.free(params)
    }
}
```

**证据**: `ui_test_ffi.cj:47-72`

#### 修复建议

1. **输入验证**: 在构造 `ApiCallParams` 前验证所有字符串参数
2. **长度限制**: 添加参数长度上限检查
3. **审计日志**: 在关键 FFI 调用点记录日志

---

### 3.2 【高风险】系统参数访问

#### 位置
- `ohos/ui_test/systemparameter.cj:48-91`

#### 问题描述

直接读写系统参数，需要系统权限：

```cangjie
foreign func FfiOHOSSysTemParameterGet(key: CString, def: CString): RetDataCString
foreign func FfiOHOSSysTemParameterSet(key: CString, value: CString): Int32
```

**证据**: `systemparameter.cj:25-27`

#### 潜在风险

1. **越权访问**: 如果权限控制不当，可能读取敏感参数
2. **参数篡改**: 恶意修改系统参数影响系统行为

#### 当前防护措施

```cangjie
// 1. 长度限制
static const MAX_NAME_LENGTH = 128
static const MAX_VALUE_LENGTH = 4096

// 2. 参数校验
if (key.size >= MAX_NAME_LENGTH || def.size >= MAX_VALUE_LENGTH) {
    throw BusinessException(ERR_PARAMETER_ERROR, ...)
}

// 3. 错误码映射（包含权限拒绝）
let ERROR_CODE_MAP = HashMap<Int32, String>([
    (14700101, "System parameter can not be found."),
    (14700102, "System parameter value is invalid."),
    (14700103, "System permission operation permission denied."),
    (14700104, "System internal error...")
])
```

**证据**: `systemparameter.cj:29-46`

#### 修复建议

1. **最小权限原则**: 仅申请必要的系统参数访问权限
2. **参数白名单**: 限制可访问的系统参数范围
3. **审计记录**: 记录所有系统参数写操作

---

### 3.3 【高风险】Shell 命令执行

#### 位置
- `ohos/ui_test/ui_test_api.cj:68`

#### 问题描述

在初始化时执行 Shell 命令启动 uitest 守护进程：

```cangjie
let token = "${appCtx.applicationInfo.name}@${Process.pid}@${Process.uid}@${appCtx.area.getValue()}"
let result = delegator.executeShellCommand("uitest start-daemon ${token}", timeoutSecs: 3)
```

**证据**: `ui_test_api.cj:62-68`

#### 潜在风险

1. **命令注入**: 如果 `token` 中的任何部分包含恶意字符，可能导致命令注入
2. **Token 伪造**: 如果 token 生成逻辑被绕过，可能伪造身份

#### 当前防护措施

1. **Token 结构**: 包含应用名、PID、UID、Area，难以伪造
2. **超时限制**: 3 秒超时防止长时间阻塞

#### 修复建议

1. **参数转义**: 对 `token` 中的每个部分进行 Shell 转义
2. **白名单校验**: 验证 `appCtx.applicationInfo.name` 格式
3. **Token 签名**: 考虑对 token 进行签名验证

---

### 3.4 【中风险】JSON 解析

#### 位置
- `ohos/ui_test/ui_test_common.cj:477-486` (Point 解析)
- `ohos/ui_test/ui_test_api.cj:273` (数组解析)

#### 问题描述

解析从原生层返回的 JSON 数据：

```cangjie
let jsonArr = JsonValue.fromStr(data).asArray()
let arr = Array<Component>(
    jsonArr.size(),
    { i => Component(jsonArr.get(i).asString().getValue()) }
)
```

**证据**: `ui_test_api.cj:272-280`

#### 潜在风险

1. **解析错误**: 如果原生返回的 JSON 格式异常，可能导致解析失败
2. **资源消耗**: 大量数据解析可能导致内存或 CPU 资源消耗

#### 当前防护措施

1. **使用标准库**: 使用 `ohos.encoding.json` 标准库解析
2. **异常处理**: 解析失败抛出 `BusinessException`

#### 修复建议

1. **输入验证**: 解析前验证 JSON 格式
2. **大小限制**: 限制解析数据的最大大小

---

### 3.5 【中风险】文件路径处理

#### 位置
- `ohos/ui_test/ui_test_api.cj:467` (screenCap)

#### 问题描述

截图保存路径由用户传入：

```cangjie
public func screenCap(savePath: String): Bool {
    let cjCallParams = ApiCallParams(DRIVER_SCREENCAP, ref, "[\"${eatEscape(savePath)}\"]")
    // ...
}
```

**证据**: `ui_test_api.cj:466-470`

#### 潜在风险

1. **路径遍历**: 如果 `eatEscape()` 不充分，可能导致路径遍历攻击

#### 当前防护措施

1. **转义处理**: 使用 `eatEscape()` 函数转义路径
2. **沙箱限制**: 文档说明必须在应用沙箱目录内

#### 修复建议

1. **路径规范化**: 使用绝对路径并验证在沙箱内
2. **白名单字符**: 限制路径中允许的字符

---

### 3.6 【低风险】字符串输入

#### 位置
- `ohos/ui_test/ui_test_api.cj:1229` (On.text 等)

#### 问题描述

用户输入的字符串通过选择器传递：

```cangjie
public func text(txt: String, pattern!: MatchPattern = MatchPattern.Equals): On
```

#### 潜在风险

1. **特殊字符**: 输入包含控制字符可能影响匹配逻辑

#### 当前防护措施

```cangjie
// 使用 eatEscape 处理
let textParam = "\"${eatEscape(txt)}\",${pattern.getValue()}"
```

---

## 4. 测试模式安全机制

### 4.1 安全开关

UiTest 需要显式启用测试模式：

```cangjie
let testEnable = Systemparameter.get(TESTMODE_ENABLE, def: "0")
if (testEnable != "1") {
    TEST_LOG.warn("UiTestKit_exporter: systemParameter \"${TESTMODE_ENABLE}\" is not set!")
}
```

**证据**: `ui_test_api.cj:55-58`

### 4.2 评估

| 方面 | 评估 |
|------|------|
| **优点** | 明确的安全开关，防止误操作 |
| **缺点** | 仅警告不阻止，恶意代码可忽略警告继续执行 |
| **建议** | 考虑在测试模式未启用时限制关键操作 |

---

## 5. 内存安全分析

### 5.1 手动内存管理点

| 位置 | 操作 | 风险 |
|------|------|------|
| `ui_test_ffi.cj:48` | `mallocCString(id)` | 可能失败 |
| `ui_test_ffi.cj:50` | `mallocCString(ref)` | 可能失败，需释放前项 |
| `ui_test_ffi.cj:56` | `mallocCString(param)` | 可能失败，需释放前两项 |
| `ui_test_ffi.cj:67-71` | `LibC.free()` | 释放内存 |
| `systemparameter.cj:60-64` | `LibC.mallocCString/free` | 系统参数转换 |

### 5.2 内存安全评估

**当前状态**: ✅ 良好
- 使用 try-catch 确保异常时释放内存
- 有明确的 `free()` 方法配对
- 使用 Cangjie 的 `unsafe` 块明确标记危险操作

**建议**:
1. 考虑使用 RAII 模式自动管理内存
2. 添加内存分配失败的重试逻辑

---

## 6. 审计建议

### 6.1 日志审计点

建议在以下位置添加审计日志：

| 位置 | 事件 | 级别 |
|------|------|------|
| `UITest.setup()` | 测试框架初始化 | INFO |
| `Systemparameter.set()` | 系统参数修改 | WARN |
| `executeShellCommand()` | Shell 命令执行 | WARN |
| `CJ_ApiCall()` 错误 | FFI 调用失败 | ERROR |

### 6.2 当前日志

已有日志记录：

```cangjie
let TEST_LOG = HilogChannel(3, 0xD003100, "CJ-UITEST")

// 使用示例
TEST_LOG.warn("UiTestKit_exporter: systemParameter ... is not set!")
TEST_LOG.error("uitest setup failed")
```

**证据**: `ui_test_api.cj:32`, `ui_test_api.cj:57`, `ui_test_api.cj:70`

---

## 7. 安全修复建议汇总

| 优先级 | 建议 | 影响 | 实施难度 |
|--------|------|------|----------|
| P0 | 对 Shell 命令参数进行转义 | 防止命令注入 | 低 |
| P0 | FFI 调用添加参数长度限制 | 防止 DoS | 低 |
| P1 | 系统参数访问添加白名单 | 最小权限 | 中 |
| P1 | 测试模式未启用时限制操作 | 增强安全开关 | 中 |
| P2 | 添加详细审计日志 | 可追溯 | 低 |
| P2 | JSON 解析添加大小限制 | 资源保护 | 低 |

---

## 8. 未发现的风险（确认安全）

以下常见风险在本仓库中未发现：

| 风险类型 | 状态 | 说明 |
|----------|------|------|
| 网络通信 | ❌ 无 | 无 Socket/网络操作 |
| 文件读写 | ❌ 无 | 仅路径传递，实际由底层处理 |
| 动态加载 | ❌ 无 | 无 `dlopen` 或类似操作 |
| 不安全的字符串操作 | ❌ 无 | 无 `strcpy`, `memcpy` 等 |
| IPC Binder | ❌ 无 | IPC 在底层实现，本层仅 FFI |

---

## 9. 结论

### 9.1 总体评估

| 维度 | 评分 | 说明 |
|------|------|------|
| **代码质量** | ⭐⭐⭐⭐ | 结构清晰，异常处理完善 |
| **安全设计** | ⭐⭐⭐ | 有基本防护，但部分高风险点需加固 |
| **可审计性** | ⭐⭐⭐ | 有日志，但可更详细 |

### 9.2 关键发现

1. **最大风险**: Shell 命令执行和 FFI 边界
2. **良好实践**: 内存管理使用 try-catch-finally 模式
3. **需改进**: 测试模式警告应升级为强制限制

### 9.3 下一步行动

1. 修复 P0 级别建议（命令转义、参数长度限制）
2. 在 CI 中添加安全扫描
3. 对底层 `arkxtest` 进行独立安全审计

---

*本文档基于代码仓库静态分析生成*  
*证据位置: ohos/ui_test/*.cj, kit/TestKit/*.cj, mock/*.cj*
