# Inner API 接口参考

## 概述

`frame_aware_sched` 对外提供 **Inner API**（系统内部接口），主要用于：
- 应用进程向系统服务上报帧事件
- 系统服务内部调用 RTG 控制接口

**API 暴露位置**: `bundle.json:43-79` (inner_kits 声明)

---

## API 清单

### 1. FrameUiIntf（UI 帧信息接口）

**命名空间**: `OHOS::RME::FrameUiIntf`  
**头文件**: `interfaces/innerkits/frameintf/frame_ui_intf.h:25-71`  
**实现文件**: `interfaces/innerkits/frameintf/frame_ui_intf.cpp`

**说明**: 用于 JS-UI 子系统上报帧绘制事件

| JS API | C++ 实现 | 参数 | 同步/异步 | 代码位置 |
|--------|----------|------|----------|---------|
| `Init()` | `FrameUiIntf::Init()` | 无 | 同步 | `frame_ui_intf.cpp:35-47` |
| `GetSenseSchedEnable()` | `FrameUiIntf::GetSenseSchedEnable()` | 无 | 同步 | `frame_ui_intf.cpp:49-55` |
| `BeginFlushAnimation()` | `BeginFlushAnimation()` | 无 | 同步 | `frame_ui_intf.cpp:57-63` |
| `EndFlushAnimation()` | `EndFlushAnimation()` | 无 | 同步 | `frame_ui_intf.cpp:65-71` |
| `BeginFlushBuild()` | `BeginFlushBuild()` | 无 | 同步 | `frame_ui_intf.cpp:73-79` |
| `EndFlushBuild()` | `EndFlushBuild()` | 无 | 同步 | `frame_ui_intf.cpp:81-87` |
| `BeginFlushLayout()` | `BeginFlushLayout()` | 无 | 同步 | `frame_ui_intf.cpp:89-95` |
| `EndFlushLayout()` | `EndFlushLayout()` | 无 | 同步 | `frame_ui_intf.cpp:97-103` |
| `BeginFlushRender()` | `BeginFlushRender()` | 无 | 同步 | `frame_ui_intf.cpp:105-111` |
| `EndFlushRender()` | `EndFlushRender()` | 无 | 同步 | `frame_ui_intf.cpp:113-119` |
| `BeginFlushRenderFinish()` | `BeginFlushRenderFinish()` | 无 | 同步 | `frame_ui_intf.cpp:121-127` |
| `EndFlushRenderFinish()` | `EndFlushRenderFinish()` | 无 | 同步 | `frame_ui_intf.cpp:129-135` |
| `BeginProcessPostFlush()` | `BeginProcessPostFlush()` | 无 | 同步 | `frame_ui_intf.cpp:137-143` |
| `ProcessCommandsStart()` | `ProcessCommandsStart()` | 无 | 同步 | `frame_ui_intf.cpp:145-151` |
| `AnimateStart()` | `AnimateStart()` | 无 | 同步 | `frame_ui_intf.cpp:153-159` |
| `RenderStart(timestamp)` | `RenderStart(uint64_t timestamp)` | 时间戳 | 同步 | `frame_ui_intf.cpp:161-167` |
| `RenderEnd()` | `RenderEnd()` | 无 | 同步 | `frame_ui_intf.cpp:169-172` |
| `SendCommandsStart()` | `SendCommandsStart()` | 无 | 同步 | `frame_ui_intf.cpp:174-180` |
| `BeginListFling()` | `BeginListFling()` | 无 | 同步 | `frame_ui_intf.cpp:182-188` |
| `EndListFling()` | `EndListFling()` | 无 | 同步 | `frame_ui_intf.cpp:190-196` |
| `ReportSchedEvent(event, payload)` | `ReportSchedEvent(...)` | 事件+负载 | 同步 | `frame_ui_intf.cpp:253-256` |

**C 语言导出接口** (`extern "C"`):  
位置: `frame_ui_intf.cpp:258-421`

所有 C++ 接口均通过 `extern "C"` 导出供其他语言调用，例如：
```cpp
// frame_ui_intf.cpp:258-261
extern "C" void Init() {
    FrameUiIntf::GetInstance().Init();
}
```

**调用链**:
```
JS-UI --> FrameUiIntf C 接口 (extern "C") --> FrameUiIntf::BeginFlushXXX()
          --> FrameMsgMgr::GetInstance().EventUpdate(FrameEvent::XXX)
          --> RmeSceneSched/RmeCoreSched
```

---

### 2. FrameMsgIntf（帧消息接口）

**命名空间**: `OHOS::RME::FrameMsgIntf`  
**头文件**: `interfaces/innerkits/frameintf/frame_msg_intf.h:26-50`  
**实现文件**: `interfaces/innerkits/frameintf/frame_msg_intf.cpp`

**说明**: 用于上报应用状态事件，内部使用 FFRT 异步队列

| API | 函数签名 | 参数 | 同步/异步 | 代码位置 |
|-----|---------|------|----------|---------|
| `GetInstance()` | `static FrameMsgIntf& GetInstance()` | 无 | 同步 | `frame_msg_intf.cpp:27-31` |
| `Init()` | `bool Init()` | 无 | 同步 | `frame_msg_intf.cpp:41-52` |
| `ReportAppInfo()` | `void ReportAppInfo(int pid, int uid, string bundleName, ThreadState)` | PID/UID/包名/状态 | 异步 (FFRT) | `frame_msg_intf.cpp:93-104` |
| `ReportProcessInfo()` | `void ReportProcessInfo(...)` | 进程信息 | 异步 (FFRT) | `frame_msg_intf.cpp:106-117` |
| `ReportCgroupChange()` | `void ReportCgroupChange(int pid, int uid, int oldGroup, int newGroup)` | Cgroup 变化 | 异步 (FFRT) | `frame_msg_intf.cpp:119-131` |
| `ReportWindowFocus()` | `void ReportWindowFocus(int pid, int uid, int isFocus, int displayId)` | 窗口焦点 | 异步 (FFRT) | `frame_msg_intf.cpp:68-78` |
| `ReportRenderThread()` | `void ReportRenderThread(int pid, int uid, int renderTid)` | 渲染线程 | 异步 (FFRT) | `frame_msg_intf.cpp:80-91` |
| `ReportContinuousTask()` | `void ReportContinuousTask(int pid, int uid, int status)` | 持续任务 | 异步 (FFRT) | `frame_msg_intf.cpp:133-143` |
| `Stop()` | `void Stop()` | 无 | 同步 | `frame_msg_intf.cpp:155-158` |

**FFRT 队列配置**:
```cpp
// frame_msg_intf.cpp:57-58
taskQueue_ = new(ffrt::queue)("frame_aware_sched_msg_queue",
    ffrt::queue_attr().qos(ffrt::qos_user_interactive));
```

**调用链**:
```
应用 --> FrameMsgIntf::ReportAppInfo() --> FFRT Queue (frame_aware_sched_msg_queue)
                                       --> IntelliSenseServer::ReportAppInfo()
                                       --> AppInfo 管理 --> RTG 控制
```

---

### 3. FrameTrace（帧追踪接口）

**命名空间**: `FRAME_TRACE::TraceHandle`  
**头文件**: `interfaces/innerkits/frameintf/frame_trace.h:16-49`  
**实现文件**: `interfaces/innerkits/frameintf/frame_trace.cpp`

**说明**: 提供帧追踪功能（创建/启用/停止 Trace）

| API | 功能 | 同步/异步 | 代码位置 |
|-----|------|----------|---------|
| `CreateTraceTag(traceTag)` | 创建 Trace 标签 | 同步 | `frame_trace.h:32` |
| `SetTraceLimit(handle, limit)` | 设置 Trace 限制 | 同步 | `frame_trace.h:33` |
| `EnableTraceForThread(handle)` | 为线程启用 Trace | 同步 | `frame_trace.h:34` |
| `StartFrameTrace(handle)` | 开始帧追踪 | 同步 | `frame_trace.h:35` |
| `StopFrameTrace(handle)` | 停止帧追踪 | 同步 | `frame_trace.h:36` |
| `TraceAndExecute(func, type)` | 追踪执行函数 | 同步 | `frame_trace.h:37` |
| `FrameAwareTraceEnable(traceTag)` | 检查 Trace 是否启用 | 同步 | `frame_trace.h:40` |
| `QuickStartFrameTrace(traceTag)` | 快速开始 Trace | 同步 | `frame_trace.h:41` |
| `QuickEndFrameTrace(traceTag)` | 快速结束 Trace | 同步 | `frame_trace.h:42` |
| `FrameAwareTraceIsOpen()` | 检查 Trace 状态 | 同步 | `frame_trace.h:43` |
| `FrameAwareTraceOpen()` | 打开 Trace | 同步 | `frame_trace.h:44` |
| `FrameAwareTraceClose()` | 关闭 Trace | 同步 | `frame_trace.h:45` |

---

### 4. RTG Interface（RTG 控制接口）

**命名空间**: `OHOS::RME`  
**头文件**: `common/include/rtg_interface.h:26-109`  
**实现文件**: `interfaces/innerkits/frameintf/rtg_interface.cpp`

**说明**: 直接调用内核 RTG 控制接口（`/proc/self/sched_rtg_ctrl`）

| API | 功能 | 内核命令 | 代码位置 |
|-----|------|----------|---------|
| `EnableRtg(flag)` | 启用/禁用 RTG | `CMD_ID_SET_ENABLE` (0xAB01) | `rtg_interface.cpp:97-119` |
| `AddThreadToRtg(tid, grpId, prioType)` | 添加线程到 RTG | `CMD_ID_SET_RTG` (0xAB02) | `rtg_interface.cpp:121-145` |
| `AddThreadsToRtg(tids, grpId, prioType)` | 批量添加线程 | `CMD_ID_SET_RTG` | `rtg_interface.cpp:147-180` |
| `RemoveRtgThread(tid)` | 从 RTG 移除线程 | `CMD_ID_SET_RTG` | `rtg_interface.cpp:182-200` |
| `RemoveRtgThreads(tids)` | 批量移除线程 | `CMD_ID_SET_RTG` | `rtg_interface.cpp:202-230` |
| `DestroyRtgGrp(grpId)` | 销毁 RTG 组 | `CMD_ID_DESTROY_RTG_GRP` | `rtg_interface.cpp:231-252` |
| `SetFrameRateAndPrioType(rtgId, rate, type)` | 设置帧率优先级 | `CMD_ID_SET_RTG_ATTR` (0xAB04) | `rtg_interface.cpp:254-281` |
| `BeginFrameFreq(stateParam)` | 开始帧率统计 | `CMD_ID_BEGIN_FRAME_FREQ` (0xAB05) | `rtg_interface.cpp:283-294` |
| `EndFrameFreq(stateParam)` | 结束帧率统计 | `CMD_ID_END_FRAME_FREQ` (0xAB06) | `rtg_interface.cpp:296-307` |
| `EndScene(grpId)` | 结束场景 | `CMD_ID_END_SCENE` (0xAB07) | `rtg_interface.cpp:309-324` |
| `SetMinUtil(stateParam)` | 设置最小利用率 | `CMD_ID_SET_MIN_UTIL` (0xAB08) | `rtg_interface.cpp:326-342` |
| `SetMaxUtil(grpId, stateParam)` | 设置最大利用率 | `CMD_ID_SET_MAX_UTIL` (0xAB15) | `rtg_interface.cpp:344-347` |
| `SetMargin(stateParam)` | 设置调度边距 | `CMD_ID_SET_MARGIN` (0xAB0A) | `rtg_interface.cpp:349-360` |
| `SearchRtgForTid(tid)` | 查找线程所属 RTG | `CMD_ID_SEARCH_RTG` (0xAB0D) | `rtg_interface.cpp:362-380` |
| `GetRtgEnable()` | 获取 RTG 使能状态 | `CMD_ID_GET_ENABLE` (0xAB0E) | `rtg_interface.cpp:382-395` |

**内核 IOCTL 命令定义** (`rtg_interface.cpp:41-65`):
```cpp
#define CMD_ID_SET_ENABLE      _IOWR(0xAB, 1, struct rtg_enable_data)
#define CMD_ID_SET_RTG         _IOWR(0xAB, 2, struct rtg_str_data)
#define CMD_ID_SET_RTG_ATTR    _IOWR(0xAB, 4, struct rtg_str_data)
#define CMD_ID_BEGIN_FRAME_FREQ _IOWR(0xAB, 5, struct proc_state_data)
#define CMD_ID_END_FRAME_FREQ  _IOWR(0xAB, 6, struct proc_state_data)
#define CMD_ID_END_SCENE       _IOWR(0xAB, 7, struct proc_state_data)
#define CMD_ID_SET_MIN_UTIL    _IOWR(0xAB, 8, struct proc_state_data)
#define CMD_ID_SET_MARGIN      _IOWR(0xAB, 9, struct proc_state_data)
#define CMD_ID_SEARCH_RTG      _IOWR(0xAB, 12, struct proc_state_data)
#define CMD_ID_GET_ENABLE      _IOWR(0xAB, 14, struct rtg_enable_data)
```

**内核节点**: `/proc/self/sched_rtg_ctrl` (`rtg_interface.cpp:68`)

**数据结构** (`common/include/rtg_interface.h`):
```cpp
struct rtg_grp_data {
    int rtg_cmd;                    // 命令类型
    int grp_id;                     // RTG 组 ID
    int prio_type;                  // 优先级类型
    int rt_cnt;
    int tid_num;                    // 线程数量
    int tids[MAX_TID_NUM];          // 线程 ID 数组 (MAX_TID_NUM=5)
};
```

---

### 5. QoS Auth 接口

**命名空间**: `OHOS::QosCommon`  
**头文件**: `qos_manager/include/qos_common.h:15-56`  
**实现文件**: `qos_manager/src/qos_common.cpp`

**说明**: QoS 授权控制接口，操作 `/dev/auth_ctrl` 设备

| API | 功能 | 代码位置 |
|-----|------|---------|
| `AuthEnable(pid, flag, status)` | 启用进程 QoS 授权 | `qos_common.cpp:38-62` |
| `AuthPause(pid)` | 暂停进程 QoS 授权 | `qos_common.cpp:64-88` |
| `AuthDelete(pid)` | 删除进程 QoS 授权 | `qos_common.cpp:90-111` |

**IOCTL 命令**:
```cpp
// qos_common.h:47-48
#define BASIC_AUTH_CTRL_OPERATION _IOWR(0xCD, 1, struct AuthCtrlData)
```

**数据结构**:
```cpp
// qos_common.h:23-29
struct AuthCtrlData {
    int pid;
    unsigned int type;      // AUTH_ENABLE=1, AUTH_DELETE=2, AUTH_GET=3, AUTH_SWITCH=4
    unsigned int rtgFlag;
    unsigned int qosFlag;   // AF_QOS_DELEGATED=0x0001
    unsigned int status;    // AUTH_STATUS_FOREGROUND=3, BACKGROUND=4
};
```

**设备节点**: `/dev/auth_ctrl` (`qos_common.cpp:33`)

---

## 错误码

| 错误码 | 说明 | 位置 |
|--------|------|------|
| `< 0` | 操作失败（`errno`） | `rtg_interface.cpp` |
| `-1` | 文件描述符无效或参数错误 | `rtg_interface.cpp:123,156,185,208,233,365` |
| `0` | 操作成功 | - |
| `ErrorCode::FAIL` | 初始化失败 | `rme_constants.h:42` |
| `ErrorCode::SUCC` | 初始化成功 | `rme_constants.h:43` |

---

## 线程安全

| 接口 | 线程安全策略 | 代码证据 |
|------|-------------|---------|
| `FrameUiIntf` | 单例模式，无显式锁 | `frame_ui_intf.cpp:29-33` (static instance) |
| `FrameMsgIntf` | `ffrt::mutex` 保护任务队列 | `frame_msg_intf.cpp:43,70,82,95,109,121,135` |
| `RTG Interface` | 无锁（全局 `g_fd`，依赖内核串行化） | `rtg_interface.cpp:38` |
| `IntelliSenseServer` | 单例模式，FFRT 队列串行化 | `intellisense_server.h:35` |

---

## 常量定义

| 常量 | 值 | 定义位置 | 说明 |
|------|-------|---------|------|
| `MAX_TID_NUM` | 5 | `rtg_interface.h:26` | 单次最大线程数 |
| `MAX_SUBPROCESS_NUM` | 8 | `rtg_interface.h:27` | 最大子进程数 |
| `MULTI_FRAME_NUM` | 5 | `rtg_interface.h:28` | 多帧数量 |
| `RTG_SCHED_IPC_MAGIC` | 0xAB | `rtg_interface.cpp:37` | IOCTL Magic |
| `MAX_LENGTH` | 100 | `rtg_interface.cpp:34` | 字符串缓冲区 |
| `AF_QOS_DELEGATED` | 0x0001 | `qos_common.cpp:26` | QoS 委托标志 |
| `RT_PRIO` | 0 | `app_info.h:24` | 实时优先级 |
| `RT_NUM` | 4 | `app_info.h:25` | 实时线程数 |
