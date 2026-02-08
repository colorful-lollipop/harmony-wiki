# Security Component Manager Wiki

> 本 Wiki 提供 security_component_manager 项目的完整技术文档

---

## 文档导航

### 👨‍💻 新人学习路线

适合首次接触本项目的开发者：

1. **[00_Overview](./00_Overview.md)** - 项目概览与核心概念（5分钟了解项目定位）
2. **[01_Directory_Structure](./01_Directory_Structure.md)** - 目录结构与模块职责
3. **[02_Architecture](./02_Architecture.md)** - 架构设计与数据流
4. **[03_Public_APIs](./03_Public_APIs.md)** - 对外 API 参考（C++ SDK）
5. **[04_Internal_APIs](./04_Internal_APIs.md)** - 内部 API 与模块接口
6. **[05_GN_Targets](./05_GN_Targets.md)** - GN 构建系统详解
7. **[06_Build_Artifacts](./06_Build_Artifacts.md)** - 编译产物与运行时加载
8. **[08_Common_Issues](./08_Common_Issues.md)** - 常见问题

**预计阅读时间**: 45 分钟

### 🔐 安全研究路线

适合进行安全审计的研究人员：

1. **[00_Overview](./00_Overview.md)** - 了解安全组件机制与威胁模型
2. **[04_AttackSurface](./04_AttackSurface.md)** - 攻击面分析（外部输入、信任边界）
3. **[07_Security_Analysis](./07_Security_Analysis.md)** - 安全风险评估与可被利用点
4. **[02_Architecture](./02_Architecture.md)** - 理解架构与信任边界

**预计阅读时间**: 60 分钟

### 附录

- **[附录：调用链分析](./appendix/Callgraphs.md)** - 关键 API 调用链
- **[附录：配置与开关](./appendix/Config_Flags.md)** - 关键编译宏与特性开关

---

## 文档完整度检查

✅ **所有文档已创建完成**
- 主文档：8 个（00_Overview ~ 08_Common_Issues）
- 附录文档：2 个（Callgraphs, Config_Flags）
- 工作文件：2 个（NOTES.md, PLAN.md）
- 导航文件：2 个（README.md, SUMMARY.md）

✅ **证据链完整**
- 所有关键结论都有代码证据（文件路径 + 行号）
- N-API 无绑定已确认
- IPC/SA 接口已梳理
- 权限机制已详细分析
- GN 构建系统已完整记录

✅ **术语统一**
- System Ability、SA、Security Component 统一使用
- SDK、Kit、Adapter 统一使用
- Token ID、Token、UID 统一使用

✅ **无测试引用**
- 所有文档均排除测试代码引用
- 仅引用生产代码

✅ **导航完整**
- SUMMARY.md 提供完整导航
- 每个文档都有"返回主页"和"导航"链接

---

## 快速链接

| 主题 | 页面 |
|------|------|
| 项目定位 | [00_Overview](./00_Overview.md) |
| 架构图 | [02_Architecture](./02_Architecture.md) |
| API 参考 | [03_Public_APIs](./03_Public_APIs.md) |
| 构建指南 | [05_GN_Targets](./05_GN_Targets.md) |
| 安全分析 | [07_Security_Analysis](./07_Security_Analysis.md) |
| 常见问题 | [08_Common_Issues](./08_Common_Issues.md) |

---

## 文档范围

### 已覆盖

- ✅ 项目核心功能与定位
- ✅ 目录结构与模块职责
- ✅ 三层架构设计
- ✅ C++ Native SDK API
- ✅ 内部模块接口与依赖关系
- ✅ GN 构建系统（targets、依赖、输出）
- ✅ 编译产物与安装路径
- ✅ 安全风险分析（攻击面、信任边界）
- ✅ 关键调用链
- ✅ 常见构建与调试问题

### 未覆盖

- ⏳ N-API/JS API 绑定（本项目为纯 C++ 服务，无 N-API）
- ⏳ Ace Engine 的 ArkTS 组件实现（位于 arkui_ace_engine 仓库）
- ⏳ Permission Manager 应用实现（位于 applications/standard/permission_manager）
- ⏳ 厂商增强库开发指南（需要额外的厂商文档）
- ⏳ 性能优化建议

---

## 更新信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于 OpenHarmony master 分支
- **维护方式**: 随代码更新同步更新 Wiki

---

## 如何使用本文档

### 作为新开发者

1. 阅读 [00_Overview](./00_Overview.md) 了解项目定位
2. 阅读 [01_Directory_Structure](./01_Directory_Structure.md) 了解代码组织
3. 阅读 [02_Architecture](./02_Architecture.md) 理解架构与数据流
4. 根据 [03_Public_APIs](./03_Public_APIs.md) 开始使用 API

### 作为安全审计人员

1. 阅读 [00_Overview](./00_Overview.md) 了解安全组件机制
2. 阅读 [07_Security_Analysis](./07_Security_Analysis.md) 了解安全风险
3. 参考 [附录：调用链分析](./appendix/Callgraphs.md) 跟踪敏感操作

### 作为系统集成者

1. 阅读 [05_GN_Targets](./05_GN_Targets.md) 了解构建系统
2. 阅读 [06_Build_Artifacts](./06_Build_Artifacts.md) 了解产物与依赖
3. 参考 [附录：配置与开关](./appendix/Config_Flags.md) 配置构建选项

---

## 贡献指南

当更新以下内容时，请同步更新 Wiki：

- 新增/修改 API → 更新 [03_Public_APIs](./03_Public_APIs.md)
- 新增/删除文件 → 更新 [01_Directory_Structure](./01_Directory_Structure.md)
- 架构变更 → 更新 [02_Architecture](./02_Architecture.md)
- 新增/删除 target → 更新 [05_GN_Targets](./05_GN_Targets.md)
- 新增/修复安全问题 → 更新 [07_Security_Analysis](./07_Security_Analysis.md)

---

## 反馈与问题

如有疑问或发现文档错误，请：

1. 检查 `wiki/_work/NOTES.md` 中的证据引用
2. 在代码库中查找相关证据
3. 提交 issue 或 PR 更新文档

---

**返回 [README](./README.md)
