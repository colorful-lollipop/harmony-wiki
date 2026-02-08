# Bootstrap_Lite 工程 Wiki

## 文档覆盖范围

本 Wiki 文档旨在帮助开发者快速理解 OpenHarmony `bootstrap_lite` 启动引导组件的架构、设计与实现。

### 已覆盖内容
- 项目概述与定位
- 系统架构与组件关系
- 初始化机制详解（链接器脚本段 + SAMGR）
- GN 构建配置与编译产物
- 安全风险分析

### 未覆盖内容
- 测试用例与测试覆盖（按规范不引用测试代码）
- 其他子系统集成细节（仅描述本组件边界）
- 历史版本变更记录

## 阅读路线

**推荐阅读顺序**：

1. **[README.md](README.md)** - 本文档，说明文档覆盖范围
2. **[SUMMARY.md](SUMMARY.md)** - 全站导航与阅读路线
3. **[00_Overview.md](00_Overview.md)** - 项目定位、核心能力、运行环境
4. **[01_Architecture.md](01_Architecture.md)** - 组件图、数据流、初始化序列
5. **[02_Build.md](02_Build.md)** - GN Targets、编译产物、运行时加载
6. **[03_Initialization.md](03_Initialization.md)** - 初始化机制详解
7. **[04_Security.md](04_Security.md)** - 安全风险评审

## 代码证据引用规范

本文档所有关键结论均基于代码证据：

```
证据格式:
- 路径: `services/source/bootstrap_service.c:38`
- 符号: `SAMGR_GetInstance()->RegisterService()`
- 代码片段: 必要时引用关键代码
```

## 更新方式

当代码变更时，需同步更新 Wiki：

1. **N-API 变更**: 更新 `02_Build.md` 的 NDK 头文件部分
2. **初始化流程变更**: 更新 `03_Initialization.md` 的调用序列
3. **构建配置变更**: 更新 `02_Build.md` 的 target 列表
4. **安全相关变更**: 更新 `04_Security.md` 的风险清单

## 文档信息

| 属性 | 值 |
|------|-----|
| 组件名 | bootstrap_lite |
| 子系统 | startup |
| 版本 | 4.0.2 |
| 最后更新 | 2024-02 |
| 维护者 | OpenHarmony Community |

## 相关链接

- **代码仓库**: https://gitee.com/openharmony/startup_bootstrap_lite
- **官方文档**: https://gitee.com/openharmony/docs
- **Issue 反馈**: https://gitee.com/openharmony/startup_bootstrap_lite/issues
