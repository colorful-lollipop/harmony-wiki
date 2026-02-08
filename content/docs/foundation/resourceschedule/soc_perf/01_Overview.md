# 01_Overview - 项目概览

## 一句话定义

SOC 统一调频部件是 OpenHarmony 资源调度子系统的核心组件，负责根据系统事件动态调整 CPU/SOC 频率策略，实现性能与功耗的智能平衡。

## 核心能力

| 能力 | 说明 | 代码证据 |
|------|------|----------|
| **性能提频** | 根据应用场景（滑动、点击等）提升 CPU 频率 | `services/core/src/socperf.cpp:45-89` |
| **功耗限频** | 热管理模块协作，限制频率上限 | `services/core/src/socperf.cpp:112-134` |
| **动态仲裁** | 多事件优先级仲裁，避免频繁调频 | `services/core/src/socperf_thread_wrap.cpp:256-312` |
| **配置驱动** | XML 配置定义所有调频行为 | `profile/socperf_boost_config.xml` |

## 能力边界

### 能做什么

- 根据预定义场景调整 CPU/GPU/DDR/NPU 频率
- 支持固定时长提频和手动开关提频
- 与热管理、电源管理模块协同限频
- 多客户端并发请求仲裁

### 不能做什么

- 不直接控制屏幕亮度（电源管理职责）
- 不管理应用生命周期（能力管理职责）
- 不处理网络通信（网络子系统职责）
- 不支持跨设备分布式调频

## 运行环境

### 系统依赖

| 依赖组件 | 用途 | 证据 |
|----------|------|------|
| `safwk` | SystemAbility 框架 | `bundle.json:34` |
| `samgr` | 系统服务管理器 | `bundle.json:35` |
| `ipc` | IPC 通信机制 | `bundle.json:32` |
| `ffrt` | 异步任务执行 | `services/BUILD.gn` |
| `hilog` | 日志输出 | `services/BUILD.gn` |
| `libxml2` | XML 配置解析 | `services/BUILD.gn` |

### 硬件要求

| 资源 | 最小需求 | 说明 |
|------|----------|------|
| RAM | 10 MB | 服务运行内存 |
| ROM | 2 MB | 代码体积 |
| CPU | ARMv8-A | 32/64位支持 |

### 系统版本

| 版本 | 支持状态 |
|------|----------|
| Standard | ✅ 支持 |
| Mini | ❌ 不支持 |
| Small | ❌ 不支持 |

## 快速开始

### 获取实例

```cpp
#include "socperf_client.h"

using namespace OHOS::SOCPERF;

int main() {
    // 获取单例实例
    SocPerfClient& client = SocPerfClient::GetInstance();
    
    // 发送性能提频请求
    client.PerfRequest(10001, "app_launch");
    
    // 带开关的提频请求
    client.PerfRequestEx(10002, true, "game_mode");
    
    // 功耗限频
    client.PowerLimitBoost(true, "high_temp");
    
    return 0;
}
```

### 编译链接

```gn
deps += [
    "//foundation/resourceschedule/soc_perf/interfaces/inner_api/socperf_client:socperf_client"
]
```

## 架构定位

```
┌─────────────────────────────────────────────────────────────────────┐
│                           应用层                                    │
│        (ReportData → 资源调度框架 → SocPerf 插件)                   │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│                     资源调度框架 (ResSched)                         │
│              事件采集 → 插件分发 → 统一调频                          │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│                        soc_perf (本组件)                           │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│   │  SocPerfClient│→│SocPerfServer │→│   SocPerf    │            │
│   │  (IPC客户端)  │  │  (SA服务端)  │  │  (核心逻辑)  │            │
│   └──────────────┘  └──────────────┘  └──────────────┘           │
└─────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────┐
│                        Linux Kernel                                 │
│                     cpufreq 接口 → 频率调节                         │
└─────────────────────────────────────────────────────────────────────┘
```

## 项目信息

| 属性 | 值 | 证据来源 |
|------|-----|----------|
| **项目名称** | SOC 统一调频部件 | `bundle.json:3` |
| **子系统** | resourceschedule | `bundle.json:14` |
| **SA ID** | 1906 | `sa_profile/1906.json:5` |
| **组件类型** | System Ability | `services/server/include/socperf_server.h:33` |
| **版本** | 3.1 | `bundle.json:4` |
| **许可证** | Apache 2.0 | `bundle.json:6` |

## 相关文档

- 架构设计：[02_Architecture](02_Architecture.md)
- 接口文档：[04_Interface](04_Interface.md)
- 安全分析：[06_SecurityReview](06_SecurityReview.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
