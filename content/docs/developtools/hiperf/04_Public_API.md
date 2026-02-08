# 对外 API

## 目的

本文档说明 hiperf 提供的 C++ Inner API，包括 hiperf_client 和 hiperf_local。

## 适用范围

- 需要集成 hiperf 功能的应用开发者
- 系统开发者

## 重要说明

**hiperf 不提供 N-API (JS API) 接口**，仅提供 C++ Inner API。

## hiperf_client API

### 概述

**路径**: `interfaces/innerkits/native/hiperf_client/`

**头文件**: `include/hiperf_client.h`

**库文件**: `libhiperf_client.so`

**命名空间**: `OHOS::Developtools::HiPerf::HiperfClient`

### RecordOption 类

**用途**: 配置采样选项

**定义位置**: `interfaces/innerkits/native/hiperf_client/include/hiperf_client.h:38-233`

#### 配置方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `SetTargetSystemWide(bool)` | enable | 全系统采样 |
| `SetCompressData(bool)` | enable | 压缩数据 |
| `SetSelectCpus(vector<int>)` | cpus | 指定 CPU |
| `SetTimeStopSec(int)` | seconds | 采样时长 |
| `SetFrequency(int)` | freq | 采样频率 |
| `SetPeriod(int)` | period | 采样周期 |
| `SetSelectEvents(vector<string>)` | events | 选择事件 |
| `SetSelectGroups(vector<string>)` | groups | 事件分组 |
| `SetNoInherit(bool)` | enable | 不继承子进程 |
| `SetSelectPids(vector<pid_t>)` | pids | 指定进程 |
| `SetSelectTids(vector<pid_t>)` | tids | 指定线程 |
| `SetExcludePerf(bool)` | exclude | 排除 hiperf 自身 |
| `SetCpuPercent(int)` | percent | CPU 使用限制 |
| `SetOffCPU(bool)` | offCPU | 追踪 off-cpu |
| `SetCallGraph(string)` | type | 调用链类型 (fp/dwarf) |
| `SetDelayUnwind(bool)` | delay | 延迟回溯 |
| `SetDisableUnwind(bool)` | disable | 禁用回溯 |
| `SetSymbolDir(string)` | dir | 符号目录 |
| `SetDataLimit(string)` | limit | 数据大小限制 |
| `SetAppPackage(string)` | package | 应用包名 |
| `SetClockId(string)` | clockId | 时钟源 |
| `SetMmapPages(int)` | pages | mmap 页数 |
| `SetReport(bool)` | report | 采样后报告 |
| `SetBackTrack(bool)` | backtrack | 回溯采集 |
| `SetBackTrackSec(int)` | seconds | 回溯时长 |

#### 代码示例

```cpp
#include "hiperf_client.h"

using namespace OHOS::Developtools::HiPerf::HiperfClient;

// 创建配置
RecordOption option;

// 基本配置
option.SetSelectPids({1234});        // 采样 PID 1234
option.SetTimeStopSec(10);            // 采样 10 秒
option.SetFrequency(4000);            // 每秒 4000 次采样

// 调用链配置
option.SetCallGraph("dwarf");         // 使用 DWARF 回溯

// 输出配置
option.SetCompressData(true);         // 压缩数据
```

### Client 类

**用途**: 控制 hiperf 采样进程

**定义位置**: `interfaces/innerkits/native/hiperf_client/include/hiperf_client.h:235-355`

#### 构造函数

```cpp
explicit Client(const std::string &outputDir = TEMP_BIN_PATH);
```

**参数**:
- `outputDir`: 输出目录，默认为 `/data/local/tmp/`

#### 控制方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `Start()` | bool | 启动采样（采样当前进程） |
| `Start(vector<string> args)` | bool | 使用参数启动 |
| `Start(RecordOption option)` | bool | 使用配置启动 |
| `RunHiperfCmdSync(RecordOption)` | bool | 同步执行 |
| `PrePare(RecordOption)` | bool | 准备采样 |
| `StartRun()` | bool | 开始采样（配合 Prepare） |
| `Pause()` | bool | 暂停采样 |
| `Resume()` | bool | 恢复采样 |
| `Output()` | bool | 输出数据 |
| `Stop()` | bool | 停止采样 |
| `IsReady()` | bool | 检查就绪状态 |
| `Setup(string outputDir)` | bool | 设置输出目录 |

#### 调试方法

| 方法 | 说明 |
|------|------|
| `SetDebugMode()` | 启用调试日志 |
| `SetDebugMuchMode()` | 启用详细日志 |
| `EnableHilog()` | 启用 HiLog |

#### 获取方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `GetOutputDir()` | string | 获取输出目录 |
| `GetCommandPath()` | string | 获取命令路径 |
| `GetOutputPerfDataPath()` | string | 获取输出文件路径 |

#### 代码示例

```cpp
#include "hiperf_client.h"
#include <unistd.h>

using namespace OHOS::Developtools::HiPerf::HiperfClient;

void SampleMyProcess() {
    // 创建客户端
    Client client("/data/local/tmp/");
    
    // 检查就绪
    if (!client.IsReady()) {
        // 错误处理
        return;
    }
    
    // 配置采样
    RecordOption option;
    option.SetSelectPids({getpid()});
    option.SetTimeStopSec(5);
    option.SetFrequency(1000);
    option.SetCallGraph("fp");
    
    // 启动采样
    if (!client.Start(option)) {
        // 错误处理
        return;
    }
    
    // 执行业务逻辑...
    
    // 停止采样
    client.Stop();
    
    // 获取输出文件路径
    std::string outputPath = client.GetOutputPerfDataPath();
    // 处理 perf.data 文件
}

void AdvancedSample() {
    Client client;
    RecordOption option;
    
    // 全系统采样配置
    option.SetTargetSystemWide(true);
    option.SetSelectCpus({0, 1, 2, 3});
    option.SetSelectEvents({"hw-cpu-cycles", "hw-cache-misses"});
    option.SetTimeStopSec(10);
    
    // 同步执行（阻塞直到完成）
    if (client.RunHiperfCmdSync(option)) {
        // 采样完成
    }
}
```

### 通信协议

Client 与 hiperf 进程通过匿名管道通信：

**命令格式**:
```
START\n    - 开始采样
PAUSE\n    - 暂停采样
RESUME\n   - 恢复采样
OUTPUT\n   - 输出数据
STOP\n     - 停止采样
```

**响应格式**:
```
OK\n      - 成功
FAIL\n    - 失败
```

**代码位置**: `interfaces/innerkits/native/hiperf_client/include/hiperf_client.h:26-35`

## hiperf_local API

### 概述

**路径**: `interfaces/innerkits/native/hiperf_local/`

**头文件**: `include/lperf.h`

**库文件**: `libhiperf_local.so`

**命名空间**: `OHOS::Developtools::HiPerf::HiPerfLocal`

### Lperf 类

**用途**: 轻量级本地堆栈采样

**定义位置**: `interfaces/innerkits/native/hiperf_local/include/lperf.h:25-42`

#### 获取实例

```cpp
static Lperf& GetInstance();
```

**说明**: 单例模式，线程安全

#### 采样方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `StartProcessStackSampling` | tids, freq, ms, parseMini | int | 启动采样 |
| `CollectSampleStackByTid` | tid, stack | int | 收集堆栈 |
| `FinishProcessStackSampling` | - | int | 完成采样 |

**参数说明**:
- `tids`: 要采样的线程 ID 列表
- `freq`: 采样频率
- `milliseconds`: 采样时长
- `parseMiniDebugInfo`: 是否解析 MiniDebugInfo
- `tid`: 线程 ID
- `stack`: 输出堆栈字符串

**返回值**:
- `0`: 成功
- 非零: 错误码

#### 代码示例

```cpp
#include "lperf.h"
#include <vector>

using namespace OHOS::Developtools::HiPerf::HiPerfLocal;

void LightweightSample() {
    // 获取实例
    Lperf& lperf = Lperf::GetInstance();
    
    // 要采样的线程
    std::vector<int> tids = {1001, 1002, 1003};
    
    // 启动采样（100Hz，持续 1000ms）
    int ret = lperf.StartProcessStackSampling(tids, 100, 1000, false);
    if (ret != 0) {
        // 错误处理
        return;
    }
    
    // 收集每个线程的堆栈
    for (int tid : tids) {
        std::string stack;
        ret = lperf.CollectSampleStackByTid(tid, stack);
        if (ret == 0) {
            // 处理堆栈信息
            ProcessStack(stack);
        }
    }
    
    // 完成采样
    lperf.FinishProcessStackSampling();
}
```

### 与 hiperf_client 对比

| 特性 | hiperf_client | hiperf_local |
|------|---------------|--------------|
| 功能 | 完整采样控制 | 轻量级堆栈采样 |
| 输出 | perf.data 文件 | 字符串堆栈 |
| 通信 | fork + 管道 | 直接调用 |
| 依赖 | hiperf 二进制 | libdfx_dumpcatcher |
| 开销 | 较高 | 较低 |
| 使用场景 | 完整性能分析 | 实时轻量级采样 |

## API 使用流程

### hiperf_client 流程

```
应用进程
    │
    ├── 1. 创建 Client
    │   Client client("/data/local/tmp/");
    │
    ├── 2. 配置选项
    │   RecordOption option;
    │   option.SetSelectPids({pid});
    │   option.SetTimeStopSec(10);
    │
    ├── 3. 启动采样
    │   client.Start(option);
    │   ├── fork()
    │   ├── 子进程 execv("/system/bin/hiperf")
    │   └── 建立管道通信
    │
    ├── 4. 执行业务逻辑
    │   ...
    │
    ├── 5. 停止采样
    │   client.Stop();
    │   └── 发送 STOP 命令
    │
    └── 6. 获取结果
        string path = client.GetOutputPerfDataPath();
```

### hiperf_local 流程

```
应用进程
    │
    ├── 1. 获取实例
    │   Lperf& lperf = Lperf::GetInstance();
    │
    ├── 2. 启动采样
    │   lperf.StartProcessStackSampling(tids, freq, ms, false);
    │   └── 直接调用 libdfx_dumpcatcher
    │
    ├── 3. 收集堆栈
    │   lperf.CollectSampleStackByTid(tid, stack);
    │
    └── 4. 完成采样
        lperf.FinishProcessStackSampling();
```

## 错误处理

### hiperf_client 错误

| 错误场景 | 返回值 | 说明 |
|----------|--------|------|
| 未就绪 | false | IsReady() 返回 false |
| 管道创建失败 | false | pipe() 失败 |
| fork 失败 | false | fork() 返回 -1 |
| 启动超时 | false | 等待响应超时 |
| 命令失败 | false | hiperf 返回错误 |

### hiperf_local 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 非零 | 具体错误（依赖 libdfx_dumpcatcher） |

## 线程安全

### hiperf_client

- `Client` 类**非线程安全**
- 单线程使用，或外部加锁

### hiperf_local

- `GetInstance()` 返回单例，线程安全
- 其他方法**非线程安全**

## 性能考虑

### hiperf_client

- fork/exec 开销较大
- 适合长时间采样（>1秒）
- 数据量大，需要存储空间

### hiperf_local

- 直接调用，开销小
- 适合短时间、高频采样
- 数据量小，内存中处理

## 相关跳转

- [项目定位](01_Overview.md) - 了解使用场景
- [目录结构](02_Directory_Structure.md) - 了解代码组织
- [架构说明](03_Architecture.md) - 了解内部实现
- [安全风险](08_Security.md) - 了解安全限制
