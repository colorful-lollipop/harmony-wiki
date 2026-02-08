# ylong_json 项目概览

## 目的

本文档提供 `ylong_json` 项目的整体介绍，帮助新用户快速理解项目定位、核心能力和运行环境。

## 适用范围

- OpenHarmony 系统服务层开发者
- 需要使用 JSON 解析能力的 C/Rust 开发者
- 安全审计人员

## 项目定位

`ylong_json` 是 OpenHarmony 的**通用 JSON 语法解析库**，位于系统服务层，为上层应用和系统服务提供 JSON 序列化与反序列化能力。

```
┌─────────────────────────────────────────┐
│           Application Layer             │
│              (应用层)                    │
├─────────────────────────────────────────┤
│         System Service Layer            │
│            (系统服务层)                  │
│  ┌─────────────────────────────────┐    │
│  │         ylong_json              │    │
│  │    (JSON 序列化/反序列化)        │    │
│  └─────────────────────────────────┘    │
│              ↑                          │
│  ┌─────────────────────────────────┐    │
│  │            serde                │    │
│  │    (第三方序列化库)              │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

## 核心能力

### 1. JSON 解析与生成

支持完整的 JSON 语法（遵循 ECMA-404 标准）：

| JSON 类型 | 支持状态 | 说明 |
|-----------|----------|------|
| null | ✅ | 空值 |
| boolean | ✅ | true/false |
| number | ✅ | 整数、浮点数 |
| string | ✅ | Unicode 字符串 |
| array | ✅ | 数组 |
| object | ✅ | 键值对对象 |

### 2. 多底层数据结构

针对不同的使用场景，提供多种底层实现：

**Array 实现**:
- `Vec`（默认）: 适合子节点数多、查找频繁的场景
- `LinkedList`: 适合子节点数少（<15）、插入删除频繁的场景

**Object 实现**:
- `Btree`（默认）: 适合子节点数多（>1024）、查找频繁的场景
- `Vec`: 适合子节点数中等（15-1024）的场景
- `LinkedList`: 适合子节点数少（<15）的场景

### 3. 接口形式

| 接口类型 | 位置 | 说明 |
|----------|------|------|
| Rust API | `src/lib.rs` 导出 | 原生 Rust 接口 |
| C FFI | `src/adapter.rs` | C 语言兼容接口 |
| Serde | `src/deserializer.rs`, `src/serializer_compact.rs` | 第三方库集成 |

### 4. 性能表现

基于 `nativejson-benchmark` 的测试数据：

| 测试文件 | ylong_json (parse) | ylong_json (stringify) | cJSON (parse) | cJSON (stringify) |
|----------|-------------------|----------------------|---------------|------------------|
| canada.json | 200 MB/s | 90 MB/s | 55 MB/s | 11 MB/s |
| citm_catalog.json | 450 MB/s | 300 MB/s | 260 MB/s | 170 MB/s |
| twitter.json | 340 MB/s | 520 MB/s | 210 MB/s | 210 MB/s |

> 测试环境: Intel Xeon Gold 6278C @ 2.60GHz, 8核, 8GB 内存

## 运行环境

### 系统要求

- **适配系统类型**: standard（标准系统）
- **ROM**: ~200KB
- **RAM**: ~200KB

### 依赖组件

- `rust_libc`: Rust libc 绑定
- `serde`: 第三方序列化库（版本 1.0.136）

## 关键概念

### JsonValue

核心数据结构，表示任意 JSON 值：

```rust
pub enum JsonValue {
    Null,
    Boolean(bool),
    Number(Number),
    String(JsonString),
    Array(Array),
    Object(Object),
}
```

**证据**: `src/value.rs:65-83`

### Number

数值类型，支持三种内部表示：

```rust
pub enum Number {
    Unsigned(u64),
    Signed(i64),
    Float(f64),
}
```

**证据**: `src/value/number.rs:28-35`

### Feature Flags

编译时配置，影响底层数据结构和行为：

| Feature | 默认 | 说明 |
|---------|------|------|
| `btree_object` | ✅ | Object 使用 BtreeMap |
| `vec_array` | ✅ | Array 使用 Vec |
| `c_adapter` | ❌ | 启用 C FFI 接口 |
| `ascii_only` | ❌ | 仅 ASCII 字符输出 |

**证据**: `Cargo.toml:17-25`

## 项目边界

### 包含的功能

- JSON 文本解析为内存结构
- 内存结构序列化为 JSON 文本
- 支持格式化/紧凑两种输出格式
- 支持通过索引访问和修改 JSON 值
- C 语言 FFI 接口

### 不包含的功能

- 文件 I/O 操作（需调用方实现）
- 网络 I/O 操作
- JSON Schema 验证
- 流式解析（当前为全量解析）
- N-API 接口（仅提供 C FFI）

## 相关跳转

- [架构说明](01_Architecture.md) - 深入了解组件设计
- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [对外 API](03_Public_API.md) - C FFI 接口详情
