# CastEngine 框架 Wiki

> 最后更新时间: 2026-02-06
> 文档版本: 1.0.0
> 覆盖范围: 完整技术文档

## 文档说明

本 Wiki 文档为 OpenHarmony CastEngine 框架（castengine_cast_framework）提供完整的技术参考，包括架构设计、API 文档、构建系统、安全评审等各个方面。

### 覆盖范围

**已覆盖**:
- ✅ 项目概览和定位
- ✅ 目录结构与模块职责
- ✅ 对外 N-API（JavaScript API）
- ✅ 内部 C++ API
- ✅ GN 构建目标和编译产物
- ✅ 服务架构和 IPC 通信
- ✅ 安全风险评审
- ✅ 常见问题和调试指南

**未覆盖**:
- ❌ 协议适配器的详细实现（DLNA/WiFi Display/Cast+Stream）
  - 这些是外部仓库（见 NOTES.md 中的相关仓库）
- ❌ 具体的网络协议细节
  - RTSP/SoftBus 协议的完整规范
- ❌ 测试代码分析
  - 按照任务要求，忽略所有 test/ 目录内容
- ❌ 第三方依赖的详细文档
  - 详见各依赖的官方文档

### 文档更新方式

当代码更新时，建议按以下步骤更新文档：

1. **更新 NOTES.md**: 在 `wiki/_work/NOTES.md` 中记录新的发现
2. **更新对应文档**: 根据变更类型更新相应的 Wiki 页面
3. **更新 PLAN.md**: 更新任务状态和进度
4. **更新本页**: 更新最后更新时间和版本号

### 生成信息

- **生成工具**: Sisyphus (OhMyOpenCode AI Agent)
- **生成日期**: 2026-02-06
- **代码库版本**: 基于 2025-02-06 的代码快照

### 如何使用本 Wiki

建议按照以下顺序阅读文档：

1. **新人入门路径** (30 分钟):
   - [00_Overview.md](00_Overview.md) - 项目概览和定位
   - [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构和模块
   - [03_N-API.md](03_N-API.md) - 如何使用 JavaScript API

2. **开发者路径** (1-2 小时):
   - [02_Architecture.md](02_Architecture.md) - 深入架构理解
   - [04_Internal_API.md](04_Internal_API.md) - 内部 C++ API
   - [05_GN_Targets.md](05_GN_Targets.md) - 构建系统理解

3. **高级主题** (按需):
   - [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物和部署
   - [07_Security_Review.md](07_Security_Review.md) - 安全风险和最佳实践
   - [08_QA.md](08_QA.md) - 常见问题

4. **附录** (参考):
   - [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链
   - [appendix/Config_Flags.md](appendix/Config_Flags.md) - 编译选项

### 文档约定

- **代码引用格式**: `path/to/file:line` - 指向具体代码证据
- **模块名称**: 使用代码中的实际类名/接口名（保持英文）
- **语言**: 主要使用中文，技术术语保留英文
- **版本说明**: 如果某个特性是特定版本引入的，会标注

### 贡献指南

如果发现文档错误或遗漏：

1. 在 `wiki/_work/NOTES.md` 中记录问题
2. 更新相应的文档页面
3. 在本 README 中更新最后修改时间

### 许可证

本文档遵循与代码相同的 Apache License 2.0 许可证。

---

**快速链接**:

- [项目概览](00_Overview.md)
- [目录结构](01_Directory_Structure.md)
- [架构设计](02_Architecture.md)
- [N-API 文档](03_N-API.md)
- [内部 API](04_Internal_API.md)
- [GN 构建系统](05_GN_Targets.md)
- [编译产物](06_Build_Artifacts.md)
- [安全评审](07_Security_Review.md)
- [常见问题](08_QA.md)
