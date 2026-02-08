# AI Engine 目录结构

## 总体结构

```
ai_engine/
├── interfaces/                    # 对外接口目录
│   └── kits/                      # SDK 头文件
│       ├── ai_datatype.h          # 基础数据类型
│       ├── ai_retcode.h           # 通用错误码
│       ├── asr/                   # 语音识别模块
│       │   └── keyword_spotting/  # 关键词检测 SDK
│       │       ├── kws_sdk.h
│       │       ├── kws_callback.h
│       │       ├── kws_retcode.h
│       │       └── kws_constants.h
│       └── cv/                    # 计算机视觉模块
│           ├── ai_image.h
│           ├── image_classification/ # 图像分类 SDK
│           │   ├── ic_sdk.h
│           │   ├── ic_callback.h
│           │   ├── ic_retcode.h
│           │   └── ic_constants.h
│           └── card_rectification/ # 卡片矫正 SDK
│               ├── cr_sdk.h
│               ├── cr_callback.h
│               └── cr_constants.h
├── services/                      # 服务端实现
│   ├── client/                   # 客户端模块
│   │   ├── client_executor/       # 客户端执行器
│   │   │   ├── include/
│   │   │   │   ├── client_factory.h
│   │   │   │   ├── i_client_cb.h
│   │   │   │   └── async_handler.h
│   │   │   └── source/
│   │   ├── communication_adapter/ # 通信适配层
│   │   │   ├── include/
│   │   │   │   ├── sa_client_adapter.h
│   │   │   │   └── sa_client_proxy.h
│   │   │   └── source/
│   │   └── algorithm_sdk/        # 算法 SDK 实现
│   │       ├── asr/keyword_spotting/
│   │       └── cv/image_classification/
│   ├── common/                   # 公共模块
│   │   ├── platform/             # 平台抽象层
│   │   │   ├── threadpool/       # 线程池
│   │   │   ├── lock/             # 读写锁
│   │   │   ├── semaphore/        # 信号量
│   │   │   ├── event/            # 事件
│   │   │   ├── queuepool/        # 队列池
│   │   │   ├── time/             # 时间工具
│   │   │   ├── dl_operation/     # 动态库操作
│   │   │   └── os_wrapper/       # OS 封装
│   │   │       ├── ipc/          # IPC 封装
│   │   │       ├── feature/      # 特征处理
│   │   │       ├── audio_loader/ # 音频加载
│   │   │       └── utils/        # 工具类
│   │   ├── protocol/            # 协议定义
│   │   │   ├── ipc_interface/    # IPC 接口
│   │   │   ├── data_channel/     # 数据通道
│   │   │   ├── struct_definition/ # 数据结构
│   │   │   └── retcode_inner/    # 错误码
│   │   └── utils/               # 通用工具
│   │       ├── encdec/           # 编解码
│   │       ├── file_operation/  # 文件操作
│   │       └── log/              # 日志
│   └── server/                  # 服务端模块
│       ├── communication_adapter/ # 通信适配层
│       ├── plugin/              # 插件接口
│       │   ├── i_plugin.h
│       │   ├── i_plugin_callback.h
│       │   ├── asr/keyword_spotting/
│       │   └── cv/image_classification/
│       ├── plugin_manager/      # 插件管理器
│       └── server_executor/     # 服务端执行器
├── CMakeLists.txt                # CMake 构建入口
├── bundle.json                   # 组件配置
├── LICENSE                       # 许可证
├── README.md                     # 项目说明
└── README_zh.md                 # 中文说明
```

**证据**：`README.md:19-40` 目录结构描述

## 模块职责

### interfaces/kits（对外接口）

| 目录 | 职责 |
|------|------|
| `asr/keyword_spotting/` | 关键词检测 SDK，提供 KWSSdk 类和 KWSCallback 回调 |
| `cv/image_classification/` | 图像分类 SDK，提供 IcSdk 类和 IcCallback 回调 |
| `ai_datatype.h` | 定义 Array<T>、DataInfo 等基础数据结构 |
| `ai_retcode.h` | 定义 AI_RETCODE_* 通用错误码 |

**证据**：`CMakeLists.txt:12-21` 定义接口头文件

### services/client（客户端模块）

| 目录 | 职责 |
|------|------|
| `client_executor/` | 客户端工厂 ClientFactory，实现 AieClient* 系列 API |
| `communication_adapter/` | SAMGR IPC 适配层，实现 SaClientAdapter |
| `algorithm_sdk/` | 各算法 SDK 的客户端实现封装 |

**证据**：`services/client/BUILD.gn` 客户端构建配置

### services/common（公共模块）

| 目录 | 职责 |
|------|------|
| `platform/threadpool/` | 线程池管理器，支持 IWorker 任务执行 |
| `platform/lock/` | 读写锁 RwLock 实现 |
| `platform/semaphore/` | 信号量接口 |
| `platform/event/` | 事件通知机制 |
| `platform/queuepool/` | 无锁环形队列 |
| `platform/os_wrapper/ipc/` | 共享内存 IPC 封装 |
| `protocol/` | 数据结构、IPC 接口、错误码定义 |
| `utils/` | 日志、编解码、文件操作等工具 |

### services/server（服务端模块）

| 目录 | 职责 |
|------|------|
| `communication_adapter/` | 服务端 SAMGR 适配，SaServerAdapter |
| `plugin/` | IPlugin/IPluginCallback 接口定义 |
| `plugin_manager/` | 插件生命周期管理 |
| `server_executor/` | 引擎管理、任务调度 |

**证据**：`services/server/BUILD.gn:15-42` 服务端组件定义

## 快速定位指南

| 需求 | 定位路径 |
|------|----------|
| 开发新插件 | `services/server/plugin/i_plugin.h` |
| 开发新 SDK | `services/client/algorithm_sdk/` |
| 理解 IPC | `services/common/protocol/ipc_interface/ai_service.h` |
| 修改构建 | `services/BUILD.gn` |
| 查看错误码 | `services/common/protocol/retcode_inner/aie_retcode_inner.h` |
| 线程池实现 | `services/common/platform/threadpool/` |
| 插件配置 | `services/ai_plugin_config.gni` |

## 代码统计

| 类型 | 数量 |
|------|------|
| BUILD.gn 文件 | 39 |
| CMakeLists.txt | 5 |
| 头文件 (h) | 100+ |
| 源文件 (cpp/c) | 100+ |

**证据**：`services/BUILD.gn` 根组件定义，`services/server/BUILD.gn` 服务端定义
