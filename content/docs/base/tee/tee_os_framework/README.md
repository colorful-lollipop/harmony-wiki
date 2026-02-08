# TEE OS Framework Wiki

## 文档说明

本文档描述 OpenHarmony TEE (Trusted Execution Environment) OS Framework 的工程实现细节。

### 覆盖范围

本文档涵盖以下内容：

- **项目定位与边界**：TEE OS Framework 在 OpenTrustee 架构中的角色
- **架构设计**：核心框架模块职责与交互
- **Native API 接口**：GP (GlobalPlatform) 标准 TEE API
- **构建系统**：Makefile 编译配置与产物
- **安全评审**：攻击面分析与风险点

### 不包含内容

- 测试代码 (`test/`)
- N-API / JavaScript 绑定层（本仓库为 Native 实现）
- OpenHarmony 上层框架（位于其他仓库）
- tee_os_kernel（内核实现，位于独立仓库）

### 代码证据原则

本文档所有关键结论均基于以下证据来源：

- 文件路径（精确到 `path:line`，如 `services/permission_service/src/main.c:538`）
- 关键符号名（函数、宏、结构体）
- 代码片段或调用链描述
- 无法确认的信息标注为 `TODO(需确认)`

### 更新方式

当代码发生变更时，需同步更新本文档：

1. 新增 API → 更新 `03_Native_API.md`
2. 新增模块 → 更新 `02_Module_Detail.md` 和 `01_Architecture.md`
3. 构建变更 → 更新 `04_Build_System.md`
4. 安全修复 → 更新 `05_Security_Review.md`

### 阅读建议

新人推荐阅读顺序：

1. `README.md` → 了解文档结构
2. `SUMMARY.md` → 全站导航
3. `00_Overview.md` → 项目概览
4. `01_Architecture.md` → 架构设计
5. `02_Module_Detail.md` → 模块详解（按需）
6. `05_Security_Review.md` → 安全考量（开发时必读）

### 相关仓库

- [tee_os_kernel](https://gitcode.com/openharmony-sig/tee_tee_os_kernel)：TEE 内核实现
- OpenHarmony 系统框架：提供 N-API 层桥接 CA 与 TEE

---

*文档生成时间：2026-02-06*
*基于代码提交：HEAD*
