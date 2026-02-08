# Hilog Lite 组件 Wiki

## 文档覆盖范围

本文档提供 OpenHarmony DFX 子系统 **hilog_lite** 组件的完整工程文档。

### 覆盖内容

| 主题 | 描述 | 状态 |
|------|------|------|
| 项目概览 | 组件定位、边界、核心能力、运行环境 | ✅ 已完成 |
| 架构设计 | 组件图、数据流、线程模型、关键时序 | ✅ 已完成 |
| Native API | 轻量系统、小型系统、JS/ACE Lite 对外接口 | ✅ 已完成 |
| 内部模块 | 核心模块职责、依赖方向、生命周期 | ✅ 已完成 |
| GN Targets | 构建目标梳理、依赖关系、产物映射 | ✅ 已完成 |
| 编译产物 | .so/.a/可执行文件、安装路径、加载关系 | ✅ 已完成 |
| 安全评审 | 攻击面、信任边界、风险点、修复建议 | ✅ 已完成 |

### 未覆盖内容

- 测试代码分析 (tests/、test/ 等)
- 第三方依赖内部实现 (bounds_checking_function, zlib 等)
- 内核态日志驱动实现 (需查看对应内核仓库)

---

## 系统类型说明

本组件支持两种 OpenHarmony 系统类型：

| 类型 | 内核 | 架构特点 |
|------|------|----------|
| **轻量系统 (mini)** | LiteOS-M | 标准 C 实现，适用于 MCU 设备 |
| **小型系统 (small)** | LiteOS-A | C/C++ 实现，适用于 Cortex-A 系列设备 |

---

## 文档更新方式

### 何时更新 Wiki

当代码发生以下变更时，应同步更新 Wiki：

| 变更类型 | 影响文档 |
|----------|----------|
| 新增/删除/修改 API | `03_Native_API.md` |
| 新增/删除构建 Target | `05_GN_Targets.md`、`06_Build_Outputs.md` |
| 架构调整 | `02_Architecture.md`、`04_Internal_API.md` |
| 安全相关修改 | `07_Security_Review.md` |
| 新增模块依赖 | `04_Internal_API.md`、`05_GN_Targets.md` |

### 更新步骤

1. 在 `wiki/_work/NOTES.md` 中记录变更事实
2. 修改对应的 Wiki 文档
3. 更新 `SUMMARY.md` (如有新增页面)
4. 执行 `Phase 7` 进行一致性校验

---

## 证据来源

所有关键结论均可在代码仓库中找到直接证据：

| 证据类型 | 来源位置 |
|----------|----------|
| API 定义 | `interfaces/native/kits/**/log.h`、`**/hiview_log.h` |
| 构建设置 | `**/BUILD.gn`、`bundle.json` |
| 实现代码 | `frameworks/**/*.{c,cpp}` |
| 服务实现 | `services/**/*.{c,cpp}` |

---

## 版本信息

- **文档生成时间**: 2026-02-06
- **hilog_lite 版本**: 4.0.2
- **OpenHarmony 分支**: master (待确认)
- **最后代码更新**: 基于当前仓库代码

---

## 反馈与贡献

如有以下问题，请通过 OpenHarmony 社区反馈：

- 文档描述与代码实现不符
- 缺少重要的 API 或配置说明
- 安全风险评估需要补充
- 需要澄清的技术细节

相关仓库: [hiviewdfx_hilog_lite](https://gitee.com/openharmony/hiviewdfx_hilog_lite)
