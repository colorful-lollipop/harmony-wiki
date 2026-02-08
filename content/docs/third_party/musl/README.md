# OpenHarmony musl Wiki

> OpenHarmony musl libc 工程文档中心
> 生成时间: 2025-02-06

---

## 文档说明

本文档集是面向 OpenHarmony 第三方库 musl 的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 接口、构建系统和安全风险。

### 覆盖范围

- **项目定位与核心能力**: musl 在 OpenHarmony 中的角色和适配特性
- **目录结构与模块职责**: 源码组织方式和各模块功能
- **架构设计**: 组件关系、数据流、线程模型
- **API 文档**: 对外接口和内部接口
- **构建系统**: GN 构建目标和编译产物
- **安全分析**: 攻击面识别和风险评估
- **问题排查**: 常见问题定位方法

### 未覆盖范围

- 测试相关代码（test/, tests/, unittest/ 等）
- 上游 musl 的通用文档（请参考 [musl 官方文档](https://musl.libc.org/)）
- 特定芯片平台的底层细节

### 如何更新本文档

本文档基于代码分析自动生成，当代码发生变更时：
1. 重新运行分析流程
2. 更新 `wiki/_work/NOTES.md` 中的事实记录
3. 同步修改相关文档章节
4. 更新 `wiki/README.md` 中的生成时间

---

## 快速导航

| 文档 | 说明 | 推荐阅读顺序 |
|------|------|-------------|
| [00_Overview.md](00_Overview.md) | 项目概览 | 1 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构 | 2 |
| [02_Architecture.md](02_Architecture.md) | 架构设计 | 3 |
| [03_Public_API.md](03_Public_API.md) | 对外 API | 4 |
| [04_Internal_API.md](04_Internal_API.md) | 内部 API | 5 |
| [05_GN_Targets.md](05_GN_Targets.md) | GN 构建目标 | 6 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物 | 7 |
| [07_Security_Analysis.md](07_Security_Analysis.md) | 安全风险分析 | 8 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 问题排查 | 9 |
| [SUMMARY.md](SUMMARY.md) | 全站导航 | - |

---

## 项目基本信息

| 属性 | 值 |
|------|-----|
| 组件名 | @ohos/musl |
| 版本 | 3.1（基于 musl 1.1.x） |
| 许可证 | MIT |
| 子系统 | thirdparty |
| 适配系统 | mini, small, standard |
| 支持架构 | arm, aarch64, x86_64, mips, riscv64, loongarch64 |

---

## OpenHarmony 适配特性

相比上游 musl，OpenHarmony 版本新增了以下关键特性：

1. **加载器地址随机化** - 增强动态链接安全性
2. **RELRO 共享机制** - 减少内存占用
3. **Namespace 机制** - 支持多 namespace 库隔离
4. **Bionic 兼容** - 支持在 OHOS 容器中运行 Android 库
5. **ICU 全球化** - 通过 ICU 实现 locale 数据能力
6. **mallocng 安全增强** - 堆内存分配器安全加固
7. **GWP-ASan 集成** - 内存错误检测
8. **Hook 机制** - 内存和 socket 操作的 hook 支持

---

## 参考链接

- [musl 官方网站](https://musl.libc.org/)
- [musl 参考手册](https://musl.libc.org/doc/1.1.24/manual.html)
- [OpenHarmony 文档中心](https://docs.openharmony.cn/)

