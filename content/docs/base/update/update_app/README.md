# OpenHarmony update_app 工程 Wiki

> 本 Wiki 为 update_app 模块提供完整的工程文档，覆盖架构设计、API 参考、构建系统和安全评审。

## 📋 文档概览

### 覆盖范围

本 Wiki 涵盖 update_app 模块的核心工程文档：

| 分类 | 内容 | 状态 |
|------|------|------|
| 项目概述 | 定位、能力、运行环境 | ✅ 完成 |
| 架构设计 | 组件图、数据流、时序 | ✅ 完成 |
| N-API 参考 | JS API 清单、参数、错误码 | ✅ 完成 |
| 内部 API | 模块接口、依赖关系 | ✅ 完成 |
| 构建系统 | GN targets、编译产物 | ✅ 完成 |
| 安全评审 | 攻击面、风险清单 | ✅ 完成 |
| 故障排查 | 常见问题、定位路径 | ✅ 完成 |

### 未覆盖范围

- 测试代码细节（遵循约束不引用测试）
- 第三方依赖内部实现
- 历史版本变更记录
- 性能基准测试数据

## 🚀 快速开始

### 新人阅读路线

```
1. wiki/index.md          → 项目概览
2. wiki/01_Project_Overview.md → 详细定位与能力
3. wiki/02_Architecture.md    → 架构设计与数据流
4. wiki/03_N-API_Reference.md → API 使用方法
5. wiki/05_Build_System.md    → 构建配置与产物
```

### 关键链接

- [API 清单](./03_N-API_Reference.md)
- [架构图示](./02_Architecture.md)
- [安全指南](./07_Security_Review.md)
- [故障排查](./08_Troubleshooting.md)

## 📖 目录结构

```
wiki/
├── README.md              ← 本文档
├── SUMMARY.md             ← 全站导航
├── index.md               ← 首页/概览
├── 01_Project_Overview.md ← 项目定位与边界
├── 02_Architecture.md     ← 架构设计
├── 03_N-API_Reference.md  ← N-API 接口文档
├── 04_Inner_API.md        ← 内部模块 API
├── 05_Build_System.md     ← GN 构建系统
├── 06_Build_Artifacts.md  ← 编译产物清单
├── 07_Security_Review.md  ← 安全风险评审
├── 08_Troubleshooting.md  ← 故障排查指南
└── appendix/
    ├── Callgraphs.md      ← 关键调用链
    └── Config_Flags.md    ← 配置开关
```

## 🔧 使用说明

### 文档更新

当代码变更涉及以下内容时，需同步更新 Wiki：

| 变更类型 | 需更新文档 |
|----------|-----------|
| 新增/修改 N-API | `03_N-API_Reference.md` |
| 新增模块/服务 | `02_Architecture.md`, `04_Inner_API.md` |
| 修改构建配置 | `05_Build_System.md` |
| 新增安全控制 | `07_Security_Review.md` |
| 新增编译产物 | `06_Build_Artifacts.md` |

### 文档规范

1. **语言**: 默认中文，代码/日志/错误码可保留英文
2. **引用格式**: `[文件:行号]` 标注代码证据
3. **术语**: 使用项目规范术语，首次出现需标注英文原文
4. **图表**: 使用 Mermaid 语法，支持流程图、时序图

### 示例

```markdown
## API 注册

模块通过 `NAPI_MODULE` 宏注册入口：
```cpp
// src/napi/native/init.cpp:25
NAPI_MODULE(update, Register, nullptr)
```
```

## 📝 版本信息

| 项目 | 值 |
|------|-----|
| 文档生成时间 | 2026-02-06 |
| 对应代码版本 | 当前 HEAD |
| 最后更新 | 2026-02-06 |
| 维护者 | update_app 团队 |

## 贡献指南

欢迎为本 Wiki 贡献内容：

1. 在 `wiki/` 目录下创建或修改 `.md` 文件
2. 确保关键结论有代码证据（路径+符号）
3. 更新 `SUMMARY.md` 添加新页面链接
4. 运行 `docs/lint.sh` 检查格式

---

*本 Wiki 由 Sisyphus 自动生成*
