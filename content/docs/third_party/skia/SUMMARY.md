# Skia Wiki - 阅读路线建议

> **最后更新**: 2026-02-08
> **目标读者**: OpenHarmony 图形系统开发者、第三方库维护者、系统架构师

---

## 快速导航

### 如果你需要...

| 目的 | 推荐阅读路径 |
|------|------------|
| **快速了解 Skia 在 OH 中的作用** | `01_Overview.md` → `04_Usage_in_OH.md` |
| **了解 OH 对 Skia 的定制化修改** | `02_Patches.md` → `03_Build_Integration.md` → `05_API_Differences.md` |
| **排查图形渲染问题** | `04_Usage_in_OH.md` → `05_API_Differences.md` → `06_Security.md` |
| **评估 Skia 版本升级** | `02_Patches.md` → `_work/ASSESSMENT.md` |
| **学习 OH 图形架构** | `01_Overview.md` → `04_Usage_in_OH.md` → `_work/ASSESSMENT.md` |

---

## 详细文档说明

### 01_Overview.md - Skia 库概览
**内容**：
- Skia 原始库简介
- 在 OpenHarmony 中的定位
- 版本信息与上游地址

**适合人群**：
- 新接触 OH 图形系统的开发者
- 需要了解技术选型的架构师

**预计阅读时间**: 5 分钟

---

### 02_Patches.md - Patch 详细分析
**内容**：
- 子库 Patch 清单（zlib、icu、expat）
- 每个 Patch 的修改目的和 OH 价值
- Patch 升级建议

**适合人群**：
- 第三方库维护者
- 需要评估版本升级影响的工程师
- 安全审计人员

**预计阅读时间**: 30 分钟

**重点章节**：
- `2.1 Zlib 性能优化 Patch`
- `2.2 ICU 本地化 Patch`
- `2.3 Patch 升级建议`

---

### 03_Build_Integration.md - OH 构建集成
**内容**：
- BUILD.gn 结构说明
- 关键编译选项（defines、configs、flags）
- OH 特定宏定义
- 性能优化配置（PGO、LTO）

**适合人群**：
- 构建系统工程师
- 需要自定义编译选项的开发者
- 性能优化工程师

**预计阅读时间**: 20 分钟

**重点章节**：
- `3.2 OH 特定宏定义`
- `3.3 性能优化配置`
- `3.4 构建变体`

---

### 04_Usage_in_OH.md - 依赖关系与使用
**内容**：
- 谁在使用 Skia（依赖者列表）
- 主要使用场景
- 依赖关系图
- 代码调用示例

**适合人群**：
- 图形应用开发者
- 系统集成工程师
- 技术架构师

**预计阅读时间**: 15 分钟

**重点章节**：
- `4.1 直接依赖者`
- `4.2 典型使用场景`
- `4.3 依赖关系图`

---

### 05_API_Differences.md - API/接口差异
**内容**：
- OH 新增的 API（fontmgr_ohos）
- 行为变更的 API
- 废弃或禁用的功能
- 使用示例

**适合人群**：
- 需要调用 Skia API 的开发者
- 图形应用开发者
- 移植开发者

**预计阅读时间**: 25 分钟

**重点章节**：
- `5.1 OH 新增 API`
- `5.2 字体管理 API`

---

### 06_Security.md - 安全风险分析
**内容**：
- Skia 已知 CVE 和 OH 版本修复状态
- OH Patch 引入的新攻击面
- 建议的安全升级策略

**适合人群**：
- 安全工程师
- 安全审计人员
- 系统管理员

**预计阅读时间**: 20 分钟

**重点章节**：
- `6.1 已知 CVE 状态`
- `6.2 OH 特定安全考虑`

---

## 工作文档（内部使用）

### _work/ASSESSMENT.md - 项目评估
**内容**：
- Phase 0 信息收集结果
- Patch 文件清单
- OH 使用情况分析
- 需进一步确认的问题

**用途**：
- 文档编写的基础数据
- 后续分析的参考
- 问题追踪

---

## 阅读建议

### 初次接触 Skia + OH
推荐顺序：
1. `01_Overview.md` - 了解 Skia 是什么
2. `04_Usage_in_OH.md` - 了解在 OH 中如何使用
3. `03_Build_Integration.md` - 了解如何编译配置

### 图形应用开发者
推荐顺序：
1. `04_Usage_in_OH.md` - 确认你的使用场景
2. `05_API_Differences.md` - 了解 OH 特有 API
3. `03_Build_Integration.md` - 配置你的构建选项

### 系统集成/移植开发者
推荐顺序：
1. `01_Overview.md` - 了解基础信息
2. `02_Patches.md` - 了解有哪些 Patch
3. `05_API_Differences.md` - 了解 API 差异
4. `06_Security.md` - 评估安全风险

### 第三方库维护者
推荐顺序：
1. `02_Patches.md` - 详细分析 Patch
2. `_work/ASSESSMENT.md` - 了解评估细节
3. `06_Security.md` - 了解安全考虑

---

## 版本说明

| 文档 | 版本 | 更新日期 | 维护者 |
|------|------|---------|--------|
| README.md | 1.0 | 2026-02-08 | Sisyphus |
| SUMMARY.md | 1.0 | 2026-02-08 | Sisyphus |
| 01_Overview.md | 待完成 | - | - |
| 02_Patches.md | 待完成 | - | - |
| 03_Build_Integration.md | 待完成 | - | - |
| 04_Usage_in_OH.md | 待完成 | - | - |
| 05_API_Differences.md | 待完成 | - | - |
| 06_Security.md | 待完成 | - | - |

---

## 贡献指南

如果发现文档错误或有改进建议，请：
1. 检查 `_work/ASSESSMENT.md` 是否包含相关数据
2. 联系 Skia 组件维护者（yangguangyu6@huawei.com）
3. 提交文档更新 PR

---

## 参考资料

- [Skia 官方文档](https://skia.org/)
- [OpenHarmony 图形子系统](https://docs.openharmony.cn/)
- [Skia 上游仓库](https://skia.googlesource.com/skia.git)
