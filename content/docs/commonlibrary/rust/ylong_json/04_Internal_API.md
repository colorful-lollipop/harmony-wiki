# ylong_json 内部 API (Rust)

## 目的

本文档描述 `ylong_json` 提供的 Rust 内部 API，供 Rust 开发者使用。

## 适用范围

- Rust 开发者
- 系统服务层开发者
- 需要深入理解实现的开发者

## 导出 API

`ylong_json` 通过 `src/lib.rs` 导出以下公共 API：

```rust
pub use error::{Error, ParseError};
pub use value::{Array, Index, JsonValue, Number, Object};
pub use deserializer::{from_reader, from_slice, from_str};
pub use serializer_compact::{to_string, to_writer};

#[cfg(feature = "c_adapter")]
pub use adapter::*;

#[cfg(any(feature = "list_array", feature = "list_object"))]
pub use linked_list::{Iter, IterMut, Node};
```

**证据**: `src/lib.rs:62-84`

## 核心类型

### JsonValue

**定义**: `src/value.rs:65-83`

```rust
#[derive(Clone)]
pub enum JsonValue {
    Null,
    Boolean(bool),
    Number(Number),
    String(JsonString),
    Array(Array),
    Object(Object),
}
```

**说明**:
- `JsonString` 根据 `c_adapter` feature 选择 `CString` 或 `String`
- 实现了 `Clone`, `Display`, `Debug` trait

### Number

**定义**: `src/value/number.rs:28-35`

```rust
#[derive(Clone)]
pub enum Number {
    Unsigned(u64),
    Signed(i64),
    Float(f64),
}
```

**方法**:
- `is_unsigned()` - 是否为无符号整数
- `is_signed()` - 是否为有符号整数
- `is_float()` - 是否为浮点数
- `try_as_u64()` - 尝试转换为 u64
- `try_as_i64()` - 尝试转换为 i64
- `try_as_f64()` - 尝试转换为 f64

### Array

**定义**: 根据 feature 选择不同实现

- `vec_array` (默认): `Vec<JsonValue>`
- `list_array`: `LinkedList<JsonValue>`

**公共方法**:
- `new()` - 创建空数组
- `push(value)` - 添加元素
- `pop()` - 移除并返回最后一个元素
- `remove(index)` - 移除指定索引元素
- `get(index)` - 获取指定索引元素引用
- `get_mut(index)` - 获取指定索引元素可变引用
- `len()` - 获取长度
- `is_empty()` - 是否为空

### Object

**定义**: 根据 feature 选择不同实现

- `btree_object` (默认): `BTreeMap<String, JsonValue>`
- `vec_object`: `Vec<(String, JsonValue)>`
- `list_object`: `LinkedList<(String, JsonValue)>`

**公共方法**:
- `new()` - 创建空对象
- `insert(key, value)` - 插入键值对
- `remove(key)` - 移除键值对
- `get(key)` - 获取值引用
- `get_mut(key)` - 获取值可变引用
- `contains_key(key)` - 检查键是否存在
- `len()` - 获取键值对数量
- `is_empty()` - 是否为空
- `iter()` / `iter_mut()` - 迭代器

## JsonValue 方法

### 构造方法

```rust
impl JsonValue {
    pub fn new_null() -> Self;
    pub fn new_boolean(boolean: bool) -> Self;
    pub fn new_number(number: Number) -> Self;
    pub fn new_string(str: &str) -> Self;
    pub fn new_array(array: Array) -> Self;
    pub fn new_object(object: Object) -> Self;
}
```

**证据**: `src/value.rs:115-200`

### 解析方法

```rust
impl JsonValue {
    pub fn from_text<T: AsRef<[u8]>>(text: T) -> Result<Self, Error>;
    pub fn from_file<P: AsRef<Path>>(path: P) -> Result<Self, Error>;
    pub fn from_reader<R: Read>(reader: &mut R) -> Result<Self, Error>;
}

impl FromStr for JsonValue {
    type Err = Error;
    fn from_str(s: &str) -> Result<Self, Error>;
}
```

**证据**: `src/value.rs:400-500`

### 类型判断方法

```rust
impl JsonValue {
    pub fn is_null(&self) -> bool;
    pub fn is_boolean(&self) -> bool;
    pub fn is_number(&self) -> bool;
    pub fn is_string(&self) -> bool;
    pub fn is_array(&self) -> bool;
    pub fn is_object(&self) -> bool;
}
```

**证据**: `src/value.rs:250-300`

### 类型转换方法

```rust
impl JsonValue {
    pub fn try_as_boolean(&self) -> Result<&bool, Error>;
    pub fn try_as_number(&self) -> Result<&Number, Error>;
    pub fn try_as_string(&self) -> Result<&JsonString, Error>;
    pub fn try_as_array(&self) -> Result<&Array, Error>;
    pub fn try_as_object(&self) -> Result<&Object, Error>;
    
    pub fn try_as_mut_boolean(&mut self) -> Result<&mut bool, Error>;
    pub fn try_as_mut_number(&mut self) -> Result<&mut Number, Error>;
    pub fn try_as_mut_string(&mut self) -> Result<&mut JsonString, Error>;
    pub fn try_as_mut_array(&mut self) -> Result<&mut Array, Error>;
    pub fn try_as_mut_object(&mut self) -> Result<&mut Object, Error>;
}
```

**证据**: `src/value.rs:300-400`

### 序列化方法

```rust
impl JsonValue {
    pub fn compact_encode<W: Write>(&self, writer: &mut W) -> Result<(), Error>;
    pub fn formatted_encode<W: Write>(&self, writer: &mut W) -> Result<(), Error>;
}
```

**证据**: `src/value.rs:500-550`

## 宏

### array! 宏

**定义**: `src/lib.rs:25-37`

```rust
#[macro_export]
macro_rules! array {
    () => ({
        Array::new()
    });
    ($($x:expr),+ $(,)?) => ({
        let mut array = Array::new();
        $(
            array.push($x.into());
        )*
        array
    });
}
```

**用法**:
```rust
use ylong_json::{array, Array, JsonValue};

let empty: Array = array![];
let arr = array![1, 2, 3];
let json_arr = JsonValue::new_array(array!["a", "b", "c"]);
```

### object! 宏

**定义**: `src/lib.rs:40-52`

```rust
#[macro_export]
macro_rules! object {
    () => ({
        Object::new()
    });
    ($($k: expr => $v: expr);+ $(;)?) => ({
        let mut object = Object::new();
        $(
           object.insert(String::from($k), $v.into());
        )*
        object
    });
}
```

**用法**:
```rust
use ylong_json::{object, Object, JsonValue};

let empty: Object = object![];
let obj = object! {
    "name" => "test";
    "value" => 123
};
```

## Serde 集成

### 反序列化

```rust
pub fn from_reader<R, T>(reader: R) -> Result<T, Error>
where
    R: Read,
    T: DeserializeOwned;

pub fn from_slice<'a, T>(slice: &'a [u8]) -> Result<T, Error>
where
    T: Deserialize<'a>;

pub fn from_str<T>(s: &str) -> Result<T, Error>
where
    T: DeserializeOwned;
```

**证据**: `src/deserializer.rs:104-150`

**用法**:
```rust
use serde::Deserialize;
use ylong_json::deserializer::from_str;

#[derive(Deserialize)]
struct Person {
    name: String,
    age: u32,
}

let json = r#"{"name":"Alice","age":30}"#;
let person: Person = from_str(json).unwrap();
```

### 序列化

```rust
pub fn to_string<T>(value: &T) -> Result<String, Error>
where
    T: Serialize;

pub fn to_writer<T, W>(value: &T, writer: &mut W) -> Result<(), Error>
where
    T: Serialize,
    W: Write;
```

**证据**: `src/serializer_compact.rs:46-68`

**用法**:
```rust
use serde::Serialize;
use ylong_json::serializer_compact::to_string;

#[derive(Serialize)]
struct Person {
    name: String,
    age: u32,
}

let person = Person { name: "Alice".to_string(), age: 30 };
let json = to_string(&person).unwrap();
```

## 索引访问

实现了 `Index` trait，支持通过索引访问 JsonValue：

```rust
impl Index<&str> for JsonValue {
    type Output = JsonValue;
    fn index(&self, index: &str) -> &Self::Output;
}

impl Index<usize> for JsonValue {
    type Output = JsonValue;
    fn index(&self, index: usize) -> &Self::Output;
}

impl IndexMut<&str> for JsonValue {
    fn index_mut(&mut self, index: &str) -> &mut Self::Output;
}

impl IndexMut<usize> for JsonValue {
    fn index_mut(&mut self, index: usize) -> &mut Self::Output;
}
```

**用法**:
```rust
use ylong_json::JsonValue;

let json = JsonValue::from_text(r#"{"name":"Alice","items":[1,2,3]}"#).unwrap();

// 访问对象字段
let name = &json["name"];

// 访问数组元素
let first = &json["items"][0];

// 修改值
let mut json_mut = json.clone();
json_mut["name"] = "Bob".into();
json_mut["items"][0] = 10.into();
```

**证据**: `src/value/index.rs`

## 错误类型

### Error

**定义**: `src/error.rs:19-46`

```rust
pub enum Error {
    Parsing(ParseError),
    Io(std::io::Error),
    ParseNumber,
    Utf8Transform,
    TypeTransform,
    Reader(Box<dyn std::error::Error>),
    IncorrectSerdeUsage,
    Custom(String),
    ExceedRecursionLimit,
}
```

### ParseError

**定义**: `src/error.rs:49-73`

```rust
pub enum ParseError {
    UnexpectedCharacter(usize, usize, char),
    InvalidUtf8Bytes(usize),
    UnexpectedEndOfJson(usize),
    TrailingBytes(usize),
    ParsingUnfinished,
    TrailingComma(usize, usize),
    MissingColon(usize, usize),
    MissingComma(usize, usize),
}
```

## 类型转换

实现了多种 From trait，支持便捷的类型转换：

```rust
impl From<bool> for JsonValue;
impl From<Number> for JsonValue;
impl From<u64> for Number;
impl From<i64> for Number;
impl From<f64> for Number;
impl From<&str> for JsonValue;
impl From<String> for JsonValue;
impl From<Array> for JsonValue;
impl From<Object> for JsonValue;
```

**用法**:
```rust
use ylong_json::{JsonValue, Number};

let bool_val: JsonValue = true.into();
let num_val: JsonValue = 42i32.into();
let float_val: JsonValue = 3.14f64.into();
let str_val: JsonValue = "hello".into();
```

## 相关跳转

- [对外 API](03_Public_API.md) - C FFI 接口
- [架构说明](01_Architecture.md) - 架构设计
- [安全风险](07_Security_Analysis.md) - 安全分析
