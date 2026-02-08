# SUMMARY.md - liburing Wiki 阅读路线

本文档提供 liburing Wiki 的结构化阅读路线，帮助不同角色的读者快速定位所需信息。

---

## 📖 阅读路线

### 🚀 快速入门路线 (5 分钟)

适合想快速了解 liburing 在 OH 中情况的读者：

1. **本页概览** - 了解本文档结构
2. **[01_Overview.md](./01_Overview.md)** - 第 1-2 节 (库简介和 OH 定位)
3. **[02_Patches.md](./02_Patches.md)** - 结论部分 (无 Patch)
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 依赖关系摘要

---

### 🔧 系统集成路线 (15 分钟)

适合需要将 liburing 集成到其他模块的开发者：

1. **[01_Overview.md](./01_Overview.md)** - 完整阅读，了解功能边界
2. **[03_Build_Integration.md](./03_Build_Integration.md)** - 重点阅读
   - BUILD.gn 结构
   - 头文件包含路径
   - 依赖声明方式
3. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 查看使用示例
4. **[05_API_Differences.md](./05_API_Differences.md)** - 确认 API 兼容性

**关键参考代码**:
```gn
# 在您的 BUILD.gn 中添加依赖
deps = [ "//third_party/liburing:liburing" ]
```

---

### 🔒 安全审计路线 (20 分钟)

适合进行安全评估的工程师：

1. **[02_Patches.md](./02_Patches.md)** - 确认无 Patch 引入
2. **[06_Security.md](./06_Security.md)** - 完整阅读
   - 已知 CVE 列表
   - 版本升级建议
   - 风险评估
3. **[01_Overview.md](./01_Overview.md)** - 了解功能范围，识别攻击面
4. **[03_Build_Integration.md](./03_Build_Integration.md)** - 检查编译选项

---

### 📚 完整学习路线 (45 分钟)

适合需要全面掌握 liburing 的开发者：

#### 第一阶段：理解 (15 分钟)
1. [01_Overview.md](./01_Overview.md) - 了解库的背景和功能
2. 阅读上游文档：https://kernel.dk/io_uring.pdf

#### 第二阶段：分析 (15 分钟)
3. [02_Patches.md](./02_Patches.md) - Patch 分析
4. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系

#### 第三阶段：深入 (15 分钟)
6. [05_API_Differences.md](./05_API_Differences.md) - API 差异
7. [06_Security.md](./06_Security.md) - 安全考量
8. 查看源码：`src/include/liburing.h`

---

## 📋 文档速查表

| 文档 | 页数 | 关键章节 | 目标读者 |
|-----|------|---------|---------|
| 01_Overview | ~2 | io_uring 简介、OH 定位 | 所有人 |
| 02_Patches | ~1 | Patch 清单、升级建议 | 维护者 |
| 03_Build_Integration | ~2 | BUILD.gn 详解、配置分析 | 集成工程师 |
| 04_Usage_in_OH | ~1 | 依赖者列表、使用场景 | 架构师 |
| 05_API_Differences | ~1 | API 变更说明 | 应用开发者 |
| 06_Security | ~1 | CVE、升级策略 | 安全工程师 |

---

## 🎯 按问题类型查找

### "liburing 是什么？"
→ [01_Overview.md - 第 1 节](./01_Overview.md#1-原始库简介)

### "OH 对 liburing 做了什么修改？"
→ [02_Patches.md - 结论](./02_Patches.md#结论)

### "如何在 OH 中构建 liburing？"
→ [03_Build_Integration.md - BUILDgn 结构](./03_Build_Integration.md#buildgn-结构说明)

### "哪些模块依赖 liburing？"
→ [04_Usage_in_OH.md - 直接依赖者](./04_Usage_in_OH.md#直接依赖者)

### "liburing 有安全漏洞吗？"
→ [06_Security.md - CVE 清单](./06_Security.md#已知-cve)

### "升级 liburing 需要注意什么？"
→ [03_Build_Integration.md - 升级建议](./03_Build_Integration.md#升级与维护建议)

---

## 🔗 外部资源

### 必读
- [io_uring 技术论文](https://kernel.dk/io_uring.pdf) - 理解 io_uring 设计原理
- [liburing GitHub](https://github.com/axboe/liburing) - 上游源码

### 参考
- [io_uring 邮件列表归档](https://lore.kernel.org/io-uring/)
- [Kernel 文档 - io_uring](https://www.kernel.org/doc/html/latest/io_uring.html)

---

## 💡 提示

- 所有代码片段均可直接复制使用
- 文档中的路径都是相对于 OH 代码根目录的
- 遇到 `TODO(需确认)` 标记表示需要人工核实的内容

---

*阅读路线版本: 1.0*  
*最后更新: 2026-02-08*
