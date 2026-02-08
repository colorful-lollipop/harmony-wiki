# atty - OpenHarmony Wiki

## 库概览

**atty** 是一个 Rust 编写的轻量级 TTY（终端）检测库，用于检测标准输入/输出/错误流是否连接到终端。

| 属性 | 值 |
|------|-----|
| **上游版本** | 0.2.14 |
| **OH 组件名** | rust_atty |
| **许可证** | MIT |
| **上游地址** | https://github.com/softprops/atty |
| **Patch 数量** | 0 |

## OpenHarmony 适配概述

该库在 OpenHarmony 中的集成非常简单：**无需任何 Patch 或特殊适配**。

### 为什么不需要 Patch？

1. **原生 Unix 兼容**: atty 在 Unix 平台仅使用标准 `libc::isatty()` 接口，OpenHarmony 基于 Linux 内核，完全兼容
2. **代码简洁**: 核心代码仅约 50 行，跨平台逻辑清晰，无复杂依赖
3. **标准构建**: 使用 OH 标准的 `ohos_cargo_crate` GN 模板即可编译

### 构建配置

```gn
ohos_cargo_crate("lib") {
    crate_name = "atty"
    crate_type = "rlib"
    sources = ["src/lib.rs"]
    edition = "2015"
    external_deps = [ "rust_libc:lib" ]
}
```

## 文档导航

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析（重点文档） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 配置详解 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异（如适用） |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 快速参考

### 核心功能
```rust
use atty::Stream;

if atty::is(Stream::Stdout) {
    // 输出到终端，可以使用颜色、进度条等
} else {
    // 输出被重定向，使用纯文本格式
}
```

### 平台支持
- ✅ Linux / Unix（含 OpenHarmony）
- ✅ Windows
- ✅ WebAssembly（始终返回 false）
- ✅ Hermit OS

### 维护建议
- **升级上游版本**: 无障碍，无 Patch 需要重新适配
- **安全更新**: 关注上游 CVE（历史记录良好，无已知严重漏洞）
