# Wiki 导航与阅读顺序

> OpenHarmony Standard Settings 应用完整文档

---

## 新人阅读顺序

### 第一阶段：快速了解（30 分钟）

1. **[00_Overview.md](00_Overview.md)** ⭐ 从这里开始
   - 项目概览
   - 核心能力
   - 运行环境

2. **[01_Positioning.md](01_Positioning.md)**
   - 项目定位与边界
   - 关键概念

3. **[02_Directory_Structure.md](02_Directory_Structure.md)**
   - 目录结构
   - 模块职责

### 第二阶段：深入理解（2-3 小时）

4. **[03_Architecture.md](03_Architecture.md)** 🏗️
   - 组件架构
   - 数据流
   - 线程模型

5. **[04_NAPI_API.md](04_NAPI_API.md)** 🔌
   - 对外 API（N-API / ANI / CJ FFI）
   - API 清单表
   - 调用链

6. **[05_Inner_API.md](05_Inner_API.md)** 🔧
   - 内部 API
   - 模块依赖
   - 稳定性说明

### 第三阶段：构建与部署（1-2 小时）

7. **[06_GN_Targets.md](06_GN_Targets.md)** 📦
   - GN Targets 清单
   - 依赖关系
   - 构建配置

8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** 📦
   - 编译产物
   - 安装路径
   - 运行时加载

### 第四阶段：安全与运维（1-2 小时）

9. **[08_Security_Audit.md](08_Security_Audit.md)** 🔒
   - 攻击面清单
   - 信任边界
   - 可被利用点

10. **[09_FAQ.md](09_FAQ.md)** ❓
    - 常见问题
    - 调试路径
    - 定位方法

### 附录（按需阅读）

- **[附录 - 调用链图](appendix/Callgraphs.md)** 📊
  - 关键调用链
  - 入口→核心逻辑

- **[附录 - 配置标志](appendix/Config_Flags.md)** ⚙️
  - 关键宏定义
  - Feature flags

---

## 文档状态

| 文档 | 状态 | 完成度 | 最后更新 |
|------|--------|---------|----------|
| README.md | ✅ 完成 | 100% | 2026-02-06 |
| SUMMARY.md | ✅ 完成 | 100% | 2026-02-06 |
| 00_Overview.md | 🚧 草稿 | 50% | 2026-02-06 |
| 01_Positioning.md | 🚧 草稿 | 50% | 2026-02-06 |
| 02_Directory_Structure.md | 🚧 草稿 | 50% | 2026-02-06 |
| 03_Architecture.md | 🚧 草稿 | 50% | 2026-02-06 |
| 04_NAPI_API.md | 🚧 草稿 | 50% | 2026-02-06 |
| 05_Inner_API.md | 🚧 草稿 | 50% | 2026-02-06 |
| 06_GN_Targets.md | 🚧 草稿 | 50% | 2026-02-06 |
| 07_Build_Artifacts.md | 🚧 草稿 | 50% | 2026-02-06 |
| 08_Security_Audit.md | 🚧 草稿 | 50% | 2026-02-06 |
| 09_FAQ.md | 🚧 草稿 | 50% | 2026-02-06 |

**图例**：
- ✅ 完成：文档已完成，包含完整内容
- 🚧 草稿：文档已创建，但内容待完善
- ⏳ 待开始：文档尚未创建

---

## 按角色查看文档

### 应用开发者

如果你是应用开发者，想使用 Settings API：

1. 阅读 **[04_NAPI_API.md](04_NAPI_API.md)** - 了解可用的 JS API
2. 阅读 **[09_FAQ.md](09_FAQ.md)** - 查看常见问题

### 系统开发者

如果你是系统开发者，想了解 Settings 内部实现：

1. 完整阅读 **第一阶段**：快速了解项目
2. 完整阅读 **第二阶段**：深入理解架构和 API
3. 阅读附录中的 **调用链图**：了解关键调用路径

### 安全研究员

如果你是安全研究员，想了解 Settings 安全机制：

1. 阅读 **[08_Security_Audit.md](08_Security_Audit.md)** - 安全评审报告
2. 阅读 **[04_NAPI_API.md](04_NAPI_API.md)** - 了解 API 权限需求

### 构建工程师

如果你是构建工程师，想了解 Settings 构建系统：

1. 阅读 **[06_GN_Targets.md](06_GN_Targets.md)** - GN Targets 清单
2. 阅读 **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 编译产物

---

## 文档规范

- 所有文档均采用中文编写（除非特别说明）
- 所有关键结论均包含代码证据（文件路径+行号）
- 忽略测试相关内容（test/、*_test.*）
- 每篇文档包含：目的 / 适用范围 / 关键结论 / 相关跳转链接

---

**最后更新**：2026-02-06 00:11:23
