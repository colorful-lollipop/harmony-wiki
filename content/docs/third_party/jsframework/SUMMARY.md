# 阅读路线建议

本文档为 OpenHarmony jsframework 第三方库/wiki提供多种阅读路线，满足不同需求的读者。

---

## 路线 A：快速概览（5 分钟）

适合：首次接触此库，想快速了解整体情况

### 必读章节

1. **[README.md](./README.md)** - 2 分钟
   - 快速了解库的基本信息
   - 导航到感兴趣的部分

2. **[01_Overview.md](./01_Overview.md)** - 3 分钟
   - 理解框架在 OH 中的定位
   - 了解与上游 Weex 的差异

### 产出

- 理解 jsframework 是什么
- 知道它在 OH 系统中的位置

---

## 路线 B：集成开发（15 分钟）

适合：要将 jsframework 集成到新模块的开发者

### 必读章节

1. **[README.md](./README.md)** - 2 分钟
2. **[01_Overview.md](./01_Overview.md)** - 3 分钟
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 5 分钟
   - 理解 BUILD.gn 配置
   - 了解构建产物
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 5 分钟
   - 了解依赖关系
   - 查看集成示例

### 产出

- 能够将 jsframework 集成到新模块
- 理解构建配置的含义

---

## 路线 C：深度理解（30 分钟）

适合：需要深入理解框架实现，或要进行二次开发的工程师

### 必读章节

1. **[README.md](./README.md)** - 2 分钟
2. **[01_Overview.md](./01_Overview.md)** - 5 分钟
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 8 分钟
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 8 分钟
5. **[05_API_Differences.md](./05_API_Differences.md)** - 7 分钟
   - 详细了解 OH 特有 API

### 产出

- 全面理解框架架构
- 能够进行二次开发

---

## 路线 D：维护升级（45 分钟）

适合：负责维护此库，或要进行版本升级的工程师

### 必读章节（全部）

1. **README.md** - 2 分钟
2. **01_Overview.md** - 5 分钟
3. **02_Patches.md** - 5 分钟（简短）
4. **03_Build_Integration.md** - 10 分钟
5. **04_Usage_in_OH.md** - 10 分钟
6. **05_API_Differences.md** - 8 分钟
7. **06_Security.md** - 5 分钟

### 附加资源

- **[\_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 项目评估详情
- **[\_work/NOTES.md](./_work/NOTES.md)** - 分析过程记录

### 产出

- 掌握所有适配细节
- 了解升级时的注意事项
- 清楚安全风险和修复策略

---

## 角色专属路线

### 前端开发者

重点关注：[04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解如何在应用中使用框架

### 系统集成工程师

重点关注：[03_Build_Integration.md](./03_Build_Integration.md) - 理解构建配置和依赖关系

### 框架维护者

全部文档 + [05_API_Differences.md](./05_API_Differences.md) - 了解 API 变更历史

### 安全工程师

重点关注：[06_Security.md](./06_Security.md) - 安全风险分析

---

## 文档更新检查清单

如果你是维护者，更新文档时请检查：

- [ ] 版本号是否更新
- [ ] 新增 API 是否添加到 [05_API_Differences.md](./05_API_Differences.md)
- [ ] 新增依赖是否更新 [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- [ ] 安全漏洞是否添加到 [06_Security.md](./06_Security.md)
- [ ] 变更是否记录在 [\_work/NOTES.md](./_work/NOTES.md)
