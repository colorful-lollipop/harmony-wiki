# connectivity_cangjie_wrapper Wiki

> OpenHarmony 蓝牙与 WLAN Cangjie API 封装层工程文档

## 文档覆盖范围

本 Wiki 旨在帮助开发者快速理解 `connectivity_cangjie_wrapper` 项目的架构、API 和使用方式。

### 已覆盖内容

| 文档 | 内容 |
|------|------|
| [项目概览](./01_Project_Overview.md) | 项目定位、核心能力、运行环境 |
| [目录结构](./02_Directory_Structure.md) | 模块划分、职责边界 |
| [架构说明](./03_Architecture.md) | 组件图、数据流、线程模型 |
| [N-API 参考](./04_N-API_Reference.md) | Cangjie API 清单、参数校验 |
| [内部 API](./05_Inner_API.md) | 模块接口、依赖方向 |
| [构建系统](./06_Build_System.md) | GN Targets、编译产物 |
| [安全评审](./07_Security_Review.md) | 攻击面、风险点、修复建议 |
| [FAQ与排错](./08_FAQ_Troubleshooting.md) | 常见问题、定位路径 |

### 未覆盖内容

- 详细 API 使用示例（请参考 [官方 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)）
- 蓝牙/WiFi 底层实现（位于 `communication_bluetooth`/`communication_wifi` 仓库）
- 测试代码（位于 `test/` 目录，不在本文档范围内）

## 文档更新方式

当项目代码发生变更时，建议同步更新以下内容：

1. **新增 API**: 在 `04_N-API_Reference.md` 中添加 API 清单表
2. **新增模块**: 更新 `02_Directory_Structure.md` 和 `05_Inner_API.md`
3. **构建变更**: 更新 `06_Build_System.md` 的 targets 列表
4. **安全相关**: 在 `07_Security_Review.md` 中添加新风险点

## 阅读建议

**新人快速上手**：

1. 先阅读 [项目概览](./01_Project_Overview.md) 了解整体定位
2. 查看 [目录结构](./02_Directory_Structure.md) 理解模块划分
3. 阅读 [N-API 参考](./04_N-API_Reference.md) 学习 API 使用
4. 参考 [FAQ与排错](./08_FAQ_Troubleshooting.md) 解决实际问题

**深入理解**：

1. 阅读 [架构说明](./03_Architecture.md) 掌握设计思想
2. 查看 [构建系统](./06_Build_System.md) 了解编译流程
3. 分析 [安全评审](./07_Security_Review.md) 确保安全使用

## 相关链接

- **项目仓库**: [connectivity_cangjie_wrapper](https://gitee.com/openharmony/connectivity_cangjie_wrapper)
- **API 文档**: [Cangjie ConnectivityKit API](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_en/apis/ConnectivityKit)
- **开发指南**: [蓝牙服务开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/connectivity/bluetooth/cj-bluetooth-overview.md)
- **依赖仓库**:
  - [communication_bluetooth](https://gitee.com/openharmony/communication_bluetooth)
  - [communication_wifi](https://gitee.com/openharmony/communication_wifi)
  - [arkcompiler_cangjie_ark_interop](https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop)

## 文档版本

| 版本 | 更新日期 | 更新内容 |
|------|----------|----------|
| 1.0 | 2026-02-06 | 初始版本 |

---

*文档由 OpenHarmony Wiki Agent 自动生成*
