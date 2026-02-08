# Cellular Call Wiki - 全站导航

## 📖 文档目录

```
cellular_call/
│
├── 📄 README.md                     # 文档说明与更新指南
│
├── 📋 SUMMARY.md                    # 本导航页
│
├── 🏠 01_Overview.md               # 项目概览
│   ├── 项目定位与目标
│   ├── 核心能力
│   ├── 系统依赖
│   └── 特性标志
│
├── 📂 02_Directory_Structure.md    # 目录结构
│   ├── 顶层结构
│   ├── 目录职责说明
│   └── 文件分类索引
│
├── 🏗️ 03_Architecture.md            # 架构说明
│   ├── 三层架构模型
│   ├── 数据流设计
│   ├── 线程模型
│   └── 时序图
│
├── 🔌 04_Interfaces.md             # Inner API 接口
│   ├── IMS Call 接口 (SAID: 4014)
│   ├── Satellite Call 接口 (SAID: 4015)
│   ├── 补充业务接口
│   └── 接口清单表
├── 🔧 05_Inner_API.md              # 内部模块接口
│   ├── 管理层 (Manager)
│   ├── 控制层 (Control)
│   ├── 连接层 (Connection)
│   └── 服务交互层
├── 🎯 06_GN_Build.md               # GN 构建配置
│   ├── 构建入口
│   ├── Targets 清单
│   ├── 条件编译
│   └── 编译产物
├── 🛡️ 07_Security_Review.md       # 安全风险评估
│   ├── 安全风险清单 (5+类)
│   ├── 漏洞触发路径
│   ├── 影响评估
│   └── 修复建议
├── 🎨 08_Internals.md              # 内部实现细节
│   ├── 核心类职责
│   ├── 内部 API 契约
│   ├── 资源生命周期
│   └── 线程模型
└── ❓ 09_Troubleshooting.md        # 常见问题与调试
    ├── 构建问题
    ├── 运行问题
    ├── 调试方法
    └── 定位路径
│
└── 📎 appendix/                    # 附录
    ├── Callgraphs.md              # 关键调用链
    └── Config_Flags.md           # 配置标志说明
```

---

## 🎯 新人阅读路线

### 路线 A：快速概览（30分钟）

```
1️⃣ 01_Overview.md         → 项目定位与能力
2️⃣ 02_Directory_Structure.md → 代码组织结构
3️⃣ 03_Architecture.md    → 核心架构理解
```

### 路线 B：开发接入（1-2小时）

```
1️⃣ 01_Overview.md         → 理解项目背景
2️⃣ 03_Architecture.md    → 掌握架构设计
3️⃣ 04_Interfaces.md      → 查看接口清单
4️⃣ 06_GN_Build.md         → 了解构建配置
5️⃣ 05_Inner_API.md        → 深入模块实现
```

### 路线 C：安全审查（2-3小时）

```
1️⃣ 01_Overview.md         → 了解系统边界
2️⃣ 04_Interfaces.md      → 检查输入验证点
3️⃣ 07_Security_Review.md → 完整安全评估
4️⃣ 05_Inner_API.md       → 分析调用链
5️⃣ 08_Internals.md       → 理解内部实现
```

---

## 📊 模块信息速查

| 属性 | 值 | 参考章节 |
|------|-----|----------|
| 模块名 | `@ohos/cellular_call` | 01_Overview |
| SA ID | 4006 | 03_Architecture |
| 子系统 | telephony | 01_Overview |
| 主要语言 | C++ | 01_Overview |
| ROM 占用 | ~1MB | 01_Overview |
| RAM 占用 | ~650KB | 01_Overview |
| N-API | ❌ 无 | 04_Interfaces |
| 条件编译 | 卫星/RTT/UT | 06_GN_Build |

---

## 🔗 外部链接

- **代码仓库**: https://gitee.com/openharmony/telephony_cellular_call
- **依赖仓库**:
  - [telephony_core_service](https://gitee.com/openharmony/telephony_core_service/blob/master/README.md)
  - [telephony_call_manager](https://gitee.com/openharmony/telephony_call_manager/blob/master/README.md)
- **OpenHarmony 文档**: https://gitee.com/openharmony/docs

---

## 📝 文档更新日志

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本 |

---

## ❓ 快速索引

### 查找特定内容

| 需求 | 前往章节 |
|------|----------|
| 项目做什么的？ | 01_Overview |
| 代码在哪里？ | 02_Directory_Structure |
| 架构怎么设计的？ | 03_Architecture |
| 有哪些 API？ | 04_Interfaces |
| 怎么构建？ | 06_GN_Build |
| 有安全问题吗？ | 07_Security_Review |
| 内部怎么实现的？ | 08_Internals |
| 遇到问题怎么办？ | 09_Troubleshooting |
| 想看调用链？ | appendix/Callgraphs.md |

---

*最后更新：2026-02-06*
