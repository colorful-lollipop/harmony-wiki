# OpenHarmony Linux Kernel - 工程 Wiki

> **生成时间**: 2026-02-06
> **项目版本**: v3.1.0
> **覆盖范围**: OpenHarmony Linux Kernel 构建系统 (kernel/linux/build)

---

## 文档目的

本文档集为 OpenHarmony Linux Kernel 构建项目提供全面的技术文档，帮助开发者快速理解项目架构、构建流程、配置管理和安全特性。

---

## 适用对象

- **内核适配工程师**: 需要理解内核构建流程和补丁管理
- **驱动开发者**: 需要了解如何添加和管理驱动补丁
- **构建系统开发者**: 需要理解 GN/Make 构建流程
- **安全审计人员**: 需要评估构建系统的安全性
- **新加入项目成员**: 快速了解项目全貌

---

## 覆盖范围

### 已覆盖内容

✅ 项目定位与架构说明
✅ GN 构建系统与 targets 清单
✅ Shell 构建脚本与 Makefile
✅ 内核配置管理（defconfig）
✅ HDF 与驱动补丁管理
✅ 编译产物与镜像类型
✅ 安全风险评审

### 未覆盖内容

❌ Linux 内核源码分析（位于 `kernel/linux/linux-4.19` 或 `kernel/linux/linux-5.10`）
❌ HDF 驱动框架详细实现（位于 `drivers/hdf_core/adapter/khdf/linux`）
❌ 具体芯片平台的硬件适配细节
❌ 测试用例与测试框架

---

## 文档维护

### 随代码更新

本文档应随代码变更同步更新。以下情况需要更新文档：

1. **BUILD.gn 变更**: 新增/删除/修改 GN target
2. **构建脚本变更**: Shell 脚本、Makefile、Python 脚本的功能修改
3. **配置文件变更**: defconfig、patch 文件的路径或命名规则变化
4. **架构调整**: 构建流程、依赖关系、产物路径变化
5. **安全特性**: 新增或修改安全相关的配置或检查点

### 更新流程

1. 修改对应章节的内容
2. 更新文档底部的"更新记录"表格
3. 运行一致性校验：检查 `SUMMARY.md` 链接、术语统一性
4. 在 PR 描述中注明文档变更

---

## 文档质量标准

所有文档遵循以下 DoD (Definition of Done):

- [ ] 每篇文档包含：目的、适用范围、关键结论、相关链接
- [ ] 关键结论可追溯到代码证据（路径:行号 / 符号名）
- [ ] 无法确认的内容标注 `TODO(需确认)` 并说明缺少的证据
- [ ] 不引用测试目录内容（test/、*_test.*、*_fuzzer.*）
- [ ] 术语统一，全站导航链接有效
- [ ] 包含适当的 Mermaid 图表说明架构或流程

---

## 快速导航

| 章节 | 内容 | 预计阅读时间 |
|------|------|-------------|
| [项目概览](01_Project_Overview.md) | 项目定位、核心能力、运行环境 | 10 分钟 |
| [目录结构](02_Directory_Structure.md) | 目录职责、文件组织 | 5 分钟 |
| [构建系统架构](03_Build_System_Architecture.md) | GN+Make+Shell 构建流程 | 15 分钟 |
| [GN Targets](04_GN_Targets.md) | targets 清单、类型、依赖 | 10 分钟 |
| [构建脚本](05_Build_Scripts.md) | Shell/Makefile/Python 脚本分析 | 15 分钟 |
| [内核配置](06_Kernel_Configuration.md) | defconfig、内核参数配置 | 10 分钟 |
| [补丁管理](07_Patch_Management.md) | HDF/驱动补丁机制 | 15 分钟 |
| [编译产物](08_Build_Artifacts.md) | 镜像类型、输出路径 | 5 分钟 |
| [安全评审](09_Security_Review.md) | 攻击面、风险点分析 | 20 分钟 |
| [故障排查](10_Troubleshooting.md) | 常见问题与定位方法 | 10 分钟 |

---

## 参考资源

### 官方文档

- [OpenHarmony Linux Kernel 文档](https://gitee.com/openharmony/kernel_linux_patches)
- [Linux 4.19 LTS](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-4.19.y)
- [Linux 5.10 LTS](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/log/?h=linux-5.10.y)

### 内部资源

- `kernel/linux/linux-4.19` - Linux 4.19 内核源码
- `kernel/linux/linux-5.10` - Linux 5.10 内核源码
- `kernel/linux/patches` - 内核补丁目录
- `kernel/linux/config` - 内核配置文件
- `drivers/hdf_core/adapter/khdf/linux` - HDF Linux 适配层

---

## 反馈与贡献

如发现文档错误或有改进建议，请：

1. 提交 Issue 说明问题
2. 创建 PR 修改文档
3. 在 PR 描述中引用相关的代码证据

---

## 更新记录

| 日期 | 版本 | 变更内容 | 贡献者 |
|------|------|---------|--------|
| 2026-02-06 | 3.1.0 | 初始版本，创建完整 Wiki 骨架 | AI Assistant |

---

## 许可证

本文档遵循 [GPL 2.0](../LICENSE) 许可证。
