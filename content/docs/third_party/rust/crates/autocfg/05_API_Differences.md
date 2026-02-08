# 05 - API/接口差异

## 执行摘要

**结论**: autocfg 在 OpenHarmony 中**没有 API 差异**

| 指标 | 状态 |
|------|------|
| OH 新增 API | **无** |
| OH 修改的 API | **无** |
| 废弃/禁用的功能 | **无** |
| 行为变更 | **无** |

autocfg 完全使用上游代码，未做任何 API 级别的修改。

---

## API 一致性说明

### 与上游 API 对比

| API 类别 | 上游 (v1.4.0) | OpenHarmony | 差异 |
|---------|---------------|-------------|------|
| `AutoCfg` 结构体 | ✅ 存在 | ✅ 存在 | **无** |
| `emit()` 函数 | ✅ 存在 | ✅ 存在 | **无** |
| `probe_*` 方法 | ✅ 存在 | ✅ 存在 | **无** |
| `emit_*` 方法 | ✅ 存在 | ✅ 存在 | **无** |
| 错误类型 `Error` | ✅ 存在 | ✅ 存在 | **无** |

### 公共 API 列表

以下是 autocfg v1.4.0 的全部公共 API（OH 版本完全一致）：

#### 核心结构体

```rust
/// 主结构体
pub struct AutoCfg { /* ... */ }

impl AutoCfg {
    /// 构造函数
    pub fn new() -> Result<Self, Error>;
    pub fn with_dir<T: Into<PathBuf>>(dir: T) -> Result<Self, Error>;
    
    /// 配置方法
    pub fn no_std(&self) -> bool;
    pub fn set_no_std(&mut self, no_std: bool);
    
    /// 版本探测
    pub fn probe_rustc_version(&self, major: usize, minor: usize) -> bool;
    pub fn emit_rustc_version(&self, major: usize, minor: usize);
    
    /// 原始探测
    pub fn probe_raw(&self, code: &str) -> Result<(), Error>;
    
    /// sysroot crate 探测
    pub fn probe_sysroot_crate(&self, name: &str) -> bool;
    pub fn emit_sysroot_crate(&self, name: &str);
    
    /// 路径探测
    pub fn probe_path(&self, path: &str) -> bool;
    pub fn emit_has_path(&self, path: &str);
    pub fn emit_path_cfg(&self, path: &str, cfg: &str);
    
    /// trait 探测
    pub fn probe_trait(&self, name: &str) -> bool;
    pub fn emit_has_trait(&self, name: &str);
    pub fn emit_trait_cfg(&self, name: &str, cfg: &str);
    
    /// 类型探测
    pub fn probe_type(&self, name: &str) -> bool;
    pub fn emit_has_type(&self, name: &str);
    pub fn emit_type_cfg(&self, name: &str, cfg: &str);
    
    /// 表达式探测
    pub fn probe_expression(&self, expr: &str) -> bool;
    pub fn emit_expression_cfg(&self, expr: &str, cfg: &str);
    
    /// 常量表达式探测
    pub fn probe_constant(&self, expr: &str) -> bool;
    pub fn emit_constant_cfg(&self, expr: &str, cfg: &str);
}
```

#### 便利函数

```rust
/// 创建 AutoCfg 实例（panic 版本）
pub fn new() -> AutoCfg;

/// 输出 cfg 标志
pub fn emit(cfg: &str);

/// 输出重运行路径
pub fn rerun_path(path: &str);

/// 输出重运行环境变量
pub fn rerun_env(var: &str);

/// 输出 cfg 可能性（Rust 1.80+ checked cfgs）
pub fn emit_possibility(cfg: &str);
```

#### 错误类型

```rust
pub struct Error { /* ... */ }

impl Error {
    // 标准 Error trait 实现
}

impl fmt::Display for Error { ... }
impl error::Error for Error { ... }
```

---

## 为什么不需要 API 修改？

### 设计哲学

autocfg 的设计是**通用构建工具**，其 API 设计遵循以下原则：

1. **平台无关**: 所有 API 基于 Rust 编译器接口，与操作系统无关
2. **功能完备**: v1.0 版本已覆盖主要探测需求
3. **向后兼容**: 1.0+ 版本保证 API 稳定

### OH 需求分析

| OH 需求 | autocfg 支持情况 | 需要修改？ |
|--------|-----------------|-----------|
| 探测 Rust 版本 | ✅ `probe_rustc_version()` | 否 |
| 探测类型支持 | ✅ `probe_type()` / `emit_has_type()` | 否 |
| 探测 trait 支持 | ✅ `probe_trait()` / `emit_has_trait()` | 否 |
| 探测模块路径 | ✅ `probe_path()` / `emit_has_path()` | 否 |
| 探测表达式 | ✅ `probe_expression()` | 否 |
| 探测常量表达式 | ✅ `probe_constant()` | 否 |
| no_std 支持 | ✅ `set_no_std()` | 否 |
| 自定义探测 | ✅ `probe_raw()` | 否 |

**结论**: autocfg 的现有 API 完全满足 OpenHarmony 的需求，无需扩展或修改。

---

## 与上游的差异（如果有）

### 当前状态

截至文档生成时间（2025-02-08），OpenHarmony 中的 autocfg **与上游 v1.4.0 完全一致**。

### 验证方法

```bash
# 在 autocfg 目录执行
git diff HEAD
# 结果：无差异（空输出）

git log --oneline -1
# 结果：对应上游 1.4.0 标签的提交
```

### 文件对比

| 文件 | OH 版本 | 上游 v1.4.0 | 差异 |
|------|---------|-------------|------|
| `src/lib.rs` | 1.4.0 | 1.4.0 | **无** |
| `src/rustc.rs` | 1.4.0 | 1.4.0 | **无** |
| `src/version.rs` | 1.4.0 | 1.4.0 | **无** |
| `src/error.rs` | 1.4.0 | 1.4.0 | **无** |
| `Cargo.toml` | 1.4.0 | 1.4.0 | **无** |
| `BUILD.gn` | OH 特有 | N/A | **OH 构建适配**（非 API 差异）|

**注意**: `BUILD.gn` 是 OpenHarmony 特有的构建配置文件，不属于 API 差异。

---

## 使用示例对比

### 上游文档示例

```rust
// 来自上游 README.md
extern crate autocfg;

fn main() {
    let ac = autocfg::new();
    ac.emit_has_type("i128");
    autocfg::rerun_path("build.rs");
}
```

### OpenHarmony 中的使用

```rust
// memoffset 的 build.rs (在 OH 中)
extern crate autocfg;

fn main() {
    let ac = autocfg::new();
    ac.emit_has_path("core::mem::offset_of");
    ac.emit_has_type("core::mem::MaybeUninit");
}
```

**对比结论**：API 使用方式完全一致，只是探测的具体内容不同。

---

## 潜在的未来 API 差异

### 场景分析

虽然当前无差异，但考虑以下潜在场景：

#### 场景 1: OH 需要额外的探测功能

**假设**: OH 需要探测特定的编译器特性。

**处理建议**:
1. **优先推向上游**: 如果是通用需求，向上游提交 PR
2. **临时方案**: 使用 `probe_raw()` 进行自定义探测
3. **避免本地修改**: 不直接修改 autocfg 源码

示例：
```rust
// 使用 probe_raw 进行自定义探测
let ac = autocfg::new();
let custom_code = r#"
    #![feature(custom_feature)]
    pub fn test() {}
"#;
if ac.probe_raw(custom_code).is_ok() {
    autocfg::emit("has_custom_feature");
}
```

#### 场景 2: OH 需要禁用某些功能

**假设**: 出于安全考虑，需要禁用某些探测功能。

**评估**: 不建议禁用 autocfg 功能，因为：
- autocfg 是构建时工具，安全风险极低
- 禁用功能可能导致下游 crate 编译失败
- 更好的方案是在具体 crate 层面控制

---

## API 稳定性保证

### 上游承诺

autocfg 遵循严格的 **semver** 规范：

| 版本类型 | API 变更 | 适用场景 |
|---------|---------|---------|
| Patch (1.4.x) | 无破坏性变更 | Bug 修复 |
| Minor (1.x.0) | 向后兼容的添加 | 新功能 |
| Major (2.0.0) | 可能有破坏性变更 | 重大重构（极少发生）|

### OH 策略

| 场景 | 建议 |
|------|------|
| Patch 升级 | 可直接同步 |
| Minor 升级 | 需验证下游 crate 兼容性 |
| Major 升级 | 需全面评估影响 |

---

## 总结

### API 差异结论表

| 类别 | 状态 | 说明 |
|------|------|------|
| 新增 API | ❌ 无 | 无需新增 |
| 修改 API | ❌ 无 | 无需修改 |
| 废弃 API | ❌ 无 | 上游未废弃 |
| 行为变更 | ❌ 无 | 与上游一致 |
| 文档差异 | ❌ 无 | 使用方式一致 |

### 核心结论

autocfg 在 OpenHarmony 中**保持与上游 100% API 兼容性**，原因如下：

1. ✅ **功能完备**: 现有 API 覆盖所有探测需求
2. ✅ **设计通用**: API 设计平台无关，无需 OH 适配
3. ✅ **向后兼容**: 1.0+ 版本保证 API 稳定
4. ✅ **无运行时**: 构建时工具，不暴露给用户代码

**维护建议**: 保持零修改策略，直接跟随上游版本升级。

---

*文档生成时间: 2025-02-08*
