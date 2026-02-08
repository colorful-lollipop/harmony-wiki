# Time Service Wiki

> OpenHarmony 时间服务子系统工程文档
> 
> 生成时间：2026-02-06
> 代码版本：3.1
> 源码路径：`/base/time/time_service`

---

## 文档范围

本文档集覆盖 OpenHarmony `time_service` 模块的完整工程实现，包括：

- **对外 API**：N-API（JS/TS）、C NDK、Cangjie FFI
- **内部架构**：SystemAbility、IPC、定时器管理、时区管理
- **构建系统**：GN 目标、编译产物、依赖关系
- **安全分析**：权限校验、攻击面、风险点

## 适用对象

- **新成员**：快速理解项目结构、入口、关键流程
- **开发者**：接口使用、调试定位、功能扩展
- **安全审计**：权限边界、攻击面、加固建议

## 文档导航

详见 [SUMMARY.md](./SUMMARY.md)

### 推荐阅读顺序

1. [项目概览](./00_Overview.md) - 了解项目定位、核心能力
2. [目录结构](./01_Directory_Structure.md) - 源码组织方式
3. [架构说明](./02_Architecture.md) - 组件关系、数据流、线程模型
4. [N-API 参考](./03_NAPI_Reference.md) - JS API 详细说明
5. [内部 API](./04_Inner_API.md) - C++ 接口与模块职责
6. [构建系统](./05_GN_Targets.md) - GN 目标与编译配置
7. [安全分析](./06_Security_Analysis.md) - 风险评审与加固建议

## 维护与更新

### 如何更新本文档

1. 修改源码后同步更新对应 `.md` 文件
2. 新增功能需在 `SUMMARY.md` 添加导航链接
3. 关键结论必须附带代码证据（路径+行号）

### 文档约束

- **仅覆盖 `wiki/` 目录**：不得修改其他仓库文件
- **忽略测试代码**：不引用 `test/` 目录内容作为业务证据
- **中文为主**：除非特殊术语保持英文

## 项目元数据

| 属性 | 值 |
|------|-----|
| 项目名 | @ohos/time_service |
| 版本 | 3.1 |
| 子系统 | time |
| 系统能力 | SystemCapability.MiscServices.Time |
| 许可证 | Apache License 2.0 |
| 仓库 | https://gitcode.com/openharmony/time_time_service/ |

## 未覆盖范围

- 单元测试实现细节（位于 `test/`）
- 上层应用使用示例（参考 README.md）
- 历史版本变更（参考 Git 历史）
