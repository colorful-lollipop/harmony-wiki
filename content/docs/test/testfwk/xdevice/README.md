# XDevice Wiki 文档

## 文档概述

本文档是 OpenHarmony 测试框架核心组件 **XDevice** 的完整工程 Wiki。

### 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目定位与核心能力 | ✅ 完成 | XDevice 是 OpenHarmony 测试框架的核心模块，提供用例执行所依赖的相关服务 |
| 目录结构与模块职责 | ✅ 完成 | 10 个核心模块的详细职责说明 |
| 架构说明 | ✅ 完成 | 组件图、数据流、线程模型、关键时序 |
| 对外 API | ✅ 完成 | Python 模块导出 API（173个公共接口） |
| 内部 API | ✅ 完成 | 模块接口、依赖方向、稳定性标注 |
| GN Targets | ✅ 完成 | 构建配置与编译产物说明 |
| 配置说明 | ✅ 完成 | XML/JSON 配置解析机制 |
| 安全风险评审 | ✅ 完成 | 6 个高危风险、5 个中危风险、2 个低危风险 |
| 使用指南 | ✅ 完成 | 安装、配置、命令参考 |

### 不包含内容

- 测试相关内容（test/ tests/ unittest/ 等）
- N-API（XDevice 为纯 Python 项目）

---

## 快速导航

### 新人阅读顺序（推荐）

```
1. README.md (本文档)
2. 00_Overview.md (项目概览)
3. 01_Directory_Structure.md (目录结构)
4. 02_Architecture.md (架构说明)
5. 05_Configuration.md (配置说明)
6. 07_Usage.md (使用指南)
```

### 按需求查找

| 需求 | 文档 |
|------|------|
| 理解项目结构 | 01_Directory_Structure.md |
| 理解架构设计 | 02_Architecture.md |
| 了解模块详情 | 03_Modules.md |
| 配置测试环境 | 05_Configuration.md |
| 执行测试用例 | 07_Usage.md |
| 安全合规检查 | 06_Security.md |
| 构建与打包 | 04_Build.md |
| 排查问题 | 08_Troubleshooting.md |

---

## 版本信息

| 项目 | 版本 |
|------|------|
| XDevice | 5.0.6.100 |
| bundle.json | 2.30.0 |
| 适配系统 | mini, small, standard |
| Python 要求 | >= 3.7.5 |

---

## 更新日志

| 日期 | 更新内容 | 更新人 |
|------|---------|--------|
| 2026-02-06 | 初始 Wiki 生成 | Sisyphus Agent |

---

## 如何贡献

本文档基于代码自动生成。如需更新：

1. **修改代码后**：运行 Wiki 生成脚本更新文档
2. **更新配置说明**：修改 `config/user_config.xml` 后更新 `05_Configuration.md`
3. **更新架构设计**：修改核心模块后更新 `02_Architecture.md`
4. **更新安全评审**：发现新安全问题后更新 `06_Security.md`

---

## 相关链接

- [OpenHarmony Testing Subsystem](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E6%B5%8B%E8%AF%95%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- [test_developertest 仓库](https://gitee.com/openharmony/test_developertest/blob/master/README_zh.md)
