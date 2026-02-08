# 文档导航 (SUMMARY)

## 快速导航

| 文档 | 说明 | 建议阅读时间 |
|------|------|--------------|
| [README](README.md) | 文档说明、覆盖范围、阅读顺序 | 5 分钟 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、运行环境 | 10 分钟 |
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构与模块职责 | 15 分钟 |
| [02_Architecture](02_Architecture.md) | 架构设计、组件交互、时序图 | 20 分钟 |
| [03_Internal_API](03_Internal_API.md) | 模块接口定义与依赖关系 | 25 分钟 |
| [04_GN_Targets](04_GN_Targets.md) | GN 构建系统、Targets、Feature flags | 20 分钟 |
| [05_Build_Artifacts](05_Build_Artifacts.md) | 编译产物、安装路径、加载关系 | 15 分钟 |
| [06_Security_Review](06_Security_Review.md) | 安全风险评审、权限控制、攻击面 | 30 分钟 |
| [07_Common_Issues](07_Common_Issues.md) | 常见构建/运行/调试问题 | 20 分钟 |
| [08_Config_Flags](08_Config_Flags.md) | Feature flags 与配置宏 | 10 分钟 |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链（附录） | 30 分钟 |

---

## 新人学习路径

### 第一天：了解项目
1. 📖 [README](README.md) (5 min) - 了解文档结构
2. 📖 [00_Overview](00_Overview.md) (10 min) - 项目定位与边界
3. 📖 [01_Directory_Structure](01_Directory_Structure.md) (15 min) - 模块职责

### 第二天：深入架构
1. 📖 [02_Architecture](02_Architecture.md) (20 min) - 分层架构与数据流
2. 📖 [03_Internal_API](03_Internal_API.md) (25 min) - 接口设计
3. 📖 [06_Security_Review](06_Security_Review.md) (30 min) - 安全机制

### 第三天：构建与调试
1. 📖 [04_GN_Targets](04_GN_Targets.md) (20 min) - GN 构建系统
2. 📖 [05_Build_Artifacts](05_Build_Artifacts.md) (15 min) - 编译产物
3. 📖 [07_Common_Issues](07_Common_Issues.md) (20 min) - 常见问题
4. 📖 [08_Config_Flags](08_Config_Flags.md) (10 min) - Feature flags

### 进阶：深度分析
1. 📖 [appendix/Callgraphs](appendix/Callgraphs.md) - 调用链分析
2. 源码阅读（结合 [02_Architecture](02_Architecture.md)）

---

## 专题导航

### 想了解 Profile 实现？
- **A2DP**: [02_Architecture.md](02_Architecture.md#a2dp-profile), [04_GN_Targets.md](04_GN_Targets.md#a2dp-features)
- **HFP**: [02_Architecture.md](02_Architecture.md#hfp-profile)
- **GATT**: [02_Architecture.md](02_Architecture.md#gatt-profile)
- **HID**: [02_Architecture.md](02_Architecture.md#hid-profile)
- **PAN**: [02_Architecture.md](02_Architecture.md#pan-profile)

### 想了解权限控制？
- [06_Security_Review.md](06_Security_Review.md#权限控制机制)
- [02_Architecture.md](02_Architecture.md#权限检查流程)

### 想了解 IPC 通信？
- [02_Architecture.md](02_Architecture.md#ipc-架构)
- [03_Internal_API.md](03_Internal_API.md#ipc-层接口)

### 想了解构建配置？
- [04_GN_Targets.md](04_GN_Targets.md) - Targets 与依赖
- [08_Config_Flags.md](08_Config_Flags.md) - Feature flags
- [05_Build_Artifacts.md](05_Build_Artifacts.md) - 编译产物

### 想排查问题？
- [07_Common_Issues.md](07_Common_Issues.md) - 常见问题
- [06_Security_Review.md](06_Security_Review.md) - 权限错误
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链分析

---

## 架构视角

### 自上而下（从应用到硬件）
1. 应用层 → System Ability (SA)
2. SA → IPC Server ([bluetooth_server](04_GN_Targets.md#bluetooth_server))
3. Server → Service Layer ([btservice](04_GN_Targets.md#btservice))
4. Service → Profile Manager
5. Profile → Protocol Stack ([btstack](04_GN_Targets.md#btstack))
6. Stack → Hardware Interface ([HDI](02_Architecture.md#hdi-层))

详见：[02_Architecture.md](02_Architecture.md#分层架构)

---

## 模块快速索引

| 模块 | 职责 | 文档链接 | 关键文件 |
|------|------|----------|----------|
| **Server 层** | IPC SA 入口、权限检查、请求路由 | [02_Architecture](02_Architecture.md#server-层) | `services/bluetooth/server/` |
| **Service 层** | 业务逻辑、Profile 管理、状态机 | [02_Architecture](02_Architecture.md#service-层) | `services/bluetooth/service/` |
| **IPC 层** | IPC Skeleton/Proxy | [03_Internal_API](03_Internal_API.md#ipc-层接口) | `services/bluetooth/ipc/` |
| **Stack 层** | 蓝牙协议栈实现 | [02_Architecture](02_Architecture.md#stack-层) | `services/bluetooth/stack/` |
| **Hardware 层** | HDI 接口 | [02_Architecture](02_Architecture.md#hdi-层) | `services/bluetooth/hardware/` |
| **权限模块** | 权限检查、Token 管理 | [06_Security_Review](06_Security_Review.md#权限控制机制) | `services/bluetooth/service/src/permission/` |

---

## 调试指南

### 日志定位
- 日志开关：[07_Common_Issues.md](07_Common_Issues.md#日志开关)
- 日志级别：[07_Common_Issues.md](07_Common_Issues.md#日志级别)
- HiSysEvent：[06_Security_Review.md](06_Security_Review.md#hisysevent-事件)

### 构建问题
- 编译错误：[07_Common_Issues.md](07_Common_Issues.md#编译错误)
- 链接错误：[07_Common_Issues.md](07_Common_Issues.md#链接错误)
- Feature flags：[08_Config_Flags.md](08_Config_Flags.md)

### 运行时问题
- 服务启动失败：[07_Common_Issues.md](07_Common_Issues.md#服务启动失败)
- Profile 连接失败：[07_Common_Issues.md](07_Common_Issues.md#profile-连接失败)
- 权限拒绝：[06_Security_Review.md](06_Security_Review.md#权限错误)

---

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| SA | System Ability | OpenHarmony 系统能力框架 |
| HDI | Hardware Device Interface | 硬件设备接口 |
| A2DP | Advanced Audio Distribution Profile | 高级音频分发配置文件 |
| HFP | Hands-Free Profile | 免提配置文件 |
| AVRCP | Audio/Video Remote Control Profile | 音视频远程控制配置文件 |
| GATT | Generic Attribute Profile | 通用属性配置文件 |
| HID | Human Interface Device | 人机接口设备 |
| PAN | Personal Area Networking | 个人区域网络 |
| HCI | Host Controller Interface | 主机控制器接口 |
| L2CAP | Logical Link Control and Adaptation Protocol | 逻辑链路控制和适配协议 |
| SMP | Security Manager Protocol | 安全管理协议 |

---

## 相关资源

### 官方文档
- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [蓝牙子系统文档](https://docs.openharmony.cn/docs/documentation/doc-releases/master/application-dev/reference/apis-ohos-bluetooth-ble-0000001778099644)

### 相关仓库
- `foundation/communication/bluetooth` - 蓝牙框架（含 N-API 绑定）
- `drivers_peripheral_bluetooth` - 蓝牙驱动实现

---

## 反馈与改进

本文档持续维护，欢迎反馈：
- 📝 文档错误或过时
- 🔍 缺少关键内容
- 💡 改进建议

反馈方式：提交 Issue 或 Pull Request
