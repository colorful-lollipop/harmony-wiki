# relational_store Wiki

## 概述

本 Wiki 文档系统化地记录了 OpenHarmony **relational_store（关系型数据库）** 组件的架构、实现细节、API 接口、构建系统和安全特性。

**组件版本**: 3.1.0
**子系统**: distributeddatamgr
**许可证**: Apache License 2.0
**生成时间**: 2026-02-06

## 覆盖范围

### 已覆盖内容

- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构说明（组件图、数据流、线程模型）
- ✅ 对外 N-API（JavaScript/TypeScript API）
- ✅ 内部 API（模块接口、依赖方向、稳定性）
- ✅ GN 构建目标与配置
- ✅ 编译产物与安装路径
- ✅ 安全风险评审（攻击面、信任边界、可被利用点）

### 未覆盖内容

- ❌ 测试用例（test/ 目录内容被排除）
- ❌ 跨平台移植细节（仅 OHOS 主分支被分析）
- ❌ 第三方依赖详细分析（如 SQLite、ICU、HUKS 内部实现）

## 如何更新文档

文档基于以下证据生成：
- **源代码文件**: 直接阅读实现
- **头文件**: 接口定义和类型声明
- **构建配置**: BUILD.gn/.gni 文件
- **配置文件**: bundle.json, trusts_config.json

当代码发生重大变更时，应更新对应章节：
1. 新增 N-API 方法 → 更新 `04_N-API.md`
2. 修改模块边界 → 更新 `05_Inner_API.md`
3. 新增/删除 GN 目标 → 更新 `06_GN_Build.md`
4. 安全相关变更 → 更新 `08_Security.md`
5. 架构调整 → 更新 `03_Architecture.md`

## 文档维护指南

### 添加新功能

在相应章节添加：
- 功能描述与目的
- 代码证据（文件路径 + 行号）
- 调用链/时序图（如有）
- 限制与注意事项

### 修改现有文档

- 保持术语一致（使用术语表统一）
- 更新证据引用（确保行号准确）
- 在末尾添加"更新日志"说明变更历史

### 验证清单

更新文档前检查：
- [ ] 所有结论都有代码证据
- [ ] 路径和行号可验证
- [ ] 图表与代码一致
- [ ] 链接有效（Summary.md 导航）
- [ ] 无测试目录引用

## 相关资源

- **官方文档**: [关系型数据库开发指导](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/database/data-persistence-by-rdb-store.md)
- **API 参考**: [关系型数据库 API 文档](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-arkdata/arkts-apis-data-relationalStore.md)
- **源仓库**: [distributeddatamgr_relational_store](https://gitcode.com/openharmony/distributeddatamgr_relational_store)
- **SQLite**: [third_party_sqlite](https://gitcode.com/openharmony/third_party_sqlite)

## 反馈与贡献

如发现问题或需要补充，请在实际代码仓库中提 Issue 或 PR。

---

**文档生成工具**: OpenCode Wiki Agent
**生成日期**: 2026-02-06
**代码版本**: Main Branch (latest)
