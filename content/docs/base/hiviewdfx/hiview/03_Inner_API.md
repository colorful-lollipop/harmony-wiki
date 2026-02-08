# 内部 API

## 概述

本文档描述 Hiview 模块的内部 C++ API，供模块内部和子系统内部使用。

---

## 插件框架 API

### Plugin 基类

**头文件**: `base/include/plugin.h`

```cpp
class Plugin {
public:
    virtual ~Plugin() = default;
    
    // 插件初始化
    virtual bool Init(const PluginConfig& config, PluginBundle& pluginBundle);
    
    // 插件销毁
    virtual void Destroy();
    
    // 获取插件名称
    virtual std::string GetName() const = 0;
    
    // 获取插件优先级
    virtual int GetPriority() const;
    
    // 处理事件
    virtual void OnEvent(const Event& event);
    
    // 插件加载完成
    virtual void OnLoad();
    
    // 插件卸载
    virtual void OnUnload();
};
```

### Event 基类

**头文件**: `base/include/event.h`

```cpp
class Event {
public:
    // 事件类型
    enum class EventType {
        MESSAGE,
        SYSEVENT,
        FAULT,
        STATS,
        // ...
    };
    
    // 获取事件类型
    EventType GetEventType() const;
    
    // 获取事件名称
    std::string GetName() const;
    
    // 获取事件源
    std::string GetSource() const;
    
    // 获取时间戳
    uint64_t GetCreateTime() const;
    
    // 设置事件参数
    void SetParam(const std::string& key, const Variant& value);
    
    // 获取事件参数
    Variant GetParam(const std::string& key) const;
};
```

### EventLoop

**头文件**: `base/include/event_loop.h`

```cpp
class EventLoop {
public:
    // 添加事件到队列
    bool PostEvent(const Event& event);
    
    // 添加延迟事件
    bool PostEvent(const Event& event, uint64_t delayMs);
    
    // 停止事件循环
    void Stop();
    
    // 运行事件循环
    void Run();
};
```

### PluginFactory

**头文件**: `base/include/plugin_factory.h`

```cpp
class PluginFactory {
public:
    // 注册插件创建函数
    bool RegisterPlugin(const std::string& name, CreateFunc createFunc);
    
    // 创建插件实例
    std::unique_ptr<Plugin> Create(const std::string& name);
    
    // 获取所有已注册的插件名
    std::vector<std::string> GetRegisteredPlugins() const;
};
```

---

## 故障日志 API

### FaultLogger Client

**头文件**: `plugins/faultlogger/interfaces/cpp/innerkits/include/faultlogger_client.h`

```cpp
class FaultLoggerClient {
public:
    // 获取单例
    static FaultLoggerClient& GetInstance();
    
    // 添加故障日志
    int AddFaultLog(const FaultLogInfo& info);
    
    // 查询自身故障日志
    int QuerySelfFaultLog(FaultType type, std::vector<FaultLogInfo>& result);
    
    // 查询所有故障日志
    int QueryFaultLog(FaultType type, std::vector<FaultLogInfo>& result);
};
```

### FaultLogInfo

**头文件**: `plugins/faultlogger/interfaces/cpp/innerkits/include/faultlog_info.h`

```cpp
struct FaultLogInfo {
    pid_t pid;              // 进程 ID
    uid_t uid;              // 用户 ID
    int type;               // 故障类型
    uint64_t timestamp;     // 时间戳
    std::string reason;     // 故障原因
    std::string module;     // 模块名
    std::string summary;    // 摘要
    std::string fullLog;    // 完整日志
};
```

---

## 统一采集器 API

### CPU Collector Client

**头文件**: `interfaces/inner_api/unified_collection/client/cpu_collector_client.h`

```cpp
class CpuCollectorClient {
public:
    // 开始采集
    int Start();
    
    // 停止采集
    int Stop();
    
    // 获取采集结果
    int GetResult(std::vector<CpuUsageInfo>& result);
};
```

### Trace Collector Client

**头文件**: `interfaces/inner_api/unified_collection/client/trace_collector_client.h`

```cpp
class TraceCollectorClient {
public:
    // 开始 Trace 采集
    int StartTrace(const TraceConfig& config);
    
    // 停止 Trace 采集
    int StopTrace();
    
    // 获取 Trace 数据
    int GetTraceData(std::string& traceData);
};
```

### 统一采集器接口

**头文件**: `interfaces/inner_api/unified_collection/utility/cpu_collector.h`

```cpp
class CpuCollector {
public:
    virtual ~CpuCollector() = default;
    
    // 初始化采集器
    virtual int Init();
    
    // 开始采集
    virtual int Start();
    
    // 停止采集
    virtual int Stop();
    
    // 获取采集配置
    virtual const CollectConfig& GetConfig() const;
};
```

---

## 功耗事件 API

### XPowerEvent

**头文件**: `interfaces/inner_api/xpower_event/include/xpower_event.h`

```cpp
class XPowerEvent {
public:
    // 事件类型
    enum class EventType {
        SCREEN,
        CPU,
        THERMAL,
        // ...
    };
    
    // 上报事件
    int Report(EventType type, const std::string& content);
    
    // 上报原始 HiSysEvent
    int ReportRaw(const HiSysEvent& event);
};
```

### XPowerEvent JS

**头文件**: `interfaces/inner_api/xpower_event/include/xpower_event_js.h`

```cpp
class XPowerEventJs {
public:
    // JS 接口：上报功耗事件
    static napi_value Report(napi_env env, napi_callback_info info);
};
```

---

## Hiview 服务 API

### HiviewServiceAbility

**头文件**: `adapter/service/server/include/hiview_service_ability.h`

```cpp
class HiviewServiceAbility : public SystemAbility {
public:
    // 构造函数，指定 SA ID
    HiviewServiceAbility();
    
    // 列出日志文件
    int ListFiles(const std::string& logType, std::vector<FileInfo>& files);
    
    // 复制日志文件
    int CopyFile(const std::string& logType, const std::string& fileName,
                 const std::string& destDir);
    
    // 移动日志文件
    int MoveFile(const std::string& logType, const std::string& fileName,
                 const std::string& destDir);
    
    // 删除日志文件
    int RemoveFile(const std::string& logType, const std::string& fileName);
    
    // 采集 Trace
    int DumpSnapshotTrace(int client, int errNo, std::vector<uint8_t>& result);
};
```

---

## 隐私控制 API

### PrivacyManager

**头文件**: `base/include/privacy_manager.h`

```cpp
class PrivacyManager {
public:
    // 检查事件是否允许
    bool IsAllowed(const Event& event);
    
    // 检查隐私级别
    bool IsPrivacyAllowed(int privacyLevel);
    
    // 检查 Bundle 是否在白名单
    bool IsBundleNameInList(const std::string& bundleName);
};
```

### IPrivacyController

**头文件**: `base/include/i_privacy_controller.h`

```cpp
class IPrivacyController {
public:
    // 检查隐私控制
    virtual bool Check(const Event& event) = 0;
    
    // 获取隐私级别
    virtual int GetPrivacyLevel(const std::string& bundleName) = 0;
};
```

---

## 依赖方向

```
                    ┌─────────────────┐
                    │   applications  │ ←─── 调用者
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   N-API Layer   │ ←─── 稳定接口
                    └────────┬────────┘
                             │
┌────────────────────────────┼────────────────────────────┐
│                            │                            │
│            ┌──────────────▼──────────────┐             │
│            │      Plugin Framework       │             │
│            │  (base/include/plugin.h)    │             │
│            └──────────────┬──────────────┘             │
│                           │                            │
│     ┌─────────────────────┼─────────────────────┐      │
│     │                     │                     │      │
│ ┌───▼───┐           ┌─────▼─────┐       ┌─────▼───┐  │
│ │Fault- │           │ Unifed    │       │  Other  │  │
│ │Logger │           │ Collector │       │ Plugins │  │
│ └───┬───┘           └─────┬─────┘       └─────┬───┘  │
│     │                     │                   │      │
│     └─────────────────────┼───────────────────┘      │
│                           │                          │
│            ┌──────────────▼──────────────┐           │
│            │     Adapter Layer          │           │
│            │  (adapter/service/server)   │           │
│            └──────────────┬──────────────┘           │
└───────────────────────────┼─────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
     ┌────────▼────────┐        ┌────────▼────────┐
     │   IPC/SAMgr    │        │   OS Services   │
     │   (系统底层)    │        │   (系统底层)    │
     └─────────────────┘        └─────────────────┘
```

---

## 接口稳定性标注

| 接口 | 路径 | 稳定性 | 说明 |
|------|------|--------|------|
| `faultlogger_client.h` | `plugins/faultlogger/interfaces/cpp/innerkits/` | 稳定 | 内部接口，供子系统使用 |
| `xpower_event.h` | `interfaces/inner_api/xpower_event/` | 稳定 | 内部接口 |
| `cpu_collector_client.h` | `interfaces/inner_api/unified_collection/client/` | 稳定 | 内部接口 |
| `plugin.h` | `base/include/` | 稳定 | 框架接口 |
| `event.h` | `base/include/` | 稳定 | 框架接口 |
| `hiview_service_ability.h` | `adapter/service/server/include/` | 稳定 | SA 接口 |
