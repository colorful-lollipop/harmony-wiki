# ability_lite Wiki

## 简介

本 Wiki 是 OpenHarmony ability_lite 组件的工程文档，旨在帮助开发者快速理解项目架构、接口定义、构建系统和安全风险。

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 3.1
- **适用系统**: mini, small
- **仓库路径**: foundation/ability/ability_lite

## 覆盖范围

### 已覆盖内容
- 项目概览与核心概念
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- 对外 Native API 与 N-API (JS API)
- 内部 API 与模块接口
- GN 构建目标与编译产物
- 安全风险评审

### 未覆盖内容
- 单元测试实现细节（test/ 目录内容）
- 具体 UI 组件实现（依赖 ui_lite）
- 分布式能力详细实现（dmsfwk_lite）

## 阅读指南

1. **新人入门**: 从 [项目概览](00_Overview.md) → [目录结构](01_Directory_Structure.md) → [架构说明](02_Architecture.md)
2. **应用开发**: 查看 [对外 Native API](03_Native_API.md) 和 [N-API/JS API](04_NAPI_JS_API.md)
3. **系统开发**: 参考 [内部 API](05_Inner_API.md) 和 [GN 构建](06_GN_Targets.md)
4. **安全审计**: 阅读 [安全评估](08_Security_Assessment.md)

## 更新方式

本文档基于代码生成，当代码变更时需要：
1. 更新 `wiki/_work/NOTES.md` 中的事实记录
2. 修改对应章节文档
3. 同步更新 `SUMMARY.md` 导航

## 文档结构

```
wiki/
├── README.md              # 本文件
├── SUMMARY.md             # 全站导航
├── 00_Overview.md         # 项目概览
├── 01_Directory_Structure.md  # 目录结构
├── 02_Architecture.md     # 架构说明
├── 03_Native_API.md       # 对外 Native API
├── 04_NAPI_JS_API.md      # N-API/JS API
├── 05_Inner_API.md        # 内部 API
├── 06_GN_Targets.md       # GN 构建目标
├── 07_Build_Artifacts.md  # 编译产物
├── 08_Security_Assessment.md  # 安全评估
├── 09_Troubleshooting.md  # 常见问题
└── _work/                 # 工作目录
    ├── NOTES.md           # 事实记录
    └── PLAN.md            # 任务计划
```

## 术语表

| 术语 | 说明 |
|------|------|
| Ability | 应用能力，系统调度的最小单元 |
| Page Ability | 带 UI 的 Ability |
| Service Ability | 后台服务 Ability，无 UI |
| AbilitySlice | Page Ability 的页面片段 |
| Want | Ability 间传递的意图信息载体 |
| AMS | Ability Manager Service，能力管理服务 |
| SAMGR | System Ability Manager，系统服务管理框架 |
| N-API | Node-API，JavaScript 绑定接口 |

## 参考资料

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Ability 框架说明](https://gitee.com/openharmony/docs/blob/master/en/readme/ability.md)
- [bundle.json](bundle.json) - 组件配置
- [ability_lite.gni](ability_lite.gni) - GN 变量定义
