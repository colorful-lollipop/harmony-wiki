# 04_Inner_API - 内部 API 参考

## 1. 模块概览

### 1.1 内部 API 目录结构

```
hidebug/interfaces/
├── js/kits/napi/           # N-API 实现 (对外)
├── ets/ani/               # ETS/ANI 接口 (对外)
├── cj/                    # CJ 接口 (对外)
└── native/kits/          # Native 接口 (对外)
    └── include/
        └── hidebug_base.h

device/plugins/api/
└── include/
    ├── hidebug_base.h     # 基础 API
    ├── writer.h           # 数据写入
    └── manager_interface.h # 插件管理
```

---

## 2. 核心内部 API

### 2.1 hidebug_base.h

**位置**: `device/plugins/api/include/hidebug_base.h`

```cpp
// 基础内存查询
uint64_t GetPss();
uint64_t GetSharedDirty();
uint64_t GetPrivateDirty();
uint64_t GetVss();
uint64_t GetNativeHeapSize();
uint64_t GetNativeHeapAllocatedSize();
uint64_t GetNativeHeapFreeSize();

// CPU 使用率
double GetCpuUsage();

// 线程 CPU 信息
int GetThreadCpuUsage(ThreadCpuInfo* infos, int maxCount);

// 系统内存信息
int GetSystemMemInfo(SystemMemInfo* memInfo);

// 应用内存限制
int GetAppMemoryLimit(AppMemLimit* limit);

// 调试状态
bool IsDebugState();

// VM 运行时统计
int GetVMRuntimeStats(VMRuntimeStats* stats);
int GetVMRuntimeStat(const char* property, uint64_t* value);
```

### 2.2 writer.h

**位置**: `device/plugins/api/include/writer.h`

```cpp
// 写入器接口
class BufferWriter {
public:
    // 构造函数
    BufferWriter(uint8_t* buffer, uint32_t size);
    
    // 写入带时间戳的数据
    int64_t Write(const uint8_t* data, size_t size);
    
    // 序列化 protobuf
    int Serialize(const google::protobuf::MessageLite& msg);
    
    // 刷新缓冲区
    void Flush();
    
    // 获取已写入大小
    uint32_t GetWrittenSize() const;
    
    // 检查空间
    bool HasSpace(size_t size) const;
};
```

### 2.3 manager_interface.h

**位置**: `device/plugins/api/include/manager_interface.h`

```cpp
// 插件管理器接口
class PluginManager {
public:
    // 单例获取
    static PluginManager& GetInstance();
    
    // 加载插件
    int LoadPlugin(const char* path);
    
    // 卸载插件
    int UnloadPlugin(const char* name);
    
    // 创建会话
    int CreateSession(const SessionConfig& config);
    
    // 启动会话
    int StartSession(uint32_t sessionId);
    
    // 停止会话
    int StopSession(uint32_t sessionId);
    
    // 获取数据
    int GetData(uint32_t sessionId, uint8_t* buffer, uint32_t* size);
};
```

---

## 3. 稳定性标注

### 3.1 接口稳定性等级

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| **N-API 接口** | 稳定 (Stablebug/interfaces/js) | `hide/kits/napi/` |
| **ETS/ANI 接口** | 稳定 (Stable) | `hidebug/interfaces/ets/ani/` |
| **CJ 接口** | 稳定 (Stable) | `hidebug/interfaces/cj/` |
| **Native 接口** | 平台接口 (Platform) | `hidebug/interfaces/native/kits/` |
| **插件 API** | 不稳定 (Unstable) | `device/plugins/api/` |

### 3.2 稳定性说明

| 等级 | 说明 | 使用限制 |
|------|------|----------|
| **Stable** | 官方支持，长期维护 | 可直接使用 |
| **Platform** | 平台接口，可能变更 | 仅系统组件使用 |
| **Unstable** | 实验性，随时变更 | 不推荐外部使用 |

---

## 4. 依赖方向

### 4.1 模块依赖图

```
┌─────────────────────────────────────────────────────────────────────┐
│                       模块依赖关系                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   host/smartperf/                                                  │
│        │                                                           │
│        │ depends                                                   │
│        ▼                                                           │
│   hidebug (N-API) ◄───────────────────────────────────────────┐   │
│        │                                                          │   │
│        │ depends                                            │   │
│        ▼                                                          │   │
│   hidebug_native.so ◄────────────────────────────────────────┐ │   │
│        │                                                      │ │   │
│        │ depends                                              │ │   │
│        ▼                                                      │ │   │
│   ability_runtime, ffrt, hitrace                            │ │   │
│                                                              │ │   │
│   device/plugins/api/ (插件框架) ────────────────────────────┤ │   │
│        │                                                      │ │   │
│        │ depends                                              │ │   │
│        ▼                                                      │ │   │
│   profiler_service ──► shared_memory ──► proto_encoder ─────┘ │   │
│        │                                                           │
│        │ depends                                                   │
│        ▼                                                           │
│   grpc, protobuf, openssl, abseil-cpp                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 循环依赖检查

```
✅ 无循环依赖
- N-API → Native 接口 → 系统组件
- 插件框架 → 服务 → 共享内存 → 编码器
```

---

## 5. 头文件包含层级

### 5.1 公开头文件 (可被外部包含)

| 头文件 | 用途 | 包含规则 |
|--------|------|----------|
| `hidebug/interfaces/js/kits/napi/*.h` | N-API 声明 | JS 应用可包含 |
| `hidebug/interfaces/native/kits/*.h` | Native 声明 | Native 应用可包含 |
| `device/plugins/api/include/*.h` | 插件 API | 插件可包含 |

### 5.2 内部头文件 (仅框架使用)

| 头文件 | 用途 |
|--------|------|
| `device/base/include/*.h` | 基础框架 |
| `device/services/*/include/*.h` | 服务内部 |
| `device/plugins/*/src/*.h` | 插件内部 |

---

## 6. 常见内部类型

### 6.1 内存信息类型

```cpp
// 进程内存信息
struct ProcessMemInfo {
    uint64_t rss;           // 常驻集大小
    uint64_t vss;           // 虚拟内存大小
    uint64_t pss;           // 比例集大小
    uint64_t sharedClean;   // 共享干净页
    uint64_t sharedDirty;   // 共享脏页
    uint64_t privateClean;  // 私有干净页
    uint64_t privateDirty;  // 私有脏页
};

// 系统内存信息
struct SystemMemInfo {
    uint64_t totalMem;      // 总内存
    uint64_t freeMem;       // 空闲内存
    uint64_t availableMem;  // 可用内存
};
```

### 6.2 配置类型

```cpp
// Session 配置
struct SessionConfig {
    uint32_t sessionId;
    uint32_t bufferPages;     // 缓冲区页数
    std::string resultFile;    // 结果文件路径
    uint32_t sampleDuration;   // 采样时长
    std::vector<PluginConfig> plugins;
};

// 插件配置
struct PluginConfig {
    std::string pluginName;
    uint32_t sampleInterval;  // 采样间隔
    std::vector<uint8_t> configData;  // Protobuf 配置
};
```

---

## 7. 相关跳转

| 主题 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| N-API 接口 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) |
| 插件系统 | [02_Plugin_System.md](./02_Plugin_System.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |

---

*最后更新: 2026-02-06*
