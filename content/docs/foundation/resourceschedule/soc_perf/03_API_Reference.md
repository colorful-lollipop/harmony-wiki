# API 参考

## Inner API

### SocPerfClient

**头文件**：`interfaces/inner_api/socperf_client/include/socperf_client.h`

**命名空间**：`OHOS::SOCPERF`

#### API 清单

| JS API | C++ 接口 | 参数 | 返回值 | 同步/异步 |
|--------|----------|------|--------|----------|
| N/A | `GetInstance()` | - | `SocPerfClient&` | 同步 |
| N/A | `PerfRequest()` | `int32_t cmdId, const std::string& msg` | `void` | 异步 IPC |
| N/A | `PerfRequestEx()` | `int32_t cmdId, bool onOffTag, const std::string& msg` | `void` | 异步 IPC |
| N/A | `PowerLimitBoost()` | `bool onOffTag, const std::string& msg` | `void` | 异步 IPC |
| N/A | `ThermalLimitBoost()` | `bool onOffTag, const std::string& msg` | `void` | 异步 IPC |
| N/A | `LimitRequest()` | `int32_t clientId, const std::vector<int32_t>& tags, const std::vector<int64_t>& configs, const std::string& msg` | `void` | 异步 IPC |
| N/A | `SetRequestStatus()` | `bool status, const std::string& msg` | `void` | 异步 IPC |
| N/A | `SetThermalLevel()` | `int32_t level` | `void` | 异步 IPC |
| N/A | `RequestDeviceMode()` | `const std::string& mode, bool status` | `void` | 异步 IPC |
| N/A | `RequestCmdIdCount()` | `const std::string& msg` | `std::string` | 异步 IPC |

#### 客户端绑定位置

| 符号 | 文件:行号 |
|------|----------|
| `SocPerfClient::GetInstance()` | `socperf_client.cpp:41-44` |
| `SocPerfClient::PerfRequest()` | `socperf_client.cpp:113-121` |
| `SocPerfClient::CheckClientValid()` | `socperf_client.cpp:47-78` |

### IPC 接口

#### ISocPerf (IDL 生成)

**接口定义**：`interfaces/inner_api/socperf_client/ISocPerf.idl`

**Stub 实现**：`socperf_server.cpp`

| 方法 | 权限 | 校验 |
|------|------|------|
| `PerfRequest` | ✅ | `HasPerfPermission()` |
| `PerfRequestEx` | ✅ | `HasPerfPermission()` |
| `PowerLimitBoost` | ✅ | `HasPerfPermission()` |
| `ThermalLimitBoost` | ✅ | `HasPerfPermission()` |
| `LimitRequest` | ✅ | `HasPerfPermission()` |
| `SetRequestStatus` | ✅ | `HasPerfPermission()` |
| `SetThermalLevel` | ✅ | `HasPerfPermission()` |
| `RequestDeviceMode` | ✅ | `HasPerfPermission()` |
| `RequestCmdIdCount` | ✅ | `HasPerfPermission()` |
| `Dump` | ✅ | `AllowDump()` |

#### IPC 调用链

```
SocPerfClient API
    ↓
CheckClientValid() → GetSystemAbility(1906) → iface_cast<ISocPerf>
    ↓
client->IPC_Method(cmdId, ...)
    ↓
SocPerfServer::OnMessage()
    ↓
HasPerfPermission() (HAP Token → System App 检查)
    ↓
socPerf.Method(cmdId, ...)
```

**证据**：`interfaces/inner_api/socperf_client/src/socperf_client.cpp:113-121`

### 参数校验

| 参数 | 校验方式 | 越界处理 |
|------|---------|---------|
| `cmdId` | 配置文件中定义 | 无效则忽略 |
| `mode` | 长度检查 `MAX_MODE_LEN=64` | `socperf_client.cpp:186` |
| `tags` | 配置文件中 resId 校验 | `SocPerfConfig::IsValidResId()` |
| `configs` | 类型/范围校验 | 按配置生效 |

**证据**：`interfaces/inner_api/socperf_client/src/socperf_client.cpp:186`

```cpp
if (!CheckClientValid() || mode.length() > MAX_MODE_LEN) {
    return;
}
```

### 错误码

| 错误类型 | 处理方式 |
|---------|---------|
| 客户端无效 | 返回，日志输出错误 |
| 权限不足 | 返回，HiSysEvent 上报 |
| 配置无效 | 忽略，日志调试 |
| IPC 失败 | 重置客户端连接 |

### 动作类型枚举

**头文件**：`interfaces/inner_api/socperf_client/include/socperf_action_type.h`

```cpp
enum ActionType : uint32_t {
    ACTION_TYPE_PERF,      // 性能
    ACTION_TYPE_POWER,     // 功耗
    ACTION_TYPE_THERMAL,   // 热管理
    ACTION_TYPE_PERFLVL,   // 性能等级
    ACTION_TYPE_BATTERY,   // 电池
    ACTION_TYPE_MAX
};
```

## 内部 API

### SocPerf 核心类

**头文件**：`services/core/include/socperf.h`

| 方法 | 职责 |
|------|------|
| `Init()` | 配置加载初始化 |
| `PerfRequest()` | 性能请求处理 |
| `PerfRequestEx()` | 带开关的性能请求 |
| `PowerLimitBoost()` | 功耗限频 boost |
| `ThermalLimitBoost()` | 热限频 boost |
| `LimitRequest()` | 限频请求 |
| `SetRequestStatus()` | 服务开关状态 |
| `SetThermalLevel()` | 热等级设置 |
| `RequestDeviceMode()` | 设备模式请求 |
| `RequestCmdIdCount()` | 命令计数查询 |

### SocPerfConfig 配置类

**头文件**：`services/core/include/socperf_config.h`

| 方法 | 职责 |
|------|------|
| `Init()` | 配置初始化 |
| `IsGovResId()` | 治理资源 ID 校验 |
| `IsValidResId()` | 有效资源 ID 校验 |
| `GetInstance()` | 单例获取 |

### SocPerfThreadWrap 线程封装

**头文件**：`services/core/include/socperf_thread_wrap.h`

| 方法 | 职责 |
|------|------|
| `Init()` | 线程初始化 |
| `Execute()` | 执行调频动作 |
