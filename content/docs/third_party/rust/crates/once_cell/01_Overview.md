# 原始库简介

## 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | once_cell |
| **上游版本** | 1.17.0 |
| **OH 版本** | 6.1 (组件版本) |
| **许可证** | Apache License 2.0 OR MIT |
| **许可证文件** | LICENSE-APACHE, LICENSE-MIT |
| **编程语言** | Rust |
| **Rust Edition** | 2021 |
| **MSRV** | 1.56.0 |
| **上游地址** | https://github.com/matklad/once_cell |
| **作者** | Aleksey Kladov <aleksey.kladov@gmail.com> |
| **OH 负责人** | fangting12@huawei.com |

## 核心功能一句话描述

once_cell 提供**单次赋值** 的线程安全单元格 (cell)，以及基于此实现的延迟初始化 (lazy initialization) 类型，用于 Rust 中优雅地管理全局状态和延迟计算。

## 原始功能概述

### 主要类型

once_cell 提供以下核心类型：

1. **`unsync::OnceCell<T>`** - 非线程安全的单次赋值单元格
   - 类似 `RefCell`，但只能赋值一次
   - 赋值后可直接获取 `&T` 引用，无需运行时借用检查

2. **`sync::OnceCell<T>`** - 线程安全的单次赋值单元格
   - 类似 `std::sync::Once`，但可以存储任意类型 `T`
   - 使用内部原子操作保证线程安全

3. **`sync::Lazy<T, F>`** - 延迟初始化包装器
   - 首次访问时通过函数 `F` 初始化值
   - 提供类似 `lazy_static!` 宏的功能，但无需宏
   - 支持 `Deref` trait，可像普通引用一样使用

### 典型使用场景

```rust
// 1. 全局静态配置
use once_cell::sync::Lazy;

static CONFIG: Lazy<Config> = Lazy::new(|| {
    Config::load_from_file("config.toml")
});

// 2. 缓存计算结果
use once_cell::sync::OnceCell;

let expensive_result: OnceCell<Result<DataType>> = OnceCell::new();

fn compute_expensive() -> &'static Result<DataType> {
    expensive_result.get_or_init(|| {
        // 只在首次访问时计算
        do_expensive_computation()
    })
}
```

## 在 OpenHarmony 中的作用和定位

### 系统定位

once_cell 在 OpenHarmony 中的定位是：**基础设施级别的 Rust 工具库**

**关键特征**:
- 属于 `thirdparty` 子系统
- 提供 `rust_once_cell` 组件
- 作为底层依赖库被其他 Rust crate 间接使用

### 在 OH 中的价值

1. **统一全局状态管理**:
   - 为 OH 的 Rust 模块提供标准的全局静态变量管理方案
   - 替代不安全的 `static mut` 或复杂的 `lazy_static!` 宏

2. **延迟初始化优化**:
   - 减少启动时间：只在首次访问时初始化资源
   - 节省内存：未使用的全局变量不占用资源

3. **跨模块共享**:
   - 支持在编译单元 (compilation unit) 之间共享只读数据
   - 为测试框架和工具库提供全局状态管理

### 为什么 OH 选择 once_cell

| 原因 | 说明 |
|------|------|
| **上游活跃** | API 正被提议纳入 Rust 标准库 (RFC 2788)，设计稳定 |
| **性能优异** | 使用零成本抽象，无运行时开销 |
| **功能完备** | 提供同步/异步、线程安全/非线程安全等多种变体 |
| **零配置** | 代码纯 Rust 实现，无需平台特定代码 |
| **易于维护** | 无需修改源代码，仅构建系统适配 |

### 与其他全局状态方案对比

| 方案 | 优点 | 缺点 | OH 使用情况 |
|------|------|------|-----------|
| **once_cell** | ✅ 类型安全<br>✅ 无宏<br>✅ 标准化 | ❌ 需要学习 | ✅ **广泛使用** |
| **lazy_static!** | ✅ 熟悉 | ❌ 需要宏<br>❌ 不支持 `const fn` | ❌ 未使用 |
| **std::sync::Once** | ✅ 标准库 | ❌ 只能执行代码，不能存储值 | ❌ 未使用 |
| **static mut** | ✅ 简单 | ❌ 不安全<br>❌ 需要手动同步 | ❌ 未使用 |

## 版本历史

### 上游版本

| 版本 | 发布日期 | 主要变更 |
|------|----------|---------|
| **1.17.0** | 2023-xx | 新增 `race::OnceRef` 用于存储 `&'a T` |
| **1.16.0** | 2023-xx | 基于 `critical-section` 添加 `no_std` 实现 |
| **1.15.0** | 2023-xx | MSRV 提升至 Rust 1.56.0 |
| **1.14.0** | 2023-xx | 新增 `Lazy::force_mut` 和 `get_mut` |
| **1.13.1** | 2022-xx | 修复 strict provenance 合规性问题 |
| **1.0.0** | 2020-xx | 第一个稳定版本 |

### OpenHarmony 版本

| 组件版本 | 上游版本 | 集成时间 | 备注 |
|---------|----------|----------|------|
| **6.1** | 1.17.0 | 2023-04 | 当前使用版本 |

## 相关链接

- **上游文档**: https://docs.rs/once_cell
- **API 参考**: https://docs.rs/once_cell/latest/once_cell/
- **RFC 2788**: https://github.com/rust-lang/rfcs/pull/2788 (提议纳入 std)
- **OpenHarmony Issue**: https://gitee.com/openharmony/build/issues/I6UFTP

## 注意事项

⚠️ **重要**: once_cell 的 API 正在被提议纳入 Rust 标准库。未来如果被采纳，OH 可能需要迁移到标准库实现 (`std::sync::OnceCell` 和 `std::sync::Lazy`)。

当前的 `once_cell` 库与未来标准库 API 的兼容性：
- API 设计基本一致
- 迁移路径预计很简单（主要是替换导入路径）
- 具体迁移计划待标准库 RFC 合并后确定
