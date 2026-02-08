# 阅读路线建议

本文档为不同角色的开发者提供针对性的阅读建议，帮助您快速找到所需信息。

---

## 场景一：了解 FreeType 在 OH 中的作用

**目标读者**: 项目经理、技术规划者、架构师

### 推荐阅读顺序

1. **README.md** - 5 分钟，了解 FreeType 在 OH 系统中的定位
2. **01_Overview.md** - 10 分钟，了解原始库功能和 OH 用途
3. **04_Usage_in_OH.md** - 10 分钟，了解依赖关系和使用场景

### 跳过内容

- **02_Patches.md** - 技术细节，可跳过
- **03_Build_Integration.md** - 构建细节，可跳过
- **05_API_Differences.md** - API 细节，可跳过

### 关键输出

了解 FreeType 是 OH 字体渲染的基础设施，其稳定性和性能直接影响 UI 体验。

---

## 场景二：进行 FreeType 升级

**目标读者**: 维护者、升级负责人、测试工程师

### 推荐阅读顺序

1. **README.md** - 快速概览
2. **02_Patches.md** - **必须仔细阅读**，了解所有 Patch 及升级影响
3. **03_Build_Integration.md** - 了解构建配置差异
4. **06_Security.md** - 检查安全风险

### 重点关注

- Patch 与上游版本的兼容性（见 02_Patches.md）
- ABI 兼容性变更（02_Patches.md 中 "升级建议" 部分）
- API 变更（05_API_Differences.md）

### 升级检查清单

- [ ] 所有 Patch 已在目标版本中实现
- [ ] 内部函数导出仍然有效
- [ ] 构建配置兼容
- [ ] 安全补丁已应用

---

## 场景三：排查字体渲染问题

**目标读者**: UI 开发工程师、图形工程师、测试工程师

### 推荐阅读顺序

1. **README.md** - 了解 FreeType 的角色
2. **04_Usage_in_OH.md** - 了解使用场景
3. **03_Build_Integration.md** - 了解编译配置

### 排查方向

- **渲染异常**: 检查子像素渲染配置（02_Patches.md）
- **字体加载失败**: 检查模块验证配置（02_Patches.md）
- **内存问题**: 检查 glyph loader 导出（02_Patches.md patch 7）

---

## 场景四：贡献代码到 FreeType OH 适配层

**目标读者**: OH 开发者、图形库维护者

### 推荐阅读顺序

1. **全部文档按顺序阅读**
2. **重点关注**:
   - 02_Patches.md: 了解现有 Patch 的设计思路
   - 03_Build_Integration.md: 了解构建系统适配
   - 05_API_Differences.md: 了解 API 扩展

### 开发规范

- 新增 Patch 应参考现有 Patch 格式
- 提交前验证所有依赖模块
- 更新相关文档

---

## 场景五：安全审计

**目标读者**: 安全工程师、审计人员

### 推荐阅读顺序

1. **06_Security.md** - 安全风险概述
2. **02_Patches.md** - 检查 Patch 引入的变更
3. **上游 CVE 列表** - 对比 OH 版本

### 审计重点

- 已知 CVE 在 OH 版本中的修复状态
- Patch 引入的新代码是否安全
- API 暴露是否合理

---

## 文档速查表

| 文档 | 目的 | 读者 | 阅读时间 |
|------|------|------|----------|
| README.md | 快速概览 | 所有人 | 5 分钟 |
| 01_Overview.md | 背景了解 | 架构师、管理者 | 10 分钟 |
| 02_Patches.md | 技术适配 | 开发者、维护者 | 30 分钟 |
| 03_Build_Integration.md | 构建配置 | 开发者、构建工程师 | 20 分钟 |
| 04_Usage_in_OH.md | 使用场景 | 开发者、集成工程师 | 15 分钟 |
| 05_API_Differences.md | API 变更 | 开发者 | 15 分钟 |
| 06_Security.md | 安全评估 | 安全工程师 | 10 分钟 |

---

## 推荐路线图

```
新开发者:
    README.md → 01_Overview.md → 04_Usage_in_OH.md
        ↓
    有具体任务时阅读对应文档

维护者:
    完整阅读 → 定期检查 02_Patches.md 和 06_Security.md

升级负责人:
    02_Patches.md (重点) → 03_Build_Integration.md → 05_API_Differences.md
```

---

*文档版本: 1.0*
*最后更新: 2025-02-08*
