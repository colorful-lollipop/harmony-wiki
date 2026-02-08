# PIN 认证模块 Wiki 文档

> 生成时间：2026-02-06 09:14:52
> 组件版本：4.0
> 子系统：useriam

---

## 文档范围

本文档为 OpenHarmony `pin_auth` 模块提供完整的技术文档，涵盖：

- ✅ 项目定位和核心能力
- ✅ 目录结构和模块职责
- ✅ 架构设计（组件、数据流、时序）
- ✅ 对外 API（Native C++ 接口，非 N-API）
- ✅ 内部 API 和依赖关系
- ✅ GN 构建目标和编译产物
- ✅ 安全风险评审（基于代码证据）
- ✅ 常见问题和调试指南

## 未覆盖范围

- ❌ JavaScript N-API 层（由 `user_auth_framework` 组件提供）
- ❌ 测试相关内容（unit test、fuzz test）
- ❌ 南向 HDI 驱动实现（厂商在 TEE 中实现）
- ❌ SELinux 策略配置（位于其他仓库）

## 文档更新方法

本文档通过以下方式保持与代码同步：

1. **自动生成**：基于代码静态分析生成，包含文件路径和行号引用
2. **证据驱动**：所有结论均附带代码证据（路径+符号）
3. **结构化更新**：每次更新遵循 Phase 0-7 的工作流程

更新文档时，请确保：
- 更新 `wiki/_work/NOTES.md` 中的证据和发现
- 更新相关 .md 文档并保留代码引用
- 运行 `wiki/_work/PLAN.md` 中对应的验证步骤

---

## 目录结构

```
wiki/
├── README.md                      # 本文档
├── SUMMARY.md                     # 全站导航
├── index.md                       # 概览页（新人推荐首先阅读）
├── 01_Overview.md                 # 项目定位、边界、核心能力、运行环境、关键概念
├── 02_Directory.md                # 目录结构与模块职责
├── 03_Architecture.md             # 架构说明（组件图、数据流、线程模型、时序图）
├── 04_Native_API.md               # 对外 Native API（C++ 接口，非 N-API）
├── 05_Inner_API.md               # 内部 API（模块接口、依赖方向、稳定性）
├── 06_GN_Targets.md              # GN 目标梳理（targets、类型、依赖、产物）
├── 07_Build_Artifacts.md          # 编译产物（.so/.a、安装路径、加载关系）
├── 08_Security_Review.md          # 安全风险评审（攻击面、可被利用点、修复建议）
├── 09_FAQ.md                     # 常见问题（构建、运行、调试）
└── _work/                        # 工作区（内部使用，不发布）
    ├── NOTES.md                   # 事实记录
    └── PLAN.md                   # 任务计划
```

---

## 新人阅读顺序

推荐按以下顺序阅读文档：

1. **快速了解**：阅读 `index.md` 获取项目概览
2. **目录熟悉**：阅读 `02_Directory.md` 了解代码组织
3. **架构理解**：阅读 `03_Architecture.md` 理解系统设计
4. **API 使用**：
   - 如果是**应用开发者**：阅读 `04_Native_API.md`
   - 如果是**系统开发者**：阅读 `05_Inner_API.md`
5. **构建调试**：阅读 `06_GN_Targets.md` 和 `09_FAQ.md`
6. **安全审查**：阅读 `08_Security_Review.md`

---

## 关键概念

### PIN 认证（Pin Auth）

PIN（Personal Identification Number）认证是 OpenHarmony 最基础的用户身份认证方式，支持：

- **PIN 设置**：用户首次设置或修改 PIN
- **PIN 删除**：删除已设置的 PIN
- **PIN 认证**：验证用户输入的 PIN 是否正确
- **PIN 修改**：配合 User IAM 框架实现 PIN 更改

### Service Ability（SA）

OpenHarmony 的系统能力机制，pin_auth 作为 SA 运行：

- **SAID**：941 (`SUBSYS_USERIAM_SYS_ABILITY_PINAUTH`)
- **进程**：`useriam`（默认模式）或 `pinauth`（动态加载模式）
- **注册**：通过 `SystemAbility::MakeAndRegisterAbility()` 自动注册

### HDI（Hardware Driver Interface）

硬件驱动接口层，由南向厂商在 TEE 或安全芯片中实现：

- `IAllInOneExecutor`：全功能执行器（注册+认证）
- `ICollector`：数据收集器
- `IVerifier`：验证器
- `IExecutorCallback`：执行器回调

### Inputer（输入器）

由系统级应用（Settings、锁屏等）实现的输入对话框：

- 通过 `RegisterInputer()` 注册到 pin_auth SA
- 通过 `UnRegisterInputer()` 注销
- 通过 `OnGetData()` 回调请求 PIN 数据
- 通过 `OnSetData()` 回调传输 PIN 数据

---

## 参考资源

- **代码仓库**：[useriam_pin_auth](https://gitee.com/openharmony/useriam_pin_auth)
- **相关模块**：
  - [useriam_user_auth_framework](https://gitee.com/openharmony/useriam_user_auth_framework) - JS API 和认证框架
  - [useriam_face_auth](https://gitee.com/openharmony/useriam_face_auth) - 人脸认证
  - [drivers_peripheral](https://gitee.com/openharmony/drivers_peripheral) - 驱动实现
  - [drivers_interface](https://gitee.com/openharmony/drivers_interface) - HDI 接口定义

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本，覆盖 Phase 0-2 内容 |

---

*文档维护者：OpenHarmony Wiki 生成 Agent*
