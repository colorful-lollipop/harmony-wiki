# hdc 文档导航

## 新人阅读路线

按顺序阅读以下文档，可快速了解 hdc 项目：

### 1. 入门文档（必读）
1. **[本页](./00_Overview.md)** - 项目定位、核心能力、运行环境
   - 了解 hdc 是什么，解决什么问题
   - 核心能力和关键概念
   - 运行环境要求

2. **[目录结构](./02_Directory_Structure.md)** - 模块职责、文件组织
   - 了解代码组织结构
   - 各模块的职责边界
   - 文件分类说明

3. **[架构说明](./03_Architecture.md)** - 组件图、数据流、线程模型
   - Client/Server/Daemon 三部分架构
   - 通信协议和数据流
   - 关键时序图

### 2. 深入文档（根据需求选择）

4. **[对外 API](./04_External_API.md)** - JDWP 注册、API 清单
   - 如果需要了解对外接口
   - JDWP 连接机制
   - API 参数和错误码

5. **[内部 API](./05_Internal_API.md)** - 模块接口、依赖方向
   - 如果需要了解模块间接口
   - Common/Host/Daemon 模块关系
   - 稳定/不稳定接口标注

6. **[GN 目标](./06_GN_Targets.md)** - 构建系统、targets、产物
   - 如果需要编译或定制构建
   - Feature flags 和配置选项
   - Target 依赖关系

7. **[编译产物](./07_Build_Artifacts.md)** - 安装路径、运行时加载
   - 如果需要了解部署
   - 输出文件和安装位置
   - 运行时加载关系

### 3. 安全文档（安全审计必读）

8. **[安全评审](./08_Security_Review.md)** - 攻击面、风险点、修复建议
   - 了解安全威胁模型
   - 信任边界和数据流
   - 可被利用点和修复建议

### 4. 参考文档

9. **[常见问题](./09_FAQ.md)** - 构建/运行/调试问题
   - 常见问题排查
   - 定位路径和方法
   - 已知限制和解决方案

### 5. 附录文档

10. **[调用链图](./appendix/Callgraphs.md)** - 关键调用链
    - 入口→核心逻辑
    - 跨模块调用流程

11. **[配置标志](./appendix/Config_Flags.md)** - 关键宏和 feature flags
    - 编译选项说明
    - 平台特定宏
    - Feature flags 列表

## 文档结构树

```
wiki/
├── README.md                      # 本文档 - 概述和导航
├── SUMMARY.md                     # 本文档 - 完整导航
├── 00_Overview.md                 # 项目定位、核心能力
├── 01_Project_Positioning.md       # 项目边界、核心能力
├── 02_Directory_Structure.md       # 目录结构、模块职责
├── 03_Architecture.md             # 架构说明（组件图、数据流）
├── 04_External_API.md             # 对外 API（JDWP）
├── 05_Internal_API.md              # 内部 API（模块接口）
├── 06_GN_Targets.md             # GN 目标梳理
├── 07_Build_Artifacts.md          # 编译产物
├── 08_Security_Review.md         # 安全风险评审
├── 09_FAQ.md                    # 常见问题
└── appendix/
    ├── Callgraphs.md              # 调用链图
    └── Config_Flags.md            # 配置标志
```

## 阅读建议

### 快速入门（30 分钟）
- 阅读 00_Overview.md
- 阅读 02_Directory_Structure.md
- 阅读 03_Architecture.md

### 功能开发（1-2 小时）
- 阅读 04_External_API.md（如果涉及对外接口）
- 阅读 05_Internal_API.md（如果修改内部模块）
- 阅读 06_GN_Targets.md（如果需要修改构建）

### 安全审计（2-3 小时）
- 阅读 08_Security_Review.md（安全必读）
- 查阅附录 Callgraphs.md（了解关键调用链）

### 问题排查（按需）
- 阅读 09_FAQ.md（遇到问题时）
- 查阅 07_Build_Artifacts.md（了解部署）

## 术语表

| 术语 | 说明 |
|------|------|
| hdc | OpenHarmony Device Connector，设备连接工具 |
| hdcd | hdc daemon，设备端守护进程 |
| Client | hdc 客户端，运行在开发机 PC 上 |
| Server | hdc 服务端，运行在开发机 PC 上，管理 client-daemon 通信 |
| Daemon | hdc 守护进程，运行在 OpenHarmony 设备上 |
| JDWP | Java Debug Wire Protocol，Java 调试线协议 |
| Session | 会话，一次完整的 client-daemon 连接 |
| Channel | 通道，会话内的逻辑通信管道 |
| PSK | Pre-Shared Key，预共享密钥 |
| UDS | Unix Domain Socket，Unix 域套接字 |
| HUKS | Huawei Universal Keystore，华为通用密钥库 |

## 版本信息

- **本文档版本**: 1.0
- **基于代码**: master 分支（2026-02-06）
- **hdc 版本**: 3.1
- **最后更新**: 2026-02-06 09:48
