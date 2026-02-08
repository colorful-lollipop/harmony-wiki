# 01_Directory_Structure - 目录结构

## 根目录结构

```
ecological_rule_manager/
├── interfaces/                    # Inner API 接口层（对外）
│   └── innerkits/
│       ├── include/              # 接口头文件
│       └── src/                  # 接口实现
├── services/                     # SA 服务实现（核心）
│   └── manager/
│       ├── include/              # 服务头文件
│       └── src/                  # 服务实现
├── utils/                        # 工具类
│   └── include/                  # 工具头文件
├── profile/                      # SA 配置文件
├── BUILD.gn                      # 根构建入口
├── bundle.json                   # 组件配置
└── LICENSE                       # Apache 2.0
```

## 目录职责说明

### interfaces/innerkits/

**职责**: 对外 Inner API 接口层，提供给其他系统服务调用。

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `include/` | 公共头文件 | `ecological_rule_mgr_service_interface.h`, `ecological_rule_mgr_service_param.h` |
| `src/` | 客户端实现 | `client.cpp`, `proxy.cpp`, `param.cpp` |

**产出**: `liberms_client.z.so` (Inner Kit)

### services/manager/

**职责**: System Ability 服务端实现，处理 IPC 请求和业务逻辑。

| 子目录 | 职责 | 关键文件 |
|--------|------|----------|
| `include/` | 服务端头文件 | `ecological_rule_mgr_service_stub.h` |
| `src/` | 服务实现 | `ecologic_rule_mgr_service.cpp`, `ecologic_rule_mgr_service_stub.cpp` |

**产出**: `libecologicalrulemgr_service.z.so` (SA 库)

### utils/

**职责**: 通用工具类，目前主要包含日志宏定义。

| 文件 | 职责 |
|------|------|
| `ecological_rule_mgr_service_logger.h` | 日志宏（LOG_INFO, LOG_ERROR 等） |

### profile/

**职责**: System Ability 配置信息。

| 文件 | 职责 |
|------|------|
| `6105.json` | SA ID、库路径、运行参数配置 |
| `BUILD.gn` | SA Profile 构建配置 |

## 代码行数统计

| 模块 | 文件数 | 主要职责 |
|------|--------|----------|
| interfaces | 5 | API 暴露、Proxy/Client 实现 |
| services | 2 | SA 主逻辑、IPC Stub |
| utils | 1 | 日志工具 |
| profile | 2 | SA 配置 |

**注意**: 不包含测试代码（test/ 目录）
