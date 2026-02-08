# 目录结构

## 顶层目录

```
//foundation/resourceschedule/frame_aware_sched
├── common                      # 公共工具类
├── interfaces                  # 接口目录
│   └── innerkits              # Inner API（系统内部接口）
├── frameworks                  # 框架核心实现
│   └── core                   # 核心组件
│       ├── frame_aware_collector   # 帧信息收集器
│       └── frame_aware_policy     # 帧感知策略
├── profiles                    # 配置文件
└── test                        # 测试目录（文档中忽略）
```

## 各目录职责

### `common/` - 公共工具类

| 文件 | 职责 | 关键符号 | 行号 |
|------|------|---------|------|
| `include/frame_info_const.h` | 帧绘制过程信息枚举 | `FrameSchedEvent`, `FrameEvent`, `SceneEvent` | 21-89 |
| `include/rme_constants.h` | 线程/窗口状态常量 | `ThreadState`, `WindowState`, `ErrorCode` | 21-45 |
| `include/rme_log_domain.h` | 日志域封装 | `RME_LOGI`, `RME_LOGE`, `RME_LOGD` | 50-54 |
| `include/single_instance.h` | 单例模式模板宏 | `DECLARE_SINGLE_INSTANCE`, `IMPLEMENT_SINGLE_INSTANCE` | 21-44 |
| `include/rtg_interface.h` | RTG 控制接口结构体与宏定义 | `MAX_TID_NUM=5`, `CMD_ADD_RTG_THREAD` | 26-106 |

### `interfaces/innerkits/` - Inner API

| 目录/文件 | 职责 | 关键函数 | 行号 |
|-----------|------|---------|------|
| `frameintf/` | Inner API 实现目录 | - | - |
| `frame_ui_intf.h` | UI 帧信息接口（帧事件上报） | `FrameUiIntf::GetInstance()`, `BeginFlushAnimation()` | 25-71 |
| `frame_ui_intf.cpp` | UI 接口实现 | 22 个 extern "C" 导出函数 | 258-421 |
| `frame_msg_intf.h` | 帧消息接口（应用状态事件） | `ReportAppInfo()`, `ReportWindowFocus()` | 26-50 |
| `frame_msg_intf.cpp` | 消息接口实现 | FFRT 任务队列封装 | 27-161 |
| `frame_trace.h` | 帧追踪接口（Trace 控制） | `CreateTraceTag()`, `StartFrameTrace()` | 32-46 |
| `rtg_interface.cpp` | RTG 控制实现 | `EnableRtg()`, `AddThreadToRtg()` | 97-395 |

### `frameworks/core/` - 核心实现

#### `frame_aware_collector/` - 帧信息收集器

| 文件 | 职责 | 关键类/函数 | 行号 |
|------|------|------------|------|
| `include/frame_msg_mgr.h` | 帧消息管理器 | `FrameMsgMgr::EventUpdate()` | 27-68 |
| `include/frame_window_mgr.h` | 窗口管理器 | `SetStartFlag()`, `GetEnable()` | 24-34 |
| `include/frame_scene_sched.h` | 场景调度接口 | `HandleBeginScene()`, `HandleEndScene()` | 28-44 |
| `include/rme_core_sched.h` | 核心调度 | `HandleBeginScene()`, `SetMargin()` | 21-56 |
| `include/rme_scene_sched.h` | 场景调度实现 | `HandleBeginScene()`, `HandleEndScene()` | 24-59 |
| `src/frame_msg_mgr.cpp` | 帧消息管理实现 | 事件映射表 `m_frameMsgKeyToFunc` | - |
| `src/rme_scene_sched.cpp` | 场景调度实现 | 滑动场景识别算法 | - |

#### `frame_aware_policy/` - 帧感知策略

| 文件 | 职责 | 关键类/函数 | 行号 |
|------|------|------------|------|
| `include/app_info.h` | 应用信息结构 | `AppInfo` 类定义 | 44-75 |
| `include/intellisense_server.h` | 智能感知服务器 | `ReportAppInfo()`, `TryCreateRtgForApp()` | 34-72 |
| `include/para_config.h` | 参数配置 | `GetGeneralConfig()`, `GetFpsList()` | 34-62 |
| `src/intellisense_server.cpp` | 服务器实现 | XML 配置读取、RTG 管理 | 39-400+ |
| `src/para_config.cpp` | 配置解析 | `SplitString()`, `GetFrameConfig()` | - |

### `profiles/` - 配置文件

| 文件 | 职责 |
|------|------|
| `hwrme.xml` | RME 智能感知配置（FPS 列表、渲染类型、帧调度参数） |

### `qos_manager/` - QoS 管理

| 文件 | 职责 | 关键函数 | 行号 |
|------|------|---------|------|
| `include/qos_common.h` | QoS 接口定义 | `AuthEnable()`, `AuthPause()`, `AuthDelete()` | 47-52 |
| `src/qos_common.cpp` | QoS 实现 | ioctl 调用 `/dev/auth_ctrl` | 31-111 |

## 模块依赖关系

```
interfaces/innerkits/
    ├── frame_ui_intf.cpp ──┬──> frameworks/core/frame_aware_collector/
    ├── frame_msg_intf.cpp ──┴──> frameworks/core/frame_aware_policy/
    ├── frame_trace.cpp ──────────> (独立追踪模块)
    └── rtg_interface.cpp ────────> common/include/rtg_interface.h
                                      └── kernel (/proc/self/sched_rtg_ctrl)
                                      └── /dev/basic_auth_ctrl (via qos_common)
```
