# 目录结构与代码地图

> 目的：帮助新人快速定位核心代码位置，理解文件组织关系
>  
> **受众**: 新人学习者、代码维护者

---

## 1. 顶层目录结构

```
/base/hiviewdfx/hidumper
├── BUILD.gn              # 根构建配置
├── bundle.json           # Bundle 配置
├── hidumper.gni          # GN 变量定义
├── hidumper.yaml         # 事件配置
├── LICENSE               # 许可证
├── OAT.xml              # OAT 配置
├── README.md            # 英文 README
├── README_zh.md         # 中文 README
│
├── client/              # 客户端代码
├── frameworks/          # 框架核心代码
├── interfaces/          # 对外接口
├── sa_profile/          # SA 配置文件
├── services/            # 服务实现
├── test/               # 测试代码 (不纳入文档)
├── utils/              # 工具类
├── figures/            # 文档图片
└── wiki/               # 本文档
```

---

## 2. 核心目录详解

### 2.1 client/ - 客户端入口

```
client/
└── native/
    ├── main.cpp                   # 程序入口 (main 函数)
    ├── dump_client_main.cpp       # CLI 参数解析与 IPC 调用
    └── dump_client_main.h         # 头文件
```

**关键文件导航**:
| 文件 | 职责 | 新人关注 |
|------|------|---------|
| `main.cpp:38` | 程序入口 | ⭐ 调试起点 |
| `dump_client_main.cpp:38` | 参数解析 | ⭐ CLI 入口 |
| `dump_client_main.cpp:65` | IPC 调用 | ⭐ 服务端调用点 |

---

### 2.2 frameworks/native/ - 框架核心

```
frameworks/native/
├── BUILD.gn                    # 框架层构建配置
├── dump_controller.h           # 控制常量定义
├── dump_utils.cpp/h            # 工具函数
│
├── include/                    # 头文件目录
│   ├── common/                # 公共数据结构
│   │   ├── dump_cfg.h         # DumpCfg 配置结构体
│   │   ├── dumper_constant.h  # 常量定义
│   │   ├── dumper_opts.h      # 命令行选项结构
│   │   ├── dumper_parameter.h # 参数结构
│   │   └── option_args.h      # 参数选项
│   │
│   ├── executor/              # Dumper 执行器
│   │   ├── hidumper_executor.h    # 执行器基类 (关键!)
│   │   ├── cmd_dumper.h          # 命令执行 Dumper
│   │   ├── cpu_dumper.h          # CPU Dumper
│   │   ├── memory_dumper.h       # 内存 Dumper
│   │   ├── file_stream_dumper.h  # 文件流 Dumper
│   │   ├── sa_dumper.h           # System Ability Dumper
│   │   ├── traffic_dumper.h      # 网络流量 Dumper
│   │   ├── fd_output.h           # FD 输出
│   │   ├── zip_output.h          # ZIP 输出
│   │   ├── dumper_group.h        # Dumper 组合
│   │   └── memory/              # 内存相关 Dumper
│   │       ├── memory_info.h
│   │       ├── dump_jsheap_info.h
│   │       └── parse/           # 内存解析器
│   │           ├── parse_smaps_info.h
│   │           └── parse_meminfo.h
│   │
│   ├── factory/               # 工厂模式
│   │   ├── executor_factory.h     # 工厂基类
│   │   ├── cmd_dumper_factory.h
│   │   ├── cpu_dumper_factory.h
│   │   └── memory_dumper_factory.h
│   │
│   ├── manager/               # 管理器
│   │   ├── dump_manager.h         # DumpManager (新架构)
│   │   ├── dump_implement.h       # DumpImplement (旧架构)
│   │   ├── cmd_parse.h            # 命令行解析器
│   │   └── dump_context.h         # Dump 上下文
│   │
│   └── util/                  # 工具类
│       ├── file_utils.h
│       ├── string_utils.h
│       └── config_utils.h
│
├── src/                       # 源文件
│   ├── common/               # 公共实现
│   ├── executor/             # Dumper 实现
│   │   ├── hidumper_executor.cpp
│   │   ├── cmd_dumper.cpp         # ⚠️ 命令执行，安全关注
│   │   ├── memory_dumper.cpp
│   │   ├── fd_output.cpp
│   │   └── zip_output.cpp
│   ├── factory/              # 工厂实现
│   ├── manager/              # 管理器实现
│   │   ├── dump_manager.cpp
│   │   └── dump_implement.cpp     # 核心实现!
│   └── util/                 # 工具实现
│
├── manager/                  # 管理器单独目录
│   ├── cmd_parse.cpp
│   └── dump_context.cpp
│
├── dump_strategy/            # 策略模式
│   ├── dump_strategy.h          # 策略基类
│   ├── dump_strategy_factory.h  # 策略工厂
│   ├── cpu_dump_strategy.h
│   ├── mem_dump_strategy.h
│   └── system_ability_dump_strategy.h
│
├── task/                    # 任务系统 (新架构)
│   ├── base/
│   │   ├── task_control.h       # 任务控制
│   │   └── task_enable_config.h
│   ├── cpu/                 # CPU 任务
│   ├── memory/              # 内存任务
│   ├── storage/             # 存储任务
│   └── writer/              # 写入器
│
└── utils/                   # 框架层工具
    └── writer_utils.cpp
```

**新人必看文件** (按优先级):

| 优先级 | 文件 | 说明 |
|--------|------|------|
| P0 | `include/executor/hidumper_executor.h` | 所有 Dumper 的基类 |
| P0 | `src/manager/dump_implement.cpp` | 核心实现入口 |
| P1 | `include/common/dumper_constant.h` | 常量定义 |
| P1 | `include/common/dump_cfg.h` | 配置数据结构 |
| P2 | `include/manager/dump_manager.h` | 新架构管理器 |
| P2 | `dump_strategy/dump_strategy.h` | 策略模式 |

---

### 2.3 interfaces/ - 对外接口

```
interfaces/
├── innerkits/                   # Inner API
│   ├── include/
│   │   └── dump_usage.h         # DumpUsage 工具类
│   └── BUILD.gn
│
└── native/
    └── innerkits/
        └── include/
            ├── dump_broker_proxy.h        # IPC Proxy
            ├── dump_manager_client.h      # 客户端封装
            ├── dump_manager_cpu_client.h  # CPU 客户端
            ├── dump_cpu_data.h            # CPU 数据结构
            ├── dump_common_utils.h        # 通用工具
            ├── idump_broker.h             # IPC 接口定义 ⭐
            ├── hidumper_service_ipc_interface_code.h
            └── hidumper_cpu_service_ipc_interface_code.h
```

**关键接口文件**:

| 文件 | 职责 | 安全关注点 |
|------|------|-----------|
| `idump_broker.h` | IPC 接口定义 | 所有 IPC 入口 |
| `dump_manager_client.h` | 客户端 API | 外部调用点 |
| `dump_usage.h` | 统计 API | 内存/CPU 查询 |

---

### 2.4 services/ - 服务实现

```
services/
├── BUILD.gn                    # 服务层构建配置
├── hidumper.map               # 符号表
├── IHidumperCpuService.idl    # CPU 服务 IDL 定义 ⭐
│
├── native/
│   ├── etc/                   # 配置文件
│   │   ├── hidumper_service.cfg    # 服务启动配置
│   │   ├── infos_config.json       # Dump 内容配置
│   │   ├── task_enable_config.json # 任务配置
│   │   └── event_reason_config.json
│   │
│   ├── include/
│   │   ├── dump_manager_service.h      # 主服务头文件 ⭐
│   │   ├── dump_manager_cpu_service.h  # CPU 服务头文件
│   │   ├── dump_common_utils.h
│   │   ├── dump_log_manager.h
│   │   ├── dump_cpu_data.h
│   │   ├── raw_param.h
│   │   └── inner/
│   │       └── dump_service_id.h       # SA ID 定义 ⭐
│   │
│   └── src/
│       ├── dump_manager_service.cpp      # 主服务实现 ⭐⭐⭐
│       ├── dump_manager_cpu_service.cpp  # CPU 服务实现 ⭐
│       ├── dump_manager_client.cpp       # 客户端实现
│       ├── dump_broker_stub.cpp          # IPC Stub (关键!)
│       ├── dump_common_utils.cpp
│       ├── dump_cpu_data.cpp
│       ├── dump_log_manager.cpp
│       └── raw_param.cpp
│
└── zidl/                       # ZIDL 实现
    ├── include/
    │   └── dump_broker_stub.h    # Stub 头文件
    └── src/
        ├── dump_broker_stub.cpp    # IPC 分发 ⭐⭐⭐
        └── dump_broker_proxy.cpp   # Proxy 实现
```

**关键服务文件**:

| 文件 | 职责 | 安全关注点 |
|------|------|-----------|
| `src/dump_manager_service.cpp:172` | 主服务实现 | 权限检查点 |
| `src/dump_manager_service.cpp:195` | 权限检查实现 | `HasDumpPermission()` |
| `zidl/src/dump_broker_stub.cpp:32` | IPC 分发 | 所有 IPC 入口 |
| `native/include/inner/dump_service_id.h:21` | SA ID | 1212, 1215 |

---

### 2.5 sa_profile/ - SA 配置

```
sa_profile/
├── BUILD.gn
└── 1212.json               # SA 1212 配置文件 ⭐
```

**文件内容**:
```json
{
    "process": "hidumper_service",
    "systemability": [{
        "name": 1212,
        "libpath": "libhidumperservice.z.so",
        "run-on-create": true
    }]
}
```

---

### 2.6 utils/ - 工具类

```
utils/
└── native/
    ├── include/
    │   ├── dump_errors.h       # 错误码定义
    │   └── hilog_wrapper.h     # 日志封装
    └── src/
        └── dump_errors.cpp
```

---

## 3. 代码导航图

### 3.1 请求处理流程

```
用户命令
    │
    ▼
┌────────────────────────────────────────────────────────────────┐
│ 1. 入口层                                                        │
│    client/native/main.cpp:38                                    │
│    └─▶ DumpClientMain::Main()                                   │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 2. 客户端层                                                      │
│    client/native/dump_client_main.cpp:65                        │
│    └─▶ DumpManagerClient::Request() ──IPC──▶                    │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 3. IPC 层                                                        │
│    services/zidl/src/dump_broker_stub.cpp:32                    │
│    └─▶ OnRemoteRequest()                                        │
│        ├─▶ RequestFileFdStub()                                  │
│        ├─▶ ScanPidOverLimitStub()                               │
│        └─▶ CountFdNumsStub()                                    │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 4. 服务层                                                        │
│    services/native/src/dump_manager_service.cpp:165             │
│    ├─▶ Request()              [权限检查]                        │
│    ├─▶ ScanPidOverLimit()     [权限检查]                        │
│    └─▶ CountFdNums()          [权限检查]                        │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 5. 框架层                                                        │
│    frameworks/native/src/manager/dump_implement.cpp:133         │
│    └─▶ DumpImplement::Main()                                    │
│        ├─▶ CmdParse()                                           │
│        ├─▶ ConfigUtils::GetDumperConfigs()                      │
│        └─▶ DumpDatas()                                          │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 6. 执行器层                                                      │
│    frameworks/native/src/executor/                              │
│    ├─▶ cmd_dumper.cpp        (命令执行)                        │
│    ├─▶ memory_dumper.cpp     (内存采集)                        │
│    ├─▶ cpu_dumper.cpp        (CPU 采集)                        │
│    └─▶ ...                                                    │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 权限检查点地图

```
┌────────────────────────────────────────────────────────────────┐
│                      权限检查点分布                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  入口                                                           │
│   │                                                             │
│   ▼                                                             │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ services/native/src/dump_manager_service.cpp              │   │
│ │ ├─▶ Request():172           ──▶ 权限检查                  │   │
│ │ ├─▶ ScanPidOverLimit():219  ──▶ 权限检查                  │   │
│ │ └─▶ CountFdNums():374       ──▶ 权限检查                  │   │
│ │                                                           │   │
│ │ HasDumpPermission():195-204                              │   │
│ │ └─▶ VerifyAccessToken(callingTokenID, "ohos.permission.DUMP")│   │
│ └──────────────────────────────────────────────────────────┘   │
│                                                                │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ services/native/src/dump_manager_cpu_service.cpp          │   │
│ │ └─▶ HasDumpPermission():460-464                          │   │
│ └──────────────────────────────────────────────────────────┘   │
│                                                                │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ frameworks/native/src/manager/dump_implement.cpp          │   │
│ │ └─▶ CheckDumpPermission():1196                           │   │
│ └──────────────────────────────────────────────────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 3.3 安全敏感代码地图

```
┌────────────────────────────────────────────────────────────────┐
│                    安全敏感代码位置                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ 🔴 命令执行 (高风险)                                            │
│   frameworks/native/src/executor/cmd_dumper.cpp:58,110         │
│   └─▶ popen() 调用                                             │
│                                                                │
│ 🟡 文件操作 (中风险)                                            │
│   frameworks/native/src/executor/fd_output.cpp:48              │
│   └─▶ open() 用户路径                                          │
│                                                                │
│   frameworks/native/src/util/file_utils.cpp:65                 │
│   └─▶ fopen() 使用 realpath                                    │
│                                                                │
│   services/native/src/dump_manager_service.cpp:244             │
│   └─▶ readlink() 读取符号链接                                  │
│                                                                │
│ 🟡 动态库加载 (中风险)                                          │
│   frameworks/native/src/executor/memory_dumper.cpp:98          │
│   └─▶ dlopen() 加载 libhidumpermemory.z.so                     │
│                                                                │
│ 🟢 权限检查 (已保护)                                            │
│   services/native/src/dump_manager_service.cpp:195             │
│   └─▶ HasDumpPermission()                                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. 快速查找指南

### 4.1 按功能查找

| 功能 | 查找路径 | 关键文件 |
|------|---------|---------|
| 新增 Dumper | `frameworks/native/include/executor/` | `xxx_dumper.h` |
| 修改 IPC 接口 | `services/zidl/`, `interfaces/native/` | `idump_broker.h` |
| 修改权限检查 | `services/native/src/` | `dump_manager_service.cpp` |
| 修改构建 | 各层 `BUILD.gn` | `BUILD.gn` |
| 修改配置 | `services/native/etc/` | `*.json` |

### 4.2 按类名查找

| 类名 | 位置 | 说明 |
|------|------|------|
| `DumpManagerService` | `services/native/src/dump_manager_service.cpp` | 主服务 |
| `DumpImplement` | `frameworks/native/src/manager/dump_implement.cpp` | 核心实现 |
| `HidumperExecutor` | `frameworks/native/include/executor/hidumper_executor.h` | 执行器基类 |
| `DumperConstant` | `frameworks/native/include/common/dumper_constant.h` | 常量 |
| `DumpCfg` | `frameworks/native/include/common/dump_cfg.h` | 配置结构 |

### 4.3 按符号查找

| 符号 | 类型 | 位置 |
|------|------|------|
| `DFX_SYS_HIDUMPER_ABILITY_ID` | 宏 | `services/native/include/inner/dump_service_id.h:22` |
| `ohos.permission.DUMP` | 字符串 | `services/native/src/dump_manager_service.cpp:198` |
| `ARG_MAX_COUNT` | 常量 | `frameworks/native/dump_controller.h:21` |
| `CMD_PREFIX` | 常量 | `frameworks/native/src/executor/cmd_dumper.cpp:22` |

---

## 5. 相关文档

- [系统架构](./01_Architecture.md) - 理解组件关系
- [API 参考](./02_API_Reference.md) - 接口定义
- [攻击面分析](./05_AttackSurface.md) - 安全关注点
- [构建系统](./03_Build_System.md) - 编译配置
