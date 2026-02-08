# 文档导航

本文档为 `interface/sdk-js` 仓库的工程 Wiki 导航页。

## 阅读路线

### 新人入门路线

```
1. 项目概览 → 了解项目定位与整体结构
   └─ 00_Overview.md

2. API 声明文件 → 理解声明文件组织方式
   └─ 01_API_Declarations.md

3. 构建工具链 → 掌握工具使用与开发
   └─ 02_Build_Tools.md

4. GN 配置 → 理解构建配置与产物
   └─ 03_Build_Configuration.md
```

### 按角色阅读

| 角色 | 推荐阅读顺序 |
|------|-------------|
| SDK 开发者 | Overview → API Declarations → Build Configuration |
| 构建工程师 | Build Configuration → Build Tools → Troubleshooting |
| 安全工程师 | Overview → Security Review → Build Tools |
| 工具开发者 | Build Tools → API Declarations → Build Configuration |

## 完整文档列表

### 核心文档

| 文档 | 说明 | 关键内容 |
|------|------|---------|
| [README](README.md) | 文档说明 | 覆盖范围、更新方式 |
| [00_Overview](00_Overview.md) | 项目概览 | 定位、目录、核心能力 |
| [01_API_Declarations](01_API_Declarations.md) | API 声明 | @ohos.*, @kit.*, @arkts.* |
| [02_Build_Tools](02_Build_Tools.md) | 工具链 | dts_parser, api_check_plugin 等 |
| [03_Build_Configuration](03_Build_Configuration.md) | GN 配置 | BUILD.gn, targets, 产物 |
| [04_Security_Review](04_Security_Review.md) | 安全评审 | 风险清单、修复建议 |
| [05_Troubleshooting](05_Troubleshooting.md) | 常见问题 | 构建问题、调试方法 |

### 附录

| 文档 | 说明 |
|------|------|
| [Callgraphs](appendix/Callgraphs.md) | 关键调用链 |

## 快速跳转

### 按主题跳转

- **API 相关**: [01_API_Declarations](01_API_Declarations.md)
- **构建相关**: [02_Build_Tools](02_Build_Tools.md) → [03_Build_Configuration](03_Build_Configuration.md)
- **安全问题**: [04_Security_Review](04_Security_Review.md)
- **问题排查**: [05_Troubleshooting](05_Troubleshooting.md)

### 按文件类型跳转

- **声明文件**: `api/@ohos.*.d.ts`, `kits/@kit.*.d.ts`
- **构建脚本**: `BUILD.gn`, `*.py`, `*.js`
- **配置文件**: `bundle.json`, `interface_config.gni`

## 版本信息

| 项目 | 值 |
|------|-----|
| 文档版本 | 1.0 |
| 最后更新 | 2026-02-06 |
| 维护者 | SDK 团队 |

## 反馈与贡献

如发现文档错误或需要补充，请：

1. 在仓库提 Issue
2. 或直接提交 PR 修改 `wiki/` 目录
