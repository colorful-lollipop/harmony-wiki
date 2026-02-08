# Wiki 导航索引

本文档为 ability_runtime 工程的完整导航索引，提供新人阅读路线和各文档间的交叉引用。

## 文档结构

```
wiki/
├── README.md                    # 文档说明、更新方式
├── SUMMARY.md                   # 本导航索引
├── index.md                     # 项目首页/概览
├── 01_Project_Overview.md       # 项目定位、边界、核心能力
├── 02_Directory_Structure.md   # 目录结构与模块职责
├── 03_Architecture.md          # 架构说明（组件图、数据流、线程模型）
├── 04_NAPI_Reference.md        # N-API 接口参考（JS API 面）
├── 05_Inner_API.md             # Inner API（内部组件间接口）
├── 06_GN_Targets.md            # GN Targets 梳理
├── 07_Build_Artifacts.md       # 编译产物说明
├── 08_Security_Review.md       # 安全风险评审
├── 09_Troubleshooting.md       # 常见问题与定位路径
└── appendix/
    ├── Callgraphs.md           # 关键调用链
    └── Config_Flags.md         # 关键宏/特性开关
```

## 新人阅读路线

### 路线一：快速入门（30分钟）

建议顺序：先理解整体，再深入细节。

1. **必读** → `index.md`（5分钟）
   - 了解项目是什么、能做什么
   - 查看架构总览图

2. **必读** → `01_Project_Overview.md`（10分钟）
   - 理解核心能力和应用模型
   - 了解关键概念和术语

3. **可选** → `02_Directory_Structure.md`（10分钟）
   - 熟悉代码组织结构
   - 知道核心模块在哪里

4. **可选** → `03_Architecture.md`（5分钟）
   - 了解模块间如何协作

### 路线二：API 开发（1小时）

面向需要开发Ability/Extension的开发者。

1. → `index.md` + `01_Project_Overview.md`（15分钟）
2. → `04_NAPI_Reference.md`（30分钟）
   - 查找可用的 JS API
   - 理解参数校验和错误码

3. → `05_Inner_API.md`（15分钟）
   - 了解内部模块接口
   - 理解稳定性标注

### 路线三：框架开发（2小时）

面向需要修改框架核心的开发者。

1. → 全部文档按顺序阅读
2. → 重点关注 `03_Architecture.md`（架构）
3. → 重点关注 `06_GN_Targets.md` + `07_Build_Artifacts.md`（构建）
4. → 重点关注 `08_Security_Review.md`（安全）

### 路线四：问题定位（30分钟）

面向需要调试或定位问题的开发者。

1. → `09_Troubleshooting.md`（20分钟）
   - 查找常见问题解决方案
   - 了解调试工具和日志位置

2. → `appendix/Callgraphs.md`（10分钟）
   - 理解关键调用链路

## 快速跳转

### 按功能查找

| 功能需求 | 跳转文档 | 相关章节 |
|---------|---------|---------|
| 理解项目是什么 | `index.md` | 全部 |
| 启动/停止 Ability | `04_NAPI_Reference.md` | ability 模块 |
| 管理应用生命周期 | `04_NAPI_Reference.md` | application 模块 |
| IPC 通信 | `03_Architecture.md` | IPC 通信机制 |
| 权限校验 | `08_Security_Review.md` | 权限相关章节 |
| 构建项目 | `06_GN_Targets.md` | 全部 |
| 问题定位 | `09_Troubleshooting.md` | 全部 |

### 按模块查找

| 模块 | 跳转文档 | 相关章节 |
|------|---------|---------|
| AbilityManagerService | `03_Architecture.md` | 系统服务 |
| AppManagerService | `03_Architecture.md` | 系统服务 |
| N-API 绑定 | `04_NAPI_Reference.md` | 全部 |
| 权限验证 | `08_Security_Review.md` | 权限模块 |
| 构建系统 | `06_GN_Targets.md` | 全部 |

## 术语表

| 术语 | 说明 | 相关文档 |
|------|------|---------|
| Ability | OpenHarmony 应用组件 | `01_Project_Overview.md` |
| UIAbility | 页面 Ability（Stage模型） | `01_Project_Overview.md` |
| ExtensionAbility | 扩展 Ability | `01_Project_Overview.md` |
| N-API | Node.js API，JS 调用原生接口 | `04_NAPI_Reference.md` |
| SA | System Ability，系统服务 | `03_Architecture.md` |
| IPC | 进程间通信 | `03_Architecture.md` |
| GN | Generate Ninja，构建系统 | `06_GN_Targets.md` |

## 版本兼容性

| 文档版本 | 适用代码版本 | 最后验证 |
|---------|------------|---------|
| v1.0 | OpenHarmony 5.0 | 2026-02-06 |

## 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|---------|
| 2026-02-06 | v1.0 | 初始版本，基于代码证据生成 |
