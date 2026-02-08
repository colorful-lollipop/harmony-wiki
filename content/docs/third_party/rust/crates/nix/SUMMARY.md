# 阅读路线建议

本文档面向不同角色的开发者提供针对性的阅读建议。

---

## 场景 1：我要在 OH 中使用 nix 库

**目标**：快速了解如何在 OpenHarmony Rust 项目中使用 nix 库

### 推荐阅读顺序

| 顺序 | 文档 | 内容 |
|------|------|------|
| 1 | [README.md](README.md) | 5 分钟快速概览 |
| 2 | [01_Overview.md](01_Overview.md) | 了解库的功能定位 |
| 3 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 查看使用示例和依赖配置 |
| 4 | [03_Build_Integration.md](03_Build_Integration.md) | 理解构建配置（可选） |

### 快速要点

- 无需 Patch，直接依赖即可使用
- 使用 `ohos_rust_shared_library` 或 `ohos_rust_static_library` 构建
- 参考 HDC 工具的实际使用方式

### 相关章节

- `04_Usage_in_OH.md` - 使用示例
- `03_Build_Integration.md` - BUILD.gn 配置

---

## 场景 2：我要维护/升级 nix 库

**目标**：了解 OH 对 nix 的定制内容，准备版本升级

### 推荐阅读顺序

| 顺序 | 文档 | 内容 |
|------|------|------|
| 1 | [README.md](README.md) | 整体架构概览 |
| 2 | [02_Patches.md](02_Patches.md) | 确认无 OH Patch |
| 3 | [03_Build_Integration.md](03_Build_Integration.md) | 构建配置差异 |
| 4 | [06_Security.md](06_Security.md) | 安全注意事项 |
| 5 | [05_API_Differences.md](05_API_Differences.md) | API 兼容性 |

### 升级检查清单

- [ ] 确认上游版本兼容性
- [ ] 检查 features 配置是否需要调整
- [ ] 验证 OH 目标平台支持状态
- [ ] 运行 OH Rust 构建测试

### 相关章节

- `02_Patches.md` - Patch 分析
- `03_Build_Integration.md` - 构建配置
- `06_Security.md` - 安全更新

---

## 场景 3：我要贡献 OH Rust 基础设施

**目标**：了解 nix 在 OH Rust 生态中的位置和作用

### 推荐阅读顺序

| 顺序 | 文档 | 内容 |
|------|------|------|
| 1 | [README.md](README.md) | 项目整体介绍 |
| 2 | [01_Overview.md](01_Overview.md) | 功能模块详解 |
| 3 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系图 |
| 4 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 完整评估报告 |

### 关键信息

- **依赖链位置**：基础系统调用绑定层
- **被依赖情况**：被 HDC 等工具使用
- **构建模板**：标准 `ohos_cargo_crate`

### 相关章节

- `04_Usage_in_OH.md` - 依赖关系
- `_work/ASSESSMENT.md` - 完整评估

---

## 场景 4：我要分析/排查问题

**目标**：定位与 nix 库相关的问题

### 推荐阅读顺序

| 顺序 | 文档 | 内容 |
|------|------|------|
| 1 | [06_Security.md](06_Security.md) | 已知问题排查 |
| 2 | [05_API_Differences.md](05_API_Differences.md) | API 兼容性问题 |
| 3 | [02_Patches.md](02_Patches.md) | Patch 相关问题 |
| 4 | [01_Overview.md](01_Overview.md) | 功能模块说明 |

### 常见问题方向

- API 行为差异
- 条件编译问题 (`target_os = "ohos"`)
- 依赖版本冲突

### 相关章节

- `05_API_Differences.md` - API 差异
- `06_Security.md` - 安全问题

---

## 文档速查表

| 文档 | 适用场景 | 复杂度 |
|------|----------|--------|
| README.md | 所有场景快速入门 | ⭐ |
| 01_Overview.md | 了解库功能 | ⭐⭐ |
| 02_Patches.md | Patch 分析/升级 | ⭐ |
| 03_Build_Integration.md | 构建配置 | ⭐⭐ |
| 04_Usage_in_OH.md | 实际使用 | ⭐⭐ |
| 05_API_Differences.md | API 兼容 | ⭐⭐ |
| 06_Security.md | 安全相关 | ⭐⭐ |
| _work/ASSESSMENT.md | 完整技术评估 | ⭐⭐⭐ |

---

## 推荐路线图

```
新用户
    │
    ▼
┌─────────────────┐
│   README.md     │  ◄── 必读
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ 01_Overview.md │  ◄── 了解功能
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ 04_Usage_in_OH │  ◄── 学习使用
└─────────────────┘
    │
    ▼
┌─────────────────┐
│   开始使用      │
└─────────────────┘

维护者
    │
    ▼
┌─────────────────┐
│ 全部文档阅读    │  ◄── 系统了解
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ _work/ASSESSMENT│  ◄── 详细评估
└─────────────────┘
```
