# 配置参数

> 本文档描述 qos_manager 项目中的关键配置参数、宏定义和 feature flags。

## 1. 编译宏定义

### 1.1 条件编译宏

| 宏定义 | 定义位置 | 启用条件 | 用途 |
|--------|----------|----------|------|
| **QOS_EXT_ENABLE** | `services/BUILD.gn:71` | `frame_aware_sched_override` 存在 | 启用 QoS 扩展功能 |
| **CROSS_PLATFORM** | 编译配置 | 跨平台构建 | 禁用平台特定代码 |

**QOS_EXT_ENABLE 证据**:

```gn
// services/BUILD.gn:69-72
if (defined(
    global_parts_info.hmosresourceschedule_frame_aware_sched_override)) {
  defines = [ "QOS_EXT_ENABLE" ]
}

// qos/BUILD.gn:46-49
if (defined(
    global_parts_info.hmosresourceschedule_frame_aware_sched_override)) {
  defines = [ "QOS_EXT_ENABLE" ]
}
```

**QOS_EXT_ENABLE 代码影响**:

```cpp
// services/include/qos_interface.h:51-55
struct QosCtrlData {
    int pid;
    unsigned int type;
    unsigned int level;
    int qos;
#ifdef QOS_EXT_ENABLE
    int staticQos;
    int dynamicQos;
    bool tagSchedEnable = false;
#endif
};
```

### 1.2 QoS 控制命令

| 宏定义 | 值 | 定义位置 | 描述 |
|--------|-----|----------|------|
| **QOS_CTRL_IPC_MAGIC** | `0xCC` | `services/include/qos_interface.h:30` | QoS ioctl 魔数 |
| **RTG_SCHED_IPC_MAGIC** | `0xAB` | `services/include/qos_interface.h:31` | RTG ioctl 魔数 |

**ioctl 命令定义**:

```cpp
// services/include/qos_interface.h:107-110
#define QOS_CTRL_BASIC_OPERATION \
    _IOWR(QOS_CTRL_IPC_MAGIC, QOS_CTRL, struct QosCtrlData)
#define QOS_CTRL_POLICY_OPERATION \
    _IOWR(QOS_CTRL_IPC_MAGIC, QOS_POLICY, struct QosPolicyDatas)

// services/include/qos_interface.h:118-119
#define CMD_ID_SET_ENABLE \
    _IOWR(RTG_SCHED_IPC_MAGIC, SET_RTG_ENABLE, struct RtgEnableData)
```

---

## 2. 系统参数

### 2.1 QoS 开关参数

| 参数名 | 类型 | 默认值 | 用途 |
|--------|------|--------|------|
| **persist.qosmanager.setQos.on** | bool | true | 全局 QoS 设置开关 |

**证据代码**:

```cpp
// qos/qos.cpp:38-43
#if !defined(CROSS_PLATFORM)
bool qosEnable = OHOS::system::GetBoolParameter("persist.qosmanager.setQos.on", true);
if (!qosEnable) {
    CONCUR_LOGD("[Qos] qoslevel %{public}d apply for tid %{public}d disable", ...);
    return 0;
}
#endif
```

### 2.2 参数文件

| 文件 | 位置 | 用途 |
|------|------|------|
| `ffrt.para` | `etc/param/` | FFRT 参数配置 |
| `ffrt.para.dac` | `etc/param/` | FFRT DAC (动态访问控制) 配置 |

---

## 3. 枚举常量

### 3.1 QoS 操作类型

```cpp
// services/include/qos_interface.h:39-44
enum class QosManipulateType {
    QOS_APPLY = 1,      // 申请 QoS
    QOS_LEAVE,          // 离开 QoS
    QOS_GET,            // 获取 QoS
    QOS_MAX_NR,
};
```

### 3.2 调度策略

```cpp
// services/include/qos_interface.h:67-72
enum SchedPolicy {
    SCHED_POLICY_OTHER = 0,   // CFS 调度
    SCHED_POLICY_FIFO = 1,    // 先入先出
    SCHED_POLICY_RR = 2,      // 时间片轮转
    SCHED_POLICY_RT_EX = 0xFF,// 实时扩展
};
```

### 3.3 QoS 策略类型

```cpp
// services/include/qos_interface.h:74-81
enum QosPolicyType {
    QOS_POLICY_DEFAULT = 1,      // 默认策略
    QOS_POLICY_SYSTEM_SERVER,    // 系统服务策略
    QOS_POLICY_FRONT,            // 前台策略
    QOS_POLICY_BACK,             // 后台策略
    QOS_POLICY_FOCUS,            // 焦点策略
    QOS_POLICY_MAX_NR,
};
```

### 3.4 QoS 策略标志

```cpp
// services/include/qos_interface.h:83-91
#define QOS_FLAG_NICE           0X01        // nice 值
#define QOS_FLAG_LATENCY_NICE   0X02        // 延迟敏感度
#define QOS_FLAG_UCLAMP         0x04        // uclamp 限制
#define QOS_FLAG_RT             0x08        // 实时优先级

#define QOS_FLAG_ALL    (QOS_FLAG_NICE | QOS_FLAG_LATENCY_NICE | QOS_FLAG_UCLAMP | QOS_FLAG_RT)
```

### 3.5 QoS 等级

```cpp
// interfaces/kits/c/qos.h:50-80
typedef enum QoS_Level {
    QOS_BACKGROUND = 0,         // 后台
    QOS_UTILITY,                // 实用工具
    QOS_DEFAULT,                // 默认
    QOS_USER_INITIATED,         // 用户主动
    QOS_DEADLINE_REQUEST,       // 截止时间
    QOS_USER_INTERACTIVE,      // 用户交互
} QoS_Level;

// interfaces/inner_api/qos.h:21-30
enum class QosLevel {
    QOS_BACKGROUND,
    QOS_UTILITY,
    QOS_DEFAULT,
    QOS_USER_INITIATED,
    QOS_DEADLINE_REQUEST,
    QOS_USER_INTERACTIVE,
    QOS_KEY_BACKGROUND,
    QOS_MAX,
};
```

### 3.6 消息类型

```cpp
// interfaces/inner_api/concurrent_task_type.h:25-44
enum MsgType {
    MSG_FOREGROUND = 0,
    MSG_BACKGROUND,
    MSG_APP_START,
    MSG_APP_KILLED,
    MSG_CONTINUOUS_TASK_START,
    MSG_CONTINUOUS_TASK_END,
    MSG_GET_FOCUS,
    MSG_LOSE_FOCUS,
    MSG_ENTER_INTERACTION_SCENE,
    MSG_EXIT_INTERACTION_SCENE,
    MSG_SUB_FOCUS,
    MSG_GROUP_CHANGE,
    MSG_SYSTEM_MAX,
    MSG_APP_START_TYPE = 100,
    MSG_REG_RENDER,
    MSG_REG_UI,
    MSG_REG_KEY_THERAD,
    MSG_TYPE_MAX
};
```

---

## 4. 常量定义

### 4.1 UID 常量

```cpp
// services/include/qos_interface.h:26-27
constexpr int SYSTEM_UID = 1000;
constexpr int ROOT_UID = 0;
```

### 4.2 QoS 数量

```cpp
// services/include/qos_interface.h:28
constexpr int NR_QOS = 7;
```

### 4.3 RTG 配置字符串

```cpp
// services/src/qos_interface.cpp:59
char configStr[] = "load_freq_switch:1;sched_cycle:1;frame_max_util:1024";
```

| 字段 | 值 | 描述 |
|------|-----|------|
| `load_freq_switch` | 1 | 负载频率开关 |
| `sched_cycle` | 1 | 调度周期 |
| `frame_max_util` | 1024 | 最大帧利用率 |

### 4.4 GEWU 常量

```cpp
// frameworks/native/qos_ndk.cpp:27
const char* GEWU_CLIENT_LIB = "libgewu_client.z.so";

// 符号名称
const char* GEWU_CREATE_SESSION_FUNC = "GewuCreateSession";
const char* GEWU_DESTROY_SESSION_FUNC = "GewuDestroySession";
const char* GEWU_SUBMIT_REQUEST_FUNC = "GewuSubmitRequest";
const char* GEWU_ABORT_REQUEST_FUNC = "GewuAbortRequest";

// 无效 ID
#define OH_QOS_GEWU_INVALID_SESSION_ID (static_cast<OH_QoS_GewuSession>(0xffffffffU))
#define OH_QOS_GEWU_INVALID_REQUEST_ID (static_cast<OH_QoS_GewuRequest>(0xffffffffU))

// dlopen 标志
const int GEWU_DLOPEN_FLAGS = RTLD_LAZY | RTLD_LOCAL;
```

---

## 5. 文件路径常量

### 5.1 内核节点

| 路径 | 用途 | 访问模式 |
|------|------|----------|
| `/proc/thread-self/sched_qos_ctrl` | QoS 控制 | O_RDWR |
| `/proc/self/sched_rtg_ctrl` | RTG 控制 | O_RDWR |

**证据代码**:

```cpp
// services/src/qos_interface.cpp:37-38
char fileName[] = "/proc/self/sched_rtg_ctrl";
int fd = open(fileName, O_RDWR);

// services/src/qos_interface.cpp:48-49
char fileName[] = "/proc/thread-self/sched_qos_ctrl";
int fd = open(fileName, O_RDWR);
```

### 5.2 系统库路径

| 路径 | 用途 |
|------|------|
| `/system/lib64/libqos.so` | NDK 库 |
| `/system/lib64/libqos.z.so` | QoS 核心库 |
| `/system/lib64/libconcurrent_task_client.z.so` | IPC 客户端库 |
| `/system/lib64/libconcurrentsvc.z.so` | 系统服务库 |
| `/system/lib64/libgewu_client.z.so` | GEWU AI 库 |

---

## 6. 配置表汇总

### 6.1 SA 配置

**文件**: `sa_profile/1912.json`

```json
{
  "process": "concurrent_task_service",
  "systemability": [{
    "name": 1912,
    "libpath": "libconcurrentsvc.z.so",
    "run-on-create": true,
    "distributed": false,
    "dump-level": 1
  }]
}
```

| 字段 | 值 | 描述 |
|------|-----|------|
| `process` | concurrent_task_service | 进程名 |
| `name` | 1912 | SA ID |
| `libpath` | libconcurrentsvc.z.so | 库路径 |
| `run-on-create` | true | 创建时运行 |
| `distributed` | false | 非分布式 |
| `dump-level` | 1 | dump 级别 |

### 6.2 Init 配置

**文件**: `etc/init/concurrent_task_service.cfg`

```json
{
  "jobs": [{
    "name": "post-fs-data",
    "cmds": ["start concurrent_task_service"]
  }],
  "services": [{
    "name": "concurrent_task_service",
    "path": ["/system/bin/sa_main", "/system/profile/concurrent_task_service.json"],
    "importance": -20,
    "uid": "system",
    "gid": ["system", "shell"],
    "secon": "u:r:concurrent_task_service:s0"
  }]
}
```

| 字段 | 值 | 描述 |
|------|-----|------|
| `name` | concurrent_task_service | 服务名 |
| `importance` | -20 | 优先级 |
| `uid` | system | 用户 ID |
| `gid` | system, shell | 组 ID |
| `secon` | u:r:concurrent_task_service:s0 | SELinux 上下文 |

---

## 7. 日志配置

### 7.1 日志标签

```cpp
// include/concurrent_task_log.h
#define CONCUR_LOGD(...)  // 调试日志
#define CONCUR_LOGE(...)  // 错误日志
```

### 7.2 日志标签定义

```cpp
// frameworks/native/qos_ndk.cpp
CONCUR_LOGE("[Gewu] failed to load symbol: %{public}s, error: %{public}s", symbolName, dlerror());
CONCUR_LOGE("[Gewu] failed to load library: %{public}s, error: %{public}s", GEWU_CLIENT_LIB, dlerror());

// services/src/qos_interface.cpp
CONCUR_LOGE("[Interface] task %{public}d belong to user %{public}d open rtg node failed", ...);
```

---

## 8. 相关文档

| 文档 | 描述 |
|------|------|
| [04_Build_Targets.md](./04_Build_Targets.md) | 构建配置 |
| [01_Architecture.md](./01_Architecture.md) | 架构图 |
| [02_NDK_API.md](./02_NDK_API.md) | API 参数 |
| [05_Security.md](./05_Security.md) | 安全配置 |
