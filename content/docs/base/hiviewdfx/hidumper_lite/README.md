# hidumper_lite 项目 Wiki

## 文档覆盖范围

本文档是 `hidumper_lite` 项目的完整技术 Wiki，旨在帮助开发者快速理解项目架构、使用方法、编译配置以及安全考量。

### 涵盖内容

- **项目概述**：定位、边界、核心能力、运行环境
- **架构说明**：组件图、数据流、线程模型、关键时序
- **接口文档**：对外 API（N-API、内部接口）、平台适配接口
- **构建配置**：GN Targets、编译产物、安装路径
- **安全评审**：攻击面、信任边界、风险识别与修复建议
- **问题排查**：常见构建、运行、调试问题

### 未涵盖内容

- **测试相关**：单元测试、集成测试代码及配置
- **历史变更**：详细的 CHANGELOG 和版本迁移指南
- **其他子系统**：仅涵盖 `hidumper_lite` 本身，不包含相关依赖子系统

## 文档更新方式

### 手动更新

当代码发生变更时，请同步更新相关 Wiki 章节：

1. **新增功能**：在 [03_API.md](03_API.md) 中添加 API 文档
2. **修改架构**：在 [02_Architecture.md](02_Architecture.md) 中更新架构图和说明
3. **调整构建**：在 [04_Build.md](04_Build.md) 中更新 Targets 列表和产物信息
4. **发现风险**：在 [05_Security.md](05_Security.md) 中添加安全风险项

### 自动化建议

建议在 CI/CD 流程中添加以下检查：

- [ ] API 文档与头文件声明一致性
- [ ] BUILD.gn 与产物文档匹配
- [ ] 链接有效性检查（SUMMARY.md 中的锚点）

## 文档结构

```
wiki/
├── README.md                 # 本文档，覆盖范围与更新指南
├── SUMMARY.md                # 全站导航，新人阅读路线
├── 01_Overview.md            # 项目概览
├── 02_Architecture.md        # 架构说明
├── 03_API.md                 # 接口文档
├── 04_Build.md               # 构建配置
├── 05_Security.md            # 安全评审
├── 06_Troubleshooting.md     # 问题排查
└── appendix/
    ├── Callgraphs.md         # 关键调用链
    └── Config_Flags.md       # 关键配置项
```

## 阅读建议

### 新人阅读路线

1. 阅读 [01_Overview.md](01_Overview.md) 了解项目定位
2. 阅读 [02_Architecture.md](02_Architecture.md) 理解整体架构
3. 阅读 [03_API.md](03_API.md) 掌握接口使用
4. 阅读 [04_Build.md](04_Build.md) 熟悉编译配置
5. 参考 [06_Troubleshooting.md](06_Troubleshooting.md) 解决实际问题

### 按需查阅

- **开发者**：主要阅读 02、03、04 章节
- **安全审计**：重点阅读 05 章节
- **问题定位**：直接查阅 06 章节

## 代码证据索引

本文档中的关键结论均可在代码仓库中找到直接证据：

| 结论类型 | 证据位置 |
|----------|----------|
| AT 命令参数 | `mini/hidumper_core.c:38-55` |
| IOCTL 命令定义 | `lite/hidumper.c:55-62` |
| 适配器注册流程 | `mini/hidumper_core.c:79-103` |
| 平台初始化 | `mini/hidumper_adapter.c:84-101` |
| 构建 Targets | `BUILD.gn`、`lite/BUILD.gn`、`mini/BUILD.gn` |
| 模块配置 | `bundle.json` |

## 文档版本

- **文档版本**：1.0.0
- **生成时间**：2026年2月6日
- **最后更新**：2026年2月6日
- **对应代码版本**：hidumper_lite v4.0.2

## 贡献指南

### 改进本文档

1. 在对应章节中添加或修改内容
2. 确保每个结论都有代码证据支持
3. 更新 SUMMARY.md（如果添加新页面）
4. 运行一致性检查（Phase 7）

### 报告问题

如果发现以下问题，请提交 Issue：

- 文档与代码不一致
- 缺少重要内容
- 链接失效
- 描述不清晰

## 相关资源

- **代码仓库**：`https://gitee.com/openharmony/hiviewdfx_hidumper_lite`
- **DFX 子系统**：[DFX 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- **OpenHarmony**：https://www.openharmony.cn
