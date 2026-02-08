# ArkCompiler Runtime Core Wiki

## 简介

本 Wiki 是 [ArkCompiler Runtime Core](https://gitee.com/openharmony/arkcompiler_runtime_core) 的工程文档，旨在帮助开发者快速理解项目架构、API、构建系统和安全风险。

## 生成信息

- **生成时间**: 2025-02-06
- **代码版本**: OpenHarmony_feature_20241108
- **覆盖范围**: arkcompiler/runtime_core 仓库

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航与阅读指南
├── 00_Overview.md         # 项目概览
├── 01_Project_Boundary.md # 项目定位与边界
├── 02_Directory_Structure.md # 目录结构与模块职责
├── 03_Architecture.md     # 架构说明（组件图、数据流、线程模型）
├── 04_Public_API.md       # 对外 API（ANI、N-API）
├── 05_Inner_API.md        # 内部 API 与模块接口
├── 06_GN_Targets.md       # GN 构建目标梳理
├── 07_Build_Artifacts.md  # 编译产物与安装路径
├── 08_Security.md         # 安全风险评审
├── 09_Troubleshooting.md  # 常见问题与调试
└── appendix/
    ├── Callgraphs.md      # 关键调用链
    └── Config_Flags.md    # 关键配置与宏
```

## 新人阅读顺序

1. **[项目概览](00_Overview.md)** - 了解 Runtime Core 是什么、能做什么
2. **[项目定位与边界](01_Project_Boundary.md)** - 明确项目范围、核心能力、运行环境
3. **[目录结构](02_Directory_Structure.md)** - 熟悉代码组织方式
4. **[架构说明](03_Architecture.md)** - 理解组件关系、数据流、线程模型
5. **根据兴趣深入**:
   - 开发 Native 扩展 → [对外 API](04_Public_API.md)
   - 参与运行时开发 → [内部 API](05_Inner_API.md)
   - 构建问题排查 → [GN Targets](06_GN_Targets.md) + [编译产物](07_Build_Artifacts.md)
   - 安全审计 → [安全风险评审](08_Security.md)

## 更新方式

本文档基于代码自动生成，建议随代码版本更新而更新：

1. 修改代码后，同步更新相关 Wiki 页面
2. 新增模块时，在对应章节添加说明
3. API 变更时，更新 [对外 API](04_Public_API.md) 章节

## 范围说明

### 已覆盖

- 项目整体架构与模块职责
- ANI (Ark Native Interface) 对外接口
- N-API 互操作机制
- GN 构建系统与关键 targets
- 主要编译产物与安装路径
- 安全风险分析与可利用点

### 未覆盖

- 具体实现细节（建议直接阅读源码）
- 测试代码相关说明（按约束忽略）
- 历史版本兼容性说明
- 性能优化最佳实践（见 docs/ 目录）

## 贡献与反馈

如发现文档错误或有改进建议，请：
1. 确认问题基于代码证据
2. 在对应文件提交修改
3. 更新 `wiki/_work/NOTES.md` 记录变更

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [ARK Runtime 子系统说明](https://gitee.com/openharmony/docs/blob/master/en/readme/ARK-Runtime-Subsystem.md)
- [Runtime Core 源码](https://gitee.com/openharmony/arkcompiler_runtime_core)
- [ETS Frontend](https://gitee.com/openharmony/arkcompiler_ets_frontend)
- [ETS Runtime](https://gitee.com/openharmony/arkcompiler_ets_runtime)
