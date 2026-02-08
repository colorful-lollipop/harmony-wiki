# TEE Client 目录结构与代码地图

## 1. 顶层目录概览

```
tee_client/
├── README.md                    # 项目说明文档（中英文）
├── bundle.json                  # OpenHarmony 组件配置
├── tee_client.gni               # GN 全局配置
├── figures/                     # 架构图目录
├── build/                       # 特定平台构建配置
├── frameworks/                  # 框架层实现
├── interfaces/                  # 接口定义
├── services/                    # 系统服务
└── test/                        # 测试代码（不纳入 Wiki）
```

## 2. 目录职责说明

### 2.1 frameworks/ - 框架层

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `include/` | 公共头文件 | `tee_client_inner.h`, `tc_ns_client.h` |
| `libteec_client/` | libteec.so 实现 | `tee_client.cpp` (IPC 客户端) |
| `libteec_vendor/` | libteec_vendor.so 实现 | `tee_client_api.c` (核心 API) |
| `tee_file/` | 文件操作封装 | `tee_file.c` |
| `build/` | 框架层构建配置 | `BUILD.gn` |

### 2.2 interfaces/ - 接口层

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `inner_api/` | 内部 API 头文件 | `tee_client_api.h`, `tee_client_type.h` |

### 2.3 services/ - 服务层

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `cadaemon/` | CA 守护进程 | `cadaemon_service.cpp`, `cadaemon_stub.cpp` |
| `teecd/` | TEE 代理服务 | `tee_ca_daemon.c`, `tee_agent.c` |
| `tlogcat/` | TEE 日志服务 | `tlogcat.c` |
| `authentication/` | CA 身份认证 | `tee_auth_system.cpp` |

## 3. 核心文件定位

### 3.1 入口文件

| 入口类型 | 文件路径 | 说明 |
|----------|----------|------|
| **API 头文件入口** | `interfaces/inner_api/tee_client_api.h` | GP 标准 API 声明 |
| **libteec 入口** | `frameworks/libteec_client/tee_client.cpp` | IPC 客户端主入口 |
| **libteec_vendor 入口** | `frameworks/libteec_vendor/tee_client_api.c` | Vendor 库主入口 |
| **cadaemon 服务入口** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp` | SA 8001 服务 |
| **teecd 入口** | `services/teecd/src/tee_ca_daemon.c` | TEE 代理服务 |
| **tlogcat 入口** | `services/tlogcat/src/tlogcat.c` | 日志服务 |

### 3.2 配置与构建

| 类型 | 文件路径 | 说明 |
|------|----------|------|
| **组件配置** | `bundle.json` | OpenHarmony 组件定义 |
| **GN 配置入口** | `tee_client.gni` | 全局 Feature 开关 |
| **框架构建** | `frameworks/build/standard/BUILD.gn` | libteec/libteec_vendor 构建 |
| **cadaemon 构建** | `services/cadaemon/build/standard/BUILD.gn` | CA 守护进程构建 |
| **teecd 构建** | `services/teecd/build/standard/BUILD.gn` | TEE 代理构建 |
| **tlogcat 构建** | `services/tlogcat/build/standard/BUILD.gn` | 日志服务构建 |
| **SA 配置** | `services/cadaemon/build/standard/sa_profile/8001.json` | SA ID 配置 |

## 4. 代码导航图

### 4.1 功能 → 文件映射

| 功能 | 文件路径 | 行号范围 |
|------|----------|----------|
| **TEEC_InitializeContext** | `frameworks/libteec_vendor/tee_client_api.c` | 932-1052 |
| **TEEC_FinalizeContext** | `frameworks/libteec_vendor/tee_client_api.c` | 1052-1121 |
| **TEEC_OpenSession** | `frameworks/libteec_vendor/tee_client_api.c` | 1121-1296 |
| **TEEC_CloseSession** | `frameworks/libteec_vendor/tee_client_api.c` | 1296-1398 |
| **TEEC_InvokeCommand** | `frameworks/libteec_vendor/tee_client_api.c` | 1398-1467 |
| **TEEC_RegisterSharedMemory** | `frameworks/libteec_vendor/tee_client_api.c` | 1467-1625 |
| **TEEC_AllocateSharedMemory** | `frameworks/libteec_vendor/tee_client_api.c` | 1625-1731 |
| **TEEC_ReleaseSharedMemory** | `frameworks/libteec_vendor/tee_client_api.c` | 1731-1800 |
| **TEEC_RequestCancellation** | `frameworks/libteec_vendor/tee_client_api.c` | 1800+ |
| **IPC 请求处理** | `services/cadaemon/src/ca_daemon/cadaemon_stub.cpp` | 30-79 |
| **CA 身份认证** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp` | 417-439 |
| **OpenSession 处理** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp` | 827-889 |
| **InvokeCommand 处理** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp` | 920-968 |
| **共享内存解码** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp` | 723-780 |
| **TA 文件加载** | `frameworks/libteec_vendor/tee_client_app_load.c` | 195-289 |
| **Socket 通信** | `frameworks/libteec_vendor/tee_client_socket.c` | 45-267 |
| **FS Agent 工作** | `services/teecd/src/fs_work_agent.c` | 1475-1540 |
| **Secfile Agent** | `services/teecd/src/secfile_load_agent.c` | 60-368 |
| **Agent 注册** | `services/teecd/src/tee_agent.c` | 40-120 |
| **CA 名称获取** | `services/authentication/tee_auth_common.c` | 25-67 |
| **日志收集** | `services/tlogcat/src/tlogcat.c` | 1000+ |

### 4.2 数据结构定义

| 结构体 | 文件路径 | 行号 |
|--------|----------|------|
| `TEEC_Context` | `interfaces/inner_api/tee_client_type.h` | 75-87 |
| `TEEC_Session` | `interfaces/inner_api/tee_client_type.h` | 94-103 |
| `TEEC_SharedMemory` | `interfaces/inner_api/tee_client_type.h` | 110-121 |
| `TEEC_Operation` | `interfaces/inner_api/tee_client_type.h` | 181-189 |
| `TEEC_UUID` | `interfaces/inner_api/tee_client_type.h` | 63-68 |
| `TEEC_ContextInner` | `frameworks/include/tee_client_inner.h` | 83-98 |
| `TEEC_SharedMemoryInner` | `frameworks/include/tee_client_inner.h` | 100-118 |
| `CaAuthInfo` | `frameworks/include/tc_ns_client.h` | - |
| `DaemonProcdata` | `frameworks/include/tc_ns_client.h` | - |

### 4.3 常量与错误码

| 常量 | 文件路径 | 说明 |
|------|----------|------|
| `TEEC_SUCCESS` | `interfaces/inner_api/tee_client_constants.h` | 成功返回值 |
| `TEEC_ERROR_*` | `interfaces/inner_api/tee_client_constants.h` | 错误码定义 |
| `TEEC_PARAM_*` | `interfaces/inner_api/tee_client_constants.h` | 参数类型 |
| `TEEC_MEMREF_*` | `interfaces/inner_api/tee_client_constants.h` | 内存引用类型 |
| `TC_NS_CLIENT_IOCTL_*` | `frameworks/include/standard/tee_ioctl_cmd.h` | ioctl 命令 |
| `CA_DAEMON_ID` | `frameworks/include/tc_ns_client.h` | SA ID 8001 |

## 5. 调用链导航

### 5.1 初始化上下文调用链

```
CA 应用
    │
    ├──► TEEC_InitializeContext() 
    │       @ frameworks/libteec_vendor/tee_client_api.c:932
    │       
    ├──► TEEC_InitializeContextInner()
    │       @ frameworks/libteec_vendor/tee_client_api.c:1700
    │       
    ├──► ConnectToCaDaemon()
    │       @ frameworks/libteec_vendor/tee_client_socket.c:45
    │       
    └──► SendCaMsg() ──► ioctl(TC_NS_CLIENT_IOCTL_LOGIN)
            @ frameworks/libteec_vendor/tee_client_socket.c:130
```

### 5.2 打开会话调用链

```
CA 应用
    │
    ├──► TEEC_OpenSession()
    │       @ frameworks/libteec_vendor/tee_client_api.c:1205
    │       
    ├──► TEEC_OpenSessionInner()
    │       @ frameworks/libteec_vendor/tee_client_api.c:1121
    │       
    ├──► TEEC_GetApp() ──► TEEC_ReadApp() / TEEC_LoadSecFile()
    │       @ frameworks/libteec_vendor/tee_client_app_load.c
    │       
    └──► ioctl(TC_NS_CLIENT_IOCTL_SES_OPEN_REQ)
            @ frameworks/libteec_vendor/tee_client_api.c:1092
```

### 5.3 IPC 调用链（libteec 模式）

```
CA 应用
    │
    ├──► TeeClient::InitializeContext() / TeeClient::OpenSession()
    │       @ frameworks/libteec_client/tee_client.cpp
    │       
    ├──► IPC::MessageParcel → CaDaemonStub::OnRemoteRequest()
    │       @ services/cadaemon/src/ca_daemon/cadaemon_stub.cpp:30
    │       
    ├──► CaDaemonService::OpenSession() / InvokeCommand()
    │       @ services/cadaemon/src/ca_daemon/cadaemon_service.cpp
    │       
    └──► TEEC_OpenSessionInner() → ioctl()
            @ frameworks/libteec_vendor/tee_client_api.c
```

### 5.4 Agent 工作调用链

```
TEE 安全世界
    │
    ├──► TZDriver (内核)
    │       
    ├──► teecd ──► TeeAgent::AgentWorkThread()
    │       @ services/teecd/src/tee_agent.c
    │       
    ├──► FS Agent ──► FsWorkThread()
    │       @ services/teecd/src/fs_work_agent.c:1475
    │       
    ├──► MISC Agent ──► MiscWorkThread()
    │       @ services/teecd/src/misc_work_agent.c
    │       
    └──► Secfile Load Agent ──► SecfileAgentWorkThread()
            @ services/teecd/src/secfile_load_agent.c:60
```

## 6. 关键符号索引

### 6.1 函数索引（按字母顺序）

| 函数名 | 文件路径 | 行号 | 说明 |
|--------|----------|------|------|
| `AddClient()` | `cadaemon_service.cpp` | 1212 | 添加 IPC 客户端 |
| `AddTidData()` | `cadaemon_service.cpp` | 245 | 添加 TID 数据 |
| `AllocateSharedMemory()` | `cadaemon_service.cpp` | 1091 | 分配共享内存 |
| `CaServerWorkThread()` | `tee_ca_daemon.c` | 200+ | teecd 主工作线程 |
| `CallGetBnContext()` | `cadaemon_service.cpp` | 547 | 获取 Context |
| `CallGetBnSession()` | `cadaemon_service.cpp` | 891 | 获取 Session |
| `CheckPermission()` | `cadaemon_stub.cpp` | 44 | IPC 权限检查 |
| `CheckSizeStatus()` | `cadaemon_service.cpp` | 643 | 大小检查 |
| `CloseSession()` | `cadaemon_service.cpp` | 970 | 关闭会话 |
| `ConnectToCaDaemon()` | `tee_client_socket.c` | 45 | Socket 连接 |
| `ConstructCaAuthInfo()` | `tee_auth_system.cpp` | - | 构造 CA 认证信息 |
| `CopyToShareMemory()` | `cadaemon_service.cpp` | 650 | 拷贝到共享内存 |
| `CreateAshmem()` | `tee_client.cpp` | 960 | 创建 Ashmem |
| `FsWorkThread()` | `fs_work_agent.c` | 1475 | FS Agent 工作线程 |
| `GetBnContext()` | `tee_client_inner.h` | - | 获取内部 Context |
| `GetBnSession()` | `tee_client_inner.h` | - | 获取内部 Session |
| `GetProcdataByPid()` | `cadaemon_service.cpp` | 165 | 通过 PID 获取进程数据 |
| `GetTeecOptMem()` | `cadaemon_service.cpp` | 723 | 获取 TEEC Operation 内存 |
| `InitializeContext()` | `cadaemon_service.cpp` | 455 | IPC 初始化上下文 |
| `InitCaAuthInfo()` | `cadaemon_service.cpp` | 417 | 初始化 CA 认证 |
| `InvokeCommand()` | `cadaemon_service.cpp` | 920 | 调用命令 |
| `IsValidContext()` | `cadaemon_service.cpp` | 295 | 验证 Context 有效性 |
| `OnRemoteRequest()` | `cadaemon_stub.cpp` | 30 | IPC 请求处理入口 |
| `OnStart()` | `cadaemon_service.cpp` | 49 | 服务启动 |
| `OpenSession()` | `cadaemon_service.cpp` | 827 | 打开会话 |
| `ProcessCaDied()` | `cadaemon_service.cpp` | 1289 | 处理 CA 死亡 |
| `RegisterSharedMemory()` | `cadaemon_service.cpp` | 1010 | 注册共享内存 |
| `ReleaseSharedMemory()` | `cadaemon_service.cpp` | 1150 | 释放共享内存 |
| `SendCaMsg()` | `tee_client_socket.c` | 130 | 发送 CA 消息 |
| `SendSecfile()` | `cadaemon_service.cpp` | 1308 | 发送安全文件 |
| `SetCallBack()` | `cadaemon_service.cpp` | 1194 | 设置回调 |
| `SetContextToProcData()` | `cadaemon_service.cpp` | 359 | 设置 Context 到进程数据 |
| `TEEC_AllocateSharedMemoryInner()` | `tee_client_api.c` | 1625 | 内部分配共享内存 |
| `TEEC_CloseSessionInner()` | `tee_client_api.c` | 1296 | 内部关闭会话 |
| `TEEC_GetApp()` | `tee_client_app_load.c` | 195 | 获取 TA 应用 |
| `TEEC_InvokeCommandInner()` | `tee_client_api.c` | 1398 | 内部调用命令 |
| `TEEC_OpenSessionInner()` | `tee_client_api.c` | 1121 | 内部打开会话 |
| `TEEC_ReadApp()` | `tee_client_app_load.c` | 85 | 读取 TA 应用 |
| `TEEC_RegisterSharedMemoryInner()` | `tee_client_api.c` | 1467 | 内部注册共享内存 |
| `TEEC_ReleaseSharedMemoryInner()` | `tee_client_api.c` | 1731 | 内部释放共享内存 |
| `TeeTuiThreadWork()` | `tee_tui_daemon.cpp` | - | TUI 工作线程 |
| `TcuAuthentication()` | `tcu_authentication.c` | - | TCU 认证 |
| `WriteOperation()` | `cadaemon_service.cpp` | 594 | 写入 Operation |
| `WriteSession()` | `cadaemon_service.cpp` | 576 | 写入 Session |
| `WriteSharedMem()` | `cadaemon_service.cpp` | 630 | 写入共享内存 |

### 6.2 宏定义索引

| 宏名 | 文件路径 | 说明 |
|------|----------|------|
| `TEEC_PARAM_TYPES()` | `tee_client_api.h` | 构造参数类型 |
| `TEEC_PARAM_TYPE_GET()` | `tee_client_api.h` | 获取参数类型 |
| `CHECK_ERR_RETURN()` | `tee_log.h` | 错误检查返回 |
| `IS_TEMP_MEM()` | `tee_client_inner.h` | 判断临时内存 |
| `IS_PARTIAL_MEM()` | `tee_client_inner.h` | 判断部分内存 |
| `IS_VALUE_MEM()` | `tee_client_inner.h` | 判断值类型 |
| `TC_NS_CLIENT_IOCTL_*` | `tee_ioctl_cmd.h` | ioctl 命令 |
| `LIST_DECLARE()` | `tee_client_list.h` | 声明链表 |
| `LIST_FOR_EACH()` | `tee_client_list.h` | 遍历链表 |
| `LIST_ENTRY()` | `tee_client_list.h` | 获取链表条目 |

## 7. 文件依赖关系

```
interfaces/inner_api/tee_client_api.h
    ├──► #include "tee_client_type.h"
    │       └──► #include "tee_client_constants.h"
    └──► tee_client_ext_api.h (扩展 API)

frameworks/libteec_vendor/tee_client_api.c
    ├──► #include "tee_client_api.h"
    ├──► #include "tee_client_inner.h"
    ├──► #include "tc_ns_client.h"
    └──► #include "tee_client_socket.h"

services/cadaemon/src/ca_daemon/cadaemon_service.cpp
    ├──► #include "cadaemon_service.h"
    ├──► #include "tc_ns_client.h"
    ├──► #include "tee_client_inner.h"
    └──► #include "tee_auth_system.h"

services/teecd/src/tee_ca_daemon.c
    ├──► #include "tee_ca_daemon.h"
    ├──► #include "tee_agent.h"
    ├──► #include "fs_work_agent.h"
    ├──► #include "secfile_load_agent.h"
    └──► #include "misc_work_agent.h"
```

## 8. 快速查找指南

### 8.1 按功能查找

| 想查找的内容 | 去这里 |
|--------------|--------|
| **API 用法示例** | `README.md` |
| **GP 标准 API 定义** | `interfaces/inner_api/tee_client_api.h` |
| **API 实现代码** | `frameworks/libteec_vendor/tee_client_api.c` |
| **IPC 通信代码** | `frameworks/libteec_client/tee_client.cpp` |
| **TEE 驱动通信** | `frameworks/libteec_vendor/tee_client_socket.c` |
| **CA 认证逻辑** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp:417` |
| **会话管理** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp:827` |
| **共享内存处理** | `services/cadaemon/src/ca_daemon/cadaemon_service.cpp:723` |
| **TA 文件加载** | `frameworks/libteec_vendor/tee_client_app_load.c` |
| **Agent 实现** | `services/teecd/src/*_agent.c` |
| **日志服务** | `services/tlogcat/src/tlogcat.c` |
| **构建配置** | `*/build/standard/BUILD.gn` |
| **错误码定义** | `interfaces/inner_api/tee_client_constants.h` |

### 8.2 按问题查找

| 遇到的问题 | 检查位置 |
|------------|----------|
| **IPC 调用失败** | `cadaemon_stub.cpp` 权限检查 |
| **TA 加载失败** | `tee_client_app_load.c` 路径处理 |
| **共享内存错误** | `cadaemon_service.cpp:723` 解码逻辑 |
| **会话打开失败** | `tee_client_api.c:1121` OpenSession 实现 |
| **权限被拒绝** | `cadaemon_service.cpp:417` CA 认证 |
| **内存泄漏** | 检查各 `malloc/free` 配对 |
| **日志不输出** | `tlogcat.c` 和日志配置 |
| **编译错误** | `bundle.json` 和 `BUILD.gn` 配置 |

---

**文档版本**: 1.0
**更新时间**: 2026-02-07
