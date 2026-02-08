# bundlemanager_cangjie_wrapper Wiki

> OpenHarmony BundleManager Cangjie API 封装层文档

## 文档覆盖范围

本文档覆盖 `bundlemanager_cangjie_wrapper` 项目的以下方面：

| 分类 | 覆盖内容 |
|------|----------|
| **项目定位** | 模块定位、能力边界、运行环境 |
| **目录结构** | 模块划分、职责定义 |
| **对外 API** | N-API 接口清单、参数校验、错误码 |
| **内部架构** | 模块依赖、线程模型、生命周期 |
| **构建配置** | GN Targets、编译产物、依赖关系 |
| **安全风险** | 攻击面分析、信任边界、风险评估 |

## 未覆盖内容

| 排除项 | 原因 |
|--------|------|
| 测试代码（`test/` 目录） | 根据项目规范，测试代码不计入业务文档 |
| 外部依赖内部实现 | 仅记录本仓库代码，外部依赖详见各自仓库 |
| 运行时动态加载细节 | 运行时行为由 ArkRuntime 负责 |

## 阅读建议

**新人推荐阅读顺序**：

1. `README.md`（本文档）
2. `SUMMARY.md`（导航索引）
3. `01_Overview.md`（项目概览）
4. `02_API_Reference.md`（API 参考）
5. `03_Architecture.md`（架构说明）
6. `04_Build.md`（构建配置）

## 文档更新方式

本文档基于代码自动生成。如需更新：

1. 修改代码后，运行文档生成工具（如果有）
2. 或手动更新对应章节，确保包含：
   - 文件路径（证据）
   - 关键符号名（函数/类/宏/target）
   - 最小必要代码片段或调用链描述

## 生成信息

- **生成时间**: 2026-02-07
- **仓库版本**: 6.1
- **代码范围**: `foundation/bundlemanager/bundlemanager_cangjie_wrapper`
- **文档规范**: OpenHarmony 工程 Wiki 生成 Agent v1.0
- **评估报告**: [_work/ASSESSMENT.md](_work/ASSESSMENT.md)

## 相关链接

- **主仓库**: [bundlemanager_bundle_framework](https://gitee.com/openharmony/bundlemanager_bundle_framework)
- **依赖仓库**:
  - [ability_ability_runtime](https://gitee.com/openharmony/ability_ability_runtime)
  - [arkcompiler_cangjie_ark_interop](https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
  - [hiviewdfx_hiviewdfx_cangjie_wrapper](https://gitee.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper)
  - [global_global_cangjie_wrapper](https://gitee.com/openharmony-sig/global_global_cangjie_wrapper)
- **API 参考**: [ohos.bundle.bundle_manager (Cangjie)](https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/AbilityKit/cj-apis-bundle_manager.md)

---

*本文档由 Sisyphus Agent 自动生成*
