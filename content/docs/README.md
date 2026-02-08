# OpenHarmony 文档仓库工程 Wiki - 任务计划

> 本 Wiki 为 OpenHarmony 文档仓库（docs.openharmony.cn）的工程文档，描述仓库结构、文档组织、贡献流程等。

## 项目概览

| 项目 | 值 |
|------|-----|
| 仓库类型 | 文档仓库 |
| 主要内容 | OpenHarmony 开发者文档（中英文） |
| 许可证 | CC BY 4.0 |
| 主分支 | master |
| 最新版本文档 | OpenHarmony 6.0 Release |

## 目录结构

```
docs/
├── wiki/                    # 本 Wiki（工程文档）
│   ├── _work/              # 工作区
│   │   ├── NOTES.md        # 事实记录
│   │   └── PLAN.md         # 本任务计划
│   ├── README.md           # Wiki 说明
│   ├── SUMMARY.md          # 全站导航
│   ├── 01_Overview.md      # 项目概览
│   ├── 02_Structure.md     # 目录结构
│   ├── 03_Content.md       # 文档内容体系
│   ├── 04_Build.md         # 构建与发布
│   ├── 05_Contribute.md    # 贡献指南
│   └── appendix/           # 附录
│       ├── History.md      # 版本历史
│       └── Templates.md    # 文档模板
├── zh-cn/                  # 中文文档
├── en/                     # 英文文档
├── docker/                 # Docker 环境
├── .gitcode/               # GitCode 配置
├── README.md               # 项目说明
├── LICENSE                 # 许可证
└── DCO.txt                # 开发者原创声明
```

## 文档内容体系

### 中文文档 (zh-cn/)

| 模块 | 描述 | 文件数(约) |
|------|------|-----------|
| readme | 文档首页导航 | 1 |
| application-dev | 应用开发指南 | 50 |
| contribute | 贡献指南 | 43 |
| design | 设计规范 | 9 |
| device-dev | 设备开发指南 | 20 |
| release-notes | 发布说明 | 67 |
| third-party-cases | 第三方案例 | 64 |
| third-party-components | 第三方组件 | 4 |
| figures | 图片资源 | 3 |

### 英文文档 (en/)

| 模块 | 描述 | 文件数(约) |
|------|------|-----------|
| readme | 文档首页导航 | 1 |
| application-dev | 应用开发指南 | 51 |
| contribute | 贡献指南 | 43 |
| design | 设计规范 | 9 |
| device-dev | 设备开发指南 | 19 |
| release-notes | 发布说明 | 65 |
| third-party-components | 第三方组件 | 4 |

## 版本发布历史

### 当前维护版本

| 版本 | API Level | 状态 |
|------|-----------|------|
| OpenHarmony 6.0 Release | 20 | ✅ 最新 |
| OpenHarmony 5.1.0 Release | 18 | ✅ 维护中 |
| OpenHarmony 5.0.3 | 15 | ✅ 维护中 |
| OpenHarmony 5.0.2 | 14 | ✅ 维护中 |
| OpenHarmony 5.0.1 | 13 | ✅ 维护中 |
| OpenHarmony 5.0.0 Release | 12 | ✅ 维护中 |

### 已停止维护

| 版本 | API Level | 状态 |
|------|-----------|------|
| OpenHarmony 4.1 Release | 11 | ❌ 已停止 |
| OpenHarmony 4.0 Release | 10 | ❌ 已停止 |
| OpenHarmony 3.2 Release | 9 | ❌ 已停止 |
| OpenHarmony 3.1 Release | - | ❌ 已停止 |
| OpenHarmony 3.0 LTS | - | ❌ 已停止 |
| OpenHarmony-1.0.1-Release | - | ❌ 已停止 |

## 贡献方式

1. **文档评价** - 对现有文档进行评价
2. **简单更改** - 进行简单修改
3. **反馈问题** - 反馈文档质量问题
4. **原创内容** - 贡献原创内容

## 贡献渠道

- **邮件列表**: docs@openharmony.io
- **Zulip 群组**: documentation_sig

## 维护策略

各版本维护策略参考：[release-management](https://gitee.com/openharmony/release-management)

## 扩展阅读

- [OpenHarmony 官网](https://www.openharmony.cn/)
- [OpenHarmony 代码仓库](https://gitee.com/openharmony)
- [贡献指南](zh-cn/contribute/参与贡献.md)
- [文档贡献说明](zh-cn/contribute/贡献文档.md)

---

*文档生成时间：2026-02-06*
*基于仓库版本：master 分支*
