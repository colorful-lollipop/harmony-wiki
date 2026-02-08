# Patch 详细分析

## 核心结论

**clang-sys 在 OpenHarmony 中未应用任何 Patch。**

这是一个**零 Patch、原生集成**的第三方库。

## Patch 文件清单

### 搜索结果

```bash
# 搜索命令
find /Volumes/lexar/code/d/work/oh/third_party/rust/crates/clang-sys -name "*.patch" -o -name "patches" -type d

# 结果
无 Patch 文件
```

### 清单表

| Patch 文件 | 修改文件 | 修改函数 | 修改目的 | 关联的 OH 需求 |
|-----------|---------|---------|---------|--------------|
| 无 | - | - | - | - |

## 无 Patch 原因深度分析

### 1. 库的本质特性

#### FFI 绑定库的定位
clang-sys 是一个**纯粹的 FFI 绑定库**，其唯一职责是将 libclang 的 C API 映射为 Rust 可调用的函数。这种定位决定了：

- **功能单一**: 只做一件事（绑定），不做业务逻辑
- **接口稳定**: FFI 绑定与 C API 一一对应，不引入新逻辑
- **无平台相关代码**: 跨平台逻辑由 libclang 处理，绑定层本身无平台差异

```
libclang C API ──clang-sys──> Rust FFI 绑定
     ↑                          ↓
   Clang 编译器            Rust 代码
```

### 2. 良好的跨平台支持

#### 上游已支持的平台
- Linux
- macOS  
- Windows (MSVC/MinGW)
- FreeBSD
- illumos
- Haiku

#### 平台适配策略
上游代码通过条件编译（`#[cfg(target_os = ...)]`）已经支持多种平台，OpenHarmony 基于 Linux，天然兼容。

### 3. 高度可配置性

#### Cargo Features 机制
上游通过 features 提供了灵活的配置能力：

```toml
[features]
clang_3_5 = []
clang_3_6 = ["clang_3_5"]
...
clang_16_0 = ["clang_15_0"]
runtime = ["libloading"]
static = []
```

OH 通过 BUILD.gn 启用适当的 features 即可满足需求，无需修改源码。

### 4. 无 OH 特有业务逻辑

clang-sys 是纯技术基础设施：
- 不处理 OH 特有的文件格式
- 不涉及 OH 特有的系统调用
- 不实现 OH 特有的功能需求

## OH 构建配置 vs Patch

虽然无 Patch 文件，但 OH 通过 **BUILD.gn 配置**实现了适配：

### 配置对比

| 方面 | 上游默认 | OH BUILD.gn | 说明 |
|-----|---------|-------------|-----|
| **Features** | 仅 `clang_3_5` | `clang_3_5` 到 `clang_6_0` + `static` + `libloading` | 启用更多版本支持 |
| **链接方式** | 动态链接 | 静态链接（`static` feature） | OH 构建策略 |
| **runtime** | 可选 | 未启用 | 不使用运行时加载 |

### 配置即适配

```gn
# BUILD.gn 节选
features = [
    "clang_3_5",
    "clang_3_6",
    "clang_3_7",
    "clang_3_8",
    "clang_3_9",
    "clang_4_0",
    "clang_5_0",
    "clang_6_0",
    "libloading",
    "static",
]
```

这种配置适配是**声明式的**，比 Patch 更优雅：
- 易于理解：一看就知道启用了哪些特性
- 易于维护：修改配置即可，无需处理代码冲突
- 易于升级：上游更新后配置通常无需变更

## 与其他库的对比

| 库 | Patch 数量 | 原因 |
|---|-----------|-----|
| **clang-sys** | 0 | 纯 FFI 绑定，无业务逻辑，配置可适配 |
| curl | 多 | 需适配 OH 网络栈 |
| openssl | 多 | 需适配 OH 加密需求 |
| libxml2 | 中 | 需适配 OH 文件系统 |

## 升级建议

### 升级路径

由于无 Patch，升级流程极为简单：

1. **替换源码**: 直接替换为新版本上游代码
2. **检查 BUILD.gn**: 确认 features 是否需要调整
3. **验证构建**: 执行构建测试
4. **运行测试**: 验证 bindgen 功能正常

### 注意事项

| 检查项 | 说明 |
|-------|-----|
| **API 变更** | 检查新版本是否有破坏性 API 变更 |
| **MSRV 变更** | 检查 Rust 最低版本要求是否提高 |
| **依赖变更** | 检查 dependencies 是否有新增或版本变更 |
| **Features 变更** | 检查 features 是否有新增或重命名 |

### 可推向上游的改进

当前无 Patch，因此无可推向上游的改进。

如需未来扩展，可考虑：
- 向上游贡献 OH 平台的 CI 测试
- 参与上游的 Clang 新版本支持开发

## 维护者清单

### 日常维护

- [ ] 关注上游安全公告
- [ ] 监控 libclang 相关 CVE
- [ ] 跟踪依赖库（libloading、libc）的更新

### 升级检查

- [ ] 检查上游 CHANGELOG
- [ ] 验证 BUILD.gn features 兼容性
- [ ] 在 OH 构建环境测试 bindgen 功能
- [ ] 验证生成的绑定代码正确性

### 故障排查

如遇到问题，优先检查：
1. libclang 是否在系统中正确安装
2. BUILD.gn 中的 features 是否正确配置
3. 依赖库（glob、libc、libloading）是否正常
