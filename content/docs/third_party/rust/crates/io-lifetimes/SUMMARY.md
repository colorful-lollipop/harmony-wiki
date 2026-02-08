# io-lifetimes Wiki 阅读指南

## 文档结构

本文档集面向希望了解 io-lifetimes 在 OpenHarmony 中集成情况的开发者和维护者。

```
wiki/
├── README.md              # 本文档：概览和导航
├── SUMMARY.md             # 阅读路线建议
├── 01_Overview.md         # 原始库简介和 OH 定位
├── 02_Patches.md          # Patch 分析（本库无 Patch）
├── 03_Build_Integration.md # BUILD.gn 详细分析
├── 04_Usage_in_OH.md      # 依赖关系和使用场景
└── _work/
    ├── ASSESSMENT.md      # 项目评估报告
    ├── NOTES.md           # 分析过程记录
    └── PLAN.md            # 任务进度
```

---

## 推荐阅读路线

### 路线 1：快速了解（5 分钟）

适合：第一次接触此库的开发者

1. **[README.md](README.md)** - 阅读库概览和 OH 适配概述
2. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解谁在使用这个库

### 路线 2：维护者指南（15 分钟）

适合：需要维护或升级此库的开发者

1. **[README.md](README.md)** - 整体了解
2. **[02_Patches.md](02_Patches.md)** - 确认无 Patch 及原因
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 了解构建配置
4. **`_work/ASSESSMENT.md`** - 查看详细评估报告

### 路线 3：深度分析（30 分钟）

适合：需要全面理解此库的架构师

1. **[01_Overview.md](01_Overview.md)** - 理解库的设计原理
2. **[02_Patches.md](02_Patches.md)** - 分析为何无需 Patch
3. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建系统适配
4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系和使用场景
5. **`_work/ASSESSMENT.md`** - 完整评估数据

---

## 关键信息速查

### 库元数据

| 项目 | 值 |
|------|-----|
| 上游版本 | 1.0.5 |
| OH 版本 | 6.1 |
| Patch 数量 | 0 |
| 主要依赖者 | rustix, is-terminal |

### 构建配置

```gn
ohos_cargo_crate("lib") {
  crate_name = "io_lifetimes"
  crate_type = "rlib"
  features = ["close", "libc", "windows-sys"]
  external_deps = ["rust_libc:lib"]
}
```

### 依赖链

```
应用 → rustix → io-lifetimes → libc
     → is-terminal → rustix → io-lifetimes
```

---

## 按角色查找信息

### 应用开发者

- 如何在我的 crate 中使用？→ [03_Build_Integration.md](03_Build_Integration.md) 的"使用示例"部分
- 有哪些类型可用？→ [01_Overview.md](01_Overview.md) 的"核心类型"部分

### 系统维护者

- 如何升级版本？→ [02_Patches.md](02_Patches.md) 的"升级策略"
- 有哪些依赖？→ [04_Usage_in_OH.md](04_Usage_in_OH.md) 的完整依赖分析

### 安全审查者

- 有安全风险吗？→ `_work/ASSESSMENT.md` 的"安全风险"部分
- 有 Patch 吗？→ [02_Patches.md](02_Patches.md)（答案：无）

---

## 外部参考

### 上游文档

- **GitHub**: https://github.com/sunfishcode/io-lifetimes
- **API 文档**: https://docs.rs/io-lifetimes
- **RFC 3128**: I/O Safety 设计文档

### 相关组件

- **rustix**: https://github.com/bytecodealliance/rustix
- **is-terminal**: https://github.com/sunfishcode/is-terminal

---

*本文档由 OpenHarmony Wiki Generator 自动生成*
