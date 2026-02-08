# MSDP 设备状态感知框架 - Wiki 文档

## 目的

本 Wiki 文档为 OpenHarmony MSDP 设备状态感知框架（device_status）提供完整的技术参考文档，帮助开发者快速理解项目结构、API 接口、编译系统和安全机制。

---

## 📚 文档导航

本章节为新人提供阅读路线：

### 快速入门路径

1. **[01_Overview](01_Overview.md)** - 项目概览与核心能力
   - 了解项目定位、核心能力、运行环境
   - 阅读时间：~5 分钟

2. **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构详解
   - 理解各模块职责和依赖关系
   - 阅读时间：~10 分钟

3. **[03_Architecture](03_Architecture.md)** - 架构设计
   - 理解组件关系、数据流、线程模型
   - 阅读时间：~15 分钟

4. **[04_N-API_Reference](04_N-API_Reference.md)** - 对外 JavaScript API
   - 查找所需 API 和调用方式
   - 阅读时间：~30 分钟

5. **[05_Inner_API](05_Inner_API.md)** - 内部 API
   - 理解模块间接口和依赖方向
   - 阅读时间：~20 分钟

6. **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标
   - 了解编译目标、依赖关系、产物
   - 阅读时间：~15 分钟

7. **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物
   - 了解输出文件、安装路径、加载关系
   - 阅读时间：~10 分钟

8. **[08_Security_Review](08_Security_Review.md)** - 安全风险评审
   - 了解攻击面、安全机制、风险点
   - 阅读时间：~20 分钟

9. **[09_FAQ](09_FAQ.md)** - 常见问题
   - 解决开发、构建、运行、调试问题
   - 阅读时间：~10 分钟

### 按主题导航

| 主题 | 相关文档 |
|------|----------|
| **项目概览** | [01_Overview](01_Overview.md) |
| **代码结构** | [02_Directory_Structure](02_Directory_Structure.md) |
| **架构设计** | [03_Architecture](03_Architecture.md) |
| **JS API** | [04_N-API_Reference](04_N-API_Reference.md) |
| **内部接口** | [05_Inner_API](05_Inner_API.md) |
| **构建系统** | [06_GN_Targets](06_GN_Targets.md) |
| **编译产物** | [07_Build_Artifacts](07_Build_Artifacts.md) |
| **安全评审** | [08_Security_Review](08_Security_Review.md) |
| **常见问题** | [09_FAQ](09_FAQ.md) |

### 附录

- **[appendix/Callgraphs](appendix/Callgraphs.md)** - 关键调用链图（可选）

---

## 📝 文档说明

### 适用范围

本 Wiki 文档涵盖 `device_status` 模块的以下方面：

- ✅ 项目概览与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构设计（组件图、数据流、线程模型）
- ✅ N-API（JavaScript）接口文档
  - ✅ 内部 API 接口定义
- ✅ GN 构建目标梳理
- ✅ 编译产物说明
- ✅ 安全风险评审（基于代码证据）

### 未覆盖范围

- ❌ 测试相关代码的详细说明
- ❌ 第三方依赖的详细文档（如需参考第三方文档）
- ❌ 硬件适配层的细节（参考硬件文档）

### 覆盖的 N-API 模块

本 Wiki 文档完整覆盖以下 **11 个 N-API 模块**：

1. **Stationary** - 设备静止状态订阅（legacy）
2. **Device Status V1** - 设备状态 v1，支持姿态获取
3. **Motion** - 运动感知（操作手、站立检测、远程拍照、握持状态）
4. **Distance Measurement** - 距离测量（BLE/WIFI/UWB）
5. **On-Screen** - 屏幕感知（控制事件、页面内容获取）
6. **Screen Event** - 屏幕事件订阅
7. **User Status (Underage)** - 用户年龄模型（儿童/其他）
8. **Boomerang** - 元数据绑定（图片编码/解码）
9. **Drag Interaction** - 拖拽交互（拖拽监听、数据摘要、状态设置）
10. **Input Device Cooperation** - 输入设备协同（跨设备鼠标键盘）
11. **Coordination (Legacy)** - 设备协同（legacy 协同意图）

### 架构概览

本模块采用 **插件化架构**：

- **Intention 框架**：核心业务逻辑层，管理所有功能插件
- **基础设施层**：提供 IPC、调度、设备管理、适配器
- **插件系统**：拖拽、协同、静止、屏幕感知、Boomerang

### 安全机制

- **AccessToken 验证**：区分 Native/Shell/HAP 应用
- **系统应用验证**：验证系统应用身份
- **权限检查**：基于 `ohos.permission.*` 命名权限
- **Socket FD 分配**：资源管理，防止耗尽攻击
- **白名单机制**：屏幕内容获取应用白名单

---

## 📖 工作区

Wiki 生成过程中的所有笔记和进度记录均保存在 `wiki/_work/` 目录：

- **NOTES.md** - 事实记录、代码证据、模块清单
- **PLAN.md** - 任务拆解、进度跟踪、阻塞处理规则

---

## 🔗 相关链接

- **OpenHarmony 文档**：https://docs.openharmony.cn/
- **MSDP 子系统文档**：https://docs.openharmony.cn/#/msdp/
- **N-API 开发指南**：https://docs.openharmony.cn/#/application-dev/napi/

---

## 📚 维护者

本 Wiki 文档基于以下代码快照生成：

- **生成时间**：2026-02-06
- **代码版本**：HEAD commit of `/base/msdp/device_status`

### 如何保持文档同步

1. **代码变更时**：更新相关章节
2. **重大功能添加时**：新增或扩展对应文档
3. **架构调整时**：更新架构图和数据流
4. **安全修复后**：更新安全风险评审章节

---

## 📖 约程问题

如需更新或维护本 Wiki 文档，请参考 `wiki/_work/PLAN.md` 中的任务列表和进度跟踪。

---

## 🎯 快速开始

**新人推荐阅读顺序**：

1. **第一步**：阅读 [01_Overview](01_Overview.md) - 快速了解项目
2. **第二步**：阅读 [02_Directory_Structure.md](02_Directory_Structure.md) - 理解代码结构
3. **第三步**：阅读 [04_N-API_Reference](04_N-API_Reference.md) - 查找所需 API
4. **第四步**：阅读 [05_Inner_API.md](05_Inner_API.md) - 了解内部接口
5. **按需深入**：阅读 [03_Architecture](03_Architecture.md) - 理解架构设计
6. **其他参考**：[06_GN_Targets](06_GN_Targets.md)、[07_Build_Artifacts](07_Build_Artifacts.md)、[08_Security_Review](08_Security_Review.md)、[09_FAQ](09_FAQ.md)

---

## 📊 文档状态

- ✅ 所有核心文档已完成
- ✅ 基于实际代码证据
- ✅ 覆盖 11 个 N-API 模块
- ✅ 包含完整架构图和组件关系
- ✅ 包含安全风险评审（6 类风险，可被利用点）

**生成信息**：
- **探索方式**：5 个并行后台探索任务
- **文档数量**：11 个主要 Wiki 文档 + 2 个工作文件
- **代码分析**：基于实际代码，无猜测内容
- **证据追溯**：所有结论都有文件路径、符号名、行号

---

**祝使用愉快！📚**
