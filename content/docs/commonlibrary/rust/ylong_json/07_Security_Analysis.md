# ylong_json 安全风险分析

## 目的

本文档对 `ylong_json` 进行安全风险评审，识别攻击面、可被利用点和修复建议。

## 适用范围

- 安全审计人员
- 安全架构师
- 开发团队

## 检查范围

| 检查项 | 范围 | 状态 |
|--------|------|------|
| C FFI 接口 | `src/adapter.rs` | ✅ 已检查 |
| 解析器 | `src/states.rs` | ✅ 已检查 |
| 内存管理 | 所有 unsafe 代码 | ✅ 已检查 |
| 输入验证 | 字符串/数字解析 | ✅ 已检查 |
| 递归安全 | 深度限制 | ✅ 已检查 |
| IPC/SA | 全仓库 | ✅ 无相关代码 |
| 文件系统 | 全仓库 | ✅ 无直接访问 |
| 网络 | 全仓库 | ✅ 无相关代码 |
| 权限校验 | 全仓库 | ✅ 无相关代码 |

## 攻击面清单

### 1. C FFI 接口（主要攻击面）

**位置**: `src/adapter.rs`（3407 行）

**风险等级**: 🔴 高

**说明**:
- 90+ 个 `unsafe extern "C"` 函数
- 大量原始指针操作
- 调用方负责内存管理

### 2. JSON 解析器

**位置**: `src/states.rs`

**风险等级**: 🟡 中

**说明**:
- 递归解析 JSON 结构
- 字符串转义处理
- 数字解析

### 3. 输入数据

**风险等级**: 🟡 中

**说明**:
- 外部传入的 JSON 文本
- 可能包含恶意构造的数据

## 可被利用点

### 🔴 风险 1: 空指针解引用

**证据**: `src/adapter.rs:40-59`

```rust
pub unsafe extern "C" fn ylong_json_parse(
    value: *mut c_char,
    err_msg: *mut *mut c_char,
) -> *mut YlongJson {
    let len = strlen(value);  // ⚠️ 如果 value 为 null，此处崩溃
    let slice = &*slice_from_raw_parts(value as *mut u8, len);
    // ...
}
```

**触发条件**:
- C 调用方传入 null 指针

**影响**:
- 进程崩溃（DoS）

**修复建议**:
```rust
pub unsafe extern "C" fn ylong_json_parse(
    value: *mut c_char,
    err_msg: *mut *mut c_char,
) -> *mut YlongJson {
    if value.is_null() {
        if !err_msg.is_null() {
            *err_msg = CString::new("Input is null").unwrap().into_raw();
        }
        return NULL_MUT_YLONG_JSON;
    }
    // ...
}
```

**当前状态**: ⚠️ 部分函数有检查，部分没有

---

### 🔴 风险 2: 内存泄漏

**证据**: `src/adapter.rs:487-508`

```rust
pub unsafe extern "C" fn ylong_json_add_item_to_array(
    array: *mut YlongJson,
    item: *mut YlongJson,
) -> c_int {
    // ...
    let value = Box::from_raw(item as *mut JsonValue);
    array_ref.push(*value);  // 值被复制，原始 Box 被释放
    // 但如果 push 失败，value 会丢失
    SUCCESS
}
```

**触发条件**:
- 内存分配失败
- 异常路径未正确处理

**影响**:
- 内存泄漏

**修复建议**:
确保所有路径都正确处理内存。

---

### 🔴 风险 3: use-after-free

**证据**: `src/adapter.rs:382-400`

```rust
pub unsafe extern "C" fn ylong_json_get_value_from_string(
    string: *const YlongJson,
    value: *mut *mut c_char,
) -> c_int {
    // ...
    let string = match string.try_as_string() {
        Ok(s) => s,
        Err(_) => return FAILURE,
    };
    // 返回内部指针，调用方不应释放
    *value = string.as_ptr() as *mut c_char;
    SUCCESS
}
```

**触发条件**:
- C 调用方释放返回的指针
- 后续访问已释放内存

**影响**:
- 内存损坏
- 可能代码执行

**修复建议**:
文档明确说明调用方不应释放此指针。

---

### 🟡 风险 4: 整数溢出

**证据**: `src/adapter.rs:445-461`

```rust
pub unsafe extern "C" fn ylong_json_get_array_size(
    array: *const YlongJson,
    size: *mut c_int,
) -> c_int {
    // ...
    *size = array.len() as c_int;  // ⚠️ 如果 len > i32::MAX，溢出
    SUCCESS
}
```

**触发条件**:
- 超大数组（超过 2^31-1 个元素）

**影响**:
- 整数溢出
- 后续操作可能出错

**修复建议**:
```rust
if array.len() > c_int::MAX as usize {
    return FAILURE;
}
*size = array.len() as c_int;
```

---

### 🟡 风险 5: 递归深度攻击

**证据**: `src/consts.rs:87`

```rust
pub(crate) const RECURSION_LIMIT: u32 = 128;
```

**触发条件**:
- 构造深度超过 128 层的嵌套 JSON

**影响**:
- 返回错误 `ExceedRecursionLimit`
- 不会栈溢出

**当前状态**: ✅ 有保护，但深度可能不足

**修复建议**:
考虑增加递归深度限制或使其可配置。

---

### 🟡 风险 6: UTF-8 处理不安全

**证据**: `src/adapter.rs:46-48, 357-365`

```rust
// 多处使用 from_utf8_unchecked
let str = from_utf8_unchecked(slice);
```

**触发条件**:
- 传入非法 UTF-8 字节序列

**影响**:
- 未定义行为

**修复建议**:
使用安全的 `from_utf8` 并处理错误。

---

### 🟡 风险 7: 拒绝服务（大输入）

**证据**: `src/states.rs`（解析逻辑）

**触发条件**:
- 超大 JSON 文件（GB 级别）
- 大量嵌套结构
- 超长字符串

**影响**:
- 内存耗尽
- CPU 耗尽

**当前状态**: ⚠️ 无输入大小限制

**修复建议**:
- 添加输入大小限制
- 流式解析（当前为全量解析）

---

### 🟢 风险 8: 类型混淆

**证据**: `src/adapter.rs`（多处类型转换）

```rust
let item = &*(item as *mut JsonValue);  // 原始指针转换
```

**触发条件**:
- C 调用方传入错误类型的指针

**影响**:
- 类型混淆
- 内存损坏

**当前状态**: ⚠️ 运行时类型检查有限

**修复建议**:
- 添加类型标记
- 使用句柄表而非原始指针

## 信任边界

```
┌─────────────────────────────────────────┐
│              不可信区域                  │
│  ┌─────────────────────────────────┐    │
│  │        C 调用方代码              │    │
│  │    （可能包含恶意输入）           │    │
│  └────────────┬────────────────────┘    │
│               │ 原始指针                  │
│               ▼                          │
│  ┌─────────────────────────────────┐    │
│  │        C FFI 接口层              │    │
│  │      (src/adapter.rs)           │    │
│  └────────────┬────────────────────┘    │
│               │ 信任边界                  │
└───────────────┼─────────────────────────┘
                │
┌───────────────┼─────────────────────────┐
│               ▼                          │
│  ┌─────────────────────────────────┐    │
│  │         可信区域                 │    │
│  │  ┌─────────────────────────┐    │    │
│  │  │    Rust 核心实现         │    │    │
│  │  │  (value.rs, states.rs)  │    │    │
│  │  └─────────────────────────┘    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

## 修复建议汇总

### 高优先级

1. **统一空指针检查**
   - 所有 FFI 函数添加 null 检查
   - 返回明确的错误码

2. **内存安全文档**
   - 明确内存所有权规则
   - 说明哪些指针需要释放

3. **UTF-8 安全处理**
   - 替换 `from_utf8_unchecked` 为安全版本

### 中优先级

4. **输入大小限制**
   - 添加最大输入大小限制
   - 防止资源耗尽攻击

5. **整数溢出检查**
   - 添加溢出检查
   - 返回错误而非静默截断

6. **增强类型安全**
   - 考虑使用句柄表
   - 添加类型标记

### 低优先级

7. **递归深度可配置**
   - 允许运行时调整递归限制

8. **安全日志**
   - 记录异常输入
   - 便于安全审计

## 局限性说明

本次审计的局限性：

1. **静态分析**: 基于代码阅读，未进行动态 fuzz 测试
2. **边界情况**: 未测试所有异常输入
3. **并发安全**: 未分析多线程场景
4. **依赖库**: 未审计 serde 等第三方库

## 总体评估

| 评估项 | 等级 | 说明 |
|--------|------|------|
| 整体安全 | 🟡 中 | 基本安全，但有改进空间 |
| C FFI 安全 | 🔴 低 | 大量 unsafe 代码，需谨慎使用 |
| 解析器安全 | 🟢 高 | 有递归保护，输入验证充分 |
| 内存安全 | 🟡 中 | 依赖调用方正确使用 |

## 相关跳转

- [对外 API](03_Public_API.md) - C FFI 接口详情
- [架构说明](01_Architecture.md) - 架构设计
- [目录结构](02_Directory_Structure.md) - 代码组织
