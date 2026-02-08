# ASSET Wiki 文档

本文档提供 ASSET（Asset Store Service）的完整技术文档，涵盖架构、API、构建、安全等方面。

## 适用范围

- **模块名称**: ASSET (@ohos/asset)
- **版本**: 4.1
- **子系统**: security
- **系统能力**: SystemCapability.Security.Asset
- **生成时间**: 2026-02-06
- **源码路径**: `/base/security/asset`

## 更新方式

本文档基于源代码自动生成。如需更新：

1. 修改对应模块的源代码
2. 检查受影响的部分（架构、API、构建、安全）
3. 更新相关文档章节
4. 验证所有链接和代码引用正确

## 文档覆盖范围

### 新人学习文档
- [项目概述](00_Overview.md) - 项目定位、边界、核心能力、运行环境
- [目录结构与模块职责](01_Directory_Structure.md) - 不含测试的目录组织
- [架构说明](02_Architecture.md) - 组件图、数据流、线程模型、关键时序
- [对外 N-API](03_NAPI_API.md) - JS API 清单、参数、错误码、绑定位置
- [常见问题](08_QA.md) - 构建/运行/调试问题与定位路径

### 安全研究文档
- [攻击面分析](05_AttackSurface.md) - 4个入口点、敏感操作、信任边界
- [安全风险评审](07_Security_Review.md) - 7个可利用点、修复建议

### 工程实现文档
- [内部 API](04_Inner_API.md) - 模块接口、依赖方向、稳定性
- [GN 目标梳理](05_GN_Targets.md) - targets 列表、类型、依赖、产物
- [编译产物](06_Build_Artifacts.md) - .so/.a/.hap/可执行文件等

### 导航与参考
- [站点导航](SUMMARY.md) - 双路线阅读指南（新人/安全）
- [代码证据库](_work/NOTES.md) - 关键代码引用汇总

## 附录

- [调用链示例](appendix/Callgraphs.md) - 关键调用链
- [配置标志](appendix/Config_Flags.md) - 关键宏/feature flags

## 未覆盖内容

- 测试相关内容（`test/` 目录）
- OpenHarmony 其他系统组件的内部实现
- 非本模块的依赖服务（如 HUKS、UserIAM）的详细文档

## 质量保证

- 所有关键结论都包含代码证据（文件路径 + 行号/符号名）
- 无测试引用作为业务证据
- 术语统一
- 链接完整可导航
