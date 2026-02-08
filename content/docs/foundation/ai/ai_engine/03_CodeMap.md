# AI Engine 代码地图

> 适用读者：需要快速定位代码位置的开发者、维护者、安全研究员
> 依赖章节：了解基本架构后阅读本章节更佳

---

## 概述

AI Engine 代码库采用**客户端-服务器架构**，代码分布清晰。本章节提供快速导航，帮助你在 1 分钟内定位任何代码。

### 顶层目录

```
ai_engine/
├── interfaces/          # 【核心】对外接口（SDK）
│   └── kits/         # 所有 SDK 头文件
├── services/           # 【核心】所有服务端/客户端实现
│   ├── client/       # 客户端模块
│   ├── common/       # 公共模块（协议、平台抽象）
│   └── server/       # 服务端模块
├── test/              # 测试代码（忽略）
└── wiki/              # 文档目录
```

---

## 快速导航表

| 想要找 | 在哪里 | 关键文件 | 相关章节 |
|--------|--------|---------|---------|
| **SDK API 定义** | `interfaces/kits/` | 所有 SDK 头文件 | [04_接口文档](04_Interface.md) |
| **KWS SDK 使用** | `interfaces/kits/asr/keyword_spotting/` | kws_sdk.h, kws_callback.h | [04_接口文档](04_Interface.md#kws-sdk) |
| **图像分类 SDK** | `interfaces/kits/cv/image_classification/` | ic_sdk.h, ic_callback.h | [04_接口文档](04_Interface.md#ic-sdk) |
| **卡证矫正 SDK** | `interfaces/kits/cv/card_rectification/` | cr_sdk.h, cr_callback.h | [04_接口文档](04_Interface.md#cr-sdk) |
| **客户端实现** | `services/client/` | client_executor, communication_adapter | [10_内部实现](10_Internals.md) |
| **服务端实现** | `services/server/` | plugin_manager, server_executor | [10_内部实现](10_Internals.md) |
| **插件接口** | `services/server/plugin/i_plugin.h` | IPlugin 接口定义 | [02_架构](02_Architecture.md#plugin-system) |
| **IPC 通信** | `services/common/protocol/` | AI service 接口定义 | [02_架构](02_Architecture.md#ipc-mechanism) |
| **插件实现示例** | `services/server/plugin/` | KWSPlugin, ICPlugin | [05_使用指南](05_Usage.md) |
| **数据结构** | `services/common/protocol/struct_definition/` | ClientInfo, AlgorithmInfo, DataInfo | [04_接口文档](04_Interface.md#data-structures) |
| **错误码** | `interfaces/kits/ai_retcode.h` | 公共错误码 | [04_接口文档](04_Interface.md#error-codes) |
| **构建配置** | `services/BUILD.gn` | GN 目标定义 | [08_构建系统](08_Build.md) |

---

## 核心模块定位

### 1. 对外接口（SDK）

#### 1.1 Keyword Spotting SDK

```
interfaces/kits/asr/keyword_spotting/
├── kws_sdk.h              # 【入口】主类定义
├── kws_callback.h         # 【回调】KWSCallback 接口
├── kws_retcode.h          # 【错误】KWSRetCode 枚举
└── kws_constants.h         # 【常量】音频处理参数
```

**定位要点**:
- `KWSSdk::Create()` — 创建会话
- `KWSSdk::SyncExecute()` — 同步推理
- `KWSSdk::SetCallback()` — 设置结果回调

**证据**: `interfaces/kits/asr/keyword_spotting/kws_sdk.h:45-107`

#### 1.2 Image Classification SDK

```
interfaces/kits/cv/image_classification/
├── ic_sdk.h               # 【入口】主类定义
├── ic_callback.h          # 【回调】IcCallback 接口
├── ic_retcode.h           # 【错误】IcRetCode 枚举
└── ic_constants.h          # 【常量】类型别名和版本号
```

**定位要点**:
- `IcSdk::Create()` — 建立连接
- `IcSdk::SyncExecute()` — 执行推理
- `IcSdk::SetCallback()` — 设置回调

**证据**: `interfaces/kits/cv/image_classification/ic_sdk.h:44-106`

#### 1.3 Card Rectification SDK

```
interfaces/kits/cv/card_rectification/
├── cr_sdk.h               # 【入口】主类定义
├── cr_callback.h          # 【回调】ICallback<T> 模板
└── cr_constants.h          # 【常量】输入输出结构体
```

**定位要点**:
- `CrSdk::Prepare()` — 建立连接
- `CrSdk::RectifyCardSync()` — 同步矫正

**证据**: `interfaces/kits/cv/card_rectification/cr_sdk.h:44-95`

### 2. 客户端模块（Client）

```
services/client/
├── client_executor/         # 【核心】ClientFactory, AsyncHandler
│   ├── include/
│   │   ├── client_factory.h         # 客户端工厂类
│   │   ├── async_handler.h          # 异步回调处理
│   │   ├── i_client_cb.h           # 回调接口定义
│   │   └── i_aie_client.inl         # 【重要】核心 AIE 客户端 API（内联函数）
│   └── source/
│       ├── client_factory.cpp       # 会话管理实现
│       └── async_handler.cpp        # 异步处理实现
│
├── communication_adapter/    # 【核心】IPC 通信层
│   ├── include/
│   │   ├── sa_client_adapter.h     # SAMGR 客户端适配器
│   │   ├── sa_client.h            # SAMGR 客户端
│   │   ├── sa_client_proxy.h       # IPC 代理
│   │   └── sa_async_handler.h      # 异步 IPC 处理
│   └── source/
│       ├── sa_client_adapter.cpp   # SAMGR 调用
│       ├── sa_client.cpp          # SAMGR 连接管理
│       └── sa_client_proxy.cpp    # IPC 序列化
│
└── algorithm_sdk/         # SDK 实现（高层 API）
    ├── asr/keyword_spotting/
    │   ├── include/kws_sdk_impl.h
    │   └── source/kws_sdk_impl.cpp   # KWS SDK 实现
    └── cv/image_classification/
        ├── include/ic_sdk_impl.h
        └── source/ic_sdk_impl.cpp   # IC SDK 实现
```

**定位要点**:
- **核心 API**: `services/client/client_executor/include/i_aie_client.inl:30-109`
  - `AieClientInit()` — 初始化
  - `AieClientPrepare()` — 准备算法
  - `AieClientSyncProcess()` — 同步推理
  - `AieClientRelease()` — 释放资源
  - `AieClientDestroy()` — 销毁客户端

- **IPC 代理**: `services/client/communication_adapter/source/sa_client_proxy.cpp:87-222`
  - `SyncExecAlgorithmProxy()` — 主同步 IPC 调用
  - `ParcelClientInfo()` — 序列化 ClientInfo
  - `ParcelDataInfo()` — 序列化数据（共享内存 >200B）

**证据**: `services/client/` 目录结构分析

### 3. 服务端模块（Server）

```
services/server/
├── communication_adapter/    # 【核心】IPC 服务端适配
│   ├── include/
│   │   ├── sa_server_adapter.h     # SAMGR 服务端适配器
│   │   ├── client_listener_handler.h  # 客户端监听器
│   │   ├── sa_async_handler.h       # 异步 IPC 处理
│   │   └── adapter_wrapper.h        # 适配器包装
│   └── source/
│       ├── sa_server_adapter.cpp   # 【关键】IPC 请求转换和路由
│       ├── client_listener_handler.cpp  # 客户端监听
│       ├── sa_async_handler.cpp      # 异步处理
│       └── adapter_wrapper.cpp       # 包装实现
│
├── plugin_manager/         # 【核心】插件生命周期管理
│   ├── include/
│   │   ├── plugin_manager.h       # IPluginManager 接口
│   │   ├── plugin.h              # Plugin 包装类
│   │   ├── plugin_label.h         # 插件路径解析
│   │   └── i_plugin_manager.h     # 插件管理器接口
│   └── source/
│       ├── plugin_manager.cpp    # 【关键】插件缓存和加载
│       ├── plugin.cpp           # Plugin 包装实现
│       └── plugin_label.cpp     # 插件路径映射
│
├── server_executor/         # 【核心】任务执行引擎
│   ├── include/
│   │   ├── server_executor.h     # ServerExecutor 单例
│   │   ├── engine_manager.h      # 引擎管理器
│   │   ├── engine.h             # Engine 实例
│   │   ├── engine_worker.h       # 工作线程
│   │   ├── sync_msg_handler.h   # 同步消息处理器
│   │   ├── async_msg_handler.h  # 异步消息处理器
│   │   ├── i_handler.h         # Handler 接口
│   │   ├── i_engine_manager.h   # 引擎管理器接口
│   │   ├── i_sync_task_manager.h # 同步任务管理器接口
│   │   ├── i_async_task_manager.h# 异步任务管理器接口
│   │   └── future.h            # Future 对象
│   └── source/
│       ├── server_executor.cpp   # 【关键】主执行器
│       ├── engine_manager.cpp    # 【关键】引擎生命周期
│       ├── engine.cpp           # 【关键】Engine 实例
│       ├── sync_msg_handler.cpp # 【关键】同步任务处理
│       └── async_msg_handler.cpp# 异步任务处理
│
└── plugin/               # 插件实现（示例和参考）
    ├── i_plugin.h             # 【关键】IPlugin 接口定义
    ├── i_plugin_callback.h    # IPluginCallback 接口定义
    ├── asr/keyword_spotting/
    │   ├── include/kws_plugin.h
    │   └── source/kws_plugin.cpp   # KWSPlugin 示例
    └── cv/image_classification/
        ├── include/ic_plugin.h
        └── source/ic_plugin.cpp   # ICPlugin 示例
```

**定位要点**:
- **插件管理器**: `services/server/plugin_manager/source/plugin_manager.cpp:25-127`
  - `GetPlugin(aid, version)` — 获取或加载插件
  - `LoadPlugin()` — 动态加载 .so 文件

- **引擎管理器**: `services/server/server_executor/source/engine_manager.cpp:57-236`
  - `StartEngine()` — 创建/启动引擎
  - `FindEngine(transactionId)` — 按事务 ID 查找引擎

- **服务端适配器**: `services/server/communication_adapter/source/sa_server_adapter.cpp:218-246`
  - `SyncExecute()` — 接收 IPC，转换为 IRequest
  - `ConvertToRequest()` — 转换客户端数据为 IRequest

**证据**: `services/server/` 目录结构分析

### 4. 公共模块（Common）

```
services/common/
├── protocol/              # 【核心】协议定义和数据结构
│   ├── ipc_interface/
│   │   └── ai_service.h          # 【关键】IPC 接口 ID 枚举
│   ├── struct_definition/
│   │   └── aie_info_define.h       # 【关键】ClientInfo, AlgorithmInfo, DataInfo
│   ├── data_channel/
│   │   ├── include/
│   │   │   ├── i_request.h           # IRequest 接口
│   │   │   └── i_response.h          # IResponse 接口
│   │   └── source/
│   │       ├── request.cpp           # Request 实现
│   │       └── response.cpp          # Response 实现
│   └── retcode_inner/
│       └── aie_retcode_inner.h   # 内部错误码
│
├── platform/              # 【核心】平台抽象层
│   ├── os_wrapper/
│   │   ├── ipc/
│   │   │   ├── include/aie_ipc.h
│   │   │   └── source/aie_ipc.cpp     # 【关键】共享内存实现（200B 阈值）
│   │   ├── dl_operation/
│   │   │   ├── include/aie_dl_operation.h
│   │   │   └── source/aie_dl_operation.cpp  # 【关键】dlopen 包装（路径验证）
│   │   └── utils/
│   │       ├── include/plugin_helper.h
│   │       └── source/plugin_helper.cpp     # 序列化辅助
│   ├── threadpool/
│   │   ├── include/thread_pool.h
│   │   └── source/thread_pool.cpp  # 线程池实现
│   ├── lock/
│   │   ├── include/rw_lock.h
│   │   └── source/rw_lock.cpp      # 读写锁
│   ├── event/
│   │   ├── include/i_event.h
│   │   └── source/event.cpp        # 事件通知
│   └── time/
│       ├── include/time.h
│       └── source/time.cpp         # 时间工具
│
└── utils/                 # 工具类
    ├── include/aie_guard.h      # RAII 内存守卫（PointerGuard, MallocPointerGuard）
    └── include/aie_macros.h     # 宏定义（CHK_RET, AIE_DELETE）
```

**定位要点**:
- **IPC 接口**: `services/common/protocol/ipc_interface/ai_service.h:16-29`
  - `ID_SYNC_EXECUTE_ALGORITHM` — 同步推理接口 ID
  - `ID_ASYNC_EXECUTE_ALGORITHM` — 异步推理接口 ID

- **共享内存**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:35-242`
  - `IpcIoPushSharedMemory()` — 创建共享内存
  - `IpcIoPopSharedMemory()` — 读取共享内存
  - 阈值 200 字节（`IPC_MAX_TRANS_CAPACITY`）

- **数据结构**: `services/common/protocol/struct_definition/aie_info_define.h:21-58`
  - `ClientInfo` — 客户端信息（clientId, sessionId, uid）
  - `AlgorithmInfo` — 算法规格（类型、版本、同步/异步）
  - `DataInfo` — IPC 数据容器（data指针 + length）

**证据**: `services/common/` 目录结构分析

---

## 关键符号索引

### 核心类与接口

| 符号名 | 位置 | 用途 |
|--------|------|------|
| `IPlugin` | `services/server/plugin/i_plugin.h:34` | 【关键】插件接口（所有插件必须实现） |
| `IPluginCallback` | `services/server/plugin/i_plugin_callback.h:30` | 异步插件回调接口 |
| `IPluginManager` | `services/server/plugin_manager/include/i_plugin_manager.h:26` | 插件管理器接口 |
| `PluginManager` | `services/server/plugin_manager/include/plugin_manager.h:56` | 插件管理器实现（单例） |
| `Plugin` | `services/server/plugin_manager/include/plugin.h:25` | 插件包装类 |
| `ServerExecutor` | `services/server/server_executor/include/server_executor.h:35` | 服务端执行器（单例） |
| `EngineManager` | `services/server/server_executor/include/engine_manager.h:53` | 引擎管理器 |
| `Engine` | `services/server/server_executor/include/engine.h:35` | 引擎实例（执行容器） |
| `IClientCb` | `services/client/client_executor/include/i_client_cb.h:23` | 客户端结果回调 |
| `IServiceDeadCb` | `services/client/client_executor/include/i_client_cb.h:37` | 服务死亡通知 |
| `IRequest` | `services/common/protocol/data_channel/include/i_request.h` | 请求数据接口 |
| `IResponse` | `services/common/protocol/data_channel/include/i_response.h` | 响应数据接口 |

### 核心函数

| 函数名 | 位置 | 用途 |
|--------|------|------|
| `AieClientInit()` | `services/client/client_executor/include/i_aie_client.inl:30` | 初始化 AI 引擎客户端 |
| `AieClientPrepare()` | `services/client/client_executor/include/i_aie_client.inl:39` | 准备算法（加载插件） |
| `AieClientSyncProcess()` | `services/client/client_executor/include/i_aie_client.inl:91` | 同步推理（主入口） |
| `AieClientRelease()` | `services/client/client_executor/include/i_aie_client.inl:82` | 释放算法资源 |
| `AieClientDestroy()` | `services/client/client_executor/include/i_aie_client.inl:57` | 销毁客户端 |
| `SyncExecAlgorithmProxy()` | `services/client/communication_adapter/source/sa_client_proxy.cpp:190` | IPC 同步执行代理（主入口） |
| `SyncMsgHandler::Process()` | `services/server/server_executor/source/sync_msg_handler.cpp:33` | 同步任务处理（调用插件） |
| `PluginManager::GetPlugin()` | `services/server/plugin_manager/source/plugin_manager.cpp:25` | 获取或加载插件 |
| `EngineManager::StartEngine()` | `services/server/server_executor/source/engine_manager.cpp:74` | 创建/启动引擎 |
| `IpcIoPushSharedMemory()` | `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:51` | 创建共享内存段 |

### 核心常量

| 常量名 | 位置 | 值 | 用途 |
|--------|------|-----|------|
| `PLUGIN_SYNC_INFER` | `services/server/plugin/i_plugin.h:47` | 同步推理模式标识 |
| `PLUGIN_ASYNC_INFER` | `services/server/plugin/i_plugin.h:47` | 异步推理模式标识 |
| `IPC_MAX_TRANS_CAPACITY` | `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:200` | 共享内存阈值（200 字节） |
| `INVALID_KWS_HANDLE` | `interfaces/kits/asr/keyword_spotting/kws_constants.h:48` | 无效 KWS 句柄（-1） |
| `INVALID_IC_HANDLE` | `interfaces/kits/cv/image_classification/ic_constants.h:68` | 无效 IC 句柄（-1） |

---

## 代码定位指南

### 场景 1：开发新插件

**目标位置**: `services/server/plugin/`

**步骤**:
1. 查看接口定义: `services/server/plugin/i_plugin.h`
2. 参考实现示例:
   - KWS: `services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp`
   - IC: `services/server/plugin/cv/image_classification/source/ic_plugin.cpp`
3. 实现 `IPlugin` 接口：
   - `GetVersion()`, `GetName()`, `GetInferMode()`
   - `SyncProcess()` 或 `AsyncProcess()`
   - `Prepare()`, `Release()`
   - `SetOption()`, `GetOption()`
4. 使用 `PLUGIN_INTERFACE_IMPL()` 宏注册

**关键文件**:
- `services/server/plugin/i_plugin.h:34-105` — 接口定义
- `services/server/plugin_manager/source/plugin.cpp:80-108` — 动态加载机制

---

### 场景 2：开发新 SDK

**目标位置**: `services/client/algorithm_sdk/`

**步骤**:
1. 查看现有 SDK 实现:
   - KWS: `services/client/algorithm_sdk/asr/keyword_spotting/source/kws_sdk_impl.cpp`
   - IC: `services/client/algorithm_sdk/cv/image_classification/source/ic_sdk_impl.cpp`
2. 封装核心客户端 API:
   - `AieClientInit()` → `AieClientPrepare()` → `AieClientSyncProcess()`
   - 序列化/反序列化输入数据
   - 设置回调
3. 导出公共 API（如 `Create()`, `SyncExecute()`, `Destroy()`）

**关键文件**:
- `services/client/client_executor/include/i_aie_client.inl:30-109` — 核心 API
- `services/client/algorithm_sdk/asr/keyword_spotting/source/kws_sdk_impl.cpp:38-82` — KWS SDK 示例

---

### 场景 3：追踪 IPC 数据流

**入口**: `services/client/communication_adapter/source/sa_client_proxy.cpp:190`
- `SyncExecAlgorithmProxy()` — 客户端发起 IPC

**传输**: `services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:211-242`
- `ParcelDataInfo()` — 序列化（使用共享内存如果 >200B）
- `IpcIoPushSharedMemory()` — 创建共享内存

**接收**: `services/server/communication_adapter/source/sa_server_adapter.cpp:218`
- `SyncExecute()` — 服务端接收 IPC
- `ConvertToRequest()` — 转换为 IRequest

**处理**: `services/server/server_executor/source/sync_msg_handler.cpp:33-64`
- `SyncMsgHandler::Process()` — 调用插件

**关键调用链**: 
```
SDK → AieClientSyncProcess() → 
    SaClientProxy::SyncExecAlgorithmProxy() → 
        IpcIoPushSharedMemory() → 
            SAMGR (Binder) → 
                SaServerAdapter::SyncExecute() → 
                    ServerExecutor::SyncExecute() → 
                        Engine::SyncExecute() → 
                            SyncMsgHandler::Process() → 
                                Plugin::SyncProcess()
```

---

### 场景 4：调试插件加载

**插件管理器**: `services/server/plugin_manager/source/plugin_manager.cpp:25-127`

**关键函数**:
- `GetPlugin(aid, version)` (25 行) — 获取或加载插件
- `LoadPlugin()` (80 行) — 动态加载机制
  - `PluginLabel::GetLibPath()` — 解析路径
  - `AieDlopen()` — 加载 .so
  - `AieDlsym()` — 获取 `PLUGIN_INTERFACE` 符号

**插件路径映射**: `services/server/plugin_manager/source/plugin_label.cpp:61-73`
- `cv_image_classification+20001001` → `/usr/lib/libcv_image_classification.so`
- `asr_keyword_spotting+20001002` → `/usr/lib/libasr_keyword_spotting.so`

**安全注意**:
- 路径验证: `services/common/platform/dl_operation/source/aie_dl_operation.cpp:35-51`
  - 必须以 `/usr/` 开头
  - 使用 `realpath()` 规范化路径

---

## 文件大小统计

| 目录 | 文件数 | 说明 |
|------|--------|------|
| `interfaces/kits/` | 15 | 公共 SDK 头文件 |
| `services/client/` | ~20 | 客户端实现 |
| `services/server/` | ~30 | 服务端实现 |
| `services/common/` | ~46 | 公共模块 |
| `services/common/protocol/` | ~8 | 协议定义 |
| `services/common/platform/` | ~38 | 平台抽象 |
| **总计（排除测试）** | ~149 | 核心代码文件 |

---

## 常见搜索关键词

| 想要找 | 搜索模式 | 建议位置 |
|--------|---------|---------|
| **所有公共 API** | `class.*Sdk` | `interfaces/kits/` |
| **插件接口** | `class IPlugin` | `services/server/plugin/i_plugin.h` |
| **IPC 接口** | `ID_SYNC_EXECUTE_ALGORITHM` | `services/common/protocol/ipc_interface/` |
| **错误码** | `RETCODE_` | `services/common/protocol/retcode_inner/` |
| **共享内存** | `shmget` | `services/common/platform/os_wrapper/ipc/` |
| **插件加载** | `dlopen` | `services/common/platform/dl_operation/` |
| **数据结构** | `struct (ClientInfo\|AlgorithmInfo\|DataInfo)` | `services/common/protocol/struct_definition/` |
| **回调接口** | `class.*Callback` | `interfaces/kits/*/` |

---

## 证据要求

本章节所有代码定位均基于实际代码分析：

- ✅ 文件路径：完整的绝对或相对路径
- ✅ 行号范围：关键符号所在的行号
- ✅ 符号名：类、函数、常量名称
- ✅ 功能说明：每个符号的用途

**未确认项标注**: [需确认] 表示需要进一步验证

---

## 相关章节

- [02_架构与数据流](02_Architecture.md) — 理解组件间关系
- [04_对外接口文档](04_Interface.md) — 详细的 API 参考
- [10_内部实现细节](10_Internals.md) — 内部实现机制
- [06_攻击面分析](06_AttackSurface.md) — 安全相关代码定位
- [08_构建系统](08_Build.md) — 构建文件定位
