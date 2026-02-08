# ylong_json 编译产物

## 目的

本文档描述 `ylong_json` 的编译输出产物、安装路径和运行时加载关系。

## 适用范围

- 系统集成工程师
- 构建系统开发者
- 发布工程师

## 产物清单

### 1. 动态库 (.so)

| 产物名 | 类型 | 说明 |
|--------|------|------|
| `libylong_json.so` | 动态库 | 核心 JSON 解析库 |

**证据**: `BUILD.gn:17-37`

### 2. 测试可执行文件

| 产物名 | 类型 | 说明 |
|--------|------|------|
| `rust_ylong_json_unit_test` | 可执行文件 | 单元测试 |

**证据**: `BUILD.gn:39-62`

## 预计输出路径

### 标准构建输出

```
out/
└── [target_arch]/
    └── commonlibrary/
        └── rust/
            └── ylong_json/
                ├── libylong_json.so          # 动态库
                └── rust_ylong_json_unit_test  # 单元测试（仅 testonly）
```

### 安装路径

根据 OpenHarmony 系统组件安装规范：

```
/system/lib/                    # 系统库目录
    └── libylong_json.so        # 运行时动态库
```

## 运行时加载关系

### 依赖链

```
应用程序/系统服务
    ↓ 依赖
libylong_json.so
    ↓ 依赖
libserde.so (third_party/rust/crates/serde)
    ↓ 依赖
libstd.so (Rust 标准库)
    ↓ 依赖
libc.so (系统 C 库)
```

### 加载顺序

1. 应用程序启动
2. 动态链接器加载 `libylong_json.so`
3. 解析并加载依赖库（`libserde.so`, `libstd.so`, `libc.so`）
4. 执行初始化代码
5. 调用 ylong_json 功能

## 产物特性

### 动态库特性

| 特性 | 值 | 说明 |
|------|-----|------|
| 类型 | dylib | Rust 动态库 |
| crate 名称 | ylong_json | Rust crate 名 |
| 导出符号 | Rust 符号 | 包含 Rust 元数据 |
| C 接口 | 未导出 | 未启用 c_adapter feature |

### 文件大小估算

| 产物 | 估算大小 | 说明 |
|------|----------|------|
| `libylong_json.so` | ~200KB | 与 bundle.json 声明一致 |
| `rust_ylong_json_unit_test` | ~500KB | 包含测试代码 |

## 使用方式

### Rust 代码中使用

```rust
// 在 Cargo.toml 中添加依赖
[dependencies]
ylong_json = { path = "//commonlibrary/rust/ylong_json" }

// 或在 GN 构建中添加
external_deps = [ "ylong_json:lib" ]
```

### 运行时链接

动态库需要在运行时可用：

```bash
# 确保库在系统库路径中
export LD_LIBRARY_PATH=/system/lib:$LD_LIBRARY_PATH
```

## 构建配置影响

### Feature Flags 对产物的影响

| Feature | 对产物的影响 |
|---------|-------------|
| `c_adapter` | 添加 C FFI 符号，增加代码大小 |
| `btree_object` | 包含 BTreeMap 实现 |
| `vec_array` | 包含 Vec 实现 |
| `ascii_only` | 简化字符串处理，减小代码大小 |
| `list_object` | 添加 LinkedList Object 实现 |
| `list_array` | 添加 LinkedList Array 实现 |

### 当前配置 (BUILD.gn)

启用的 features:
- `default`
- `vec_array`
- `btree_object`
- `ascii_only`

未启用的 features:
- `c_adapter` - 无 C 接口
- `list_object` - 无 LinkedList Object
- `list_array` - 无 LinkedList Array
- `vec_object` - 无 Vec Object

## 发布说明

### 发布内容

根据 `bundle.json` 配置：

- **发布方式**: `code-segment`（代码片段）
- **目标路径**: `commonlibrary/rust/ylong_json`
- **版本**: 4.0

### 兼容性

- **适配系统**: standard（标准系统）
- **架构**: 所有 Rust 支持的架构
- **API 稳定性**: 内部组件，API 可能变化

## 相关跳转

- [GN Targets](05_GN_Targets.md) - 构建配置详情
- [配置选项](appendix/Config_Flags.md) - Feature flags
- [对外 API](03_Public_API.md) - API 说明
