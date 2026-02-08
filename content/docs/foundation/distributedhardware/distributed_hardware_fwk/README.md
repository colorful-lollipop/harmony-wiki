# 分布式硬件管理框架 Wiki

**生成时间**: 2025-02-06
**最后更新**: 2025-02-07
**源码仓库**: [distributed_hardware_fwk](https://gitee.com/openharmony/distributed_hardware_fwk)

---

## 覆盖范围

本 Wiki 全面覆盖 OpenHarmony 分布式硬件管理框架（Distributed Hardware Framework）的工程文档，包括：

### 已覆盖内容
- ✅ 项目评估与受众分析 (`_work/ASSESSMENT.md`)
- ✅ 项目定位、核心能力与系统架构
- ✅ 目录结构与模块职责划分
- ✅ N-API 接口规范（JavaScript/TypeScript API）
- ✅ 内部 Inner API 与模块接口
- ✅ GN 构建配置与 Targets
- ✅ 编译产物与运行时加载关系
- ✅ 安全风险评审与威胁模型 (含5条详细风险分析)
- ✅ IPC 通信与 System Ability 架构
- ✅ 双路线导航（新人学习 + 安全研究）

### 未覆盖内容
- ❌ 测试相关内容（单元测试、模糊测试）
- ❌ 具体硬件驱动实现细节
- ❌ 运行时性能调优数据
- ❌ 历史版本变更记录

---

## 使用说明

### 新人阅读顺序（推荐）

1. **[概览](index.md)** - 快速理解项目定位与核心能力
2. **[目录结构](01_Directory_Structure.md)** - 了解模块划分
3. **[架构说明](02_Architecture.md)** - 理解系统架构与数据流
4. **[N-API 接口](03_NAPI_Reference.md)** - 查阅 JS API 使用方法
5. **[内部 API](04_Inner_API.md)** - 了解模块间接口
6. **[GN 构建配置](05_GN_Build.md)** - 理解构建系统
7. **[编译产物](06_Build_Artifacts.md)** - 了解输出文件
8. **[安全评审](07_Security_Review.md)** - 了解安全考量

### 快速导航

| 主题 | 文档链接 |
|------|----------|
| 项目概览 | [index.md](index.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 系统架构 | [02_Architecture.md](02_Architecture.md) |
| N-API 参考 | [03_NAPI_Reference.md](03_NAPI_Reference.md) |
| 内部 API | [04_Inner_API.md](04_Inner_API.md) |
| GN 构建 | [05_GN_Build.md](05_GN_Build.md) |
| 编译产物 | [06_Build_Artifacts.md](06_Build_Artifacts.md) |
| 安全评审 | [07_Security_Review.md](07_Security_Review.md) |
| 附录 | [appendix/](appendix/) |

---

## 更新方式

### 当代码变更时

1. **修改 N-API**：更新 `03_NAPI_Reference.md` 中的 API 清单
2. **修改模块结构**：更新 `01_Directory_Structure.md` 和 `04_Inner_API.md`
3. **修改构建配置**：更新 `05_GN_Build.md` 和 `06_Build_Artifacts.md`
4. **修改安全策略**：更新 `07_Security_Review.md`

### 质量检查清单

- [ ] 所有关键结论有源码证据（文件路径 + 行号）
- [ ] N-API 文档包含完整的参数/返回值/错误码
- [ ] GN 文档包含 Targets 清单和依赖关系
- [ ] 安全文档至少包含 5 条风险点
- [ ] SUMMARY.md 链接全部有效

---

## 文档规范

### 证据标注格式

所有关键结论必须标注源码证据：

```markdown
**证据**: `path/to/file:line-number`
**符号**: `SymbolName`
**代码片段**:
```cpp
// 关键代码
```
```

### 术语统一

| 术语 | 说明 |
|------|------|
| DHFwk / dhardware | 分布式硬件管理框架 |
| SA | System Ability |
| N-API | Native API（JavaScript bindings） |
| Inner Kit | 内部接口（供其他子系统使用） |
| Taihe | ANI 现代化绑定 |
| ACL | Access Control List |

---

## 贡献指南

如需贡献文档，请：

1. 在对应文档中新增/修改内容
2. 确保所有结论有源码证据支撑
3. 更新 SUMMARY.md 添加必要链接
4. 运行一致性检查（TODO: 添加检查脚本）

---

## 许可证

本 Wiki 内容基于 [Apache License 2.0](../../LICENSE) 许可证。
