# 阅读路线建议

## 根据角色选择阅读路线

### 我是架构师/技术管理者

**推荐路线**: 概览 → 依赖关系 → 风险评估

```
README.md
    ↓
01_Overview.md (重点: OH 定位和作用)
    ↓
04_Usage_in_OH.md (重点: 依赖关系图)
    ↓
06_Security.md (重点: 当前安全状态)
```

**阅读时间**: 约 20 分钟

**关键产出**:
- 了解 Python 在 OH 中的定位
- 掌握依赖关系
- 评估安全风险

---

### 我是构建工程师

**推荐路线**: 构建系统 → Patch 分析 → 升级策略

```
README.md
    ↓
03_Build_Integration.md (重点: configure.ac 修改)
    ↓
02_Patches.md (重点: Patch 1 cross_compile_support_ohos)
    ↓
_work/ASSESSMENT.md (参考: 0.2 Patch 分析)
```

**阅读时间**: 约 30 分钟

**关键产出**:
- 理解构建系统适配
- 掌握 Patch 内容
- 了解升级流程

---

### 我是开发者 (使用 Python 工具)

**推荐路线**: 概览 → API 差异 → 依赖使用

```
README.md
    ↓
01_Overview.md (重点: 禁用模块)
    ↓
05_API_Differences.md (重点: 替代方案)
    ↓
04_Usage_in_OH.md (参考: 典型使用场景)
```

**阅读时间**: 约 15 分钟

**关键产出**:
- 了解禁用的模块
- 掌握 API 替代方案
- 理解使用场景

---

### 我是安全工程师

**推荐路线**: 安全文档 → Patch 分析 → CVE 状态

```
README.md
    ↓
06_Security.md (重点: 所有章节)
    ↓
02_Patches.md (参考: Patch 引入的新攻击面)
    ↓
_work/ASSESSMENT.md (参考: git log 安全修复)
```

**阅读时间**: 约 25 分钟

**关键产出**:
- 了解当前 CVE 状态
- 评估 Patch 风险
- 建立监控流程

---

### 我是新维护者

**推荐路线**: 完整阅读

```
README.md
    ↓
01_Overview.md
    ↓
02_Patches.md (重点)
    ↓
03_Build_Integration.md
    ↓
04_Usage_in_OH.md
    ↓
05_API_Differences.md
    ↓
06_Security.md
    ↓
_work/ASSESSMENT.md (参考)
```

**阅读时间**: 约 60 分钟

**关键产出**:
- 全面理解项目
- 掌握维护要点
- 了解升级策略

---

## 按主题阅读

### 主题: Patch 维护

**相关文档**:
- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [02_Patches.md#patch-升级策略](02_Patches.md#patch-升级策略) - 升级流程
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - Patch 清单

### 主题: 构建系统

**相关文档**:
- [03_Build_Integration.md](03_Build_Integration.md) - 构建适配
- [03_Build_Integration.md#关键配置文件](03_Build_Integration.md#关键配置文件) - 配置说明
- [02_Patches.md#patch-1-cross_compile_support_ohospatch](02_Patches.md#patch-1-cross_compile_support_ohospatch) - 交叉编译 Patch

### 主题: 依赖关系

**相关文档**:
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用
- [04_Usage_in_OH.md#依赖关系图](04_Usage_in_OH.md#依赖关系图) - 可视化
- [01_Overview.md#在-oh-中的角色](01_Overview.md#在-oh-中的角色) - 定位说明

### 主题: 安全

**相关文档**:
- [06_Security.md](06_Security.md) - 完整分析
- [06_Security.md#已知-cve-和修复状态](06_Security.md#已知-cve-和修复状态) - CVE 列表
- [02_Patches.md#patch-引入的新攻击面](02_Patches.md#patch-引入的新攻击面) - 风险评估

---

## 快速参考

### 常见问题

| 问题 | 答案位置 |
|-----|---------|
| 哪些模块被禁用? | [01_Overview.md#模块禁用](01_Overview.md#模块禁用) |
| 如何升级 Patch? | [02_Patches.md#patch-升级策略](02_Patches.md#patch-升级策略) |
| 有哪些 CVE? | [06_Security.md#已知-cve-和修复状态](06_Security.md#已知-cve-和修复状态) |
| 谁依赖这个库? | [04_Usage_in_OH.md#直接依赖者清单](04_Usage_in_OH.md#直接依赖者清单) |
| 如何构建? | [03_Build_Integration.md#构建流程](03_Build_Integration.md#构建流程) |

### 关键文件位置

| 文件 | 位置 | 说明 |
|-----|------|-----|
| `README.OpenSource` | 根目录 | 原始库信息 |
| `bundle.json` | 根目录 | OH 组件信息 |
| `patches/*.patch` | patches/ | OH 适配 Patch |
| `configure.ac` | 根目录 | 自动配置脚本源 |
| `setup.py` | 根目录 | 模块构建配置 |

---

## 反馈和改进

如发现文档问题或需要补充内容，请:

1. 提交 Issue 到本仓库
2. 关联相关文档和具体段落
3. 提供改进建议或补充信息

---

**文档版本**: v1.0 (对应 Python 3.11.4)
**最后更新**: 2026-02-08
