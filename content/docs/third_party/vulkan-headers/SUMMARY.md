# 阅读路线指南

## 读者类型

### 北向应用开发者

如果你是 OpenHarmony 北向应用开发者，使用 Vulkan 进行图形渲染：

1. **快速入门** → 阅读 [01_Overview.md](./01_Overview.md)
2. **API 参考** → 参考 [05_API_Differences.md](./05_API_Differences.md)
3. **使用示例** → 查看 [04_Usage_in_OH.md](./04_Usage_in_OH.md)

### 南向驱动开发者

如果你是 SoC 厂商，需要实现 Vulkan 驱动：

1. **扩展要求** → 重点阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 和 [05_API_Differences.md](./05_API_Differences.md)
2. **构建配置** → 查看 [03_Build_Integration.md](./03_Build_Integration.md)
3. **接口规范** → 参考 Vulkan 官方文档和 [02_Patches.md](./02_Patches.md)

### 系统集成开发者

如果你负责 OpenHarmony 图形子系统集成：

1. **完整流程** → 按顺序阅读所有文档
2. **依赖关系** → 重点关注 [04_Usage_in_OH.md](./04_Usage_in_OH.md)
3. **适配要点** → 查看 [03_Build_Integration.md](./03_Build_Integration.md)

## 推荐阅读顺序

### 场景 1：了解 Vulkan-Headers 在 OH 中的作用

```
README.md → 01_Overview.md → 04_Usage_in_OH.md
```

### 场景 2：理解 OH 特有扩展

```
README.md → 01_Overview.md → 05_API_Differences.md
```

### 场景 3：进行版本升级

```
README.md → 02_Patches.md → 03_Build_Integration.md → 06_Security.md
```

### 场景 4：排查依赖问题

```
README.md → 04_Usage_in_OH.md → 03_Build_Integration.md
```

## 文档速查

| 文档 | 主要内容 | 适合场景 |
|------|---------|---------|
| README.md | 项目概览和快速导航 | 初次了解 |
| 01_Overview.md | 功能描述和 OH 定位 | 理解用途 |
| 02_Patches.md | Patch 分析和适配方式 | 版本升级 |
| 03_Build_Integration.md | 构建配置说明 | 系统集成 |
| 04_Usage_in_OH.md | 依赖关系和使用场景 | 排查依赖 |
| 05_API_Differences.md | OH 特有 API 详解 | 开发参考 |
| 06_Security.md | 安全风险和升级建议 | 安全评估 |
