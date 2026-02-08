# 目录结构与模块职责

## 顶层目录

```
samgr_lite/
├── interfaces/          # 对外接口层（API 头文件）
│   └── kits/
│       ├── samgr_lite/      # Samgr 核心 API
│       ├── registry/        # IPC 跨进程接口
│       └── communication/   # 广播服务接口
├── services/            # 服务实现层
│   └── samgr_lite/
│       ├── samgr/            # Samgr 核心服务
│       │   ├── source/       # 核心实现源码
│       │   ├── adapter/      # 平台适配层
│       │   └── registry/     # M核服务注册桩
│       ├── samgr_client/     # IPC 客户端
│       ├── samgr_server/     # IPC 服务端
│       ├── samgr_endpoint/   # IPC 通信层
│       └── communication/    # 广播服务
├── config/              # 配置文件
├── BUILD.gn            # 根构建入口
├── bundle.json         # 组件配置
└── config.gni         # GN 配置参数
```

## interfaces/ 对外接口层

### interfaces/kits/samgr/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `samgr_lite.h` | SamgrLite 管理类、服务注册发现 API | `interfaces/kits/samgr/samgr_lite.h:98-288` |
| `iunknown.h` | IUnknown 接口基类与默认实现 | `interfaces/kits/samgr/iunknown.h:150-160` |
| `service.h` | Service 生命周期接口定义 | `interfaces/kits/samgr/service.h:149-205` |
| `feature.h` | Feature 生命周期接口定义 | `interfaces/kits/samgr/feature.h:65-124` |
| `message.h` | 消息通信结构体与 API | `interfaces/kits/samgr/message.h:95-242` |
| `common.h` | 向量容器等公共工具 | `interfaces/kits/samgr/common.h:105-126` |

### interfaces/kits/registry/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `registry.h` | 客户端代理工厂注册 API | `interfaces/kits/registry/registry.h:104` |
| `iproxy_client.h` | IClientProxy 客户端代理定义 | `interfaces/kits/registry/iproxy_client.h:91-114` |
| `iproxy_server.h` | IServerProxy 服务端代理定义 | `interfaces/kits/registry/iproxy_server.h:84-106` |

### interfaces/kits/communication/broadcast/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `broadcast_interface.h` | 发布/订阅接口定义 | `interfaces/kits/communication/broadcast/broadcast_interface.h:223-227` |

## services/ 服务实现层

### services/samgr_lite/samgr/source/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `samgr_lite.c` | SamgrBootstrap、SAMGR_GetInstance 实现 | `services/samgr_lite/samgr/source/samgr_lite.c` |
| `service.c` | 服务注册/发现核心逻辑 | `services/samgr_lite/samgr/source/service.c` |
| `feature.c` | Feature 管理实现 | `services/samgr_lite/samgr/source/feature.c` |
| `iunknown.c` | IUnknown 默认实现 | `services/samgr_lite/samgr/source/iunknown.c` |
| `message.c` | 消息处理与路由 | `services/samgr_lite/samgr/source/message.c` |
| `task_manager.c` | 任务池管理 | `services/samgr_lite/samgr/source/task_manager.c` |
| `common.c` | 公共工具实现 | `services/samgr_lite/samgr/source/common.c` |

### services/samgr_lite/samgr/adapter/

| 目录 | 平台 | 职责 |
|------|------|------|
| `posix/` | A-core | POSIX 平台线程、队列、时间适配 |
| `cmsis/` | M-core | CMSIS 平台线程、队列、时间适配 |

**证据位置**：`services/samgr_lite/samgr/adapter/BUILD.gn:26-51`

### services/samgr_lite/samgr/registry/

| 文件 | 职责 | 平台 |
|------|------|------|
| `service_registry.c` | M核服务注册桩 | M-core |

### services/samgr_lite/samgr_client/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `remote_register_rpc.c` | RPC 远程服务注册发现 | `services/samgr_lite/samgr_client/source/remote_register_rpc.c` |

### services/samgr_lite/samgr_server/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `samgr_server_rpc.c` | RPC 服务端实现、权限控制 | `services/samgr_lite/samgr_server/source/samgr_server_rpc.c` |

### services/samgr_lite/samgr_endpoint/

| 文件 | 职责 |
|------|------|
| `endpoint_rpc.c` | IPC 端点管理 |
| `default_client_rpc.c` | 默认客户端实现 |
| `sa_store.c` | System Ability 存储 |
| `token_bucket.c` | 令牌桶限流 |
| `client_factory.c` | 客户端工厂 |

### services/samgr_lite/communication/broadcast/

| 文件 | 职责 | 证据位置 |
|------|------|----------|
| `broadcast_service.c` | 广播服务核心 | `services/samgr_lite/communication/broadcast/source/broadcast_service.c` |
| `pub_sub_feature.c` | 发布/订阅 Feature | `services/samgr_lite/communication/broadcast/source/pub_sub_feature.c` |
| `pub_sub_implement.c` | 发布/订阅实现 | `services/samgr_lite/communication/broadcast/source/pub_sub_implement.c` |

## 模块依赖关系

```
                    ┌─────────────────────┐
                    │   应用层/Feature     │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
    ┌─────────────────┐ ┌──────────────┐ ┌───────────────┐
    │  Broadcast API │ │ SamgrLite API│ │ Message API   │
    └────────┬────────┘ └──────┬───────┘ └───────┬───────┘
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                 ▼
    ┌──────────────────┐ ┌──────────────┐ ┌───────────────┐
    │   samgr_lite     │ │samgr_client │ │samgr_endpoint │
    │   (核心服务)      │ │(IPC客户端)   │ │(IPC通信层)    │
    └────────┬─────────┘ └──────┬─────┘ └───────┬───────┘
             │                   │               │
             └───────────────────┼───────────────┘
                                 │
              ┌──────────────────┴──────────────────┐
              ▼                                   ▼
    ┌──────────────────┐               ┌──────────────────┐
    │   samgr_server   │               │   IPC 底层       │
    │   (IPC服务端)     │               │ (Binder/DBinder) │
    └──────────────────┘               └──────────────────┘
```

## 下一章

- [系统架构](./02_Architecture.md) - 深入理解组件交互
- [SamgrLite API](./04_SamgrLite_API.md) - 服务注册与发现 API
