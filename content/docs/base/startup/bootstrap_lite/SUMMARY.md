# Bootstrap_Lite Wiki - 导航与阅读路线

## 文档结构

```
bootstrap_lite Wiki
├── README.md                    # 文档说明、覆盖范围、更新方式
├── SUMMARY.md                   # 本导航页
├── 00_Overview.md              # 项目定位、核心能力、运行环境
├── 01_Architecture.md           # 组件架构、初始化序列、数据流
├── 02_Build.md                  # GN 构建配置、编译产物
├── 03_Initialization.md        # 初始化机制详解
├── 03_CodeMap.md               # 代码地图、文件导航、符号索引
├── 04_Security.md               # 安全风险评审
└── _work/                        # 工作区
    ├── ASSESSMENT.md           # 项目评估报告
    ├── NOTES.md                # 事实记录
    └── PLAN.md                 # 任务计划
```

## 新人阅读路线

### 路线 A：快速概览（5分钟）
1. → `00_Overview.md` - 了解组件定位
2. → `01_Architecture.md` - 查看架构图

### 路线 B：开发者入门（15分钟）
1. → `00_Overview.md` - 项目定位
2. → `01_Architecture.md` - 架构设计
3. → `03_Initialization.md` - 初始化机制
4. → `02_Build.md` - 构建配置

### 路线 C：深度理解（30分钟）
1. → `00_Overview.md` - 项目定位
2. → `01_Architecture.md` - 架构设计
3. → `03_Initialization.md` - 初始化机制（详细）
4. → `02_Build.md` - 构建与产物
5. → `04_Security.md` - 安全考量
6. → 源码阅读：`services/source/`

## 按角色索引

### 系统开发者
- `01_Architecture.md` - 理解组件在系统中的位置
- `03_Initialization.md` - 了解启动流程

### 应用开发者
- `02_Build.md` - 了解如何集成组件

### 安全工程师
- `04_Security.md` - 安全风险评审

## 快速跳转

| 主题 | 文档章节 | 关键证据 |
|------|----------|----------|
| 项目定位 | `00_Overview.md` | bundle.json |
| 架构设计 | `01_Architecture.md` | bootstrap_service.c |
| 构建配置 | `02_Build.md` | BUILD.gn |
| 初始化机制 | `03_Initialization.md` | core_main.h, bootstrap_service.h |
| 安全风险 | `04_Security.md` | 威胁模型 |

## 版本信息

| 属性 | 值 |
|------|-----|
| 当前版本 | 4.0.2 |
| 最后更新 | 2024-02 |
| 兼容性 | mini, small 系统 |
