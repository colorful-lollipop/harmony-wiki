# Permission Lite Wiki 导航

## 核心文档

| 文档 | 描述 | 必读 |
|------|------|------|
| [README](README.md) | 文档说明与阅读指南 | ✅ |
| [01_Overview](01_Overview.md) | 项目定位、边界、核心能力 | ✅ |
| [02_Architecture](02_Architecture.md) | 组件关系、数据流、线程模型 | ✅ |
| [03_APIs](03_APIs.md) | C 接口、JS API、IPC 认证 | ✅ |
| [04_Build](04_Build.md) | GN Targets、编译产物、依赖 | ✅ |
| [05_Security](05_Security.md) | 攻击面、信任边界、风险分析 | ✅ |

## 附录（可选）

| 文档 | 描述 | 主题 |
|------|------|------|
| [06_Inner_APIs](appendix/06_Inner_APIs.md) | 内部模块接口详解 | 内部架构 |
| [07_Callgraphs](appendix/07_Callgraphs.md) | 关键调用链图谱 | 数据流 |
| [08_Config_Flags](appendix/08_Config_Flags.md) | 宏定义与 Feature Flags | 配置 |

---

## 新人阅读顺序

### 快速入门（30 分钟）

1. `README.md` - 了解文档结构
2. `01_Overview.md` - 理解项目定位
3. `03_APIs.md` - 掌握核心 API 使用

### 深入理解（1-2 小时）

完成快速入门后：

1. `02_Architecture.md` - 理解架构设计
2. `04_Build.md` - 了解编译配置
3. `appendix/07_Callgraphs.md` - 查看调用链

### 安全评审

如有安全相关需求：

1. `05_Security.md` - 阅读安全风险报告
2. 查看代码中的鉴权逻辑

---

## 模块速查

| 模块 | 路径 | 职责 |
|------|------|------|
| interfaces/kits | 对外 C 接口 | NDK API 暴露 |
| interfaces/innerkits | 内部接口 | 系统服务间调用 |
| services/pms | 权限管理服务端 | 权限存储、校验、授予 |
| services/pms_client | PMS 客户端 | IPC 调用 PMS |
| services/pms_base | 服务注册 | SAMGR Feature 注册 |
| services/ipc_auth | IPC 认证 | 进程间通信权限校验 |
| services/js_api | JS API 层 | ACE Lite JSI 封装 |

---

## 文档贡献

如需修改本文档，请遵循以下规范：

1. 所有关键结论必须可追溯到代码证据
2. 接口文档需包含参数说明和返回值
3. 安全分析需包含风险等级和修复建议
4. 保持术语统一（见 `01_Overview.md` 术语表）
