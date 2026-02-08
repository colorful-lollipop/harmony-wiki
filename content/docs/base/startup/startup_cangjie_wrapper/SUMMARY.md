# startup_cangjie_wrapper Wiki 文档目录

## 文档说明

本文档为 OpenHarmony startup_cangjie_wrapper（启动恢复仓颉封装）的完整技术文档，帮助开发者快速理解项目架构、API 设计、构建流程和安全风险。

## 适用对象

- 使用仓颉（Cangjie）语言开发 OpenHarmony 应用的开发者
- 参与 startup_cangjie_wrapper 开发和维护的工程师
- 需要理解设备信息查询服务架构的技术人员

## 文档更新时间

生成时间: 2026-02-06

---

## 阅读路线（推荐顺序）

### 新人入门（30分钟）

1. **[README.md](README.md)** - 文档说明和更新方式
2. **[00_Overview.md](00_Overview.md)** - 项目概览和快速了解
3. **[01_Project_Positioning.md](01_Project_Positioning.md)** - 项目定位、边界、核心能力
4. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 目录结构与模块职责

### 深入理解（2小时）

5. **[03_Architecture.md](03_Architecture.md)** - 架构说明：组件图/数据流/线程模型
6. **[04_Cangjie_API.md](04_Cangjie_API.md)** - 仓颉 API 清单和调用链
7. **[05_Internal_API.md](05_Internal_API.md)** - 内部 API 和依赖关系

### 构建与部署（1小时）

8. **[06_GN_Targets.md](06_GN_Targets.md)** - GN 目标梳理和依赖关系
9. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 编译产物和安装路径

### 安全与运维（1小时）

10. **[08_Security_Review.md](08_Security_Review.md)** - 安全风险评审和修复建议
11. **[09_Common_Issues.md](09_Common_Issues.md)** - 常见问题和定位路径

### 附录（按需查阅）

- **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
- **[appendix/API_Reference.md](appendix/API_Reference.md)** - API 完整参考

---

## 文档结构

```
wiki/
├── README.md                    # 文档说明（当前文件）
├── SUMMARY.md                   # 文档目录（当前文件）
├── 00_Overview.md               # 项目概览
├── 01_Project_Positioning.md    # 项目定位
├── 02_Directory_Structure.md    # 目录结构
├── 03_Architecture.md           # 架构说明
├── 04_Cangjie_API.md            # 仓颉 API 清单
├── 05_Internal_API.md           # 内部 API
├── 06_GN_Targets.md             # GN 目标
├── 07_Build_Artifacts.md        # 编译产物
├── 08_Security_Review.md        # 安全风险评审
├── 09_Common_Issues.md          # 常见问题
└── appendix/                    # 附录
    ├── Callgraphs.md            # 调用链
    └── API_Reference.md        # API 参考
```

---

## 关键术语

| 术语 | 说明 |
|------|------|
| 仓颉（Cangjie） | OpenHarmony 推出的编程语言 |
| FFI（Foreign Function Interface） | 外部函数接口，用于跨语言调用 |
| SA（System Ability） | 系统能力服务，OpenHarmony 的 IPC 机制 |
| SA 服务 | init 组件提供的设备信息查询服务 |
| API Level | 22（当前版本） |
| SysCap | SystemCapability.Startup.SystemInfo |

---

## 与外部资源的链接

- [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - 仓颉互操作库
- [startup_init](https://gitcode.com/openharmony/startup_init) - init 组件
- [仓颉设备信息 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-device_info.md)
- [仓颉设备信息开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/basic-services/device-info/cj-device-info-development-guide.md)

---

## 文档维护

### 如何更新文档

1. 在 `wiki/_work/` 目录中更新 NOTES.md（事实记录）
2. 更新对应的 .md 文件
3. 确保所有结论都有代码证据（路径+符号+行号）
4. 运行一致性校验（Phase 7）

### 文档生成工具

本 Wiki 使用 AI 辅助工具生成，基于代码分析自动提取关键信息。

---

## 覆盖范围

### 已覆盖

✓ 项目架构和目录结构
✓ 仓颉 API 清单和调用链
✓ GN 构建目标和产物
✓ 安全风险评审
✓ 常见问题和解决方案

### 未覆盖

⊗ 测试用例和测试流程（按需求忽略）
⊗ 详细的 FFI 实现细节（依赖外部 init 组件）
⊗ 性能分析和优化建议

---

## 反馈与贡献

如果您发现文档中的错误或有改进建议，欢迎提交 Issue 或 Pull Request。

---

*最后更新: 2026-02-06*
