# ylong_json 对外 API (C FFI)

## 目的

本文档详细描述 `ylong_json` 提供的 C FFI 接口，包括函数清单、参数说明、错误处理和内存管理。

## 适用范围

- C/C++ 开发者
- 需要调用 JSON 功能的系统服务开发者
- 安全审计人员

## 接口概述

ylong_json 通过 `src/adapter.rs` 提供 C FFI 接口，共 **90+ 个函数**。所有函数均为 `unsafe extern "C"`。

**关键设计**:
- 使用 `*mut c_void` 作为 `YlongJson` 句柄
- 使用 `*mut c_char` 表示 C 字符串
- 返回码：`1` 表示成功，`0` 表示失败
- 调用方负责内存管理

## 接口分类

### 1. 解析与输出

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_parse` | `value: *mut c_char, err_msg: *mut *mut c_char` | `*mut YlongJson` | 解析 JSON 字符串 |
| `ylong_json_print_unformatted` | `item: *const YlongJson` | `*mut c_char` | 输出紧凑格式 JSON |
| `ylong_json_free_string` | `string: *mut c_char` | - | 释放字符串内存 |
| `ylong_json_delete` | `item: *mut YlongJson` | - | 释放 JSON 对象 |

**证据**: `src/adapter.rs:40-101`

**调用示例**:
```c
char* json_text = "{\"key\":\"value\"}";
char* err_msg = NULL;
YlongJson* json = ylong_json_parse(json_text, &err_msg);
if (json != NULL) {
    char* output = ylong_json_print_unformatted(json);
    // 使用 output...
    ylong_json_free_string(output);
    ylong_json_delete(json);
} else {
    // 处理错误: err_msg
    ylong_json_free_string(err_msg);
}
```

### 2. 类型创建

#### Null
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_create_null` | - | `*mut YlongJson` | 创建 null 值 |

**证据**: `src/adapter.rs:129-132`

#### Boolean
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_create_bool` | `boolean: c_int` | `*mut YlongJson` | 创建布尔值 |

**证据**: `src/adapter.rs:148-152`

#### Number
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_create_double_number` | `number: c_double` | `*mut YlongJson` | 创建浮点数 |
| `ylong_json_create_int_number` | `number: c_longlong` | `*mut YlongJson` | 创建整数 |

**证据**: `src/adapter.rs:208-220`

#### String
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_create_string` | `string: *const c_char` | `*mut YlongJson` | 创建字符串 |

**证据**: `src/adapter.rs:354-366`

#### Array
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_create_array` | - | `*mut YlongJson` | 创建空数组 |

**证据**: `src/adapter.rs:424-428`

#### Object
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_create_object` | - | `*mut YlongJson` | 创建空对象 |

**证据**: `src/adapter.rs:688-692`

### 3. 类型判断

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_is_null` | `item: *mut YlongJson` | `c_int` | 是否为 null |
| `ylong_json_is_bool` | `item: *const YlongJson` | `c_int` | 是否为布尔 |
| `ylong_json_is_number` | `item: *const YlongJson` | `c_int` | 是否为数字 |
| `ylong_json_is_double_number` | `item: *const YlongJson` | `c_int` | 是否为浮点数 |
| `ylong_json_is_int_number` | `item: *const YlongJson` | `c_int` | 是否为整数 |
| `ylong_json_is_string` | `item: *const YlongJson` | `c_int` | 是否为字符串 |
| `ylong_json_is_array` | `item: *const YlongJson` | `c_int` | 是否为数组 |
| `ylong_json_is_object` | `item: *const YlongJson` | `c_int` | 是否为对象 |

**证据**: `src/adapter.rs:136-164, 224-262, 370-440, 696-704`

### 4. 值获取

#### Boolean
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_value_from_bool` | `boolean: *const YlongJson, value: *mut c_int` | `c_int` | 获取布尔值 |

**证据**: `src/adapter.rs:166-185`

#### Number
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_double_value_from_number` | `number: *const YlongJson, value: *mut c_double` | `c_int` | 获取浮点值 |
| `ylong_json_get_int_value_from_number` | `number: *const YlongJson, value: *mut c_longlong` | `c_int` | 获取整数值 |

**证据**: `src/adapter.rs:264-312`

#### String
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_value_from_string` | `string: *const YlongJson, value: *mut *mut c_char` | `c_int` | 获取字符串值 |

**注意**: 返回的指针指向内部数据，**不应被释放或修改**。

**证据**: `src/adapter.rs:380-400`

### 5. 值设置

#### Boolean
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_set_value_to_bool` | `boolean: *mut YlongJson, value: c_int` | `c_int` | 设置布尔值 |

**证据**: `src/adapter.rs:187-206`

#### Number
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_set_double_value_to_number` | `number: *mut YlongJson, value: c_double` | `c_int` | 设置浮点值 |
| `ylong_json_set_int_value_to_number` | `number: *mut YlongJson, value: c_longlong` | `c_int` | 设置整数值 |

**证据**: `src/adapter.rs:314-352`

#### String
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_set_value_to_string` | `string: *mut YlongJson, value: *const c_char` | `c_int` | 设置字符串值 |

**证据**: `src/adapter.rs:402-422`

### 6. 数组操作

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_array_size` | `array: *const YlongJson, size: *mut c_int` | `c_int` | 获取数组大小 |
| `ylong_json_get_array_item` | `array: *const YlongJson, index: c_int` | `*mut YlongJson` | 获取数组元素 |
| `ylong_json_add_item_to_array` | `array: *mut YlongJson, item: *mut YlongJson` | `c_int` | 添加元素到数组 |
| `ylong_json_replace_array_item_by_index` | `array: *mut YlongJson, index: c_int, new_item: *mut YlongJson` | `c_int` | 替换数组元素 |
| `ylong_json_remove_array_item_by_index` | `array: *mut YlongJson, index: c_int` | `*mut YlongJson` | 移除并返回数组元素 |
| `ylong_json_delete_array_item_by_index` | `array: *mut YlongJson, index: c_int` | - | 删除数组元素 |

**证据**: `src/adapter.rs:442-578`

### 7. 对象操作

| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_object_size` | `object: *mut YlongJson, size: *mut c_int` | `c_int` | 获取对象大小 |
| `ylong_json_has_object_item` | `object: *mut YlongJson, string: *const c_char` | `c_int` | 检查键是否存在 |
| `ylong_json_get_object_item` | `object: *const YlongJson, string: *const c_char` | `*mut YlongJson` | 获取对象元素 |
| `ylong_json_add_item_to_object` | `object: *mut YlongJson, string: *const c_char, item: *mut YlongJson` | `c_int` | 添加键值对 |
| `ylong_json_replace_object_item_by_index` | `object: *mut YlongJson, index: *const c_char, new_item: *mut YlongJson` | `c_int` | 替换键值对 |
| `ylong_json_remove_object_item_by_index` | `object: *mut YlongJson, index: *const c_char` | `*mut YlongJson` | 移除并返回键值对 |
| `ylong_json_delete_object_item_by_index` | `object: *mut YlongJson, index: *const c_char` | - | 删除键值对 |
| `ylong_json_get_all_object_items` | `object: *mut YlongJson, key: *mut *mut c_char, value: *mut *mut YlongJson, len: *mut c_int` | `c_int` | 获取所有键值对 |
| `ylong_json_for_each_object_item` | `object: *mut YlongJson, func: unsafe extern "C" fn(*mut YlongJson)` | `c_int` | 遍历对象元素 |

**证据**: `src/adapter.rs:706-958`

### 8. 链表优化接口（list_array/list_object feature）

当启用 `list_array` 或 `list_object` feature 时，提供额外的链表操作接口：

#### Array 链表操作
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_array_node` | `array: *mut YlongJson, index: c_int` | `*mut YlongJson` | 获取数组节点 |
| `ylong_json_get_item_from_array_node` | `array_node: *mut YlongJson` | `*mut YlongJson` | 从节点获取元素 |
| `ylong_json_add_item_to_array_then_get_node` | `array: *mut YlongJson, item: *mut YlongJson` | `*mut YlongJson` | 添加元素并返回节点 |
| `ylong_json_replace_item_of_array_node` | `array_node: *mut YlongJson, new_item: *mut YlongJson` | `c_int` | 替换节点元素 |
| `ylong_json_remove_array_node` | `array_node: *mut YlongJson` | `*mut YlongJson` | 移除节点 |
| `ylong_json_delete_array_node` | `array_node: *mut YlongJson` | - | 删除节点 |

**证据**: `src/adapter.rs:580-686`

#### Object 链表操作
| 函数 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `ylong_json_get_object_node` | `object: *const YlongJson, string: *const c_char` | `*mut YlongJson` | 获取对象节点 |
| `ylong_json_get_item_from_object_node` | `object_node: *mut YlongJson` | `*mut YlongJson` | 从节点获取元素 |
| `ylong_json_add_item_to_object_then_get_node` | `object: *mut YlongJson, string: *const c_char, item: *mut YlongJson` | `*mut YlongJson` | 添加键值对并返回节点 |
| `ylong_json_replace_item_of_object_node` | `object_node: *mut YlongJson, new_item: *mut YlongJson` | `c_int` | 替换节点元素 |
| `ylong_json_remove_object_node` | `object_node: *mut YlongJson` | `*mut YlongJson` | 移除节点 |
| `ylong_json_delete_object_node` | `object_node: *mut YlongJson` | - | 删除节点 |

**证据**: `src/adapter.rs:960-1090`

## 参数校验规则

### 空指针检查

所有函数都进行空指针检查：

```rust
if item.is_null() {
    return FAILURE; // 或 NULL_MUT_YLONG_JSON
}
```

**证据**: `src/adapter.rs:40-44` (ylong_json_parse), `src/adapter.rs:74-77` (ylong_json_print_unformatted)

### 类型检查

类型获取函数进行运行时类型检查：

```rust
let item = &*(item as *mut JsonValue);
let item = match item.try_as_array() {
    Ok(a) => a,
    Err(_) => return FAILURE,
};
```

**证据**: `src/adapter.rs:552-558` (ylong_json_get_array_item)

### 边界检查

数组索引进行边界检查：

```rust
if index as usize >= array_ref.len() {
    return NULL_MUT_YLONG_JSON;
}
```

**证据**: `src/adapter.rs:480-482`

## 错误码说明

| 常量 | 值 | 说明 |
|------|-----|------|
| `SUCCESS` | 1 | 操作成功 |
| `FAILURE` | 0 | 操作失败 |
| `TRUE` | 1 | 布尔真 |
| `FALSE` | 0 | 布尔假 |

**证据**: `src/adapter.rs:23-27`

## 内存管理

### 创建

以下函数创建的对象需要手动释放：

- `ylong_json_parse`
- `ylong_json_create_null`
- `ylong_json_create_bool`
- `ylong_json_create_double_number`
- `ylong_json_create_int_number`
- `ylong_json_create_string`
- `ylong_json_create_array`
- `ylong_json_create_object`
- `ylong_json_duplicate`
- `ylong_json_remove_array_item_by_index`
- `ylong_json_remove_object_item_by_index`

### 释放

- `ylong_json_delete(item)` - 释放 JSON 对象
- `ylong_json_free_string(string)` - 释放字符串

### 所有权转移

添加到数组/对象的元素所有权会转移：

```c
YlongJson* item = ylong_json_create_null();
ylong_json_add_item_to_array(array, item);
// 此时 item 的所有权已转移给 array
// 不应再调用 ylong_json_delete(item)
```

## 递归深度限制

解析器限制递归深度为 **128 层**：

```rust
pub(crate) const RECURSION_LIMIT: u32 = 128;
```

超过限制会返回 `Error::ExceedRecursionLimit`。

**证据**: `src/consts.rs:87`

## 线程安全

**ylong_json 不是线程安全的**：

- 不提供并发访问保护
- 调用方需要自行保证线程安全
- 不要在多线程间共享 YlongJson 指针

## 相关跳转

- [架构说明](01_Architecture.md) - 接口架构
- [安全风险](07_Security_Analysis.md) - 接口安全风险
- [内部 API](04_Internal_API.md) - Rust API
