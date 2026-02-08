# Frame Aware Sched Wiki

## 文档覆盖范围

本 Wiki 全面介绍 OpenHarmony `frame_aware_sched` 组件，包含以下内容：

- **项目概览**：组件定位、核心能力、运行环境
- **目录结构**：模块划分与职责说明（不含测试）
- **架构设计**：Collector 与 Policy 两大组件、数据流、Mermaid 图示
- **对外 API**：Inner API（`frame_ui_intf`、`frame_msg_intf`、`frame_trace`、`rtg_interface`）接口清单
- **内部架构**：核心模块职责、依赖关系、生命周期
- **构建产物**：GN Targets 梳理、`.so` 产物、运行时加载关系
- **安全评审**：攻击面分析、信任边界、风险点与修复建议
- **附录**：关键配置、FAQ

## 更新方式

当代码发生以下变更时，请同步更新 Wiki：

1. 新增/删除/修改 Inner API 接口
2. 新增/删除/修改 GN Targets 或依赖
3. 新增安全风险或修复现有风险
4. 修改配置文件（`profiles/*.xml`）

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: OpenHarmony 3.1+ (基于 bundle.json 版本号)
- **维护者**: OpenHarmony resourceschedule 子系统团队

## 相关链接

- [OpenHarmony 官网](https://www.openharmony.cn/)
- [Resource Schedule Subsystem](https://gitee.com/openharmony/resourceschedule_resource_schedule_service)
- [Frame Aware Sched 仓库](https://gitee.com/openharmony/frame_aware_sched)
