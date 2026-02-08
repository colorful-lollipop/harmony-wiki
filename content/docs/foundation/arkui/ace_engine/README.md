# Ace Engine Wiki 文档

> **文档版本**: v1.0  
> **更新时间**: 2026-02-06  
> **源码版本**: OpenHarmony ace_engine (master 分支)

---

## 📚 文档概述

本文档是 OpenHarmony **ace_engine** 项目的工程 Wiki，旨在帮助开发者快速理解项目架构、N-API 接口、构建系统和安全风险。

### 覆盖范围

| 文档 | 内容 |
|------|------|
| [README](README.md) | 本文档，说明文档覆盖范围和更新方式 |
| [SUMMARY](SUMMARY.md) | 全站导航和新人阅读顺序 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、关键概念 |
| [01_Architecture](01_Architecture.md) | 架构图、数据流、线程模型 |
| [02_NAPI](02_NAPI.md) | N-API 接口清单和调用链 |
| [03_Build](03_Build.md) | GN Targets 和编译产物 |
| [04_Security](04_Security.md) | 安全风险评审 |
| [05_FAQ](05_FAQ.md) | 常见问题与调试指南 |

### 未覆盖范围

- **测试相关内容**：`test/` 目录不作为文档证据来源
- **历史变更记录**：仅记录当前代码状态，不追溯历史
- **详细 API 文档**：仅覆盖关键 N-API 接口，完整 API 参看 SDK 文档

---

## 📖 阅读路线（新人推荐）

```
1. 先读 00_Overview → 了解项目定位和核心能力
2. 读 01_Architecture → 理解整体架构和数据流
3. 根据需要查看 02_NAPI → 了解接口细节
4. 如需构建/编译 → 查看 03_Build
5. 安全评估 → 查看 04_Security
6. 遇到问题 → 查看 05_FAQ
```

---

## 🔄 文档更新方式

### 何时需要更新 Wiki

| 变更类型 | 是否需要更新 |
|---------|------------|
| 新增/删除 N-API 接口 | ✅ 必须更新 02_NAPI |
| 修改 GN targets | ✅ 必须更新 03_Build |
| 架构变更（新增模块、修改数据流） | ✅ 必须更新 01_Architecture |
| 新增安全攻击面 | ✅ 必须更新 04_Security |
| 修复已知问题/FAQ | ✅ 必须更新 05_FAQ |
| 仅修改测试代码 | ❌ 不需要 |

### 更新流程

1. **确认变更**：在 `wiki/_work/NOTES.md` 中记录发现
2. **更新对应文档**：修改相应 Markdown 文件
3. **更新导航**：如新增文档，需更新 `SUMMARY.md`
4. **验证链接**：确保所有内部链接有效

---

## 📁 Wiki 目录结构

```
wiki/
├── README.md              # 本文档，覆盖范围和更新方式
├── SUMMARY.md             # 全站导航
├── 00_Overview.md         # 项目概览
├── 01_Architecture.md     # 架构说明
├── 02_NAPI.md            # N-API 接口
├── 03_Build.md           # 构建系统
├── 04_Security.md        # 安全评审
├── 05_FAQ.md             # 常见问题
└── _work/
    ├── NOTES.md           # 事实记录（仅供开发维护）
    └── PLAN.md            # 任务计划（仅供开发维护）
```

---

## 🔗 相关资源

- **源码仓库**: `OpenHarmony/foundation/arkui/ace_engine`
- **官方文档**: [ArkUI 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/ui/README.md)
- **SDK API**: [ArkUI 组件接口](https://gitee.com/openharmony/interface_sdk-js/tree/master/api/arkui)
