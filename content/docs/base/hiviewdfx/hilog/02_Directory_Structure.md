# HiLog 目录结构与模块职责

> 生成时间: 2026-02-06

---

## 目的

本文档描述 HiLog 项目的目录组织、各模块职责、关键文件定位。

## 适用范围

涵盖 `base/hiviewdfx/hilog/` 下的所有非测试目录和文件。

---

## 目录树（忽略测试）

```
/base/hiviewdfx/hilog
├── frameworks/                    # 框架层实现
│   ├── hilog_ndk/            # NDK 接口层
│   │   └── BUILD.gn          # hilog_ndk target
│   ├── include/                # 公共头文件
│   │   ├── hilog_base.h       # 基础定义（HilogMsg、常量）
│   │   ├── hilog_cmd.h         # 命令结构定义
│   │   ├── hilog_common.h      # 通用定义（错误码、常量）
│   │   └── hilog_types.h       # 类型定义
│   └── libhilog/              # libhilog 实现
│       ├── BUILD.gn             # libhilog source templates
│       ├── base/                # 基础实现
│       │   └── hilog_base.c  # hilog_base 静态库源码
│       ├── snapshot/             # 快照功能
│       │   └── hilog_snapshot.c # 崩溃时日志快照
│       ├── utils/               # 工具函数
│       │   ├── hilog_utils.cpp  # 日志工具函数
│       │   └── log_utils.cpp   # 日志辅助函数（GetPPidByPid）
│       ├── vsnprintf/            # 安全格式化
│       │   └── vsnprintf_s_p.c # vsnprintf_s 实现
│       ├── param/                # 参数配置
│       │   └── properties.cpp    # 参数读取（隐私模式）
│       ├── ioctl/                # IO 控制包装
│       │   └── log_ioctl.cpp     # LogIoctl 类实现
│       ├── socket/               # Unix Domain Socket 实现
│       │   ├── include/
│       │   │   ├── socket.h               # Socket 基类
│       │   │   ├── socket_client.h         # Socket 客户端基类
│       │   │   ├── socket_server.h          # Socket 服务端基类
│       │   │   ├── dgram_socket_client.h     # DGRAM 客户端
│       │   │   ├── dgram_socket_server.h     # DGRAM 服务端
│       │   │   ├── seq_packet_socket_client.h # SeqPacket 客户端
│       │   │   ├── seq_packet_socket_server.h # SeqPacket 服务端
│       │   │   ├── hilog_input_socket_client.h     # 日志输入客户端
│       │   │   └── hilog_input_socket_server.h     # 日志输入服务端
│       │   ├── socket.cpp                   # Socket 基类实现
│       │   ├── dgram_socket_client.cpp       # DGRAM 客户端实现
│       │   ├── dgram_socket_server.cpp       # DGRAM 服务端实现
│       │   ├── seq_packet_socket_client.cpp # SeqPacket 客户端实现
│       │   ├── seq_packet_socket_server.cpp # SeqPacket 服务端实现
│       │   ├── hilog_input_socket_client.cpp    # 日志输入客户端
│       │   └── hilog_input_socket_server.cpp    # 日志输入服务端
│       ├── hilog.cpp             # C++ HiLog API 实现
│       ├── hilog_printf.cpp      # 格式化日志实现
│       ├── log_print.cpp          # 日志打印入口
│       └── log_utils.cpp          # 工具函数
├── interfaces/                    # 接口层
│   ├── native/
│   │   ├── kits/                # 对外 C/C++ 接口
│   │   │   ├── BUILD.gn
│   │   └── include/        # 对外头文件目录
│   │       └── hilog/
│   │           ├── log.h          # 对外日志头文件（最简接口）
│   │           ├── log_c.h        # C API 完整定义
│   │           ├── log_cpp.h      # C++ API 类定义
│   │           └── log_inner.h   # 内部 API
│   │   └── ndk/            # NDK 专用头文件
│   │       └── libhilog.ndk.json # NDK 接口定义
│   ├── innerkits/             # 内部子系统接口
│   │   ├── include/            # 内部头文件
│   │   │   ├── hilog/
│   │   │   ├── hilog_base/
│   │   │   └── hilog_snapshot/
│   │   └── BUILD.gn            # libhilog、libhilog_base、libhilog_snapshot targets
│   ├── js/                    # JavaScript 接口
│   │   ├── BUILD.gn            # hilog_napi group
│   │   └── kits/
│   │       └── napi/
│   │           ├── BUILD.gn    # libhilognapi target
│   │           └── src/
│   │               ├── common/
│   │               │   └── napi/        # N-API 工具类
│   │               │       ├── n_val.h/cpp      # 值包装
│   │               │       ├── n_func_arg.h/cpp  # 参数处理
│   │               │       ├── n_class.h/cpp      # 类管理
│   │               │       └── n_exporter.h   # 导出器基类
│   │               └── hilog/
│   │                   ├── include/context/
│   │                   │   ├── hilog_napi.h    # HilogNapi 类
│   │                   │   └── hilog_napi_base.h # HilogNapiBase 类
│   │                   └── src/
│   │                       ├── module.cpp       # N-API 模块注册
│   │                       └── src/
│   │                           ├── hilog_napi.cpp       # 导出方法
│   │                           └── hilog_napi_base.cpp  # 核心实现
│   ├── rust/                  # Rust 接口
│   │   ├── BUILD.gn            # hilog_rust target
│   │   └── src/
│   │       ├── lib.rs             # Rust FFI 绑定
│   │       └── macros.rs         # Rust 宏定义
│   └── ets/                   # ArkTS/ETS 接口
│       └── ani/
│           ├── BUILD.gn            # ani_hilog_package group
│           └── hilog/
│               ├── BUILD.gn        # hilog_ani、hilog、hilog_etc targets
│               ├── src/
│               │   ├── ani_util.cpp        # ANI 工具函数
│               │   ├── hilog_ani.cpp       # ANI 实现
│               │   └── hilog_ani_base.cpp  # ANI 基类
│               └── ets/
│                   └── @ohos.hilog.ets   # ETS 类型定义
├── services/                      # 服务层
│   ├── hilogd/                # 日志守护进程
│   │   ├── BUILD.gn            # hilogd executable
│   │   ├── include/
│   │   │   ├── log_data.h          # HilogData 结构
│   │   │   ├── log_buffer.h        # HilogBuffer 类
│   │   │   ├── log_collector.h     # LogCollector 类
│   │   │   ├── service_controller.h # ServiceController 类
│   │   │   ├── cmd_executor.h      # CmdExecutor 类
│   │   │   ├── log_persister.h     # LogPersister 类
│   │   │   ├── flow_control.h      # FlowControl 类
│   │   │   ├── log_stats.h         # LogStats 类
│   │   │   ├── log_domains.h      # LogDomains 类
│   │   │   └── log_kmsg.h         # LogKmsg 类
│   │   ├── etc/
│   │   │   └── hilogd.cfg      # init 服务配置
│   │   ├── main.cpp             # hilogd 入口，初始化所有组件
│   │   ├── service_controller.cpp # IOCTL 命令处理
│   │   ├── cmd_executor.cpp      # Socket 服务端，每个客户端一个线程
│   │   ├── log_buffer.cpp        # 环形缓冲区实现
│   │   ├── log_collector.cpp     # 日志接收和流控
│   │   ├── log_persister.cpp     # 日志落盘和压缩
│   │   ├── flow_control.cpp      # Domain 流控实现
│   │   ├── log_stats.cpp         # 统计信息
│   │   ├── log_domains.cpp      # Domain 验证和管理
│   │   ├── log_kmsg.cpp         # 内核消息读取
│   │   ├── kmsg_parser.cpp      # kmsg 解析
│   │   └── log_compress.cpp      # 压缩接口实现
│   └── hilogtool/             # 日志命令行工具
│       ├── BUILD.gn            # hilog executable
│       ├── include/
│       │   └── log_display.h      # 日志显示接口
│       ├── main.cpp             # hilog 工具入口
│       └── log_display.cpp      # 日志格式化输出
├── platform/                       # 平台相关
│   ├── BUILD.gn                # 平台构建配置
│   ├── hilog.gni               # 全局配置（platforms、feature flags）
│   ├── platformlog.gni         # 平台路径定义
│   └── interface/
│       └── native/
│           └── log.cpp     # 平台日志实现
├── wiki/                         # 本文档（生成的）
│   ├── README.md               # 文档首页
│   ├── SUMMARY.md              # 导航目录
│   ├── 01_Overview.md         # 项目概览
│   ├── 02_Directory_Structure.md # 目录结构
│   ├── 03_Architecture.md       # 架构说明
│   ├── 04_NAPI_Interface.md    # N-API 接口
│   ├── 05_Internal_API.md       # 内部 API
│   ├── 06_GN_Targets.md       # GN Targets
│   ├── 07_Build_Artifacts.md   # 编译产物
│   ├── 08_Security_Analysis.md  # 安全分析
│   ├── 09_Troubleshooting.md   # 常见问题
│   ├── appendix/
│   │   ├── Callgraphs.md        # 调用链图
│   │   └── Config_Flags.md     # 配置标志
│   └── _work/
│       ├── NOTES.md             # 事实记录和证据索引
│       └── PLAN.md             # 任务计划
├── figures/                       # 图片资源
├── README_zh.md                # 中文 README
├── README.md                  # 英文 README
├── LICENSE                    # Apache 2.0 许可证
├── bundle.json                # 模块定义（依赖、targets、能力）
└── test/                      # 测试代码（本文档不引用）
    ├── BUILD.gn
    ├── fuzztest/
    ├── moduletest/
    └── unittest/
```

---

## 模块职责

### frameworks/ 框架层

#### libhilog/
**职责**: 客户端日志库实现

| 子模块 | 职责 | 关键文件 |
|--------|------|---------|
| socket/ | Unix Domain Socket 客户端实现 | socket.h, socket.cpp, hilog_input_socket_client.cpp |
| ioctl/ | IO 控制包装类 | log_ioctl.cpp |
| param/ | 参数配置读取 | properties.cpp |
| utils/ | 工具函数 | hilog_utils.cpp, log_utils.cpp |
| vsnprintf/ | 安全格式化 | vsnprintf_s_p.c |
| base/ | 基础库 | hilog_base.c |
| snapshot/ | 快照功能 | hilog_snapshot.c |

#### hilog_ndk/
**职责**: NDK 接口层

| 文件 | 职责 |
|------|------|
| BUILD.gn | hilog_ndk shared library target |
| （待有其他源文件） | NDK 封装层 |

### interfaces/ 接口层

#### native/kits/ 对外接口
**职责**: 提供 C/C++ 头文件，供应用使用

| 头文件 | 职责 |
|--------|------|
| hilog/log.h | 对外日志头文件（最简接口） |
| hilog/log_c.h | C API 完整定义（类型、宏） |
| hilog/log_cpp.h | C++ API 类定义 |
| hilog/log_inner.h | 内部 API |

#### native/innerkits/ 内部接口
**职责**: 静态库，供内部子系统使用

| 目标 | 类型 | 职责 |
|------|------|------|
| libhilog | shared_library | 主要对外库 |
| libhilog_base | static_library | 基础库（无动态分配） |
| libhilog_snapshot | static_library | 崩溃日志快照 |

#### js/ JavaScript 接口
**职责**: N-API 绑定，供 JS 应用使用

| 子模块 | 职责 | 关键文件 |
|--------|------|---------|
| napi/common/napi/ | N-API 工具类 | n_val, n_func_arg, n_class |
| napi/hilog/ | N-API 导出 | module.cpp, hilog_napi.cpp, hilog_napi_base.cpp |

#### rust/ Rust 接口
**职责**: Rust FFI 绑定

| 文件 | 职责 |
|------|------|
| lib.rs | Rust FFI 绑定到 libhilog |
| macros.rs | Rust 宏定义 |

#### ets/ ArkTS 接口
**职责**: ANI 绑定，供 ArkTS 使用

| 子模块 | 职责 | 关键文件 |
|--------|------|---------|
| hilog/ani_util.cpp | ANI 工具函数 |
| hilog/hilog_ani.cpp | ANI 实现 |
| hilog/hilog_ani_base.cpp | ANI 基类 |
| hilog/ets/@ohos.hilog.ets | ETS 类型定义 |

### services/ 服务层

#### hilogd/
**职责**: 日志常驻服务

| 组件 | 职责 | 关键文件 |
|------|------|---------|
| HilogBuffer | 环形缓冲区管理 | log_buffer.cpp, log_buffer.h |
| LogCollector | 日志接收和流控 | log_collector.cpp, log_collector.h |
| ServiceController | IOCTL 命令处理 | service_controller.cpp, service_controller.h |
| CmdExecutor | Socket 服务端 | cmd_executor.cpp, cmd_executor.h |
| LogPersister | 日志落盘和压缩 | log_persister.cpp, log_persister.h |
| FlowControl | Domain 流控 | flow_control.cpp, flow_control.h |
| LogStats | 统计信息 | log_stats.cpp, log_stats.h |
| LogDomains | Domain 管理 | log_domains.cpp, log_domains.h |
| LogKmsg | 内核消息读取 | log_kmsg.cpp, kmsg_parser.cpp, log_kmsg.h |
| main | 服务入口 | main.cpp |

#### hilogtool/
**职责**: 命令行日志工具

| 组件 | 职责 | 关键文件 |
|------|------|---------|
| main | 工具入口 | main.cpp |
| log_display | 日志显示和格式化 | log_display.cpp, log_display.h |

---

## 依赖关系

### 纵向依赖（高层 → 低层）

```
JS 应用
    ↓
hilog_napi (N-API)
    ↓
libhilog (client library)
    ↓
Unix Domain Socket
    ↓
hilogd (daemon service)
    ↓
HilogBuffer → LogPersister → /data/log/hilog/*.gz
```

### 横向依赖（同层级）

```
libhilog
    ├─→ socket/
    ├─→ ioctl/
    ├─→ param/
    ├─→ utils/
    └─→ vsnprintf/

hilogd
    ├─→ log_buffer (core)
    ├─→ log_collector (input)
    ├─→ service_controller (command)
    ├─→ log_persister (output)
    ├─→ flow_control (policy)
    └─→ log_stats (monitor)
```

---

## 关键文件快速定位

| 功能需求 | 文件路径 | 说明 |
|---------|---------|------|
| 理解项目定位 | README_zh.md | 项目说明 |
| N-API 调用链 | interfaces/js/kits/napi/src/hilog/module.cpp | N-API 注册入口 |
| N-API 实现 | interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp | 所有 N-API 方法 |
| Socket 通信 | frameworks/libhilog/socket/ | 所有 Socket 实现 |
| 日志服务入口 | services/hilogd/main.cpp | hilogd 初始化 |
| 命令处理 | services/hilogd/service_controller.cpp | IOCTL 处理逻辑 |
| 缓冲区管理 | services/hilogd/log_buffer.cpp | 环形缓冲区 |
| 日志落盘 | services/hilogd/log_persister.cpp | 持久化实现 |
| 构建配置 | hilog.gni | platforms、feature flags |
| 模块定义 | bundle.json | 依赖、targets、能力 |

---

## 相关跳转链接

- [项目概览](01_Overview.md)
- [架构说明](03_Architecture.md)
- [N-API 接口](04_NAPI_Interface.md)
- [内部 API](05_Internal_API.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| 目录树 | - | - |
| libhilog BUILD.gn | frameworks/libhilog/BUILD.gn | 全文 |
| hilogd BUILD.gn | services/hilogd/BUILD.gn | 全文 |
| N-API BUILD.gn | interfaces/js/kits/napi/BUILD.gn | 全文 |
| N-API module | interfaces/js/kits/napi/src/hilog/module.cpp | NAPI_MODULE |
| libhilog source | frameworks/libhilog/BUILD.gn | libhilog_source_* |
