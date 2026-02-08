# libbpf Wiki 阅读路线建议

> 本文档提供针对不同读者的推荐阅读路径，帮助你快速找到所需信息。

---

## 按角色分类

### 新用户 / 首次接触 libbpf

**目标**: 了解 libbpf 是什么，在 OH 中如何使用

**推荐路径**:
```
1. README.md              (5分钟)  - 快速了解 libbpf 和 OH 适配概述
2. SUMMARY.md             (2分钟)  - 选择适合自己的阅读路径
3. 01_Overview.md         (10分钟) - 详细了解 libbpf 功能和定位
4. 04_Usage_in_OH.md      (15分钟) - 查看 OH 中的实际使用案例
```

**收获**: 能够判断是否需要使用 libbpf，以及如何在 OH 中集成

---

### 应用开发者

**目标**: 学习如何在 OH 应用中使用 libbpf

**推荐路径**:
```
1. README.md              (5分钟)  - 快速开始示例
2. 03_Build_Integration.md (10分钟) - 了解如何正确引入依赖
3. 04_Usage_in_OH.md      (15分钟) - 参考 OH 系统模块的使用方式
4. 05_API_Differences.md  (5分钟)  - 确认 API 兼容性
```

**收获**: 能够在应用中正确使用 libbpf，避免常见陷阱

---

### 系统集成开发者

**目标**: 深入理解 libbpf 的 OH 适配细节

**推荐路径**:
```
1. 01_Overview.md         (10分钟) - libbpf 核心功能和 OH 定位
2. 03_Build_Integration.md (20分钟) - BUILD.gn 配置详解
3. 02_Patches.md         (10分钟) - 确认是否有需要关注的 Patch
4. 04_Usage_in_OH.md      (15分钟) - 了解依赖关系
```

**收获**: 理解 libbpf 的适配策略，能够正确集成或升级

---

### 维护者 / 版本升级者

**目标**: 升级 libbpf 版本或修复问题

**推荐路径**:
```
1. _work/ASSESSMENT.md   (10分钟) - 完整的项目评估
2. 02_Patches.md         (15分钟) - Patch 升级建议
3. 03_Build_Integration.md (20分钟) - 构建配置变更点
4. 06_Security.md        (15分钟) - 安全风险和升级策略
```

**收获**: 知道如何安全升级版本，以及需要保留哪些配置

---

### 安全工程师

**目标**: 评估 libbpf 的安全风险

**推荐路径**:
```
1. 01_Overview.md         (10分钟) - 了解 libbpf 的安全相关功能
2. 06_Security.md        (30分钟) - CVE 分析和安全建议
3. 03_Build_Integration.md (10分钟) - 构建时的安全配置
```

**收获**: 了解 libbpf 的安全风险点，知道如何安全使用

---

## 按问题分类

### 我想了解 libbpf 是什么？

→ 阅读 **[README.md](README.md)** 的快速导航部分
→ 深入阅读 **[01_Overview.md](01_Overview.md)**

---

### 我想在 OH 应用中使用 libbpf？

→ 阅读 **[README.md](README.md)** 的快速开始部分
→ 参考 **[04_Usage_in_OH.md](04_Usage_in_OH.md)** 的使用案例
→ 检查 **[05_API_Differences.md](05_API_Differences.md)** 的兼容性说明

---

### 我遇到了构建问题？

→ 详细阅读 **[03_Build_Integration.md](03_Build_Integration.md)**
→ 检查 BUILD.gn 配置
→ 确认依赖的 elfio 和 zlib 组件可用

---

### 我需要升级 libbpf 版本？

→ 阅读 **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** 的升级建议
→ 参考 **[02_Patches.md](02_Patches.md)** 确认 Patch 状态
→ 检查 **[06_Security.md](06_Security.md)** 的安全建议

---

### 我担心安全问题？

→ 重点阅读 **[06_Security.md](06_Security.md)**
→ 了解 CVE 修复状态
→ 查看安全使用建议

---

## 文档结构说明

### 必读文档 (📋)

- **README.md**: 库概览，包含快速开始和导航
- **SUMMARY.md**: 本文档，阅读路线建议
- **01_Overview.md**: libbpf 详细介绍和 OH 定位

### 核心文档 (📚)

- **03_Build_Integration.md**: OH 构建系统适配详解（BUILD.gn）
- **04_Usage_in_OH.md**: 依赖关系和使用场景

### 参考文档 (📖)

- **02_Patches.md**: Patch 分析（说明无实际 Patch）
- **05_API_Differences.md**: API 差异（说明无差异）
- **06_Security.md**: 安全风险分析

### 工作文档 (📝)

- **_work/ASSESSMENT.md**: 项目评估结果
- **_work/PLAN.md**: 任务进度
- **_work/NOTES.md**: 分析过程记录

---

## 常见问题

### Q: libbpf 在 OH 中有源代码修改吗？

**A**: 没有。libbpf 是纯净的上游版本，所有适配通过 BUILD.gn 实现。
详见：[03_Build_Integration.md](03_Build_Integration.md)

---

### Q: libbpf 依赖哪些 OH 组件？

**A**: elfio（ELF 文件解析）和 zlib（压缩）。
详见：[03_Build_Integration.md](03_Build_Integration.md)

---

### Q: 哪些 OH 模块在使用 libbpf？

**A**: netmanager_base（网络防火墙）、hiebpf（性能追踪）、smartperf_host（性能数据流）。
详见：[04_Usage_in_OH.md](04_Usage_in_OH.md)

---

### Q: 如何升级 libbpf 版本？

**A**: 参考 `scripts/sync-kernel.sh` 和 `SYNC.md`，保留 BUILD.gn 配置。
详见：[_work/ASSESSMENT.md](_work/ASSESSMENT.md) 的升级建议

---

### Q: libbpf 在 OH 中的 API 与上游一致吗？

**A**: 一致。没有 OH 特定的 API 差异。
详见：[05_API_Differences.md](05_API_Differences.md)

---

## 反馈与贡献

如果你发现文档有错误或需要补充的内容，欢迎反馈：

- 提交 Issue 到 OpenHarmony 仓库
- 联系维护者: xiazhonglin@huawei.com

---

**最后更新**: 2026-02-07
**libbpf 版本**: v1.3.4
