# 附录：关键调用链

## 目的

本文档描述 `ylong_json` 的关键调用链，帮助理解代码执行流程。

## C FFI 调用链

### 1. 解析 JSON

```
ylong_json_parse (adapter.rs:40)
    ├── strlen (libc) - 计算字符串长度
    ├── slice_from_raw_parts - 创建字节切片
    ├── JsonValue::from_text (value.rs)
    │   └── start_parsing (states.rs:86)
    │       └── parse_value (states.rs:99)
    │           ├── parse_null
    │           ├── parse_bool
    │           ├── parse_number
    │           ├── parse_string
    │           ├── parse_array
    │           │   └── parse_value (递归)
    │           └── parse_object
    │               └── parse_value (递归)
    └── Box::into_raw - 转换为原始指针
```

### 2. 输出 JSON

```
ylong_json_print_unformatted (adapter.rs:74)
    ├── 空指针检查
    ├── 解引用 *mut JsonValue
    ├── JsonValue::compact_encode (value.rs)
    │   └── CompactEncoder::encode (encoder.rs)
    │       └── encode_value (递归)
    ├── CString::from_vec_unchecked
    └── into_raw - 返回 *mut c_char
```

### 3. 创建数组并添加元素

```
ylong_json_create_array (adapter.rs:424)
    ├── Array::new
    ├── JsonValue::Array
    └── Box::into_raw

ylong_json_add_item_to_array (adapter.rs:490)
    ├── 空指针检查
    ├── try_as_mut_array - 类型检查
    ├── Box::from_raw - 获取所有权
    ├── Array::push
    └── 所有权转移完成
```

### 4. 访问对象元素

```
ylong_json_get_object_item (adapter.rs:757)
    ├── 空指针检查
    ├── strlen - 计算 key 长度
    ├── slice_from_raw_parts
    ├── from_utf8_unchecked
    ├── try_as_mut_object - 类型检查
    ├── Object::get_mut
    └── 返回 *mut YlongJson
```

## Rust API 调用链

### 5. Serde 反序列化

```
from_str<T> (deserializer.rs:138)
    ├── Deserializer::new_from_slice
    │   └── SliceReader::new
    ├── T::deserialize
    │   └── Deserializer 实现
    │       └── parse_value (states.rs)
    │           └── 递归解析
    └── 转换为 T
```

### 6. Serde 序列化

```
to_string<T> (serializer_compact.rs:46)
    ├── AuxiliaryWriter::new
    ├── to_writer
    │   ├── Serializer::new
    │   └── T::serialize
    │       └── Serializer 实现
    │           └── 写入 JSON 文本
    └── String::from_utf8_unchecked
```

### 7. 索引访问

```
json["key"] (value/index.rs)
    ├── Index::index
    ├── try_as_object
    ├── Object::get
    └── 返回 &JsonValue（不存在返回 &JsonValue::Null）

json["key"] = value (value/index.rs)
    ├── IndexMut::index_mut
    ├── try_as_mut_object
    └── Object::insert 或 创建 Null 后修改
```

## 内部实现调用链

### 8. 状态机解析

```
start_parsing (states.rs:86)
    ├── check_recursion - 检查递归深度
    ├── parse_value
    │   ├── eat_whitespace_until_not - 跳过空白
    │   ├── 根据首字符分发:
    │   │   ├── 'n' -> parse_null
    │   │   ├── 't'/'f' -> parse_bool
    │   │   ├── '"' -> parse_string
    │   │   ├── '[' -> parse_array
    │   │   ├── '{' -> parse_object
    │   │   ├── '-'/'0'-'9' -> parse_number
    │   │   └── _ -> unexpected_character!
    │   └── 返回 JsonValue
    └── eat_whitespace_until_not - 检查尾部
```

### 9. 字符串编码

```
encode_string (encoder.rs)
    ├── 写入 '"'
    ├── 遍历字符:
    │   ├── 控制字符 -> 转义序列 (\\b, \\t, \\n, etc.)
    │   ├── '"' -> \\"
    │   ├── '\\' -> \\\\
    │   └── 其他 -> 直接写入
    └── 写入 '"'
```

## 内存管理调用链

### 10. 对象生命周期

```
创建:
ylong_json_create_*
    ├── JsonValue::new_*
    ├── Box::new
    └── Box::into_raw (返回 *mut YlongJson)

使用:
ylong_json_* 操作函数
    └── 解引用 *mut JsonValue

释放:
ylong_json_delete
    ├── Box::from_raw (获取所有权)
    └── Drop (自动释放内存)
```

## 错误处理调用链

### 11. 解析错误

```
parse_value 遇到错误
    ├── unexpected_character! 宏
    │   ├── 读取位置信息
    │   └── ParseError::UnexpectedCharacter
    ├── unexpected_eoj! 宏
    │   └── ParseError::UnexpectedEndOfJson
    └── Error::Parsing
        └── Display/Debug 格式化输出
```

## 相关跳转

- [对外 API](03_Public_API.md) - C FFI 接口
- [内部 API](04_Internal_API.md) - Rust API
- [架构说明](01_Architecture.md) - 架构设计
