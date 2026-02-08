# NFC Wiki 文档导航

## 新手指南

建议按以下顺序阅读：

1. **[概述](00_Overview.md)** - 了解 NFC 组件是什么、能做什么
2. **[目录结构](02_Directory_Structure.md)** - 熟悉代码组织方式
3. **[N-API 接口](03_NAPI_Interfaces.md)** - 如果是应用开发者，重点阅读
4. **[架构设计](01_Architecture.md)** - 理解系统架构设计
5. **[构建系统](05_GN_Targets.md)** - 了解如何构建
6. **[安全评估](06_Security_Assessment.md)** - 了解安全注意事项

## 完整文档索引

### 核心文档

| 文档 | 路径 | 描述 |
|------|------|------|
| [概述](00_Overview.md) | ./00_Overview.md | 项目定位、核心能力、运行环境 |
| [架构设计](01_Architecture.md) | ./01_Architecture.md | 组件架构、数据流、线程模型、时序图 |
| [目录结构](02_Directory_Structure.md) | ./02_Directory_Structure.md | 源码目录组织与模块职责 |
| [N-API 接口](03_NAPI_Interfaces.md) | ./03_NAPI_Interfaces.md | JS API 参考、参数、错误码 |
| [内部接口](04_Inner_API.md) | ./04_Inner_API.md | 内部模块接口与依赖关系 |
| [构建系统](05_GN_Targets.md) | ./05_GN_Targets.md | GN 构建目标、依赖、配置 |
| [安全评估](06_Security_Assessment.md) | ./06_Security_Assessment.md | 攻击面分析、风险点、修复建议 |
| [构建产物](07_Build_Artifacts.md) | ./07_Build_Artifacts.md | 输出文件、安装路径、运行时加载 |
| [问题排查](08_Troubleshooting.md) | ./08_Troubleshooting.md | 常见问题与调试方法 |

### 附录

| 文档 | 路径 | 描述 |
|------|------|------|
| [调用链附录](appendix/Callgraphs.md) | ./appendix/Callgraphs.md | 关键调用链详细分析 |
| [配置标志附录](appendix/Config_Flags.md) | ./appendix/Config_Flags.md | 编译期配置选项说明 |

## 按角色导航

### 应用开发者
重点关注 N-API 接口文档：
- [N-API 接口 - Controller](03_NAPI_Interfaces.md#controller) - NFC 开关、状态监听
- [N-API 接口 - Tag](03_NAPI_Interfaces.md#tag) - 标签读写
- [N-API 接口 - CardEmulation](03_NAPI_Interfaces.md#cardemulation) - 卡模拟

### 系统开发者
重点关注内部实现：
- [架构设计](01_Architecture.md) - 系统架构
- [内部接口](04_Inner_API.md) - 模块间接口
- [安全评估](06_Security_Assessment.md) - 安全机制

### 构建/集成工程师
重点关注构建相关：
- [构建系统](05_GN_Targets.md) - GN 构建配置
- [构建产物](07_Build_Artifacts.md) - 输出文件说明

### 安全审计人员
重点关注安全相关：
- [安全评估](06_Security_Assessment.md) - 完整安全分析
- [N-API 接口 - 权限](03_NAPI_Interfaces.md#权限说明) - 接口权限要求

## 关键快捷链接

### 常用接口
- [getNfcState](03_NAPI_Interfaces.md#getnfcstate) - 获取 NFC 状态
- [enableNfc/disableNfc](03_NAPI_Interfaces.md#enablenfcdisablenfc) - 开关 NFC
- [on/off](03_NAPI_Interfaces.md#onoff) - 注册/注销状态监听
- [Tag 读写操作](03_NAPI_Interfaces.md#tag-操作) - 标签操作
- [HCE 卡模拟](03_NAPI_Interfaces.md#hce-卡模拟) - 卡模拟功能

### 关键错误码
- [错误码汇总](03_NAPI_Interfaces.md#错误码汇总) - 完整错误码列表
- [NFC 状态错误](03_NAPI_Interfaces.md#nfc-状态错误) - 3100100+
- [标签 I/O 错误](03_NAPI_Interfaces.md#标签-io-错误) - 3100200+
- [卡模拟错误](03_NAPI_Interfaces.md#卡模拟错误) - 3100300+

### 架构组件
- [NfcService](01_Architecture.md#nfcservice) - 核心服务
- [NCI Adapter](01_Architecture.md#nci-adapter) - 硬件适配层
- [TagDispatcher](01_Architecture.md#tagdispatcher) - 标签分发
- [CeService](01_Architecture.md#ceservice) - 卡模拟服务

