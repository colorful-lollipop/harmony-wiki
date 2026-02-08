# API 参考

> **重要说明**: HiDumper 是 **Native C++ 服务**，**不提供 N-API (JS API)**。  
> 本文档描述的是 Native 接口，用于 C/C++ 组件集成。

## 1. Native API 概览

| 接口名称 | 头文件 | 稳定性 | 用途 |
|---------|-------|-------|------|
| DumpManagerClient | `dump_manager_client.h` | 稳定 | Dump 请求入口 |
| IDumpBroker | `idump_broker.h` | 稳定 | IPC Broker 接口 |
| DumpUsage | `dump_usage.h` | 稳定 | 内存/CPU 使用统计 |
| DumpCommonUtils | `dump_common_utils.h` | 稳定 | 进程/CPU 工具 |
| DumpManagerCpuClient | `dump_manager_cpu_client.h` | 稳定 | CPU 服务客户端 |

## 2. DumpManagerClient 接口

**头文件**: `interfaces/native/innerkits/include/dump_manager_client.h`

### 2.1 类定义

```cpp
class DumpManagerClient : public RemoteAPIHelper<DumpManagerClient> {
public:
    static DumpManagerClient &GetInstance();
    
    int Request(int argc, const char *argv[], const std::string& option,
                std::string &result);
    
private:
    DumpManagerClient();
    ~DumpManagerClient();
};
```

### 2.2 方法说明

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|-------|------|
| GetInstance | - | DumpManagerClient& | 获取单例 |
| Request | argc, argv, option, result | int | 发起 Dump 请求 |

### 2.3 使用示例

```cpp
#include "dump_manager_client.h"

std::string result;
int ret = DumpManagerClient::GetInstance().Request(
    argc, argv, "-c", result
);
if (ret == 0) {
    printf("%s", result.c_str());
}
```

## 3. IDumpBroker 接口 (IPC)

**头文件**: `interfaces/native/innerkits/include/idump_broker.h`

### 3.1 接口定义

```cpp
class IDumpBroker : public IRemoteBroker {
public:
    virtual int Request(int fd, const std::vector<std::string> &args) = 0;
    virtual bool ScanPidOverLimit() = 0;
    virtual int CountFdNums() = 0;
    
    DECLARE_INTERFACE_DESCRIPTOR(u"IDumpBroker");
};
```

### 3.2 方法说明

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|-------|------|
| Request | fd, args | int | 通用 Dump 请求 |
| ScanPidOverLimit | - | bool | FD 泄漏检测 |
| CountFdNums | - | int | 统计 FD 数量 |

## 4. DumpUsage 接口

**头文件**: `interfaces/innerkits/include/dump_usage.h`

### 4.1 类定义

```cpp
class DumpUsage {
public:
    static int GetMemInfo(int32_t pid, std::string &memInfo);
    static int GetPss(int32_t pid, std::string &pssInfo);
    static int GetCpuUsage(double &cpuUsage);
    static int GetProcessName(int32_t pid, std::string &processName);
};
```

### 4.2 方法说明

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|-------|------|
| GetMemInfo | pid, memInfo | int | 获取进程内存信息 |
| GetPss | pid, pssInfo | int | 获取 PSS 内存 |
| GetCpuUsage | cpuUsage | int | 获取系统 CPU 使用率 |
| GetProcessName | pid, processName | int | 获取进程名 |

## 5. DumpCommonUtils 接口

**头文件**: `interfaces/native/innerkits/include/dump_common_utils.h`

### 5.1 工具方法

| 方法 | 功能 |
|-----|------|
| GetPidInfos() | 获取所有进程信息 |
| GetAllPids() | 获取所有 PID 列表 |
| GetProcessNameByPid(pid) | 根据 PID 查进程名 |
| GetCpuInfoByPid(pid) | 根据 PID 查 CPU 信息 |

## 6. CPU 服务客户端

**头文件**: `interfaces/native/innerkits/include/dump_manager_cpu_client.h`

### 6.1 接口定义

```cpp
class DumpManagerCpuClient {
public:
    int Request(DumpCpuData &dumpCpuData);
    int GetCpuUsageByPid(int pid, double &cpuUsage);
};
```

## 7. IPC 接口码

### 7.1 主服务接口码

**头文件**: `interfaces/native/innerkits/include/hidumper_service_ipc_interface_code.h`

| 接口码 | 值 | 功能 |
|-------|-----|------|
| DUMP_REQUEST_FILEFD | 0 | 文件描述符请求 |
| SCAN_PID_OVER_LIMIT | 1 | 扫描超限 PID |
| COUNT_FD_NUMS | 2 | 统计 FD 数量 |

### 7.2 CPU 服务接口码

**头文件**: `interfaces/native/innerkits/include/hidumper_cpu_service_ipc_interface_code.h`

| 接口码 | 值 | 功能 |
|-------|-----|------|
| DUMP_REQUEST_CPUINFO | 0 | 请求 CPU 信息 |
| DUMP_USAGE_ONLY | 1 | 仅请求使用率 |

## 8. System Ability ID

| SA ID | 服务名 | 头文件 |
|-------|-------|-------|
| 1212 | DumpManagerService | `services/native/include/inner/dump_service_id.h` |
| 1215 | DumpManagerCpuService | `services/native/include/inner/dump_service_id.h` |

## 9. 错误码

**头文件**: `utils/native/include/dump_errors.h`

| 错误码 | 定义 | 说明 |
|-------|-----|------|
| ERR_OK | 0 | 成功 |
| ERR_DUMP_FAILED | -1 | Dump 失败 |
| ERR_INVALID_PARAM | -2 | 参数无效 |
| ERR_NO_PERMISSION | -3 | 无权限 |
| ERR_SERVICE_NOT_READY | -4 | 服务未就绪 |

## 相关文档

- [系统架构](./01_Architecture.md)
- [构建系统](./03_Build_System.md)
- [安全评审](./05_Security_Review.md)
