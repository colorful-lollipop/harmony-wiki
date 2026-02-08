# OpenHarmony Linux Config Wiki - 目录

## 快速入门

对于首次接触本仓库的读者，建议按照以下顺序阅读以快速建立整体认知：首先阅读 [项目概览](./01_Overview.md) 了解项目定位和核心能力，然后阅读 [目录结构](./02_Directory_Structure.md) 掌握仓库的组织方式，接着阅读 [配置分层架构](./03_Config_Layers.md) 理解配置的设计理念，最后阅读 [构建集成说明](./04_Build_Integration.md) 了解如何在 OpenHarmony 构建系统中使用本仓库。

对于有特定需求的读者，可以直接跳转到相关章节：如果需要进行内核安全评估，请直接阅读 [安全评审](./05_Security_Review.md)；如果需要移植到新的开发板，请参考 [附录：开发板移植指南](./appendix/Board_Porting.md)；如果需要查阅所有配置项清单，请参考 [附录：配置项清单](./appendix/Config_Details.md)。

## 文档索引

### 入门篇

| 文档 | 说明 | 预计阅读时间 |
|------|------|-------------|
| [README](./README.md) | 本 Wiki 的说明、适用范围和更新方式 | 2 分钟 |
| [SUMMARY](./SUMMARY.md) | 全站导航和新人阅读路线 | 1 分钟 |
| [项目概览](./01_Overview.md) | 项目定位、核心能力、关键概念 | 5 分钟 |
| [目录结构](./02_Directory_Structure.md) | 仓库目录组织方式 | 3 分钟 |

### 核心篇

| 文档 | 说明 | 预计阅读时间 |
|------|------|-------------|
| [配置分层架构](./03_Config_Layers.md) | 五层配置设计理念和加载顺序 | 10 分钟 |
| [构建集成说明](./04_Build_Integration.md) | 与 OpenHarmony 构建系统的集成方式 | 5 分钟 |
| [安全评审](./05_Security_Review.md) | 安全配置分析和风险评估 | 10 分钟 |

### 附录篇

| 文档 | 说明 |
|------|------|
| [配置项清单](./appendix/Config_Details.md) | 所有 defconfig 文件的详细配置项说明 |
| [开发板移植指南](./appendix/Board_Porting.md) | 新增开发板配置的步骤和注意事项 |
| [术语表](./appendix/Glossary.md) | 本 Wiki 使用的专业术语解释 |

## 相关链接

- **OpenHarmony 官方文档**：[https://docs.openharmony.cn](https://docs.openharmony.cn)
- **内核源码仓库**：`kernel/linux/linux-5.10`
- **构建脚本仓库**：`kernel/linux/build`
- **HDF 驱动框架**：`drivers/hdf_core`
- **代码仓库地址**：[https://gitee.com/openharmony/kernel_linux_config](https://gitee.com/openharmony/kernel_linux_config)

## 阅读建议

新手建议完整阅读全部文档以建立系统认知，重点关注配置分层架构章节以理解 OpenHarmony 内核配置的设计哲学。有经验者可选择性阅读，核心文档为配置分层架构和构建集成说明。安全评审章节建议周期性阅读，特别是在进行安全合规评估时。所有文档中的配置项都可以在仓库中找到对应的代码证据，建议结合源码阅读以加深理解。
