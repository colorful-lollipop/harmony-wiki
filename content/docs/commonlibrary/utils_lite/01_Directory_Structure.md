# 目录结构

> 了解 utils_lite 仓库的目录布局与各模块职责。

## 目录树

```
commonlibrary/utils_lite/
├── .git/                        # Git 版本控制
├── .gitee/                      # Gitee 配置
├── file/                        # 文件系统 C API 实现
│   ├── src/
│   │   └── file_impl_hal/
│   │       └── file.c          # UtilsFile API 实现
│   └── BUILD.gn
├── hals/                        # 硬件抽象层
│   └── file/
│       ├── hal_file.c           # HAL File 实现
│       ├── hal_file.h           # HAL File 头文件
│       └── BUILD.gn
├── include/                     # 外部 API 头文件
│   ├── kv_store.h               # KV 存储 API
│   ├── utils_file.h             # 文件操作 API
│   ├── utils_list.h             # 双向链表 API
│   ├── ohos_types.h             # OpenHarmony 类型定义
│   ├── ohos_errno.h              # 错误码定义
│   ├── ohos_init.h              # 初始化框架
│   └── utils_config.h           # 配置项
├── js/                          # JavaScript API (JSI)
│   └── builtin/
│       ├── common/              # 公共工具
│       │   ├── include/
│       │   │   ├── nativeapi_common.h
│       │   │   └── nativeapi_config.h
│       │   ├── src/
│       │   │   └── nativeapi_common.cpp
│       │   └── BUILD.gn
│       ├── deviceinfokit/       # 设备信息套件
│       │   ├── include/
│       │   │   └── nativeapi_deviceinfo.h
│       │   ├── src/
│       │   │   ├── nativeapi_deviceinfo.cpp
│       │   │   └── nativeapi_ohos_deviceinfo.cpp
│       │   └── BUILD.gn
│       ├── filekit/             # 文件套件
│       │   ├── include/
│       │   │   ├── nativeapi_fs.h
│       │   │   └── nativeapi_fs_impl.h
│       │   ├── src/
│       │   │   ├── nativeapi_fs.cpp
│       │   │   └── nativeapi_fs_impl.c
│       │   └── BUILD.gn
│       ├── kvstorekit/          # KV 存储套件
│       │   ├── include/
│       │   │   ├── nativeapi_kv.h
│       │   │   └── nativeapi_kv_impl.h
│       │   ├── src/
│       │   │   ├── nativeapi_kv.cpp
│       │   │   └── nativeapi_kv_impl.c
│       │   └── BUILD.gn
│       └── simulator/           # SDK 模拟器（预览用）
│           └── BUILD.gn
├── kal/                         # 内核抽象层
│   └── timer/
│       ├── include/
│       │   └── kal.h            # KAL Timer API
│       ├── src/
│       │   └── kal.c            # KAL Timer 实现
│       └── BUILD.gn
├── memory/                      # 内存池管理
│   └── include/
│       └── ohos_mem_pool.h      # 内存池 API
├── timer_task/                  # 定时器任务
│   ├── include/
│   │   └── nativeapi_timer_task.h
│   ├── src/
│   │   └── nativeapi_timer_task.c
│   └── BUILD.gn
├── BUILD.gn                     # 根构建文件
├── bundle.json                  # Bundle 配置
├── README.md                    # 英文说明
├── README_zh.md                 # 中文说明
└── LICENSE                      # Apache 2.0 许可证
```

## 模块职责

### file/ - 文件系统 C API

**职责**：提供统一的文件操作 C API

| 文件 | 职责 |
|------|------|
| `src/file_impl_hal/file.c` | UtilsFile API 实现，调用 HAL 层 |
| `BUILD.gn` | 编译配置，输出 `libnative_file.a` |

**证据来源**：`file/src/file_impl_hal/file.c`

### hals/file/ - 硬件抽象层

**职责**：封装底层 POSIX 文件操作，屏蔽硬件差异

| 文件 | 职责 |
|------|------|
| `hal_file.h` | HAL File 接口声明 |
| `hal_file.c` | POSIX 文件操作封装 |
| `BUILD.gn` | 静态库编译配置 |

**证据来源**：`hals/file/hal_file.h:21-37`

### include/ - 外部 API 头文件

**职责**：声明供外部使用的 C API

| 头文件 | 职责 |
|--------|------|
| `utils_file.h` | 文件操作 API（7 个函数） |
| `kv_store.h` | KV 存储 API（3 个函数） |
| `utils_list.h` | 双向链表 API |
| `ohos_mem_pool.h` | 内存池 API |
| `ohos_init.h` | 分层初始化框架 |
| `ohos_types.h` | 类型定义 |
| `ohos_errno.h` | 错误码 |
| `utils_config.h` | 配置宏 |

**证据来源**：`include/` 目录

### js/builtin/ - JavaScript API

**职责**：提供 JS 运行时所需的 Native API

| 目录 | 职责 | JS 方法/属性 |
|------|------|-------------|
| `common/` | 公共回调处理 | `FailCallBack`, `SuccessCallBack` |
| `deviceinfokit/` | 设备信息 | `getInfo` + 40+ 属性 |
| `filekit/` | 文件操作 | `move`, `copy`, `delete`, `list`, `get`, `readText`, `writeText`, `access`, `mkdir`, `rmdir` |
| `kvstorekit/` | KV 存储 | `get`, `set`, `delete`, `clear` |
| `simulator/` | SDK 预览模拟器 | 4 个模拟器库 |

**证据来源**：`js/builtin/` 各模块

### kal/timer/ - 内核抽象层

**职责**：封装 POSIX 定时器，提供统一 KAL 接口

| 文件 | 职责 |
|------|------|
| `include/kal.h` | KAL Timer 接口声明 |
| `src/kal.c` | POSIX 定时器封装 |
| `BUILD.gn` | 编译配置 |

**证据来源**：`kal/timer/include/kal.h`

### memory/ - 内存池管理

**职责**：提供内存池按类型分配

| 头文件 | 职责 |
|--------|------|
| `ohos_mem_pool.h` | 内存池 API |

**证据来源**：`memory/include/ohos_mem_pool.h`

### timer_task/ - 定时器任务

**职责**：提供定时器任务管理，封装 KAL

| 文件 | 职责 |
|------|------|
| `include/nativeapi_timer_task.h` | Timer Task 接口 |
| `src/nativeapi_timer_task.c` | 实现 |
| `BUILD.gn` | 编译配置 |

**证据来源**：`timer_task/include/nativeapi_timer_task.h`

## 模块依赖关系

```
JS Builtin (js/builtin/)
    │
    ├── deviceinfokit/ ────────────────┐
    │                                  │
    ├── filekit/ ──────────────────────┼──> 外部依赖
    │                                  │
    └── kvstorekit/ ───────────────────┘
           │
           │   public_deps
           ▼
    ┌─────────────────────────────────────────┐
    │           C/C++ API Layer               │
    │  ┌────────────┐  ┌─────────────────┐    │
    │  │ utils_file │  │   kv_store      │    │
    │  └────┬───────┘  └────────┬────────┘    │
    │       │                   │             │
    └───────┼───────────────────┼─────────────┘
            │                   │
            ▼                   ▼
    ┌─────────────────────────────────────────┐
    │           HAL / KAL Layer               │
    │  ┌────────────┐  ┌─────────────────┐    │
    │  │ hal_file   │  │   kal_timer     │    │
    │  └────────────┘  └─────────────────┘    │
    └─────────────────────────────────────────┘
                    │
                    ▼
            POSIX / 文件系统 / 定时器
```

**证据来源**：`timer_task/BUILD.gn:32`，`js/builtin/` 各模块 BUILD.gn

## 相关跳转

- [概述](00_Overview.md) - 项目定位与能力
- [架构说明](02_Architecture.md) - 组件关系图
- [N-API 参考](03_NAPI_Reference.md) - JS API 详情
- [内部 API](04_Inner_API.md) - C/C++ 接口
- [GN 构建](05_GN_Build.md) - 构建配置
