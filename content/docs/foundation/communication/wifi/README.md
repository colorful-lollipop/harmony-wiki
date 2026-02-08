# WLAN Wiki 文档

**覆盖范围**: OpenHarmony WLAN 组件 (`communication_wifi`) 的完整工程文档

**最后更新**: 2026-02-06

**文档目的**: 为开发者提供一套完整、易懂的 WLAN 组件文档，包括架构、API、构建系统、安全风险等。

---

## 文档内容

本 Wiki 包含以下文档：

### 入门与概览
- [00_Overview.md](00_Overview.md) - 项目概览、定位、边界、核心能力
- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构与模块职责

### 架构与设计
- [02_Architecture.md](02_Architecture.md) - 架构说明：组件图、数据流、线程模型、关键时序

### 对外接口
- [03_Public_API_NAPI.md](03_Public_API_NAPI.md) - 对外 N-API（JS API 面）、导出符号、权限/参数/错误码

### 内部接口
- [04_Inner_API.md](04_Inner_API.md) - 内部 API：模块接口、依赖方向、稳定性、可替换点

### 构建系统
- [05_GN_Targets.md](05_GN_Targets.md) - GN 目标梳理：targets 列表、类型、依赖、产物、开关

### 编译产物
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物：`.so/.a/.hap/可执行文件`、安装路径、运行时加载关系

### 安全与风险
- [07_Security_Review.md](07_Security_Review.md) - 安全风险评审：攻击面、信任边界、可被利用点、修复建议

### 常见问题
- [08_FAQ.md](08_FAQ.md) - 常见构建/运行/调试问题与定位路径（基于代码与脚本）

### 附录
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链（入口→核心逻辑）
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 关键宏/feature flags

---

## 新人阅读顺序

如果您是首次接触 WLAN 组件的开发者，建议按以下顺序阅读：

1. **快速入门**（30 分钟）
   - [00_Overview.md](00_Overview.md) - 了解项目定位和核心能力
   - [01_Directory_Structure.md](01_Directory_Structure.md) - 熟悉目录结构和模块职责

2. **理解架构**（60 分钟）
   - [02_Architecture.md](02_Architecture.md) - 理解组件架构、数据流和时序

3. **学习 API**（120 分钟）
   - [03_Public_API_NAPI.md](03_Public_API_NAPI.md) - 学习如何使用对外 N-API
   - [04_Inner_API.md](04_Inner_API.md) - 了解内部接口（如需参与开发）

4. **构建与部署**（60 分钟）
   - [05_GN_Targets.md](05_GN_Targets.md) - 理解构建系统
   - [06_Build_Artifacts.md](06_Build_Artifacts.md) - 了解编译产物和部署

5. **安全与调试**（60 分钟）
   - [07_Security_Review.md](07_Security_Review.md) - 了解安全考虑
   - [08_FAQ.md](08_FAQ.md) - 查看常见问题和解决方案

**总计阅读时间**: 约 6-7 小时

---

## 如何更新文档

### 代码变更时
当修改代码后，请同步更新相关 Wiki 文档：

1. **API 变更**：更新 [03_Public_API_NAPI.md](03_Public_API_NAPI.md) 和 [04_Inner_API.md](04_Inner_API.md)
2. **架构变更**：更新 [02_Architecture.md](02_Architecture.md)
3. **构建变更**：更新 [05_GN_Targets.md](05_GN_Targets.md) 和 [06_Build_Artifacts.md](06_Build_Artifacts.md)
4. **安全变更**：如影响安全，需更新 [07_Security_Review.md](07_Security_Review.md)

### 更新标准
- **证据先行**：所有关键结论必须能在仓库内找到直接证据（文件路径、行号、符号名）
- **避免测试引用**：不得引用 `test/`、`tests/`、`unittest/` 目录内容作为业务证据
- **术语统一**：使用统一的术语和命名约定
- **链接有效**：确保所有文档内链接正确且有效

---

## 文档维护说明

本文档由 OpenHarmony WLAN 工程团队维护。

**生成方式**：
- 基于代码库静态分析
- 使用 [wiki/_work/NOTES.md](wiki/_work/NOTES.md) 记录的事实证据
- 参考官方文档和代码注释

**更新机制**：
- 代码变更触发对应文档更新
- 定期进行一致性检查
- 根据开发者反馈持续改进

---

## 反馈与贡献

如有文档问题或改进建议，请通过以下方式反馈：

- 提交 Issue 到代码仓库
- 提交 Pull Request 到 Wiki 文档
- 联系 WLAN 组件维护者

---

**注意**: 本 Wiki 专注于代码层面的工程文档，如需了解产品规格或用户使用指南，请参考产品文档。
