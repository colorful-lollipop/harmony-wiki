# SmartPerf Inner Kit API

## 概述

device_command 模块通过 **Inner Kit** 机制对外暴露 C++ 接口，供其他系统组件调用。Inner Kit 包含三个核心头文件，定义在 `smartperf_device/device_command/interface/` 目录。

**Inner Kit 配置来源**: `bundle.json`

```json
"inner_kits": [
  {
    "header": {
      "header_base": "//developtools/smartperf_host/smartperf_device/device_command/interface",
      "header_files": [
        "GameServicePlugin.h",
        "GameEventCallback.h",
        "GpuCounterCallback.h"
      ]
    },
    "name": "//developtools/smartperf_host/smartperf_device/device_command:smartperf_daemon"
  }
]
```

**代码位置**: `smartperf_device/device_command/interface/`

## Inner Kit 清单

| 头文件 | 职责 | 关键类/接口 |
|--------|------|------------|
| `GameServicePlugin.h` | 游戏服务插件主接口 | `GameServicePlugin` |
| `GameEventCallback.h` | 游戏事件回调接口 | `GameEventCallback` |
| `GpuCounterCallback.h` | GPU 计数器回调接口 | `GpuCounterCallback`, `GpuPerfInfo` |

## 1. GameServicePlugin 接口

**代码位置**: `smartperf_device/device_command/interface/GameServicePlugin.h`

### 接口定义

```cpp
namespace OHOS {
    namespace SmartPerf {
        class GameServicePlugin {
        public:
            uint32_t version;
            const char *pluginName;

            // GPU 性能信息采集
            virtual int32_t StartGetGpuPerfInfo(int64_t duration, int64_t collectDur,
                std::unique_ptr<GpuCounterCallback> callback) = 0;
            virtual int32_t StopGetGpuPerfInfo() = 0;

            // 系统功能状态查询
            virtual std::map<std::string, std::string> GetSystemFunctionStatus(
                std::map<std::string, std::string> &queryParams) = 0;

            // 游戏事件监听
            virtual int32_t RegisterGameEventListener(std::unique_ptr<GameEventCallback> callback) = 0;
            virtual int32_t UnregisterGameEventListener() = 0;
        };
    }
}
```

### 接口方法说明

| 方法 | 返回类型 | 参数 | 用途 |
|------|----------|------|------|
| `StartGetGpuPerfInfo` | `int32_t` | `duration`, `collectDur`, `callback` | 开始 GPU 性能采集 |
| `StopGetGpuPerfInfo` | `int32_t` | 无 | 停止 GPU 性能采集 |
| `GetSystemFunctionStatus` | `std::map<std::string, std::string>` | `queryParams` | 查询系统功能状态 |
| `RegisterGameEventListener` | `int32_t` | `callback` | 注册游戏事件监听 |
| `UnregisterGameEventListener` | `int32_t` | 无 | 取消游戏事件监听 |

### 使用示例

```cpp
// 获取 GameServicePlugin 实例（通过 samgr 获取）
auto& samgr = OHOS::SamgrProxy::GetInstance();
auto plugin = samgr.GetService<GameServicePlugin>("GameServicePlugin");

if (plugin != nullptr) {
    // 注册 GPU 性能回调
    auto gpuCallback = std::make_unique<GpuCounterCallbackImpl>();
    plugin->StartGetGpuPerfInfo(10800000, 1000, std::move(gpuCallback));
    
    // 注册游戏事件回调
    auto eventCallback = std::make_unique<MyGameEventCallback>();
    plugin->RegisterGameEventListener(std::move(eventCallback));
}
```

## 2. GameEventCallback 接口

**代码位置**: `smartperf_device/device_command/interface/GameEventCallback.h`

### 接口定义

```cpp
namespace OHOS {
    namespace SmartPerf {
        class GameEventCallback {
        public:
            GameEventCallback() = default;
            virtual ~GameEventCallback() = default;
            virtual void OnGameEvent(int32_t type, std::map<std::string, std::string> &params) = 0;
        };
    }
}
```

### 回调方法说明

| 方法 | 返回类型 | 参数 | 用途 |
|------|----------|------|------|
| `OnGameEvent` | `void` | `type`, `params` | 游戏事件回调 |

### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `int32_t` | 事件类型 |
| `params` | `std::map<std::string, std::string>` | 事件参数键值对 |

### 实现示例

```cpp
class MyGameEventCallback : public OHOS::SmartPerf::GameEventCallback {
public:
    void OnGameEvent(int32_t type, std::map<std::string, std::string>& params) override {
        std::cout << "Game event type: " << type << std::endl;
        for (const auto& [key, value] : params) {
            std::cout << "  " << key << ": " << value << std::endl;
        }
    }
};
```

## 3. GpuCounterCallback 接口

**代码位置**: `smartperf_device/device_command/interface/GpuCounterCallback.h`

### 接口定义

```cpp
namespace OHOS {
    namespace SmartPerf {
        struct GpuPerfInfo {
            int64_t remainTime = 0;
            int64_t startTime = 0;
            int32_t duration = 0;
            uint32_t gpuActive = 0;
            uint32_t drawCalls = 0;
            uint64_t primitives = 0;
            uint64_t vertexCounts = 0;
            uint64_t totalInstruments = 0;
            uint32_t gpuLoadPercentage = 0;
            uint32_t vertexLoadPercentage = 0;
            uint32_t fragmentLoadPercentage = 0;
            uint32_t computeLoadPercentage = 0;
            uint32_t textureLoadPercentage = 0;
            uint32_t memoryReadBandwidth = 0;
            uint32_t memoryWriteBandwidth = 0;
            uint32_t memoryBandwidthPercentage = 0;
        };

        class GpuCounterCallback {
        public:
            GpuCounterCallback() = default;
            virtual ~GpuCounterCallback() = default;
            virtual int OnGpuData(std::vector<GpuPerfInfo> &gpuPerfInfos) = 0;
        };
    }
}
```

### GpuPerfInfo 字段说明

| 字段 | 类型 | 单位 | 说明 |
|------|------|------|------|
| `remainTime` | `int64_t` | ms | 剩余时间 |
| `startTime` | `int64_t` | ms | 开始时间 |
| `duration` | `int32_t` | ms | 采集持续时间 |
| `gpuActive` | `uint32_t` | % | GPU 活跃度 |
| `drawCalls` | `uint32_t` | count | 绘制调用次数 |
| `primitives` | `uint64_t` | count | 图元数量 |
| `vertexCounts` | `uint64_t` | count | 顶点数量 |
| `totalInstruments` | `uint64_t` | count | 仪器总数 |
| `gpuLoadPercentage` | `uint32_t` | % | GPU 负载百分比 |
| `vertexLoadPercentage` | `uint32_t` | % | 顶点负载百分比 |
| `fragmentLoadPercentage` | `uint32_t` | % | 片元负载百分比 |
| `computeLoadPercentage` | `uint32_t` | % | 计算负载百分比 |
| `textureLoadPercentage` | `uint32_t` | % | 纹理负载百分比 |
| `memoryReadBandwidth` | `uint32_t` | MB/s | 内存读取带宽 |
| `memoryWriteBandwidth` | `uint32_t` | MB/s | 内存写入带宽 |
| `memoryBandwidthPercentage` | `uint32_t` | % | 内存带宽百分比 |

### 回调方法说明

| 方法 | 返回类型 | 参数 | 用途 |
|------|----------|------|------|
| `OnGpuData` | `int` | `gpuPerfInfos` | GPU 数据回调 |

### 实现示例

```cpp
class GpuCounterCallbackImpl : public OHOS::SmartPerf::GpuCounterCallback {
public:
    int OnGpuData(std::vector<OHOS::SmartPerf::GpuPerfInfo>& gpuPerfInfos) override {
        for (const auto& info : gpuPerfInfos) {
            std::cout << "GPU Load: " << info.gpuLoadPercentage << "%" << std::endl;
            std::cout << "Draw Calls: " << info.drawCalls << std::endl;
            std::cout << "Memory Read: " << info.memoryReadBandwidth << " MB/s" << std::endl;
        }
        return 0;
    }
};
```

## SP_daemon 其他内部 API

### 采集器基类

**代码位置**: `smartperf_device/device_command/include/sp_profiler.h`

```cpp
namespace OHOS {
    namespace SmartPerf {
        class SpProfiler {
        public:
            virtual ~SpProfiler() = default;
            virtual bool Start() = 0;
            virtual bool Stop() = 0;
            virtual std::map<std::string, std::string> GetOne() = 0;
            virtual std::string GetType() = 0;
        };
    }
}
```

### 采集器列表

| 采集器类 | 头文件 | 职责 |
|----------|--------|------|
| `CPU` | `collector/include/CPU.h` | CPU 使用率、频率 |
| `GPU` | `collector/include/GPU.h` | GPU 负载 |
| `FPS` | `collector/include/FPS.h` | 帧率监控 |
| `RAM` | `collector/include/RAM.h` | 内存使用 |
| `Power` | `collector/include/Power.h` | 功耗数据 |
| `Temperature` | `collector/include/Temperature.h` | 温度传感器 |
| `DDR` | `collector/include/DDR.h` | 内存带宽 |
| `Network` | `collector/include/Network.h` | 网络流量 |
| `ByTrace` | `collector/include/ByTrace.h` | ftrace 录制 |
| `Hiperf` | `collector/include/hiperf.h` | 性能采样 |
| `GpuCounter` | `collector/include/GpuCounter.h` | GPU 计数器 |
| `GameEvent` | `collector/include/GameEvent.h` | 游戏事件 |

## 相关文档

- [项目概览](00_Overview.md)
- [系统架构](01_Architecture.md)
- [WASM 接口](02_WASM_API.md)
- [GN 构建配置](04_GNBuild.md)
