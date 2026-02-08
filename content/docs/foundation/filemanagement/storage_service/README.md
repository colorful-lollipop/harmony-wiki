# Storage Service Wiki

> 本 Wiki 由代码自动生成，覆盖 `@ohos/storage_service` 存储管理服务的完整工程文档。

## 文档覆盖范围

### 已覆盖

- [项目概览](./01_Overview.md) - 组件定位、能力边界、核心功能
- [架构设计](./02_Architecture.md) - 分层架构、组件关系、数据流
- [N-API 接口](./03_NAPI.md) - JS API 清单、绑定位置、调用链
- [内部 API](./04_InnerAPI.md) - Inner API 模块职责、依赖方向
- [GN 构建](./05_GN.md) - Targets 列表、构建配置、产物映射
- [编译产物](./06_Artifacts.md) - `.so`/可执行文件/配置文件
- [安全评审](./07_Security.md) - 攻击面、信任边界、风险点

### 未覆盖

- 详细 API 实现逻辑（代码量过大，文档化不切实际）
- 测试用例说明（按规范不引用测试代码）

## 更新方式

当代码变更时：

1. **N-API 变更**：修改 `interfaces/kits/js/storage_manager/` 下的 `_n_exporter.cpp` 文件后，更新 [03_NAPI.md](./03_NAPI.md) 的 API 清单表
2. **GN 构建变更**：修改 BUILD.gn 后，更新 [05_GN.md](./05_GN.md) 的 targets 列表
3. **架构变更**：调整 SA/IPC 结构后，更新 [02_Architecture.md](./02_Architecture.md)

## 生成信息

| 属性 | 值 |
|------|-----|
| 生成时间 | 2026-02-06 |
| 仓库路径 | `foundation/filemanagement/storage_service` |
| 组件版本 | 3.1 |
| 文档版本 | 1.0 |

## 相关资源

- [bundle.json](../bundle.json) - 组件配置
- [README_zh](../README_zh.md) - 中文简介
- [OpenHarmony Docs](https://gitee.com/openharmony/docs) - 官方文档
