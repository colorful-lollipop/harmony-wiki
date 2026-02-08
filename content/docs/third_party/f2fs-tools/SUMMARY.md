# 阅读路线建议

## 根据您的角色选择阅读路径

### 👨‍💻 系统开发者（开发/调试）
**推荐路径**：
1. [01_Overview.md](./01_Overview.md) - 了解基础信息（10分钟）
2. [03_Build_Integration.md](./03_Build_Integration.md) - 理解 BUILD.gn 结构（15分钟）
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖关系和使用场景（10分钟）
4. [02_Patches.md](./02_Patches.md) - 深入了解代码修改（20分钟）

### 🔧 集成工程师（移植/适配）
**推荐路径**：
1. [01_Overview.md](./01_Overview.md) - 快速了解（5分钟）
2. [02_Patches.md](./02_Patches.md) - 重点阅读 WITH_OHOS 适配点（30分钟）
3. [03_Build_Integration.md](./03_Build_Integration.md) - 理解构建配置（15分钟）
4. [05_API_Differences.md](./05_API_Differences.md) - 查看新增接口（10分钟）

### 🔒 安全工程师（审计/加固）
**推荐路径**：
1. [01_Overview.md](./01_Overview.md) - 基础信息（5分钟）
2. [06_Security.md](./06_Security.md) - 安全风险分析（15分钟）
3. [02_Patches.md](./02_Patches.md) - 关注安全相关的修改（15分钟）
4. [03_Build_Integration.md](./03_Build_Integration.md) - 检查编译配置（10分钟）

### 📚 全面了解（完整阅读）
**完整路径**（约 2 小时）：
1. [01_Overview.md](./01_Overview.md)
2. [02_Patches.md](./02_Patches.md)
3. [03_Build_Integration.md](./03_Build_Integration.md)
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md)
5. [05_API_Differences.md](./05_API_Differences.md)
6. [06_Security.md](./06_Security.md)

---

## 关键文档速查

| 问题 | 查看文档 |
|-----|---------|
| f2fs-tools 是什么？ | [01_Overview.md](./01_Overview.md) |
| OH 做了哪些修改？ | [02_Patches.md](./02_Patches.md) |
| BUILD.gn 怎么配置的？ | [03_Build_Integration.md](./03_Build_Integration.md) |
| 谁在使用这个库？ | [04_Usage_in_OH.md](./04_Usage_in_OH.md) |
| 有哪些新增 API？ | [05_API_Differences.md](./05_API_Differences.md) |
| 有安全漏洞吗？ | [06_Security.md](./06_Security.md) |

---

## 文档依赖关系

```
README.md (入口)
    │
    ├── 01_Overview.md ─────┐
    │                        │
    ├── 02_Patches.md ──────┼── 03_Build_Integration.md
    │                        │
    ├── 03_Build_Integration.md
    │
    ├── 04_Usage_in_OH.md ──┤
    │                        │
    ├── 05_API_Differences.md
    │
    └── 06_Security.md
```
