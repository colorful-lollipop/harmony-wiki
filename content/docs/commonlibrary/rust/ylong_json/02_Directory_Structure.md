# ylong_json 目录结构

## 目的

本文档详细描述 `ylong_json` 项目的目录结构和各模块职责。

## 适用范围

- 新加入的开发者
- 需要定位特定功能的开发者
- 代码审查人员

## 顶层目录

```
ylong_json/
├── benches/           # 性能测试（忽略）
├── docs/              # 说明文档
│   ├── user_guide.md      # 英文用户指南
│   └── user_guide_zh.md   # 中文用户指南
├── examples/          # 代码示例
│   ├── ylong_json_example.rs   # 基础示例
│   └── ylong_json_perf.rs      # 性能测试示例
├── figures/           # 架构图
├── src/               # 源代码（核心）
└── tests/             # 测试代码（忽略）

├── BUILD.gn           # GN 构建配置
├── Cargo.toml         # Rust 包配置
├── LICENSE            # Apache 2.0 许可证
├── OAT.xml            # 开源合规检查配置
├── README.md          # 英文 README
├── README_zh.md       # 中文 README
├── RELEASE_NOTE.md    # 发布说明
└── bundle.json        # OpenHarmony 组件配置
```

## src/ 目录详解

### 核心模块

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `lib.rs` | 85 | 库入口，模块组织，宏定义 | `array!`, `object!` |
| `value.rs` | 1600+ | JsonValue 枚举定义和实现 | `JsonValue`, `JsonString` |
| `error.rs` | 439 | 错误类型定义 | `Error`, `ParseError` |
| `consts.rs` | 213 | 常量和查找表 | `RECURSION_LIMIT`, `ESCAPE` |

### 序列化/反序列化

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `states.rs` | 1000+ | JsonValue 反序列化状态机 | `start_parsing`, `parse_value` |
| `encoder.rs` | 500+ | JsonValue 序列化实现 | `FormattedEncoder`, `CompactEncoder` |
| `deserializer.rs` | 800+ | Serde 反序列化实现 | `Deserializer`, `from_str` |
| `serializer_compact.rs` | 500+ | Serde 序列化实现 | `to_string`, `to_writer` |

### C FFI 接口

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `adapter.rs` | 3407 | C 语言接口实现 | `ylong_json_*` 系列函数 |

### 数据结构实现

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `linked_list.rs` | 600+ | 自定义 LinkedList 实现 | `LinkedList`, `Node` |
| `value/number.rs` | 400+ | Number 类型实现 | `Number` |
| `value/index.rs` | 500+ | 索引访问实现 | `Index` trait |

### 读取器

| 文件 | 行数 | 职责 | 关键符号 |
|------|------|------|----------|
| `reader/mod.rs` | 100+ | 读取器 trait 定义 | `BytesReader`, `Cacheable` |
| `reader/slice_reader.rs` | 200+ | 字节切片读取器 | `SliceReader` |
| `reader/io_reader.rs` | 200+ | IO 流读取器 | `IoReader` |

## value/ 子目录

### 条件编译结构

```
value/
├── array.rs           # Array 模块入口（条件编译选择实现）
├── array/
│   ├── vec.rs         # Vec 实现（feature = "vec_array"）
│   └── linked_list.rs # LinkedList 实现（feature = "list_array"）
├── object.rs          # Object 模块入口（条件编译选择实现）
├── object/
│   ├── btree.rs       # BTreeMap 实现（feature = "btree_object"）
│   ├── vec.rs         # Vec 实现（feature = "vec_object"）
│   └── linked_list.rs # LinkedList 实现（feature = "list_object"）
├── number.rs          # Number 枚举
└── index.rs           # 索引访问
```

### Array 实现选择逻辑

**`src/value/array.rs`**:
```rust
#[cfg(feature = "list_array")]
mod linked_list;
#[cfg(feature = "list_array")]
pub use linked_list::Array;

#[cfg(feature = "vec_array")]
mod vec;
#[cfg(feature = "vec_array")]
pub use vec::Array;
```

### Object 实现选择逻辑

**`src/value/object.rs`**:
```rust
#[cfg(feature = "btree_object")]
mod btree;
#[cfg(feature = "btree_object")]
pub use btree::Object;

#[cfg(feature = "list_object")]
mod linked_list;
#[cfg(feature = "list_object")]
pub use linked_list::Object;

#[cfg(feature = "vec_object")]
mod vec;
#[cfg(feature = "vec_object")]
pub use vec::Object;
```

## 模块职责详解

### lib.rs

**职责**: 库入口，组织模块，导出公共 API

**关键内容**:
- 定义 `array!` 和 `object!` 宏
- 条件编译导入 `adapter` 模块（c_adapter feature）
- 导出核心类型：`JsonValue`, `Array`, `Object`, `Number`, `Index`
- 导出 Serde 函数：`from_reader`, `from_slice`, `from_str`, `to_string`, `to_writer`

**证据**: `src/lib.rs:1-85`

### value.rs

**职责**: JsonValue 枚举定义和所有方法实现

**关键内容**:
- `JsonValue` 枚举（6 种 JSON 类型）
- `Display` 和 `Debug` trait 实现
- 类型判断方法：`is_null`, `is_boolean`, `is_number`, `is_string`, `is_array`, `is_object`
- 类型转换方法：`try_as_*` 系列
- 解析方法：`from_str`, `from_text`, `from_file`, `from_reader`
- 序列化方法：`compact_encode`, `formatted_encode`

**证据**: `src/value.rs:1-1600+`

### adapter.rs

**职责**: C FFI 接口实现

**关键内容**:
- 90+ 个 C 接口函数
- 类型创建函数：`ylong_json_create_*`
- 类型判断函数：`ylong_json_is_*`
- 值获取/设置函数：`ylong_json_get_*`, `ylong_json_set_*`
- 数组操作函数：`ylong_json_add_item_to_array`, `ylong_json_get_array_item`, 等
- 对象操作函数：`ylong_json_add_item_to_object`, `ylong_json_get_object_item`, 等
- 内存管理函数：`ylong_json_parse`, `ylong_json_delete`, `ylong_json_free_string`

**证据**: `src/adapter.rs:1-3407`

### states.rs

**职责**: JSON 解析状态机实现

**关键内容**:
- `start_parsing` 函数：解析入口
- `parse_value` 函数：递归解析 JSON 值
- `parse_object` 函数：解析对象
- `parse_array` 函数：解析数组
- `parse_string` 函数：解析字符串
- `parse_number` 函数：解析数字
- 宏定义：`unexpected_character!`, `eat_whitespace_until_not!`, 等

**证据**: `src/states.rs:1-1000+`

### encoder.rs

**职责**: JsonValue 序列化实现

**关键内容**:
- `FormattedEncoder`: 格式化输出（带缩进）
- `CompactEncoder`: 紧凑输出
- `encode_value` 函数：递归编码
- 字符串转义处理

**证据**: `src/encoder.rs:1-500+`

### error.rs

**职责**: 错误类型定义

**关键内容**:
- `Error` 枚举：解析错误、IO 错误、类型转换错误等
- `ParseError` 枚举：具体解析错误（意外字符、非法 UTF-8、缺少冒号等）
- 错误格式化实现

**证据**: `src/error.rs:1-439`

### consts.rs

**职责**: 常量和查找表定义

**关键内容**:
- JSON 语法字符常量：`COLON`, `COMMA`, `LEFT_CURLY_BRACKET`, 等
- 转义字符映射：`BS_UNICODE`, `HT_UNICODE`, 等
- 递归深度限制：`RECURSION_LIMIT = 128`
- 查找表：`ESCAPE`（256项布尔表），`PRINT_MAP`（输出映射表）

**证据**: `src/consts.rs:1-213`

## 文件大小统计

| 文件 | 行数 | 大小 |
|------|------|------|
| adapter.rs | 3407 | ~118KB |
| value.rs | ~1600 | ~60KB |
| states.rs | ~1000 | ~42KB |
| deserializer.rs | ~800 | ~34KB |
| encoder.rs | ~500 | ~16KB |
| serializer_compact.rs | ~500 | ~22KB |
| linked_list.rs | ~600 | ~26KB |
| error.rs | 439 | ~13KB |
| consts.rs | 213 | ~9KB |
| lib.rs | 85 | ~2KB |

## 代码组织原则

1. **单一职责**: 每个文件负责一个明确的功能领域
2. **条件编译**: 通过 feature flags 选择不同实现，避免运行时开销
3. **模块化**: 核心类型、序列化、反序列化、FFI 分离
4. **可测试性**: 单元测试内联在源文件中（`#[cfg(test)]` 模块）

## 相关跳转

- [架构说明](01_Architecture.md) - 整体架构设计
- [对外 API](03_Public_API.md) - C FFI 接口详情
- [内部 API](04_Internal_API.md) - Rust API 详情
