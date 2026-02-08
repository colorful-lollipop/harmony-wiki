# 03 - 目录结构与代码地图

**文档目的**: 帮助读者在 15 分钟内找到核心代码位置  
**目标受众**: 新人学习者  
**阅读时间**: 约 10 分钟

---

## 顶层目录结构

```
/base/request/request
├── bundle.json              # 组件配置（依赖、syscap、构建组）
├── request_aafwk.gni        # GN 构建配置（feature 开关）
├── hisysevent.yaml          # 系统事件定义
├── figures/                 # 架构图资源
├── common/                  # 公共组件
├── etc/                     # 配置文件（init、SA profile）
├── frameworks/              # 框架实现（N-API、Native）
├── interfaces/              # 接口定义
├── services/                # 服务核心实现（Rust）
└── test/                    # 测试代码 [本文档忽略]
```

---

## 核心目录详解

### 1. services/ - 服务核心（Rust + C++）

**职责**: System Ability 3706 的实现，处理 IPC 请求、任务调度、网络传输

```
services/
├── BUILD.gn                    # 服务构建配置
├── libdownload_server.map      # 动态库符号导出控制
├── include/                    # C++ 头文件
│   └── request_utils.h         # 工具函数
├── src/
│   ├── lib.rs                  # Rust Library Root [入口]
│   ├── ability.rs              # SA 生命周期管理
│   ├── error.rs                # 错误定义
│   ├── macros.rs               # Rust 宏
│   ├── trace.rs                # 性能跟踪
│   ├── sys_event.rs            # 系统事件上报
│   │
│   ├── cxx/                    # C++ 互操作层
│   │   ├── account.cpp         # 账号相关
│   │   ├── bundle.cpp          # Bundle 信息获取
│   │   ├── network.cpp         # 网络状态
│   │   ├── request_utils.cpp   # 工具函数 [权限检查]
│   │   ├── request_cert_mgr_adapter.cpp  # 证书管理
│   │   └── ...
│   │
│   ├── manage/                 # 管理层
│   │   ├── mod.rs              # 模块入口
│   │   ├── task_manager.rs     # [核心] 任务管理器
│   │   ├── database.rs         # [核心] 数据库操作
│   │   ├── network.rs          # 网络状态监听
│   │   ├── network_manager.rs  # 网络管理
│   │   ├── query.rs            # 查询实现
│   │   ├── notifier.rs         # 通知管理
│   │   ├── app_state.rs        # 应用状态监听
│   │   ├── account.rs          # 账号管理
│   │   ├── config/             # 配置管理
│   │   │   ├── cert_manager.rs # 证书配置
│   │   │   └── system_proxy.rs # 系统代理
│   │   ├── scheduler/          # 调度器
│   │   │   ├── mod.rs          # 调度器入口
│   │   │   ├── sql.rs          # SQL 操作
│   │   │   ├── queue/          # 任务队列
│   │   │   │   ├── mod.rs
│   │   │   │   ├── keeper.rs   # 队列管理
│   │   │   │   └── running_task.rs
│   │   │   ├── state/          # 状态管理
│   │   │   │   ├── mod.rs
│   │   │   │   └── recorder.rs
│   │   │   └── qos/            # QoS 策略
│   │   │       ├── mod.rs
│   │   │       ├── apps.rs     # 应用级 QoS
│   │   │       ├── direction.rs
│   │   │       └── rss.rs
│   │   └── events/             # 事件处理
│   │       ├── mod.rs
│   │       ├── construct.rs    # 创建任务
│   │       ├── start.rs        # 启动任务
│   │       ├── pause.rs        # 暂停任务
│   │       ├── resume.rs       # 恢复任务
│   │       ├── stop.rs         # 停止任务
│   │       ├── remove.rs       # 移除任务
│   │       ├── query.rs        # 查询任务
│   │       └── dump.rs         # Dump 信息
│   │
│   ├── service/                # 服务层
│   │   ├── mod.rs              # 模块入口
│   │   ├── interface.rs        # [关键] IPC 命令码定义
│   │   ├── stub.rs             # [关键] IPC Stub 实现
│   │   ├── permission.rs       # [关键] 权限检查
│   │   ├── active_counter.rs   # 活跃任务计数
│   │   ├── client/             # 客户端管理
│   │   │   ├── mod.rs
│   │   │   └── manager.rs
│   │   ├── run_count/          # 运行计数
│   │   │   ├── mod.rs
│   │   │   └── manager.rs
│   │   ├── notification_bar/   # 通知栏
│   │   │   ├── mod.rs
│   │   │   ├── database.rs
│   │   │   ├── notify_flow.rs
│   │   │   ├── progress_size.rs
│   │   │   ├── publish.rs
│   │   │   └── task_handle.rs
│   │   └── command/            # 命令处理
│   │       ├── mod.rs
│   │       ├── construct.rs
│   │       ├── start.rs
│   │       ├── pause.rs
│   │       ├── resume.rs
│   │       ├── stop.rs
│   │       ├── remove.rs
│   │       ├── query.rs
│   │       ├── search.rs
│   │       ├── show.rs
│   │       ├── touch.rs
│   │       ├── get_task.rs
│   │       ├── open_channel.rs
│   │       ├── subscribe.rs
│   │       ├── set_mode.rs
│   │       └── dump.rs
│   │
│   ├── task/                   # 任务实现
│   │   ├── mod.rs              # 模块入口
│   │   ├── config.rs           # 任务配置
│   │   ├── info.rs             # 任务信息
│   │   ├── reason.rs           # 任务原因/错误码
│   │   ├── download.rs         # [核心] 下载实现
│   │   ├── upload.rs           # [核心] 上传实现
│   │   ├── request_task.rs     # 请求任务
│   │   ├── task_control.rs     # 任务控制
│   │   ├── operator.rs         # 任务操作
│   │   ├── notify.rs           # 任务通知
│   │   ├── ffi.rs              # FFI 接口
│   │   ├── files.rs            # 文件操作
│   │   ├── bundle.rs           # Bundle 操作
│   │   └── client.rs           # 客户端操作
│   │
│   └── utils/                  # 工具
│       ├── mod.rs
│       ├── common_event.rs     # 公共事件
│       ├── form_item.rs        # 表单项
│       ├── task_id_generator.rs
│       └── url_policy.rs       # URL 策略
```

**关键文件速查**:

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **服务入口** | `services/src/lib.rs:14` | Rust Library Root |
| **SA 生命周期** | `services/src/ability.rs` | System Ability 启动/停止 |
| **IPC 命令码** | `services/src/service/interface.rs:21` | 22 个 IPC 操作码定义 |
| **IPC 处理** | `services/src/service/stub.rs` | OnRemoteRequest 实现 |
| **权限检查** | `services/src/service/permission.rs:37` | 权限校验逻辑 |
| **任务管理** | `services/src/manage/task_manager.rs` | 任务生命周期管理 |
| **数据库** | `services/src/manage/database.rs` | SQLite 操作 |
| **下载实现** | `services/src/task/download.rs` | HTTP 下载逻辑 |
| **上传实现** | `services/src/task/upload.rs` | HTTP 上传逻辑 |
| **URL 策略** | `services/src/utils/url_policy.rs` | URL 安全策略 |

---

### 2. frameworks/js/napi/request/ - N-API 实现（C++）

**职责**: JS API 到 Native 的桥接

```
frameworks/js/napi/request/
├── BUILD.gn
├── include/                    # 头文件
│   ├── constant.h              # 常量定义
│   ├── js_task.h               # JsTask 类
│   ├── request_event.h         # 事件定义
│   └── ...
├── src/
│   ├── request_module.cpp      # [关键] N-API 模块注册
│   ├── js_task.cpp             # [关键] JsTask 实现
│   ├── js_initialize.cpp       # [关键] 参数初始化与校验
│   ├── napi_utils.cpp          # N-API 工具
│   ├── async_call.cpp          # 异步调用
│   ├── listener_list.cpp       # 监听器列表
│   ├── request_event.cpp       # 事件处理
│   ├── notification_bar.cpp    # 通知栏
│   ├── app_state_callback.cpp  # 应用状态回调
│   ├── js_response_listener.cpp
│   ├── js_notify_data_listener.cpp
│   ├── path_utils.cpp          # 路径工具
│   └── legacy/                 # 遗留接口
│       ├── download_task.cpp
│       └── request_manager.cpp
└── upload/                     # 上传专用
    ├── obtain_file.cpp         # 文件获取
    ├── file_adapter.cpp        # 文件适配
    ├── curl_adp.cpp            # Curl 适配
    └── upload_task.cpp         # 上传任务
```

**关键文件速查**:

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **N-API 注册** | `frameworks/js/napi/request/src/request_module.cpp:276` | 模块注册 |
| **JS 方法** | `frameworks/js/napi/request/src/js_task.cpp` | download/upload/create 等 |
| **参数校验** | `frameworks/js/napi/request/src/js_initialize.cpp:235` | URL/Token/Title 校验 |
| **上传文件** | `frameworks/js/napi/request/src/upload/obtain_file.cpp` | 文件路径处理 |

---

### 3. frameworks/native/request/ - Native 框架（C++）

**职责**: Native 客户端库，提供 IPC 代理

```
frameworks/native/request/
├── BUILD.gn
├── include/
│   ├── i_request_manager.h
│   ├── i_request_manager_listener.h
│   ├── request_common_utils.h
│   ├── request_manager.h
│   ├── request_service_interface.h   # [关键] IPC 接口定义
│   ├── request_service_proxy.h       # [关键] IPC Proxy
│   ├── runcount_notify_stub.h
│   └── response_message.h
└── src/
    ├── request.cpp
    ├── request_manager.cpp
    ├── request_manager_impl.cpp      # [关键] 管理器实现
    ├── request_service_proxy.cpp     # IPC Proxy 实现
    ├── runcount_notify_stub.cpp      # IPC Stub 实现
    ├── response_message_receiver.cpp # 消息接收
    └── parcel_helper.cpp
```

**关键文件速查**:

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **IPC 接口** | `frameworks/native/request/include/request_service_interface.h` | IInterface 定义 |
| **IPC Proxy** | `frameworks/native/request/src/request_service_proxy.cpp` | 客户端代理 |
| **IPC Stub** | `frameworks/native/request/src/runcount_notify_stub.cpp:44` | 服务端 Stub |

---

### 4. frameworks/native/request_action/ - Request Action（C++）

**职责**: 高级请求动作封装

```
frameworks/native/request_action/
├── BUILD.gn
├── include/
│   └── request_action.h
└── src/
    ├── request_action.cpp        # 动作实现
    ├── task_builder.cpp          # [关键] 任务构建与校验
    └── path_control.cpp          # 路径控制
```

**关键文件速查**:

| 功能 | 文件路径 | 说明 |
|------|----------|------|
| **任务构建** | `frameworks/native/request_action/src/task_builder.cpp` | URL/Proxy/Token 校验 |
| **路径控制** | `frameworks/native/request_action/src/path_control.cpp` | 文件路径安全 |

---

### 5. common/ - 公共组件

**职责**: 跨模块共享的基础组件

```
common/
├── database/                   # 数据库组件（Rust）
├── ffrt_rs/                    # FFRT Rust 绑定
├── netstack_rs/                # NetStack Rust 绑定
├── request_core/               # 核心定义（C++）
├── sys_event/                  # 系统事件（C++）
├── utf8_utils/                 # UTF8 工具（C++）
└── utils/                      # 通用工具（C++）
```

---

### 6. etc/ - 配置文件

```
etc/
├── init/
│   ├── BUILD.gn
│   └── downloadservice.cfg     # Init 进程配置
├── sa_profile/
│   ├── BUILD.gn
│   └── 3706.json               # SA 3706 配置
└── icon/
    └── notification_xmark.png  # 通知图标
```

**关键配置**:

| 配置 | 文件 | 说明 |
|------|------|------|
| **SA 配置** | `etc/sa_profile/3706.json:5` | SA ID: 3706 |
| **Init 配置** | `etc/init/downloadservice.cfg` | 进程启动配置 |

---

### 7. interfaces/inner_kits/ - 内部接口

```
interfaces/inner_kits/
├── cache_download/
│   ├── native/include/request_preload.h
│   └── napi/include/preload_napi.h
├── request_action/include/request_action.h
└── running_count/include/running_task_count.h
```

---

## 代码导航图

### 按功能查找代码

#### 我要找下载功能
1. **JS 入口**: `frameworks/js/napi/request/src/request_module.cpp:264`
2. **参数校验**: `frameworks/js/napi/request/src/js_initialize.cpp`
3. **IPC 调用**: `frameworks/native/request/src/request_service_proxy.cpp`
4. **服务端处理**: `services/src/service/stub.rs`
5. **任务创建**: `services/src/manage/events/construct.rs`
6. **下载实现**: `services/src/task/download.rs`

#### 我要找权限检查
1. **权限定义**: `services/src/service/permission.rs:24`
2. **权限校验**: `services/src/cxx/request_utils.cpp:92`
3. **N-API 检查**: `frameworks/js/napi/cache_download/src/preload_module.cpp:317`

#### 我要找 URL 校验
1. **URL 长度限制**: `frameworks/native/request_action/src/task_builder.cpp:232`
2. **URL 格式检查**: `frameworks/native/request_action/src/task_builder.cpp:237`
3. **URL 策略**: `services/src/utils/url_policy.rs`

#### 我要找文件操作
1. **文件路径处理**: `frameworks/js/napi/request/src/upload/obtain_file.cpp:97`
2. **文件创建**: `frameworks/cj/ffi/src/cj_initialize.cpp:719`

---

## 快速定位表

### 核心数据结构

| 结构 | 文件路径 |
|------|----------|
| TaskConfig | `services/src/task/config.rs` |
| TaskInfo | `services/src/task/info.rs` |
| DownloadConfig | `frameworks/js/napi/request/include/js_task.h` |

### 错误码定义

| 错误类型 | 文件路径 |
|----------|----------|
| 服务错误 | `services/src/error.rs` |
| N-API 错误 | `frameworks/js/napi/request/include/constant.h` |

### 常量定义

| 常量 | 文件路径 | 行号 |
|------|----------|------|
| URL_MAXIMUM (8192) | `frameworks/native/request_action/src/task_builder.cpp` | 232 |
| TITLE_MAXIMUM (256) | `frameworks/native/request_action/src/task_builder.cpp` | 326 |
| TOKEN_MAX_BYTES (2048) | `frameworks/native/request_action/src/task_builder.cpp` | 340 |

---

## 相关文档

- **架构理解**: [02_Architecture.md](02_Architecture.md)
- **API 详情**: [04_Interface.md](04_Interface.md)
- **安全分析**: [05_AttackSurface.md](05_AttackSurface.md)
