# 安全风险评审

## 评审范围

本评审覆盖 `frame_aware_sched` 组件的以下部分：
- `interfaces/innerkits/frameintf/rtg_interface.cpp` - RTG 控制接口
- `interfaces/innerkits/frameintf/frame_ui_intf.cpp` - UI 帧接口
- `interfaces/innerkits/frameintf/frame_msg_intf.cpp` - 帧消息接口
- `qos_manager/src/qos_common.cpp` - QoS 管理
- `frameworks/core/frame_aware_policy/src/intellisense_server.cpp` - 智能感知服务器
- `frameworks/core/frame_aware_policy/src/para_config.cpp` - 参数配置解析
- `profiles/hwrme.xml` - 配置文件

---

## 攻击面分析

| 攻击面 | 描述 | 风险等级 | 代码位置 |
|--------|------|----------|---------|
| **ioctl 系统调用** | 直接调用 `/proc/self/sched_rtg_ctrl` 和 `/dev/auth_ctrl` | **高** | `rtg_interface.cpp:108`, `qos_common.cpp:56` |
| **用户输入** | `ReportAppInfo` 等接口接收 PID/UID/bundleName | **中** | `frame_msg_intf.cpp:93` |
| **XML 配置** | `hwrme.xml` 配置文件解析 | **中** | `para_config.cpp` |
| **字符串处理** | sprintf_s 格式化 RTG 属性字符串 | **低** | `rtg_interface.cpp:261` |
| **进程间通信** | FFRT 任务队列（内部使用） | **低** | `frame_msg_intf.cpp:57` |

---

## 信任边界

```
┌─────────────────────────────────────────────────────┐
│                   用户空间                           │
│  ┌───────────────┐      ┌───────────────┐          │
│  │ 应用进程      │      │ 系统服务       │          │
│  │ (Collector)  │      │ (Policy)      │          │
│  │               │      │               │          │
│  │ libframe_ui_  │      │ libframe_msg_ │          │
│  │ intf.z.so     │      │ intf.z.so     │          │
│  └───────┬───────┘      └───────┬───────┘          │
│          │                      │                  │
│          │  ReportAppInfo()     │                  │
│          │  (PID/UID/bundleName)│                  │
│          └──────┬───────────────┘                  │
│                 │                                  │
│         ┌──────▼──────┐                           │
│         │  RTG Interface │                        │
│         │  (librtg_      │                        │
│         │   interface.z.so)                       │
│         └──────┬──────┘                           │
│                │ ioctl                            │
│    ┌───────────▼───────────┐                       │
│    │ /proc/self/sched_rtg │ ←── 内核接口          │
│    │ _ctrl                │                        │
│    │ /dev/auth_ctrl       │ ←── QoS 设备          │
│    └───────────────────────┘                       │
│                   内核                              │
└─────────────────────────────────────────────────────┘
```

---

## 风险点与修复建议

### R1: 【高危】ioctl 调用缺乏权限校验

**位置**: `interfaces/innerkits/frameintf/rtg_interface.cpp:97-119`

**证据**:
```cpp
// rtg_interface.cpp:97-119
int EnableRtg(bool flag)
{
    int ret = 0;
    struct rtg_enable_data enableData;
    char configStr[] = "load_freq_switch:1;sched_cycle:1";
    enableData.enable = flag;
    enableData.len = sizeof(configStr);
    enableData.data = configStr;
    if (g_fd < 0) {
        return g_fd;
    }
    ret = ioctl(g_fd, CMD_ID_SET_ENABLE, &enableData);  // 第108行：无权限检查
    if (ret != 0) {
        RME_LOGE("set rtg config to [%{public}d] failed...", flag);
    }
    return 0;
}
```

**问题描述**:  
`rtg_interface.cpp` 中所有 15 个 ioctl 调用均未检查调用者权限：
- `EnableRtg()` (行 108)
- `AddThreadToRtg()` (行 134)
- `AddThreadsToRtg()` (行 169)
- `RemoveRtgThread()` (行 193)
- `RemoveRtgThreads()` (行 222)
- `DestroyRtgGrp()` (行 241)
- `SetFrameRateAndPrioType()` (行 266)
- `BeginFrameFreq()` (行 292)
- `EndFrameFreq()` (行 305)
- `EndScene()` (行 319)
- `SetMinUtil()` (行 335)
- `SetMargin()` (行 358)
- `SearchRtgForTid()` (行 375)
- `GetRtgEnable()` (行 389)

**触发路径**:
```
应用/服务 ──> AddThreadToRtg(tid, grpId) 
    ──> ioctl(g_fd, CMD_ID_SET_RTG, &grp_data)
    ──> 内核 /proc/self/sched_rtg_ctrl
```

**影响评估**:
- **可利用性**: 高 - 任何加载 librtg_interface.z.so 的进程都可调用
- **权限提升**: 中 - 可修改系统调度策略，干扰其他进程
- **DoS**: 高 - 错误 RTG 配置可导致系统卡顿

**修复建议**:
```cpp
// 在 rtg_interface.cpp 中添加权限检查
#include <unistd.h>

static bool CheckPrivilege() {
    // 仅允许 system 用户或具有 RESOURCE_SCHEDULE_MANAGER 权限的进程
    uid_t uid = getuid();
    return (uid == 0 || uid == 1000);  // root 或 system
}

int EnableRtg(bool flag) {
    if (!CheckPrivilege()) {
        RME_LOGE("Permission denied for RTG operation");
        return -EPERM;
    }
    // ... 原有代码
}
```

---

### R2: 【高危】QoS Auth ioctl 同样缺乏权限校验

**位置**: `qos_manager/src/qos_common.cpp:38-62`

**证据**:
```cpp
// qos_common.cpp:38-62
int AuthEnable(int pid, unsigned int flag, unsigned int status)
{
    struct AuthCtrlData data;
    int fd;
    int ret;

    fd = TrivalOpenAuthCtrlNode();  // 行44：打开 /dev/auth_ctrl
    if (fd < 0) {
        RME_LOGE("thread %{public}d open auth node failed", gettid());
        return fd;
    }

    data.pid = pid;
    data.rtgFlag = flag;
    data.qosFlag = AF_QOS_DELEGATED;
    data.status = status;
    data.type = AUTH_ENABLE;

    ret = ioctl(fd, BASIC_AUTH_CTRL_OPERATION, &data);  // 行56：无权限检查
    close(fd);
    return ret;
}
```

**证据2** (`qos_common.h:47-48`):
```cpp
#define BASIC_AUTH_CTRL_OPERATION \
    _IOWR(0xCD, 1, struct AuthCtrlData)
```

**问题描述**:  
`qos_common.cpp` 中的 `AuthEnable()`、`AuthPause()`、`AuthDelete()` 函数直接调用 `/dev/auth_ctrl` 的 ioctl，无权限校验。

**触发路径**:
```
FrameMsgIntf::ReportAppInfo() 
    ──> IntelliSenseServer::AuthForeground()
    ──> AuthEnable(pid, flag, status)
    ──> ioctl(/dev/auth_ctrl)
```

**影响评估**:
- **可利用性**: 中 - 需通过 FrameMsgIntf 接口
- **权限提升**: 高 - 可授权任意进程 QoS 优先级
- **DoS**: 中 - 可禁用关键系统服务 QoS

**修复建议**: 同 R1，添加 UID 校验

---

### R3: 【中危】未捕获的 std::stoi() 异常

**位置**: `frameworks/core/frame_aware_policy/src/intellisense_server.cpp:59`

**证据**:
```cpp
// intellisense_server.cpp:53-63
void IntelliSenseServer::Init()
{
    if (!ReadXml()) {
        RME_LOGI("[Init]: readXml failed!");
        return;
    }
    m_switch = std::stoi(m_generalPara["enable"]);  // 行59：可能抛出异常
    if (!m_switch) {
        RME_LOGI("[Init]:xml switch close!");
        return;
    }
    // ...
}
```

**问题描述**:  
`std::stoi()` 在以下情况抛出异常：
- 字符串为空 → `std::invalid_argument`
- 包含非数字字符 → `std::invalid_argument`
- 超出 `int` 范围 → `std::out_of_range`

**影响评估**:
- **可用性**: 高 - 未捕获异常导致进程终止
- **利用难度**: 低 - 需篡改配置文件

**修复建议**:
```cpp
try {
    m_switch = std::stoi(m_generalPara["enable"]);
} catch (const std::exception& e) {
    RME_LOGE("Invalid enable value in config: %s", e.what());
    m_switch = 0;  // 默认值
}
```

---

### R4: 【中危】输入验证逻辑错误

**位置**: `frameworks/core/frame_aware_policy/src/para_config.cpp:133`

**证据**:
```cpp
// para_config.cpp:131-136
int curVal = atoi(toSplitStr.substr(0, pos).c_str());
if (curVal <= 0 && curVal > maxVal) {  // 行133：逻辑错误！
    RME_LOGE("[SplitString]:get data error!");
    return;
}
```

**问题描述**:  
条件 `curVal <= 0 && curVal > maxVal` 永远为假（一个数不可能同时 <=0 且 > maxVal）。验证永远不会触发，允许无效值通过。

**修复建议**:
```cpp
if (curVal <= 0 || curVal > maxVal) {  // 改为 ||
    RME_LOGE("[SplitString]:get data error!");
    return;
}
```

---

### R5: 【中危】PID/UID 输入未验证

**位置**: `interfaces/innerkits/frameintf/frame_msg_intf.cpp:93-104`

**证据**:
```cpp
// frame_msg_intf.cpp:93-104
void FrameMsgIntf::ReportAppInfo(const int pid, const int uid, 
                                 const std::string bundleName, ThreadState state)
{
    std::lock_guard<ffrt::mutex> autoLock(frameMsgIntfMutex_);
    if (taskQueue_ == nullptr) {
        return;
    }
    RME_LOGI("ReportProcessInfo pid is %{public}d, uid is %{public}d", pid, uid);
    // 无验证！
    taskQueue_>-submit([pid, uid, bundleName, state] {
        IntelliSenseServer::GetInstance().ReportAppInfo(pid, uid, bundleName, state);
    });
}
```

**问题描述**:  
所有 `ReportXXX` 函数接受 PID/UID 参数但未验证：
- 负数 PID/UID
- 极大值（超出有效范围）
- 非当前进程 PID

**影响评估**:
- **信息泄露**: 可查询其他进程 RTG 状态
- **DoS**: 可操作其他进程的调度组

**修复建议**:
```cpp
void FrameMsgIntf::ReportAppInfo(const int pid, const int uid, ...) {
    // 验证 PID/UID 范围
    if (pid <= 0 || uid < 0) {
        RME_LOGE("Invalid pid/uid: %d/%d", pid, uid);
        return;
    }
    // 验证 PID 属于当前进程或子进程
    if (pid != getpid() && pid != getppid()) {
        RME_LOGW("PID %d does not belong to current process", pid);
        return;
    }
    // ... 原有代码
}
```

---

### R6: 【低危】字符串缓冲区潜在截断

**位置**: `interfaces/innerkits/frameintf/rtg_interface.cpp:260-261`

**证据**:
```cpp
// rtg_interface.cpp:254-266
int SetFrameRateAndPrioType(int rtgId, int rate, int rtgType, int realInterval)
{
    // ...
    char str_data[MAX_LENGTH] = {};  // MAX_LENGTH = 100
    (void)sprintf_s(str_data, sizeof(str_data), 
                    "rtgId:%d;rate:%d;type:%d", rtgId, rate, rtgType);
    // ...
}
```

**问题描述**:  
使用 `sprintf_s` 是安全的，但如果格式化后的字符串超过 100 字节会被截断。

**修复建议**:  
检查返回值确保无截断：
```cpp
int ret = sprintf_s(str_data, sizeof(str_data), 
                    "rtgId:%d;rate:%d;type:%d", rtgId, rate, rtgType);
if (ret < 0 || ret >= MAX_LENGTH) {
    RME_LOGE("String truncation in SetFrameRateAndPrioType");
    return -1;
}
```

---

### R7: 【低危】全局文件描述符竞争条件

**位置**: `interfaces/innerkits/frameintf/rtg_interface.cpp:38-39`

**证据**:
```cpp
// rtg_interface.cpp:38-39
static int g_fd = -1;       // 全局文件描述符
static FILE* g_f = nullptr; // 全局文件指针
```

**问题描述**:  
- 全局 `g_fd` 在多线程环境下存在竞争条件
- 构造函数 (`BasicOpenRtgNode`) 和析构函数 (`BasicCloseRtgNode`) 无锁保护
- 一个线程关闭 fd 时，另一线程可能正在使用

**修复建议**:  
使用 RAII 封装或添加互斥锁：
```cpp
static std::mutex g_rtgMutex;
static std::unique_ptr<RtgFileHandle> g_rtgHandle;

class RtgFileHandle {
    int fd_ = -1;
public:
    RtgFileHandle() { fd_ = open("/proc/self/sched_rtg_ctrl", O_RDWR); }
    ~RtgFileHandle() { if (fd_ >= 0) close(fd_); }
    int get() { return fd_; }
};
```

---

### R8: 【低危】配置文件路径硬编码

**位置**: `frameworks/core/frame_aware_policy/src/intellisense_server.cpp:39`

**证据**:
```cpp
// intellisense_server.cpp:39
static std::string configFilePath = "/system/etc/frame_aware_sched/hwrme.xml";
```

**问题描述**:  
- 硬编码路径可能被符号链接攻击（如果目录权限不当）
- 无签名验证，配置文件可被篡改

**修复建议**:  
1. 使用 `O_NOFOLLOW` 打开文件
2. 添加配置文件签名验证
3. 限制配置文件写权限（仅 system 可写）

---

### R9: 【低危】BundleName 未验证

**位置**: `interfaces/innerkits/frameintf/frame_msg_intf.cpp:93`

**证据**:
```cpp
void FrameMsgIntf::ReportAppInfo(const int pid, const int uid, 
                                 const std::string bundleName, ThreadState state)
{
    // bundleName 直接传递，无长度/字符集验证
    taskQueue_>-submit([pid, uid, bundleName, state] {
        IntelliSenseServer::GetInstance().ReportAppInfo(pid, uid, bundleName, state);
    });
}
```

**问题描述**:  
- BundleName 长度无限制
- 可能包含特殊字符（如换行符）
- 注入风险（如果用于日志输出或文件路径）

**修复建议**:
```cpp
bool ValidateBundleName(const std::string& name) {
    if (name.empty() || name.length() > 256) return false;
    // 只允许字母、数字、点、下划线
    return std::all_of(name.begin(), name.end(), 
        [](char c) { return isalnum(c) || c == '.' || c == '_'; });
}
```

---

### R10: 【低危】atoi() 整数溢出

**位置**: `frameworks/core/frame_aware_policy/src/para_config.cpp:132`

**证据**:
```cpp
// para_config.cpp:132
int curVal = atoi(toSplitStr.substr(0, pos).c_str());
```

**问题描述**:  
- `atoi()` 溢出时行为未定义
- 无法区分 "0" 和无效输入

**修复建议**:  
使用 `strtol` 并检查错误：
```cpp
char* endptr;
long val = strtol(str.c_str(), &endptr, 10);
if (endptr == str.c_str() || *endptr != '\0' || val > INT_MAX) {
    RME_LOGE("Invalid integer value");
    return;
}
```

---

## 风险汇总

| 风险编号 | 风险描述 | 严重程度 | 可能性 | 风险等级 | 代码位置 |
|----------|----------|----------|--------|----------|---------|
| R1 | ioctl 调用缺乏权限校验 | 高 | 中 | **高危** | `rtg_interface.cpp:108` 等 15 处 |
| R2 | QoS Auth ioctl 同样缺乏权限校验 | 高 | 中 | **高危** | `qos_common.cpp:56,82,105` |
| R3 | 未捕获的 std::stoi() 异常 | 中 | 中 | **中危** | `intellisense_server.cpp:59` |
| R4 | 输入验证逻辑错误 | 中 | 高 | **中危** | `para_config.cpp:133` |
| R5 | PID/UID 输入未验证 | 中 | 中 | **中危** | `frame_msg_intf.cpp:93` |
| R6 | 字符串缓冲区潜在截断 | 低 | 低 | **低危** | `rtg_interface.cpp:261` |
| R7 | 全局文件描述符竞争条件 | 低 | 低 | **低危** | `rtg_interface.cpp:38-39` |
| R8 | 配置文件路径硬编码 | 低 | 低 | **低危** | `intellisense_server.cpp:39` |
| R9 | BundleName 未验证 | 低 | 低 | **低危** | `frame_msg_intf.cpp:93` |
| R10 | atoi() 整数溢出 | 低 | 低 | **低危** | `para_config.cpp:132` |

---

## 总结

`frame_aware_sched` 组件存在 **2 个高危**、**3 个中危**、**5 个低危** 安全问题：

### 关键风险
1. **R1/R2 (高危)**: ioctl 接口缺乏权限校验，任何进程都可能操纵系统调度策略
2. **R3 (中危)**: 配置解析异常可导致服务崩溃
3. **R4 (中危)**: 验证逻辑错误导致无效配置被接受

### 建议修复优先级

**P0 (立即修复)**:
- R1: 添加 RTG ioctl 权限检查
- R2: 添加 QoS Auth 权限检查

**P1 (短期修复)**:
- R3: 添加 try-catch 块捕获 stoi 异常
- R4: 修复验证逻辑 (`&&` → `||`)
- R5: 添加 PID/UID 输入验证

**P2 (长期改进)**:
- R6-R10: 代码质量改进

---

## 后续行动

- [ ] 确认 `/proc/sched_rtg_ctrl` 和 `/dev/auth_ctrl` 的访问控制机制
- [ ] 添加调用者身份验证（UID/PID 检查）
- [ ] 评估 SELinux/SMACK 策略
- [ ] 对配置文件添加完整性校验
- [ ] 修复验证逻辑错误 (`para_config.cpp:133`)
- [ ] 添加异常处理 (`intellisense_server.cpp:59`)
