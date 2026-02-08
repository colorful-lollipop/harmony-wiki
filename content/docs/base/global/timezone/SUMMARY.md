# 文档导航

本文档为 OpenHarmony `base/global/timezone` 模块的工程 Wiki 导航页。

## 新人阅读路线

建议按以下顺序阅读：

1. **[00_Overview.md](./00_Overview.md)** - 项目概览
   - 了解模块定位、核心能力、依赖关系
   - 适合首次接触本模块的开发者

2. **[01_Architecture.md](./01_Architecture.md)** - 架构说明
   - 深入理解组件结构、数据流
   - 适合需要修改核心逻辑的开发者

3. **[02_Build.md](./02_Build.md)** - 构建指南
   - 掌握 GN 构建配置、编译产物
   - 适合需要构建或部署的开发者

4. **[04_Usage.md](./04_Usage.md)** - 使用指南
   - 了解下载、编译、部署流程
   - 适合需要更新时区数据的运维人员

5. **[03_Security.md](./03_Security.md)** - 安全评审
   - 查看安全风险分析
   - 适合安全审计和代码审查

## 目录结构

```
wiki/
├── README.md              # 文档说明（本文档）
├── SUMMARY.md            # 导航页（本文档）
├── 00_Overview.md        # 项目概览
├── 01_Architecture.md   # 架构说明
├── 02_Build.md           # 构建指南
├── 03_Security.md        # 安全评审
└── 04_Usage.md          # 使用指南
```

## 快速索引

### 按功能查找

| 功能 | 章节 | 关键文件 |
|------|------|----------|
| 时区数据下载 | [04_Usage.md](./04_Usage.md) | `tool/update_tool/download_iana.py` |
| 时区数据编译 | [04_Usage.md](./04_Usage.md) | `tool/compile_tool/compile.sh` |
| 构建配置 | [02_Build.md](./02_Build.md) | `data/BUILD.gn` |
| 安全风险 | [03_Security.md](./03_Security.md) | - |

### 按角色查找

| 角色 | 推荐阅读 |
|------|----------|
| 新入职开发者 | Overview → Architecture → Usage |
| 构建工程师 | Build → Usage |
| 安全审计 | Security → Architecture |
| 运维人员 | Usage → Build |

## 版本信息

- **当前 Wiki 版本**: 1.0.0
- **最后更新**: 2024-02-06
- **兼容模块版本**: 1.0.0+
