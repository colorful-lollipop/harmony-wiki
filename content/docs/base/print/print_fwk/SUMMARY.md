# OpenHarmony Print Scan Framework - 文档导航

**建议阅读顺序**: 1 → 2 → 3 → ...

---

## 📚 目录

### 核心文档（必读）

- [项目概览](00_Overview.md) - 项目定位、边界、核心能力、运行环境、关键概念
- [目录结构](02_Directory_Structure.md) - 目录树与模块职责映射
- [架构设计](03_Architecture.md) - 组件图、数据流、线程模型、关键时序

### API 文档（开发者必读）

- [对外 API](04_External_API.md) - **N-API（JS API 面）**、导出符号、权限、参数、错误码
- [内部 API](05_Internal_API.md) - 模块接口、依赖方向、稳定性、可替换点

### 构建与运行时

- [GN Targets](06_GN_Targets.md) - Targets 列表、类型、依赖、产物、编译开关
- [编译产物](07_Build_Artifacts.md) - .so/.a/.hap/可执行文件；安装路径；运行时加载关系

### 安全与运维

- [安全风险评审](08_Security_Review.md) - 攻击面、信任边界、可被利用点、修复建议
- [常见问题](09_Common_Issues.md) - 常见构建/运行/调试问题与定位路径

### 附录（可选）

- [关键调用链](appendix/Callgraphs.md) - 入口→核心逻辑的调用链（Mermaid）
- [关键配置项](appendix/Config_Flags.md) - 关键宏、Feature Flags、配置项说明

---

## 📖 新人阅读路线图

### 路线 1：快速了解（15 分钟）

```
00_Overview.md (5 分钟)
    ↓
02_Directory_Structure.md (5 分钟)
    ↓
03_Architecture.md (5 分钟)
```

### 路线 2：开发者深入（60 分钟）

```
04_External_API.md (30 分钟)
    ↓
05_Internal_API.md (15 分钟)
    ↓
06_GN_Targets.md (10 分钟)
    ↓
07_Build_Artifacts.md (5 分钟)
```

### 路线 3：安全与运维（30 分钟）

```
08_Security_Review.md (20 分钟)
    ↓
09_Common_Issues.md (10 分钟)
```

### 按角色阅读

| 角色 | 建议阅读顺序 | 预计时间 |
|------|--------------|---------|
| **新人/项目理解** | 00 → 02 → 03 | 15 分钟 |
| **应用开发者** | 00 → 02 → 04 | 40 分钟 |
| **系统开发者** | 05 → 06 → 07 | 30 分钟 |
| **安全审计员** | 00 → 02 → 08 | 35 分钟 |
| **运维工程师** | 07 → 09 | 15 分钟 |

---

## 🔍 快速查找

| 我想了解 | 查看文档 |
|---------|---------|
| 如何调用打印 API？ | [04_External_API.md](#打印-n-api-模块-print) |
| 如何调用扫描 API？ | [04_External_API.md](#扫描-n-api-模块-scan) |
| 打印任务状态有哪些？ | [04_External_API.md](#打印-napi-枚举常量) |
| 架构分层是怎样的？ | [03_Architecture.md](#架构分层) |
| 如何编译项目？ | [06_GN_Targets.md](#构建目标清单) |
| 安全风险有哪些？ | [08_Security_Review.md](#攻击面清单) |
| 打印机发现失败怎么办？ | [09_Common_Issues.md](#设备发现问题) |
| 权限不足怎么办？ | [09_Common_Issues.md](#权限相关) |

---

## 📝 文档更新日志

| 日期 | 更新内容 | 贡献者 |
|------|---------|---------|
| 2026-02-05 | 初始创建文档框架 | AI Wiki Generator |

---

## ⚠️ 重要提示

1. **权限要求**: 使用完整打印功能需要申请 `ohos.permission.PRINT` 和 `ohos.permission.MANAGE_PRINT_JOB` 权限
2. **系统版本**: 本文档基于版本 3.1，其他版本可能有差异
3. **代码证据**: 所有关键技术结论都附有代码证据（路径+符号+行号）
4. **测试代码**: 文档中不引用测试目录代码，仅关注生产代码
5. **第三方库**: CUPS 和 SANE 是外部依赖，文档不深入其内部实现
