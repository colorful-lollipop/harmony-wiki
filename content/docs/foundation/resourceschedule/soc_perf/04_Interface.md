# 04_Interface - 对外接口文档

本文档详细描述 SOC 统一调频部件的所有对外接口，包括 IPC 接口定义、C++ 客户端 API 使用方式。

## 接口概述

本模块提供 **IPC 接口**（无 N-API），通过 SystemAbility 机制对外暴露功能。所有接口均需权限验证。

| 接口类型 | 通信方式 | 权限要求 |
|----------|----------|----------|
| IPC 接口 | Binder IPC | `ohos.permission.REPORT_RESOURCE_SCHEDULE_EVENT` |

## IPC 接口定义

### ISocPerf.idl

**文件**：`interfaces/inner_api/socperf_client/ISocPerf.idl`

```idl
interface OHOS.SOCPERF.ISocPerf {
   [oneway] void PerfRequest([in] int cmdId, [in] String msg);
   [oneway] void PerfRequestEx([in] int cmdId, [in] boolean onOffTag, [in] String msg);
   [oneway] void SetRequestStatus([in] boolean status, [in] String msg);
   [oneway] void SetThermalLevel([in] int level);
   [oneway] void PowerLimitBoost([in] boolean onOffTag, [in] String msg);
   [oneway] void RequestDeviceMode([in] String mode, [in] boolean status);
   void RequestCmdIdCount([in] String msg, [out] String funcResult);
   [oneway] void ThermalLimitBoost([in] boolean onOffTag, [in] String msg);
   [oneway] void LimitRequest([in] int clientId, [in] int[] tags, [in] long[] configs, [in] String msg);
}
```

## C++ 客户端 API

### SocPerfClient 类

**文件**：`interfaces/inner_api/socperf_client/include/socperf_client.h`

**设计模式**：单例模式

```cpp
namespace OHOS {
namespace SOCPERF {
class SocPerfClient {
public:
    static SocPerfClient& GetInstance();
    // ... 其他接口
};
}
}
```

### API 清单

| API | 参数 | 返回 | 同步/异步 | 说明 |
|-----|------|------|----------|------|
| `PerfRequest` | `cmdId: int32_t`, `msg: std::string` | `void` | 异步 | 性能提频请求 |
| `PerfRequestEx` | `cmdId: int32_t`, `onOffTag: bool`, `msg: std::string` | `void` | 异步 | 带开关的提频 |
| `PowerLimitBoost` | `onOffTag: bool`, `msg: std::string` | `void` | 异步 | 功耗限频 |
| `ThermalLimitBoost` | `onOffTag: bool`, `msg: std::string` | `void` | 异步 | 热限频 |
| `LimitRequest` | `clientId: int32_t`, `tags: std::vector<int32_t>`, `configs: std::vector<int64_t>`, `msg: std::string` | `void` | 异步 | 多项限频请求 |
| `SetRequestStatus` | `status: bool`, `msg: std::string` | `void` | 异步 | 服务开关 |
| `SetThermalLevel` | `level: int32_t` | `void` | 异步 | 设置热等级 |
| `RequestDeviceMode` | `mode: std::string`, `status: bool` | `void` | 异步 | 设备模式请求 |
| `RequestCmdIdCount` | `msg: std::string` | `std::string` | 同步 | 命令计数查询 |

## 接口详细说明

### PerfRequest - 性能提频请求

**功能**：根据预定义的 cmdId 触发性能提频场景

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `cmdId` | `int32_t` | 场景 ID，定义在 XML 配置中 |
| `msg` | `std::string` | 附加信息（自动添加 pid/tid） |

**使用示例**：

```cpp
#include "socperf_client.h"

void Example() {
    SocPerfClient& client = SocPerfClient::GetInstance();
    
    // 应用启动场景
    client.PerfRequest(10001, "app_launch");
    
    // 页面滑动场景
    client.PerfRequest(10002, "page_scroll");
}
```

### PerfRequestEx - 带开关的提频请求

**功能**：支持手动开启/关闭的长期提频事件

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `cmdId` | `int32_t` | 场景 ID |
| `onOffTag` | `bool` | `true`=开启，`false`=关闭 |
| `msg` | `std::string` | 附加信息 |

**使用示例**：

```cpp
void GameModeExample() {
    SocPerfClient& client = SocPerfClient::GetInstance();
    
    // 进入游戏模式 - 开启
    client.PerfRequestEx(20001, true, "game_enter");
    
    // 退出游戏模式 - 关闭
    client.PerfRequestEx(20001, false, "game_exit");
}
```

### PowerLimitBoost - 功耗限频

**功能**：由电源管理模块调用，限制 boost 无法突破功耗限频

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `onOffTag` | `bool` | `true`=开启限频，`false`=关闭限频 |
| `msg` | `std::string` | 附加信息（原因） |

**使用示例**：

```cpp
void ThermalManagementExample() {
    SocPerfClient& client = SocPerfClient::GetInstance();
    
    // 功耗过高 - 开启限频
    client.PowerLimitBoost(true, "high_power_consumption");
    
    // 功耗恢复正常 - 关闭限频
    client.PowerLimitBoost(false, "power_normal");
}
```

### ThermalLimitBoost - 热限频

**功能**：由热管理模块调用，限制 boost 无法突破热限频

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `onOffTag` | `bool` | `true`=开启限频，`false`=关闭限频 |
| `msg` | `std::string` | 附加信息（原因） |

**使用示例**：

```cpp
void ThermalLimitExample() {
    SocPerfClient& client = SocPerfClient::GetInstance();
    
    // 温度过高 - 开启热限频
    client.ThermalLimitBoost(true, "temperature_over_45c");
    
    // 温度恢复 - 关闭热限频
    client.ThermalLimitBoost(false, "temperature_normal");
}
```

### LimitRequest - 多项限频请求

**功能**：热或功耗模块的限频，支持多项值一同设置

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `clientId` | `int32_t` | 调用者标识（如热模块=1，电量模块=2） |
| `tags` | `std::vector<int32_t>` | 资源标签列表 |
| `configs` | `std::vector<int64_t>` | 限频值列表 |
| `msg` | `std::string` | 附加信息 |

**使用示例**：

```cpp
void LimitRequestExample() {
    SocPerfClient& client = SocPerfClient::GetInstance();
    
    // 限频请求
    std::vector<int32_t> tags = {1001, 1002};  // CPU, GPU
    std::vector<int64_t> configs = {1500000, 500000000};  // 频率限制值
    
    client.LimitRequest(1, tags, configs, "thermal_limit");
}
```

### SetThermalLevel - 设置热等级

**功能**：内部接口，用于 perfrequest 提频

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `level` | `int32_t` | 热等级值 |

### RequestDeviceMode - 设备模式请求

**功能**：请求设备进入/退出特定模式

**参数**：

| 参数 | 类型 | 约束 | 说明 |
|------|------|------|------|
| `mode` | `std::string` | 最大 64 字符 | 模式名称 |
| `status` | `bool` | - | `true`=进入，`false`=退出 |

### RequestCmdIdCount - 命令计数查询

**功能**：获取 cmdId 触发次数统计

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `msg` | `std::string` | 查询原因 |

**返回**：

| 类型 | 格式 | 说明 |
|------|------|------|
| `std::string` | `10000:xx,10001:xx` | 逗号分隔的 cmdId:count 对 |

### SetRequestStatus - 服务开关

**功能**：内部接口，启用或禁用 socperf server

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `status` | `bool` | `true`=启用，`false`=禁用 |
| `msg` | `std::string` | 状态变更原因 |

## 客户端实现分析

### 连接管理

**文件**：`interfaces/inner_api/socperf_client/src/socperf_client.cpp`

```cpp
bool SocPerfClient::CheckClientValid()
{
    if (client_ == nullptr) {
        // 获取 SystemAbilityManager
        auto samgr = SystemAbilityManagerClient::GetSystemAbilityManager();
        // 获取 SA 代理
        auto remote = samgr->GetSystemAbility(SOC_PERF_SERVICE_SA_ID);
        // 接口转换
        client_ = iface_cast<ISocPerf>(remote);
        // 注册死亡回调
        if (client_ != nullptr && recipient_ == nullptr) {
            recipient_ = new SocPerfDeathRecipient(*this);
            client_->AsObject()->AddDeathRecipient(recipient_);
        }
    }
    return client_ != nullptr;
}
```

### 线程安全

所有公共 API 方法均使用 `std::lock_guard<std::mutex>` 保护：

```cpp
void SocPerfClient::PerfRequest(int32_t cmdId, const std::string& msg)
{
    std::lock_guard<std::mutex> lock(mutex_);
    if (!CheckClientValid()) {
        return;
    }
    std::string fullMsg = AddPidAndTidInfo(msg);
    client_->PerfRequest(cmdId, fullMsg);
}
```

### PID/TID 注入

自动为每条消息追加调用者进程/线程信息：

```cpp
std::string SocPerfClient::AddPidAndTidInfo(const std::string& msg)
{
    int32_t pid = getpid();
    int32_t tid = gettid();
    return msg + ";pid=" + std::to_string(pid) + ";tid=" + std::to_string(tid);
}
```

## 编译链接

### GN 依赖

```gn
deps += [
    "//foundation/resourceschedule/soc_perf/interfaces/inner_api/socperf_client:socperf_client"
]
```

### 头文件

```cpp
#include "socperf_client.h"
#include "socperf_action_type.h"
```

---

## 相关文档

- 项目概览：[01_Overview](01_Overview.md)
- 架构设计：[02_Architecture](02_Architecture.md)
- 安全风险：[06_SecurityReview](06_SecurityReview.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
