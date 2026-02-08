# 文档导航

## 快速导航

| 章节 | 文档 | 说明 |
|------|------|------|
| **概览** | [01_Overview.md](./01_Overview.md) | 项目定位、核心能力、运行环境 |
| **结构** | [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构、模块职责 |
| **架构** | [03_Architecture.md](./03_Architecture.md) | 组件图、数据流、状态机、时序图 |
| **API** | [04_External_API.md](./04_External_API.md) | N-API、Taihe 接口清单 |
| **Inner API** | [05_Inner_API.md](./05_Inner_API.md) | Inner Kits 接口、依赖方向 |
| **构建** | [06_Build.md](./06_Build.md) | GN Targets、编译产物、Feature Flags |
| **安全** | [07_Security_Review.md](./07_Security_Review.md) | 攻击面、风险点、修复建议 |

## 新人阅读路线图

```
┌─────────────────────────────────────────────────────────────┐
│  1. 概览                                                      │
│     ↓                                                         │
│  2. 目录结构                                                  │
│     ↓                                                         │
│  3. 架构说明（理解状态机）                                     │
│     ↓                                                         │
│  4. 对外 API（根据需求选择接口）                               │
│     ↓                                                         │
│  5. 内部 API（扩展开发参考）                                   │
│     ↓                                                         │
│  6. 编译构建（开发环境搭建）                                   │
│     ↓                                                         │
│  7. 安全评审（安全开发参考）                                    │
└─────────────────────────────────────────────────────────────┘
```

## 附录

- [调用链图谱](./appendix/Callgraphs.md) - 关键调用链路
- [配置开关说明](./appendix/Config_Flags.md) - Feature Flags 详解
