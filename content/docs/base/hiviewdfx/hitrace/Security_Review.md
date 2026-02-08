# 安全风险评审

## 评审概述

本文档对 HiTrace 框架进行安全风险分析，识别潜在攻击面和可利用点，并提供修复建议。

**评审范围**: `/base/hiviewdfx/hitrace` (排除 test 目录)

**评审日期**: 2024-02-06

---

## 攻击面清单

### 1. N-API 接口 (外部输入)

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `begin(name, flags)` | 接收字符串名称和整型标志位 | 中 |
| `startTrace(name, taskId)` | 接收字符串和数值参数 | 中 |
| `tracepoint(mode, type, id, description)` | 接收多个参数 | 低 |
| `traceByValue(name, count)` | 接收字符串和数值 | 低 |

**证据来源**: `interfaces/js/kits/napi/src/napi_hitrace_js.cpp` - N-API 参数解析

### 2. Native API 接口

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `HiTraceChainBegin()` | 接收 C 字符串 | 中 |
| `HiTraceMeterStartTrace()` | 接收名称和任务 ID | 低 |
| 序列化/反序列化 | `HiTraceChainIdToBytes()` / `HiTraceChainBytesToId()` | 高 |

**证据来源**: `interfaces/native/innerkits/include/hitrace/hitracechainc.h` - C API 声明

### 3. 命令行工具

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `hitrace` 命令参数 | 用户输入的命令行参数 | 低 |

**证据来源**: `cmd/hitrace_cmd.cpp` - 命令行解析

### 4. 配置文件

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `hitrace.para` | 系统参数文件 | 低 |
| `hitrace.cfg` | 配置文件 | 低 |

**证据来源**: `config/` 目录

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      不可信区域                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  外部应用   │  │  IPC 远程   │  │  文件输入   │          │
│  │  (JS/N-API) │  │  进程数据   │  │  (命令行)   │          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
└─────────┼────────────────┼────────────────┼──────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────┐
│                      信任边界 (N-API 层)                      │
│  - 参数类型校验                                               │
│  - 长度边界检查                                               │
│  - 空指针检查                                                 │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      可信区域 (Native 层)                    │
│  - TLS 存储                                                   │
│  - 内部数据转换                                               │
│  - 日志输出                                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 可被利用点分析

### 风险 1: 缓冲区溢出风险 (低风险)

**证据**: `napi_hitrace_js.cpp:45-55`

```cpp
bool ParseStringParam(const napi_env& env, const napi_value& origin, std::string& dest)
{
    if (!NapiHitraceUtil::CheckValueTypeValidity(env, origin, napi_valuetype::napi_string)) {
        return false;
    }
    char buf[BUF_SIZE_64] = {0};  // 固定 64 字节缓冲区
    size_t bufLength = 0;
    napi_get_value_string_utf8(env, origin, buf, BUF_SIZE_64, &bufLength);
    dest = std::string {buf};
    return true;
}
```

**问题描述**:
- 固定大小缓冲区 `BUF_SIZE_64 = 64` 字节
- 超过 63 字节的字符串会被截断
- 依赖 `napi_get_value_string_utf8` 自动截断，未返回实际长度

**触发条件**:
```javascript
hiTraceChain.begin("VeryLongNameStringThatExceeds64CharactersLimitAndWillBeTruncated...", flags);
```

**影响**: 字符串截断可能导致追踪名称不完整，影响问题定位

**修复建议**:
```cpp
// 使用动态分配获取实际长度
size_t actualLength = 0;
napi_get_value_string_utf8(env, origin, nullptr, 0, &actualLength);
if (actualLength > 0 && actualLength < MAX_ALLOWED_LENGTH) {
    std::string buf(actualLength + 1, '\0');
    napi_get_value_string_utf8(env, origin, buf.data(), buf.size(), &bufLength);
    dest = buf;
}
```

---

### 风险 2: 整数溢出风险 (低风险)

**证据**: `napi_hitrace_meter.cpp:78-90`

```cpp
bool GetStringParam(const napi_env& env, const napi_value& value, std::string& dest)
{
    constexpr int nameMaxSize = 1024;
    char buf[nameMaxSize] = {0};  // VLA (Variable Length Array)
    size_t len = 0;
    napi_status status = napi_get_value_string_utf8(env, value, buf, nameMaxSize, &len);
    dest = std::string {buf};
    return status == napi_ok;
}
```

**问题描述**:
- 使用 VLA (Variable Length Array) 分配栈空间
- 极端情况下可能栈溢出

**影响**: 栈溢出可能导致程序崩溃

**修复建议**:
```cpp
// 使用 std::vector 或动态分配
std::vector<char> buf(nameMaxSize);
napi_get_value_string_utf8(env, value, buf.data(), nameMaxSize, &len);
```

---

### 风险 3: 序列化/反序列化风险 (高风险)

**证据**: `hitracechainc.h:228-257`

```c
static inline int HiTraceChainIdToBytes(const HiTraceIdStruct* pId, uint8_t* pIdArray, int len)
{
    // ... 序列化逻辑
    return HITRACE_INFO_ALL_VALID;
}

static inline HiTraceIdStruct HiTraceChainBytesToId(const uint8_t* pIdArray, int len)
{
    HiTraceIdStruct id;
    // 反序列化逻辑
    return id;
}
```

**问题描述**:
- `len` 参数未严格校验范围
- 反序列化时 `len` 可能为负数或超过缓冲区大小
- 恶意构造的字节流可能导致数据越界读取

**触发条件**:
```cpp
// 恶意构造的字节流
uint8_t maliciousData[256];
HiTraceChainBytesToId(maliciousData, 256);  // 超出预期长度
```

**影响**: 信息泄露、内存访问越界

**修复建议**:
```c
static inline HiTraceIdStruct HiTraceChainBytesToId(const uint8_t* pIdArray, int len)
{
    // 严格校验长度范围
    if (pIdArray == nullptr || len < (int)sizeof(HiTraceIdStruct)) {
        HiTraceIdStruct invalid = {0};
        return invalid;
    }
    // ...
}
```

---

### 风险 4: 空指针解引用 (中风险)

**证据**: `napi_hitrace_js.cpp:117-121`

```cpp
static napi_value End(napi_env env, napi_callback_info info)
{
    // ...
    HiTraceId traceId;
    if (!ParseTraceIdObject(env, params[ParamIndex::PARAM_FIRST], traceId)) {
        HILOG_ERROR(LOG_CORE, "hitarce id type must be object.");
        return nullptr;  // 返回 nullptr
    }
    HiTraceChain::End(traceId, LOG_DOMAIN);
    return nullptr;
}
```

**问题描述**:
- 参数校验失败时返回 `nullptr`
- 调用方未检查返回值，可能导致后续空指针解引用

**触发条件**:
```typescript
const invalidId = null;
hiTraceChain.end(invalidId);  // JS 层传入 null
```

**影响**: 潜在空指针解引用（但 N-API 层有保护）

**修复建议**: JS 层已做类型检查，风险可控

---

### 风险 5: 日志注入 (低风险)

**证据**: `napi_hitrace_js.cpp:169-207`

```cpp
static napi_value Tracepoint(napi_env env, napi_callback_info info)
{
    // ...
    HiTraceChain::Tracepoint(communicationMode, tracePointType, traceId, LOG_DOMAIN, "%s", description.c_str());
    return nullptr;
}
```

**问题描述**:
- `description` 参数直接作为格式化字符串传递
- 可能包含日志注入字符

**影响**: 日志输出被篡改，可能影响调试和问题定位

**修复建议**:
```cpp
// 对特殊字符进行转义或过滤
std::string safeDescription = SanitizeLogString(description);
HiTraceChain::Tracepoint(..., "%s", safeDescription.c_str());
```

---

### 风险 6: 竞态条件 (低风险)

**证据**: `hitracechainc.c:90`

```c
static __thread HiTraceIdStructInner g_hiTraceId = {{0, 0, 0, 0, 0, 0}, {0, 0}};
```

**问题描述**:
- TLS 存储是线程安全的
- 但跨线程传递 TraceId 时存在时间窗口

**触发条件**:
```cpp
HiTraceId id = HiTraceChain::GetId();  // 获取当前线程的 TraceId
// ... 时间窗口，其他线程可能修改 TLS
HiTraceChain::SetId(id);  // 设置的可能是旧值
```

**影响**: 追踪数据不一致

**修复建议**: 使用 `SaveAndSet()` 和 `Restore()` 配对操作

---

## 已确认的安全机制

### 1. 类型校验

**证据**: `napi_hitrace_util.cpp`

```cpp
static bool CheckValueTypeValidity(const napi_env& env, const napi_value& value, napi_valuetype expectType)
{
    napi_valuetype valueType;
    napi_status status = napi_typeof(env, value, &valueType);
    if (status != napi_ok || valueType != expectType) {
        return false;
    }
    return true;
}
```

**评估**: ✅ 良好 - 所有 N-API 参数都进行类型校验

### 2. 参数计数校验

**证据**: `napi_hitrace_js.cpp:83-87`

```cpp
if (paramNum != ParamNum::TOTAL_ONE && paramNum != ParamNum::TOTAL_TWO) {
    HILOG_ERROR(LOG_CORE,
        "failed to begin a new trace, count of parameters is not equal to 1 or 2");
    return val;
}
```

**评估**: ✅ 良好 - 检查参数数量

### 3. 栈保护编译选项

**证据**: `interfaces/native/innerkits/BUILD.gn:33`

```gn
cflags = [ "-fstack-protector-strong" ]
```

**评估**: ✅ 良好 - 启用栈保护

### 4. 版本号校验

**证据**: `hitracechainc.c:82-102`

```c
#if __BYTE_ORDER == __LITTLE_ENDIAN
    uint64_t valid : 1;        // Validity flag
    uint64_t ver : 3;          // Version (HITRACE_VER_1)
    // ...
#endif
```

**评估**: ✅ 良好 - 数据结构包含版本和有效性标志

---

## 风险汇总表

| 风险编号 | 风险类型 | 风险等级 | 影响范围 | 状态 |
|----------|----------|----------|----------|------|
| R-01 | 缓冲区溢出 | 低 | 字符串截断 | 已缓解 |
| R-02 | 整数溢出 | 低 | 栈溢出 | 需重构 |
| R-03 | 序列化越界 | 高 | 内存访问 | 需修复 |
| R-04 | 空指针 | 中 | 潜在崩溃 | 已缓解 |
| R-05 | 日志注入 | 低 | 日志篡改 | 需关注 |
| R-06 | 竞态条件 | 低 | 数据不一致 | 需文档 |

---

## 修复优先级

### 高优先级 (建议立即修复)

| 问题 | 风险编号 | 修复建议 |
|------|----------|----------|
| 序列化越界 | R-03 | 添加 `len` 参数范围校验 |

### 中优先级 (建议下个版本修复)

| 问题 | 风险编号 | 修复建议 |
|------|----------|----------|
| VLA 使用 | R-02 | 使用 `std::vector` 替代 |

### 低优先级 (建议文档说明)

| 问题 | 风险编号 | 建议 |
|------|----------|------|
| 字符串截断 | R-01 | 文档说明限制 |
| 日志注入 | R-05 | 使用时注意过滤 |
| 竞态条件 | R-06 | 使用规范 API |

---

## 安全使用建议

### 1. N-API 调用规范

```typescript
// ✅ 正确：检查返回值
const traceId = hiTraceChain.begin("operation");
if (hiTraceChain.isValid(traceId)) {
    hiTraceChain.end(traceId);
}

// ❌ 错误：不做检查
hiTraceChain.begin("operation");
hiTraceChain.end(null);
```

### 2. 避免过长追踪名称

```typescript
// ✅ 正确：限制名称长度
const shortName = longName.substring(0, 63);
hiTraceChain.begin(shortName);
```

### 3. 跨进程传递使用序列化

```typescript
// ✅ 正确：使用 IPC 框架自动处理
// IPC 框架会自动处理 TraceId 的序列化/反序列化

// ❌ 错误：手动处理可能出错
const bytes = traceId.toBytes();  // 内部可能有问题
ipc.sendBytes(bytes);
```

---

## 检查范围与局限性

### 已检查内容

- ✅ N-API 参数解析 (`interfaces/js/kits/napi/`)
- ✅ C API 入口 (`interfaces/native/innerkits/`)
- ✅ 核心框架实现 (`frameworks/native/`)
- ✅ 序列化/反序列化逻辑
- ✅ 构建配置安全选项

### 未检查内容

- ❌ 测试代码 (`test/`)
- ❌ IPC 框架集成（由其他模块负责）
- ❌ 内核 tracepoint 实现
- ❌ 运行时动态加载

---

## 参考资料

- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs)
- [N-API 安全最佳实践](https://nodejs.org/api/n-api.html)
- [CWE-120: Buffer Copy without Checking Size](https://cwe.mitre.org/data/definitions/120.html)
- [CWE-787: Out-of-bounds Write](https://cwe.mitre.org/data/definitions/787.html)
