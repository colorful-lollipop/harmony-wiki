# HiCollie Wiki

> HiCollie 软件看门狗组件 - 完整工程文档
>
> 生成时间: 2026-02-06
> 版本: 基于 OpenHarmony 源码
> 目的: 为新人提供完整的项目理解、开发指南和安全分析

---

## 文档范围

本文档档覆盖 HiCollie 组件的以下方面：

- ✅ **项目定位与核心能力** - 组件功能、运行环境、关键概念
- ✅ **目录结构** - 模块组织、职责划分
- ✅ **架构设计** - 组件图、数据流、线程模型、关键时序
- ✅ **对外 API** - NDK C API、Native C++ API、参数、错误码
- ✅ **内部 API** - 模块接口、依赖方向、稳定性标识
- ✅ **GN 构建系统** - Targets 列表、类型、依赖、产物映射
- ✅ **编译产物** - 库文件、安装路径、运行时加载关系
- ✅ **安全风险评审** - 攻击面、信任边界、可被利用点
- ✅ **常见问题** - 构建、运行、调试问题与定位路径

---

## 未覆盖内容

- ❌ N-API (JavaScript 绑定) - HiCollie 不提供 N-API 接口，仅提供 NDK C API 和 Native C++ API
- ❌ 测试代码实现细节 - 文档忽略测试目录内容
- ❌ 与其他 DFX 组件的交互细节（如 HiLog、HiSysEvent）- 仅概述依赖关系

---

## 如何更新文档

当 HiCollie 代码发生变更时，应更新以下文档章节：

1. **接口变更**: 更新 `03_NDK_API.md` 和 `04_Internal_API.md`
2. **新增模块**: 更新 `01_Directory_Structure.md` 和 `02_Architecture.md`
3. **构建目标变更**: 更新 `05_GN_Targets.md` 和 `06_Build_Artifacts.md`
4. **安全修复**: 更新 `07_Security_Review.md`

更新步骤：
1. 重新运行 Phase 1 扫描收集最新代码信息
2. 更新 `wiki/_work/NOTES.md` 记录新发现
3. 修改相关 `.md` 文件
4. 更新本文档的"生成时间"戳

---

## 文档维护者

本文档由自动化工具生成，基于源代码静态分析。如有疑问或发现错误，请：

1. 查看 `wiki/_work/NOTES.md` 了解分析过程
2. 检查源代码验证文档中的路径、行号、符号名是否正确
3. 提交修复到源码仓库

---

## 快速开始

新人阅读建议：

1. 从 `00_Overview.md` 开始了解项目定位
2. 阅读 `01_Directory_Structure.md` 理解代码组织
3. 查看 `02_Architecture.md` 理解核心架构
4. 根据开发需求选择：
   - 使用 NDK C API → `03_NDK_API.md`
   - 使用 Native C++ API → `04_Internal_API.md`
   - 理解构建系统 → `05_GN_Targets.md`
5. 遇到问题时查看 `08_Troubleshooting.md`
6. 安全审计参考 `07_Security_Review.md`

---

## 相关资源

- [OpenHarmony DFX 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- [HiCollie 源码仓库](https://gitee.com/openharmony/hiviewdfx_hicollie)
- [其他 DFX 组件](../)
  - [HiLog](https://gitee.com/openharmony/hiviewdfx_hilog)
  - [HiSysEvent](https://gitee.com/openharmony/hiviewdfx_hisysevent)
  - [HiTrace](https://gitee.com/openharmony/hiviewdfx_hitrace)
  - [HiDumper](https://gitee.com/openharmony/hiviewdfx_hidumper)
