# DHCP 组件文档目录

本文档目录为 OpenHarmony DHCP 组件（@ohos/dhcp）提供完整的导航索引。

---

## 新人阅读路线

### 第一步：快速入门
1. **[README](README.md)** - 文档说明与使用指南
2. **[00_Overview](00_Overview.md)** - 项目概览（5分钟了解核心概念）

### 第二步：深入理解
3. **[01_Project_Positioning](01_Project_Positioning.md)** - 项目定位、边界、核心能力
4. **[03_Architecture](03_Architecture.md)** - 系统架构、组件交互、数据流

### 第三步：接口开发
5. **[04_C_API_Reference](04_C_API_Reference.md)** - C API 完整参考
6. **[05_Inner_API](05_Inner_API.md)** - 内部接口与稳定性

### 第四步：构建与集成
7. **[06_Build_System](06_Build_System.md)** - GN 构建系统详解
8. **[07_Compile_Artifacts](07_Compile_Artifacts.md)** - 编译产物与运行时

### 第五步：安全与调试
9. **[08_Security_Review](08_Security_Review.md)** - 安全风险评审
10. **[09_Common_Issues](09_Common_Issues.md)** - 常见问题与调试

---

## 完整文档列表

### 基础文档
| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [README](README.md) | 文档说明与使用指南 | 5分钟 |
| [00_Overview](00_Overview.md) | 项目概览 | 5分钟 |
| [01_Project_Positioning](01_Project_Positioning.md) | 项目定位与边界 | 10分钟 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构详解 | 5分钟 |

### 架构与接口
| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [03_Architecture](03_Architecture.md) | 系统架构、数据流、线程模型 | 20分钟 |
| [04_C_API_Reference](04_C_API_Reference.md) | C API 完整参考（17个方法） | 30分钟 |
| [05_Inner_API](05_Inner_API.md) | 内部 API 与接口稳定性 | 15分钟 |

### 构建与部署
| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [06_Build_System](06_Build_System.md) | GN 构建系统、Targets、依赖 | 20分钟 |
| [07_Compile_Artifacts](07_Compile_Artifacts.md) | 编译产物、安装路径、运行时 | 15分钟 |

### 安全与运维
| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [08_Security_Review](08_Security_Review.md) | 安全风险评审（6大类风险） | 25分钟 |
| [09_Common_Issues](09_Common_Issues.md) | 常见问题与调试指南 | 15分钟 |

### 附录
| 文档 | 说明 | 阅读时间 |
|------|------|----------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 | 20分钟 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 关键配置与编译开关 | 10分钟 |

---

## 按角色阅读

### 接口开发者
核心路径：
- [00_Overview](00_Overview.md) → [04_C_API_Reference](04_C_API_Reference.md) → [05_Inner_API](05_Inner_API.md)

### 系统集成开发者
核心路径：
- [00_Overview](00_Overview.md) → [03_Architecture](03_Architecture.md) → [06_Build_System](06_Build_System.md) → [07_Compile_Artifacts](07_Compile_Artifacts.md)

### 安全审计员
核心路径：
- [00_Overview](00_Overview.md) → [03_Architecture](03_Architecture.md) → [08_Security_Review](08_Security_Review.md)

### 测试工程师
核心路径：
- [03_Architecture](03_Architecture.md) → [04_C_API_Reference](04_C_API_Reference.md) → [09_Common_Issues](09_Common_Issues.md)

---

## 关键概念索引

### 核心组件
- **DHCP Client**: DHCP 客户端服务（SA ID: 1126）
- **DHCP Server**: DHCP 服务器服务（SA ID: 1127）
- **dhcp_sdk**: 应用接口 SDK
- **dhcp_utils**: 工具库（权限、SA管理、ARP检查）

### 接口类型
- **C API**: 对外 C 语言接口（interfaces/kits/c/）
- **C++ SDK**: 对外 C++ 接口（frameworks/native/）
- **IPC 接口**: 跨进程通信接口（IRemoteBroker）

### 构建产物
- **共享库**: libdhcp_sdk.z.so, libdhcp_client.z.so, libdhcp_server.z.so
- **静态库**: libdhcp_client_static.a, libdhcp_server_static.a
- **SA 配置**: 1126.json, 1127.json

### 安全相关
- **权限**: ohos.permission.NETWORK_DHCP
- **IPC 安全**: TokenID 检查、权限验证
- **数据安全**: 敏感信息脱敏、日志控制

---

## 快速查找

### 查找 API 文档
- C API: [04_C_API_Reference](04_C_API_Reference.md)
- 内部接口: [05_Inner_API](05_Inner_API.md)

### 查找构建信息
- Targets 列表: [06_Build_System](06_Build_System.md) - Section 2
- 依赖关系: [06_Build_System](06_Build_System.md) - Section 3
- 产物清单: [07_Compile_Artifacts](07_Compile_Artifacts.md) - Section 1

### 查找安全信息
- 风险清单: [08_Security_Review](08_Security_Review.md) - Section 2
- 攻击面分析: [08_Security_Review](08_Security_Review.md) - Section 3

### 查找调试信息
- 常见问题: [09_Common_Issues](09_Common_Issues.md)
- 调试路径: [09_Common_Issues](09_Common_Issues.md) - Section 2

---

## 文档更新记录

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2025-02-06 | 初始版本，覆盖核心内容 |

---

## 相关资源

- **仓库**: https://gitee.com/openharmony/communication_dhcp
- **组件描述**: bundle.json
- **源代码**: /foundation/communication/dhcp
