# 目录结构

## 顶层结构

```
soc_perf/
├── common/                    # 公共代码
│   └── include/              # 公共头文件
├── interfaces/               # 接口定义
│   └── inner_api/            # Inner API
│       └── socperf_client/    # 客户端实现
├── profile/                  # XML 配置文件
├── sa_profile/                # SA 配置
├── services/                 # 服务实现
│   ├── core/                 # 核心业务逻辑
│   ├── dfx/                  # DFX 功能
│   └── server/               # 服务端代码
├── test/                     # 测试代码（本文档不覆盖）
└── wiki/                     # 文档目录
```

## 目录职责

### common/include/ - 公共模块

| 文件 | 职责 |
|------|------|
| `socperf_log.h` | 日志宏定义 |
| `socperf_trace.h` | 性能追踪宏 |
| `socperf_lru_cache.h` | LRU 缓存实现 |

**证据**：`common/include/socperf_lru_cache.h`

### interfaces/inner_api/socperf_client/ - 客户端模块

| 文件 | 职责 |
|------|------|
| `ISocPerf.idl` | IPC 接口定义 |
| `socperf_client.h` | 客户端 API 头文件 |
| `socperf_client.cpp` | 客户端 IPC 调用实现 |
| `socperf_action_type.h` | 动作类型枚举 |

**证据**：`interfaces/inner_api/socperf_client/include/socperf_client.h:26-139`

### services/core/ - 核心业务

| 文件 | 职责 |
|------|------|
| `socperf.h` | 核心类声明 |
| `socperf.cpp` | 调频仲裁与生效 |
| `socperf_config.h` | 配置加载 |
| `socperf_config.cpp` | XML 解析 |
| `socperf_thread_wrap.h` | 线程封装 |
| `socperf_thread_wrap.cpp` | 内核调用 |

**证据**：`services/core/include/socperf.h:26-81`

### services/server/ - 服务端

| 文件 | 职责 |
|------|------|
| `socperf_server.h` | SA 服务端声明 |
| `socperf_server.cpp` | SA 生命周期与 IPC 接口实现 |

**证据**：`services/server/include/socperf_server.h:30-34`

### services/dfx/ - DFX

| 文件 | 职责 |
|------|------|
| `socperf_hitrace_chain.h` | 性能追踪链 |
| `socperf_hitrace_chain.cpp` | 追踪实现 |

### profile/ - 配置文件

| 文件 | 职责 |
|------|------|
| `socperf_resource_config.xml` | 资源定义（CPU/GPU/DDR/NPU） |
| `socperf_boost_config.xml` | 性能提频配置 |

### sa_profile/ - SA 配置

| 文件 | 职责 |
|------|------|
| `1906.json` | SA ID 与启动配置 |
| `BUILD.gn` | SA profile 构建脚本 |

**证据**：`sa_profile/1906.json`

```
{
    "name": 1906,
    "libpath": "libsocperf_server.z.so",
    "run-on-create": true
}
```

## 稳定性标注

| 层级 | 目录 | 稳定性 | 说明 |
|------|------|--------|------|
| 1 | `interfaces/inner_api` | 稳定 | 对外 Inner API |
| 2 | `services/server` | 稳定 | SA 服务实现 |
| 3 | `services/core` | 稳定 | 核心业务逻辑 |
| 4 | `services/dfx` | 内部 | DFX 追踪功能 |
| 5 | `common` | 内部 | 公共基础库 |
