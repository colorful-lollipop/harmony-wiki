# PIN 认证模块 - 全站导航

> 最后更新：2026-02-06

---

## 新人快速开始

推荐阅读顺序：

1. 📖 [概览页](index.md) - 5 分钟了解项目全貌
2. 📁 [目录结构](02_Directory.md) - 10 分钟熟悉代码组织
3. 🏗️ [架构设计](03_Architecture.md) - 15 分钟理解系统架构
4. 📝 根据 role 选择：
   - **应用开发者** → [对外 Native API](04_Native_API.md)
   - **系统开发者** → [内部 API](05_Inner_API.md)
5. 📦 [构建系统](06_GN_Targets.md) - 理解编译配置
6. 🔒 [安全评审](08_Security_Review.md) - 了解安全机制

---

## 完整目录

### 基础篇

- 📖 [概览页](index.md)
  - 项目定位与边界
  - 核心能力
  - 运行环境
  - 关键概念

- 📁 [目录结构与模块职责](02_Directory.md)
  - 顶层目录树
  - 模块分类（interfaces/frameworks/services/common/sa_profile）
  - 关键文件索引

### 架构篇

- 🏗️ [架构设计](03_Architecture.md)
  - 组件图（Mermaid）
  - 数据流
  - 线程模型
  - 关键时序（注册、认证流程）

- 📝 [内部 API](05_Inner_API.md)
  - 模块接口定义
  - 依赖方向图
  - 稳定性标注
  - 可替换点

### API 篇

- 🔌 [对外 Native API](04_Native_API.md)
  - **注意**：本模块不提供 N-API，为 Native C++ 接口
  - PinAuthRegister 接口
  - IInputer 接口
  - IInputerData 接口
  - 参数校验与错误码
  - 调用示例

### 构建篇

- 📦 [GN Targets](06_GN_Targets.md)
  - 关键 BUILD.gn 文件
  - Target 类型与列表
  - 依赖关系图
  - 特性开关

- 💾 [编译产物](07_Build_Artifacts.md)
  - 产物清单（.so/.a）
  - 安装路径
  - 运行时加载关系

### 安全篇

- 🔒 [安全风险评审](08_Security_Review.md)
  - 攻击面清单
  - 信任边界
  - 可被利用点（5+ 条）
  - 修复建议

### 实战篇

- ❓ [常见问题](09_FAQ.md)
  - 构建问题
  - 运行问题
  - 调试技巧
  - 定位路径

### 附录

- 📋 [工作区说明](README.md)
  - 文档范围
  - 更新方法
  - 版本历史

---

## 按主题快速查找

### 我需要...

| 需求 | 文档 |
|------|------|
| 集成 PIN 认证到应用 | [对外 Native API](04_Native_API.md) |
| 理解 PIN 认证流程 | [架构设计](03_Architecture.md) |
| 编译 pin_auth 模块 | [GN Targets](06_GN_Targets.md) + [编译产物](07_Build_Artifacts.md) |
| 调试 PIN 认证问题 | [常见问题](09_FAQ.md) |
| 了解安全机制 | [安全风险评审](08_Security_Review.md) |
| 添加新功能或修改 | [内部 API](05_Inner_API.md) + [目录结构](02_Directory.md) |
| 理解 IPC 通信 | [架构设计](03_Architecture.md) → IPC 章节 |
| 实现 HDI 驱动 | [对外 Native API](04_Native_API.md) → HDI 章节 |

### 文件位置速查

| 文件类型 | 路径 | 说明 |
|---------|------|------|
| 公共接口头文件 | `interfaces/inner_api/` | 对外暴露的 API |
| 客户端实现 | `frameworks/client/` | PinAuthRegister 实现 |
| IPC 定义 | `frameworks/ipc/` | Proxy/Stub 实现 |
| 服务入口 | `services/sa/` | PinAuthService（SAID: 941） |
| HDI 执行器 | `services/modules/executors/` | 与硬件交互 |
| 输入管理 | `services/modules/inputters/` | Inputer 注册管理 |
| 构建配置 | `bundle.json`, `pin_auth.gni` | 组件描述和特性开关 |

---

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [User IAM 子系统文档](https://docs.openharmony.cn/docs/application-dev/security/pin-auth-overview)
- [Gitee 代码仓库](https://gitee.com/openharmony/useriam_pin_auth)

---

*本文档由 OpenHarmony Wiki 生成 Agent 自动维护*
