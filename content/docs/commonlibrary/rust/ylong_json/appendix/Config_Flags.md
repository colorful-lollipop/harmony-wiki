# 附录：配置选项

## 目的

本文档详细描述 `ylong_json` 的所有配置选项（Feature Flags）。

## Feature Flags 列表

### 数据结构 Features

#### `btree_object`（默认启用）

**说明**: Object 使用 `BTreeMap<String, JsonValue>` 实现

**适用场景**:
- Object 子节点数较多（>1024）
- 查找操作频繁
- 需要按键排序

**性能特点**:
- 查找: O(log n)
- 插入: O(log n)
- 内存: 较高

**证据**: `src/value/object/btree.rs`

---

#### `vec_object`

**说明**: Object 使用 `Vec<(String, JsonValue)`> 实现

**适用场景**:
- Object 子节点数中等（15-1024）
- 查找较少
- 内存敏感

**性能特点**:
- 查找: O(n)
- 插入: 平摊 O(1)
- 内存: 中等

**证据**: `src/value/object/vec.rs`

---

#### `list_object`

**说明**: Object 使用 `LinkedList<(String, JsonValue)`> 实现

**适用场景**:
- Object 子节点数较少（<15）
- 插入删除频繁
- 不需要随机访问

**性能特点**:
- 查找: O(n)
- 插入: O(1)（已知位置）
- 内存: 较低

**额外 API**: 启用后提供节点操作接口

**证据**: `src/value/object/linked_list.rs`

---

#### `vec_array`（默认启用）

**说明**: Array 使用 `Vec<JsonValue>` 实现

**适用场景**:
- Array 子节点数较多（>15）
- 随机访问频繁
- 需要索引操作

**性能特点**:
- 索引: O(1)
- 尾部插入: 平摊 O(1)
- 中间插入: O(n)

**证据**: `src/value/array/vec.rs`

---

#### `list_array`

**说明**: Array 使用 `LinkedList<JsonValue>` 实现

**适用场景**:
- Array 子节点数较少（<15）
- 插入删除频繁
- 不需要随机访问

**性能特点**:
- 索引: O(n)
- 已知位置插入: O(1)
- 内存: 较低

**额外 API**: 启用后提供节点操作接口

**证据**: `src/value/array/linked_list.rs`

---

### 功能 Features

#### `c_adapter`

**说明**: 启用 C FFI 接口

**影响**:
- 编译 `src/adapter.rs`
- 导出所有 `ylong_json_*` 函数
- 字符串内部使用 `CString` 而非 `String`

**依赖**: `libc` crate

**证据**: `src/lib.rs:68-71`

---

#### `ascii_only`

**说明**: 仅使用 ASCII 字符输出

**影响**:
- 正常解析 Unicode 字符
- 超出 ASCII 的 UTF-8 字符在输出时保持不变（不转义）
- 减小二进制大小
- 提高输出性能

**注意**: 与标准 JSON 行为略有不同

**证据**: `src/consts.rs:159-157` (条件编译)

---

#### `default`

**说明**: 默认 feature 集合

**包含**:
```toml
default = ["btree_object", "vec_array"]
```

---

## 配置组合建议

### 高性能查找场景

```toml
[dependencies]
ylong_json = { features = ["btree_object", "vec_array"] }
```

### 低内存占用场景

```toml
[dependencies]
ylong_json = { features = ["list_object", "list_array", "ascii_only"] }
```

### C 接口场景

```toml
[dependencies]
ylong_json = { features = ["c_adapter", "btree_object", "vec_array"] }
```

### 纯 Rust 场景（最小化）

```toml
[dependencies]
ylong_json = { default-features = false, features = ["vec_array"] }
```

## GN 构建配置

### BUILD.gn 默认配置

```gn
features = [
  "default",
  "vec_array",
  "btree_object",
  "ascii_only",
]
```

**注意**: GN 配置额外启用了 `ascii_only`，而 Cargo.toml 默认不启用。

### 自定义配置

如需修改 feature 配置，编辑 `BUILD.gn`:

```gn
ohos_rust_shared_library("lib") {
  # ...
  features = [
    "c_adapter",      # 启用 C 接口
    "vec_array",      # Array 使用 Vec
    "btree_object",   # Object 使用 BTree
    # "ascii_only",   # 禁用 ascii_only
  ]
}
```

## 性能对比

### Object 实现对比

| 操作 | BTree | Vec | LinkedList |
|------|-------|-----|------------|
| 查找 | O(log n) | O(n) | O(n) |
| 插入 | O(log n) | O(1)* | O(1) |
| 删除 | O(log n) | O(n) | O(1) |
| 内存 | 高 | 中 | 低 |

*Vec 尾部插入为 O(1)，中间插入为 O(n)

### Array 实现对比

| 操作 | Vec | LinkedList |
|------|-----|------------|
| 索引 | O(1) | O(n) |
| 尾部插入 | O(1)* | O(1) |
| 头部插入 | O(n) | O(1) |
| 中间插入 | O(n) | O(n)** |
| 内存 | 中 | 低 |

*平摊 O(1)
**已知位置时为 O(1)

## 相关跳转

- [GN Targets](05_GN_Targets.md) - 构建配置
- [编译产物](06_Build_Artifacts.md) - 输出文件
- [架构说明](01_Architecture.md) - 架构设计
