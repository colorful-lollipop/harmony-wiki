# 03_CodeMap - 目录结构与代码地图

本文档帮助开发者快速定位 SOC 统一调频部件的核心代码文件，理解目录结构与模块职责的对应关系。

## 顶层目录概览

```
soc_perf/                          # 组件根目录
├── common/                        # 公共头文件目录
├── figures/                       # 架构图等资源文件
├── interfaces/                    # 对外接口定义
│   └── inner_api/
│       └── socperf_client/       # IPC 客户端库
├── profile/                       # XML 配置文件
├── sa_profile/                   # SA 配置
├── services/                     # 核心服务实现
│   ├── core/                     # 核心业务逻辑
│   ├── server/                   # SA 服务端实现
│   └── dfx/                      # 诊断追踪
├── test/                         # 测试代码（仅用于参考）
├── BUILD.gn                       # GN 构建入口
├── bundle.json                    # 组件配置
└── README_ZH.md                  # 中文项目说明
```

## 核心代码目录详解

### services/core - 核心业务逻辑

**职责**：调频仲裁、配置解析、内核接口封装

| 文件路径 | 类型 | 职责描述 |
|----------|------|----------|
| `services/core/include/socperf.h` | header | SocPerf 核心类声明 |
| `services/core/src/socperf.cpp` | source | PerfRequest/LimitBoost 主逻辑实现 |
| `services/core/include/socperf_config.h` | header | SocPerfConfig 配置管理类声明 |
| `services/core/src/socperf_config.cpp` | source | XML 配置解析实现（993行） |
| `services/core/include/socperf_common.h` | header | 公共数据结构定义 |
| `services/core/include/socperf_thread_wrap.h` | header | FFRT 线程队列包装声明 |
| `services/core/src/socperf_thread_wrap.cpp` | source | 调频仲裁与内核调用实现 |

### services/server - 服务端实现

**职责**：IPC Stub、权限校验、SystemAbility 生命周期

| 文件路径 | 类型 | 职责描述 |
|----------|------|----------|
| `services/server/include/socperf_server.h` | header | SocPerfServer SA 类声明 |
| `services/server/src/socperf_server.cpp` | source | IPC 接口实现与权限检查 |

### services/dfx - 诊断追踪

**职责**：HiTrace 跟踪链支持

| 文件路径 | 类型 | 职责描述 |
|----------|------|----------|
| `services/dfx/include/socperf_hitrace_chain.h` | header | SocPerfHiTraceChain 声明 |
| `services/dfx/src/socperf_hitrace_chain.cpp` | source | HiTrace RAII 封装实现 |

### interfaces/inner_api/socperf_client - 客户端接口

**职责**：IPC 客户端代理、连接管理、单例封装

| 文件路径 | 类型 | 职责描述 |
|----------|------|----------|
| `interfaces/inner_api/socperf_client/include/socperf_client.h` | header | SocPerfClient 类声明 |
| `interfaces/inner_api/socperf_client/include/socperf_action_type.h` | header | ActionType 枚举定义 |
| `interfaces/inner_api/socperf_client/src/socperf_client.cpp` | source | IPC 代理实现 |
| `interfaces/inner_api/socperf_client/ISocPerf.idl` | IDL | IPC 接口定义 |
| `interfaces/inner_api/socperf_client/libsocperf_client.versionscript` | version | 符号版本脚本 |

### profile - 配置文件

**职责**：XML 配置定义

| 文件路径 | 用途 |
|----------|------|
| `profile/socperf_resource_config.xml` | 资源定义（CPU/GPU/DDR/NPU等） |
| `profile/socperf_boost_config.xml` | 提频场景配置 |

### sa_profile - SA 配置

| 文件路径 | 用途 |
|----------|------|
| `sa_profile/1906.json` | SA ID 1906 注册信息 |

## 代码导航图

### 快速定位

| 需求 | 文件路径 | 行号 |
|------|----------|------|
| 添加新的调频接口 | `services/core/include/socperf.h` | 28-39 |
| 修改权限检查逻辑 | `services/server/src/socperf_server.cpp` | 185-212 |
| 添加新的 cmdId | `profile/socperf_boost_config.xml` | - |
| 修改配置解析 | `services/core/src/socperf_config.cpp` | 1-993 |
| 线程安全修改 | `services/core/include/socperf.h` | 57-60 |
| IPC 接口定义 | `interfaces/inner_api/socperf_client/ISocPerf.idl` | - |
| 客户端连接管理 | `interfaces/inner_api/socperf_client/src/socperf_client.cpp` | 47-78 |

### 功能到代码的映射

| 功能 | 关键文件 | 关键类/函数 |
|------|----------|-------------|
| 性能提频 | `socperf.cpp` | `PerfRequest()`, `PerfRequestEx()` |
| 功耗限频 | `socperf.cpp` | `PowerLimitBoost()` |
| 热限频 | `socperf.cpp` | `ThermalLimitBoost()` |
| 限频请求 | `socperf.cpp` | `LimitRequest()` |
| 配置加载 | `socperf_config.cpp` | `Init()`, `LoadConfigXmlFile()` |
| 线程封装 | `socperf_thread_wrap.cpp` | `SocPerfThreadWrap` |
| 权限校验 | `socperf_server.cpp` | `HasPerfPermission()` |
| 客户端连接 | `socperf_client.cpp` | `CheckClientValid()` |

## 关键类索引

### SocPerf - 核心调频类

**文件**：`services/core/include/socperf.h`

| 成员类型 | 成员名称 | 用途 |
|----------|----------|------|
| 公共方法 | `Init()`, `PerfRequest()`, `PerfRequestEx()` | 初始化与调频接口 |
| 私有方法 | `DoFreqActions()`, `CheckTimeInterval()` | 仲裁与执行 |
| 成员变量 | `socperfThreadWrap_` | 线程封装实例 |
| 成员变量 | `socPerfConfig_` | 配置管理实例 |
| 成员变量 | `perfRequestEnable_` | 提频使能标志 |

### SocPerfServer - SA 服务类

**文件**：`services/server/include/socperf_server.h`

| 成员类型 | 成员名称 | 用途 |
|----------|----------|------|
| 继承 | `SystemAbility` | SA 框架基类 |
| 继承 | `SocPerfStub` | IPC Stub 接口 |
| 私有成员 | `socPerf_` | 核心调频实例 |
| 私有成员 | `permissionCache_` | 权限缓存 |
| 私有方法 | `HasPerfPermission()` | 权限检查 |

### SocPerfClient - 客户端代理类

**文件**：`interfaces/inner_api/socperf_client/include/socperf_client.h`

| 成员类型 | 成员名称 | 用途 |
|----------|----------|------|
| 静态方法 | `GetInstance()` | 单例获取 |
| 公共方法 | `PerfRequest()`, `LimitRequest()` | IPC 调用接口 |
| 私有方法 | `CheckClientValid()` | 连接有效性检查 |
| 私有方法 | `AddPidAndTidInfo()` | PID/TID 注入 |
| 成员变量 | `mutex_` | 线程安全锁 |

## 数据结构速查

### 动作类型枚举

**文件**：`interfaces/inner_api/socperf_client/include/socperf_action_type.h`

```cpp
enum ActionType : uint32_t {
    ACTION_TYPE_PERF,      // 性能提频
    ACTION_TYPE_POWER,     // 功耗限频
    ACTION_TYPE_THERMAL,   // 热限频
    ACTION_TYPE_PERFLVL,   // 性能级别
    ACTION_TYPE_BATTERY,   // 电池相关
    ACTION_TYPE_MAX
};
```

### 配置常量

**文件**：`services/core/include/socperf_common.h`

| 常量 | 值 | 用途 |
|------|-----|------|
| `SOCPERF_RESOURCE_CONFIG_XML` | `etc/soc_perf/socperf_resource_config.xml` | 资源配置文件路径 |
| `SOCPERF_BOOST_CONFIG_XML` | `etc/soc_perf/socperf_boost_config.xml` | 提频配置文件路径 |
| `RES_ID_MIN` | 1000 | 最小资源 ID |
| `RES_ID_MAX` | 5999 | 最大资源 ID |

---

## 相关文档

- 项目概览：[01_Overview](01_Overview.md)
- 架构设计：[02_Architecture](02_Architecture.md)
- 内部实现：[08_Internals](08_Internals.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
