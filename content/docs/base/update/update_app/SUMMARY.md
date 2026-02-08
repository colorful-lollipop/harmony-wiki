# OpenHarmony update_app Wiki 导航

> 本文档提供全站导航链接，按新人阅读顺序组织。

## 📖 文档阅读顺序（推荐）

```
入门路径
├── 1. index.md              → 首页，快速了解项目
├── 2. 01_Project_Overview.md → 项目定位、边界、核心能力
└── 3. 02_Architecture.md     → 整体架构、组件图、数据流
```

```
进阶路径
├── 4. 03_N-API_Reference.md  → JS API 详细用法
├── 5. 04_Inner_API.md        → 内部模块接口
├── 6. 05_Build_System.md     → GN 构建配置
└── 7. 06_Build_Artifacts.md  → 编译产物与安装
```

```
专项深入
├── 8. 07_Security_Review.md  → 安全风险分析
├── 9. 08_Troubleshooting.md  → 故障排查指南
└── appendix/
    ├── Callgraphs.md         → 关键调用链图
    └── Config_Flags.md       → 配置开关参考
```

---

## 📋 完整目录

### 入门文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [index.md](./index.md) | 首页 | 项目概述、快速链接 |
| [01_Project_Overview.md](./01_Project_Overview.md) | 项目定位 | 边界、能力、运行环境 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计 | 组件图、数据流、时序图 |

### API 文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [03_N-API_Reference.md](./03_N-API_Reference.md) | N-API 参考 | JS API 清单、参数、错误码 |
| [04_Inner_API.md](./04_Inner_API.md) | 内部 API | 模块接口、依赖关系 |

### 构建文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [05_Build_System.md](./05_Build_System.md) | 构建系统 | GN targets、配置选项 |
| [06_Build_Artifacts.md](./06_Build_Artifacts.md) | 编译产物 | .so/.a/.hap 清单、路径 |

### 专项文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [07_Security_Review.md](./07_Security_Review.md) | 安全评审 | 攻击面、风险清单、修复建议 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 故障排查 | 常见问题、调试方法 |

### 附录

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 调用链 | 入口→核心逻辑调用图 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 配置开关 | 宏定义、feature flags |

---

## 🔍 快速索引

### 按功能索引

| 功能 | 文档位置 |
|------|----------|
| 如何调用更新接口 | [03_N-API_Reference.md](./03_N-API_Reference.md) |
| 理解模块依赖 | [04_Inner_API.md](./04_Inner_API.md) |
| 编译项目 | [05_Build_System.md](./05_Build_System.md) |
| 安全问题定位 | [07_Security_Review.md](./07_Security_Review.md) |
| 构建失败排查 | [08_Troubleshooting.md](./08_Troubleshooting.md) |

### 按代码位置索引

| 代码位置 | 文档位置 |
|----------|----------|
| src/napi/ | [03_N-API_Reference.md](./03_N-API_Reference.md) |
| src/base/update/ | [04_Inner_API.md](./04_Inner_API.md) |
| BUILD.gn | [05_Build_System.md](./05_Build_System.md) |
| interfaces/ | [03_N-API_Reference.md](./03_N-API_Reference.md) |

---

## 📊 文档统计

| 分类 | 数量 |
|------|------|
| 主文档 | 9 |
| 附录 | 2 |
| 总计 | 11 |

---

## 🔗 相关链接

- [项目 GitHub](https://gitee.com/openharmony/update_app)
- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [N-API 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-overview.md)
