# SUMMARY - pin-utils Wiki

## 建议阅读路线

### 快速了解 (5 分钟)
适合: 需要快速了解该库在 OH 中的作用

1. [README.md](./README.md) - 库概览和 OH 适配概述
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 谁在使用、如何使用

### 深度理解 (15 分钟)
适合: 需要全面了解该库的 OH 集成细节

1. [README.md](./README.md) - 库概览
2. [01_Overview.md](./01_Overview.md) - 库功能和技术背景
3. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统适配
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系和使用场景

### 维护者路线 (20 分钟)
适合: 需要升级、维护或排查问题

1. [README.md](./README.md) - 库概览和维护建议
2. [02_Patches.md](./02_Patches.md) - Patch 分析 (本库无 Patch)
3. [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置详情
4. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整评估报告

---

## 文档索引

| 文档 | 适用场景 | 阅读时间 |
|-----|---------|---------|
| [README.md](./README.md) | 入口、导航、概览 | 3 分钟 |
| [01_Overview.md](./01_Overview.md) | 了解库功能和 OH 角色 | 5 分钟 |
| [02_Patches.md](./02_Patches.md) | Patch 分析 | 3 分钟 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建系统适配 | 5 分钟 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系分析 | 5 分钟 |
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 完整评估数据 | 10 分钟 |

---

## 关键信息速查

### 基础信息
```yaml
库名称: pin-utils
版本: 0.1.0
上游: https://github.com/rust-lang/pin-utils
许可证: Apache-2.0 OR MIT
```

### OH 集成状态
```yaml
Patch 数量: 0
OH 特有代码: 无
特殊构建配置: 无
依赖者数量: 1 (nix 库)
适配复杂度: 极低
```

### 核心功能
- `pin_mut!` - 栈上固定值
- `unsafe_pinned!` - 固定字段投影
- `unsafe_unpinned!` - 非固定字段投影

---

## 常见问题 FAQ

**Q: 为什么 pin-utils 没有 Patch？**  
A: 该库功能完整且通用，原生代码即可满足 OH 需求，无需修改。

**Q: 该库在 OH 中重要吗？**  
A: 中等重要性。它是 nix 库 AIO 功能的依赖，但使用场景有限。

**Q: 升级该库有风险吗？**  
A: 风险极低。无 Patch，标准构建，且代码简单稳定。

**Q: 可以移除该库吗？**  
A: 需先确认 nix 库是否仍需要它。若 nix AIO 功能不再使用，可考虑移除。

---

*最后更新: 2026-02-08*
