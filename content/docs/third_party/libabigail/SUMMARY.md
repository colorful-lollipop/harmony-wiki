# 阅读路线建议

## 根据您的角色选择阅读路径

### 🔧 如果你是 OH 开发者（使用 libabigail）

**阅读顺序**:
1. [README.md](README.md) - 快速了解
2. [01_Overview.md](01_Overview.md) - 了解库的基本功能
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解如何在 OH 中使用

**预计阅读时间**: 15 分钟

---

### 🔨 如果你是构建系统维护者

**阅读顺序**:
1. [README.md](README.md) - 快速了解
2. [01_Overview.md](01_Overview.md) - 了解库的基本功能
3. [03_Build_Integration.md](03_Build_Integration.md) - 详细了解 BUILD.gn 配置
4. [02_Patches.md](02_Patches.md) - 了解为何无 Patch

**预计阅读时间**: 30 分钟

---

### 🛡️ 如果你是安全工程师

**阅读顺序**:
1. [README.md](README.md) - 快速了解
2. [06_Security.md](06_Security.md) - 安全风险分析
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解使用场景

**预计阅读时间**: 20 分钟

---

### 📚 如果你需要全面了解

**阅读顺序**:
1. [README.md](README.md)
2. [01_Overview.md](01_Overview.md)
3. [02_Patches.md](02_Patches.md)
4. [03_Build_Integration.md](03_Build_Integration.md)
5. [04_Usage_in_OH.md](04_Usage_in_OH.md)
6. [05_API_Differences.md](05_API_Differences.md)
7. [06_Security.md](06_Security.md)

**预计阅读时间**: 60 分钟

---

## 文档速查表

| 你想了解 | 阅读文档 |
|----------|----------|
| 这是什么库？ | [01_Overview.md](01_Overview.md) |
| OH 用它来做什么？ | [01_Overview.md](01_Overview.md) + [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| 有哪些 Patch？ | [02_Patches.md](02_Patches.md)（无 Patch） |
| BUILD.gn 怎么配置的？ | [03_Build_Integration.md](03_Build_Integration.md) |
| 谁依赖这个库？ | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| API 有变化吗？ | [05_API_Differences.md](05_API_Differences.md) |
| 安全吗？ | [06_Security.md](06_Security.md) |

---

## 关键结论速览

- **无 Patch**: 直接使用上游 2.8 版本
- **Host 工具**: 不随设备镜像发布
- **单一用途**: 仅用于 SA 独立升级的 ABI 检查
- **核心工具**: abidiff（对比 ABI）、abidw（提取 ABI）
