# 智能语音框架 Wiki 导航

> 本文档为 Intelligent Voice Framework Wiki 的全局导航，提供文档索引和新人阅读路线。

---

## 文档结构概览

```
wiki/
├── README.md                    # 项目说明、更新方式
├── SUMMARY.md                   # 本导航文件
├── 01_Overview.md              # 项目概览、定位、概念
├── 02_Architecture.md          # 架构设计、组件图、数据流
├── 03_Directory_Structure.md   # 目录结构、模块职责
├── 04_NAPI_Reference.md        # N-API 接口详解、API清单
├── 05_Inner_API.md             # 内部 API、模块接口
├── 06_Build_System.md          # GN 构建系统、Targets
├── 07_Artifacts.md             # 编译产物、安装路径
├── 08_Security_Review.md       # 安全评审、风险分析
├── _work/                      # 工作区（生成过程记录）
│   ├── NOTES.md                # 事实记录、代码证据
│   └── PLAN.md                # 任务计划、进度跟踪
└── appendix/
    ├── Callgraphs.md           # 关键调用链
    └── Config_Flags.md        # 配置开关、Feature Flags
```

---

## 新人阅读路线（推荐）

### 路线一：快速入门（30分钟）

| 顺序 | 文档 | 目的 |
|------|------|------|
| 1 | `README.md` | 了解 Wiki 覆盖范围和使用方式 |
| 2 | `01_Overview.md` | 理解项目定位和核心能力 |
| 3 | `03_Directory_Structure.md` | 熟悉代码目录结构 |
| 4 | `04_NAPI_Reference.md` | 掌握 API 使用方法 |

### 路线二：深入开发（2小时）

| 顺序 | 文档 | 目的 |
|------|------|------|
| 1 | `README.md` | 项目整体认知 |
| 2 | `01_Overview.md` | 业务场景和技术选型 |
| 3 | `02_Architecture.md` | 理解架构设计和数据流 |
| 4 | `03_Directory_Structure.md` | 熟悉模块边界 |
| 5 | `04_NAPI_Reference.md` | API 详细用法 |
| 6 | `05_Inner_API.md` | 内部模块接口 |
| 7 | `06_Build_System.md` | 构建配置 |
| 8 | `appendix/Callgraphs.md` | 关键调用链 |

### 路线三：安全评估（1小时）

| 顺序 | 文档 | 目的 |
|------|------|------|
| 1 | `01_Overview.md` | 理解攻击面 |
| 2 | `02_Architecture.md` | 信任边界分析 |
| 3 | `08_Security_Review.md` | 详细安全评审 |
| 4 | `appendix/Callgraphs.md` | 调用链安全检查 |

---

## API 参考快速导航

### 语音注册引擎 API

| API | 说明 | 章节 |
|-----|------|------|
| `createEnrollIntelligentVoiceEngine()` | 创建注册引擎 | 4.2.1 |
| `init()` | 初始化引擎 | 4.2.2 |
| `enrollForResult()` | 执行注册 | 4.2.3 |
| `stop()` | 停止注册 | 4.2.4 |
| `commit()` | 提交注册数据 | 4.2.5 |
| `setWakeupHapInfo()` | 设置应用信息 | 4.2.6 |
| `setSensibility()` | 设置灵敏度 | 4.2.7 |
| `release()` | 释放引擎 | 4.2.8 |

### 语音唤醒引擎 API

| API | 说明 | 章节 |
|-----|------|------|
| `createWakeupIntelligentVoiceEngine()` | 创建唤醒引擎 | 4.3.1 |
| `setWakeupHapInfo()` | 设置应用信息 | 4.3.2 |
| `setSensibility()` | 设置灵敏度 | 4.3.3 |
| `on('wakeupIntelligentVoiceEvent')` | 订阅唤醒事件 | 4.3.4 |
| `release()` | 释放引擎 | 4.3.5 |

### 管理器 API

| API | 说明 | 章节 |
|-----|------|------|
| `getIntelligentVoiceManager()` | 获取管理器 | 4.1.1 |
| `getWakeupManager()` | 获取唤醒管理器 | 4.1.2 |
| `getCapabilityInfo()` | 查询能力信息 | 4.1.3 |
| `on/off('serviceChange')` | 服务状态监听 | 4.1.4 |

---

## 构建相关导航

| 需求 | 文档 | 关键章节 |
|------|------|---------|
| 了解 Targets | `06_Build_System.md` | 全部 |
| 了解产物 | `07_Artifacts.md` | 全部 |
| 配置编译选项 | `appendix/Config_Flags.md` | 全部 |
| 添加新模块 | `06_Build_System.md` | 6.4 |

---

## 安全相关导航

| 需求 | 文档 | 关键章节 |
|------|------|---------|
| 权限声明 | `08_Security_Review.md` | 3.1 |
| 访问控制 | `08_Security_Review.md` | 3.2 |
| 风险清单 | `08_Security_Review.md` | 4 |
| 修复建议 | `08_Security_Review.md` | 5 |

---

## 版本与兼容性

| Wiki 版本 | 对应框架版本 | 更新日期 |
|-----------|--------------|----------|
| 4.0 | 4.0 | 2026-02-06 |

---

## 符号约定

| 符号 | 含义 |
|------|------|
| `code` | 代码片段、文件路径、API 名称 |
| **粗体** | 重要概念、模块名称 |
| [链接]() | 文档内部引用 |
| `TODO` | 待确认/待补充内容 |
| `NOTE` | 注意事项 |

---

## 贡献指南

### 文档更新流程

1. 在 `wiki/_work/NOTES.md` 中记录发现
2. 更新对应的 Markdown 文件
3. 确保链接正确跳转
4. 更新 `SUMMARY.md`（如有新增）

### 代码证据要求

所有关键结论必须包含：
- 文件路径（必要时含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段
