# Vulkan-Loader Wiki 阅读路线

## 推荐阅读顺序

### 第一遍：快速了解（10分钟）
1. **[README.md](README.md)** - 本文档简介和导航
2. **[01_Overview.md](01_Overview.md)** - 原始库简介与 OH 定位
   - 了解 Vulkan-Loader 是什么
   - 了解它在 OpenHarmony 中的作用

### 第二遍：核心适配（20分钟）
3. **[02_Patches.md](02_Patches.md)** - Patch 与适配分析 ⭐
   - **重点阅读**：虽然没有传统 Patch，但本节详细分析了 OH 特有的代码级适配
   - 包括 Bundle 管理器、日志系统、WSI 扩展等

4. **[03_Build_Integration.md](03_Build_Integration.md)** - OH 构建适配
   - 了解 BUILD.gn 的关键配置
   - 编译选项和依赖关系

### 第三遍：使用场景（15分钟）
5. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用场景
   - 谁在调用 libvulkan.so
   - 典型使用场景和依赖图

### 第四遍：进阶参考（按需阅读）
6. **[05_API_Differences.md](05_API_Differences.md)** - API/接口差异
   - OH 新增的 API
   - 行为变更的 API

7. **[06_Security.md](06_Security.md)** - 安全风险分析
   - 已知 CVE
   - OH Patch 引入的攻击面

## 按角色阅读

### 图形应用开发者
推荐阅读：
- 01_Overview.md（了解 NDK 接口）
- 04_Usage_in_OH.md（了解依赖关系）

### GPU 驱动开发者
推荐阅读：
- 01_Overview.md（了解 Loader 如何加载驱动）
- 02_Patches.md（了解 WSI 扩展实现）
- 03_Build_Integration.md（了解驱动 JSON 配置）

### 系统维护者
推荐阅读：
- 全文阅读，特别是：
  - 02_Patches.md（升级注意事项）
  - 03_Build_Integration.md（构建配置）
  - 06_Security.md（安全分析）

### 安全审计人员
推荐阅读：
- 02_Patches.md（了解 OH 特有修改）
- 06_Security.md（风险评估）
- 04_Usage_in_OH.md（了解攻击面）

## 文档状态

| 文档 | 状态 | 备注 |
|------|------|------|
| README.md | ✅ 完成 | - |
| 01_Overview.md | ✅ 完成 | - |
| 02_Patches.md | ✅ 完成 | 核心文档 |
| 03_Build_Integration.md | ✅ 完成 | - |
| 04_Usage_in_OH.md | 🔄 部分完成 | 依赖列表待补充 |
| 05_API_Differences.md | ✅ 完成 | - |
| 06_Security.md | ⏳ 待完成 | CVE 分析待补充 |

---

*阅读建议：首次接触建议按顺序阅读 01-04，有具体问题时可按需查阅 05-06。*
