# 阅读路线建议

根据你的角色和需求，选择最适合的阅读路径：

---

## 路线 A：快速了解 (5 分钟)

**适用人群**: 想快速了解这个库的作用

1. [README.md](./README.md) - 查看概览和关键信息

---

## 路线 B：系统开发者 (15 分钟)

**适用人群**: 需要在自己的模块中使用 lzma

1. [README.md](./README.md) - 了解基本信息
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解如何依赖和使用
3. [03_Build_Integration.md](./03_Build_Integration.md) - 了解头文件引用和编译配置

**关键信息**:
- 依赖声明: `external_deps += [ "lzma:lzma_shared" ]`
- 头文件路径: `//third_party/lzma/C`
- 主要 API: `LzmaDec.h`, `LzmaEnc.h`, `7z.h`

---

## 路线 C：构建/移植工程师 (30 分钟)

**适用人群**: 需要修改构建配置或适配新架构

1. [README.md](./README.md) - 了解基本信息
2. [02_Patches.md](./02_Patches.md) - 了解当前 Patch 和修改原因
3. [03_Build_Integration.md](./03_Build_Integration.md) - 详细了解 BUILD.gn 配置
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解依赖关系，评估改动影响

**关键配置**:
- `BUILD.gn`: 主构建配置
- `lzma.gni`: 源文件列表
- 架构特定配置: `lzma_source_arm64`, `lzma_source_x86_host` 等

---

## 路线 D：升级维护者 (45 分钟)

**适用人群**: 需要同步上游新版本

1. [README.md](./README.md) - 确认当前版本
2. [01_Overview.md](./01_Overview.md) - 了解上游版本差异
3. [02_Patches.md](./02_Patches.md) - **重点**：了解所有 Patch，评估升级影响
4. [03_Build_Integration.md](./03_Build_Integration.md) - 检查 BUILD.gn 是否需要调整
5. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解使用场景，规划测试策略
6. [06_Security.md](./06_Security.md) - 检查安全更新

**升级检查清单**:
- [ ] Patch 是否需要重新生成？
- [ ] API 是否有变化？
- [ ] 汇编代码是否兼容？
- [ ] faultloggerd 功能是否正常？

---

## 路线 E：安全审计 (20 分钟)

**适用人群**: 进行安全审计或漏洞评估

1. [README.md](./README.md) - 了解基本信息
2. [06_Security.md](./06_Security.md) - 安全风险分析
3. [02_Patches.md](./02_Patches.md) - 检查 Patch 引入的新代码
4. [03_Build_Integration.md](./03_Build_Integration.md) - 检查编译选项和 flags

**关注点**:
- CVE 列表和修复状态
- 编译选项的安全性 (`-Werror`, `-Wall`)
- ARM64 PAC-RET 保护
- 多线程代码安全性

---

## 文档索引

| 文档 | 主要内容 | 阅读时间 |
|------|----------|----------|
| [README.md](./README.md) | 概览、导航 | 5 min |
| [01_Overview.md](./01_Overview.md) | 原始库介绍、OH 定位 | 10 min |
| [02_Patches.md](./02_Patches.md) | **核心文档**：Patch 分析 | 15 min |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 详解 | 15 min |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景 | 10 min |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异 | 5 min |
| [06_Security.md](./06_Security.md) | 安全分析 | 10 min |

---

*最后更新: 2025-02-08*
