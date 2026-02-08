# 附录 B: 配置参数与宏

## 1. 编译宏

### 1.1 功能开关

| 宏 | 默认值 | 用途 | 文件位置 |
|----|--------|------|----------|
| `HAVE_HILOG` | 开启 | 启用 hilog 日志 | 插件 BUILD.gn |
| `HOOK_ENABLE` | 开启 | 启用 Hook 功能 | native_hook BUILD.gn |
| `NO_PROTOBUF` | 关闭 | 禁用 protobuf | 轻量插件 BUILD.gn |
| `LITE_PROTO` | 关闭 | 使用 lite protobuf | ftrace BUILD.gn |
| `HIDEBUG_IN_INIT` | 关闭 | Init 阶段编译 | init 相关 BUILD.gn |
| `ENABLE_HAP_EXTRACTOR` | 开启 | 启用 HAP 提取 | native_daemon BUILD.gn |
| `PERFORMANCE_DEBUG` | 关闭 | 性能调试 | 测试相关 |
| `HAVE_LIBUNWINDER` | 1 | 使用 libunwinder | base BUILD.gn |

### 1.2 架构相关

| 宏 | 条件 | 说明 |
|----|------|------|
| `target_cpu_${target_cpu}` | 动态 | 目标 CPU 架构 |
| `is_musl` | musl 系统 | musl libc |
| `is_linux` | Linux 系统 | Linux 系统 |
| `arm_arch` | ARM 架构 | ARM 架构版本 |

### 1.3 安全相关

| 宏 | 默认值 | 说明 |
|----|--------|------|
| `OPENSSL_SUPPRESS_DEPRECATED` | 开启 | 抑制 OpenSSL 弃用警告 |

---

## 2. Profiler 配置参数

### 2.1 Session 配置

```protobuf
message SessionConfig {
    // 缓冲区配置
    Buffers buffers = 1;
    
    // 结果文件路径
    string result_file = 2;
    
    // 采样时长 (毫秒)
    uint32 sample_duration = 3;
    
    // 插件配置列表
    repeated PluginConfig plugin_configs = 4;
}

message Buffers {
    // 缓冲区页数 (4KB * pages)
    uint32 pages = 1;
}
```

### 2.2 插件通用配置

```protobuf
message PluginConfig {
    // 插件名称
    string plugin_name = 1;
    
    // 采样间隔 (毫秒)
    uint32 sample_interval = 2;
    
    // 插件特定配置 (protobuf 二进制)
    bytes config_data = 3;
}
```

---

## 3. 插件特定配置

### 3.1 CPU 插件配置

```protobuf
message CpuPluginConfig {
    // 采样间隔 (毫秒)
    uint32 sample_interval = 1;
    
    // 是否采样 CPU 使用率
    bool enable_cpu_usage = 2;
    
    // 是否采样线程统计
    bool enable_thread_stats = 3;
    
    // 是否采样调度信息
    bool enable_scheduler = 4;
}
```

### 3.2 内存插件配置

```protobuf
message MemoryPluginConfig {
    // 采样间隔 (毫秒)
    uint32 sample_interval = 1;
    
    // 是否上报进程树
    bool report_process_tree = 2;
    
    // 是否上报系统内存信息
    bool report_sysmem_mem_info = 3;
    
    // 系统内存计数器列表
    repeated SysMeminfoCounters sys_meminfo_counters = 4;
    
    // 是否上报虚拟内存信息
    bool report_sysmem_vmem_info = 5;
    
    // 虚拟内存计数器列表
    repeated VmeminfoCounters sys_vmeminfo_counters = 6;
    
    // 是否上报进程内存信息
    bool report_process_mem_info = 7;
    
    // 是否上报应用内存信息
    bool report_app_mem_info = 8;
}
```

### 3.3 Ftrace 插件配置

```protobuf
message TracePluginConfig {
    // ftrace 事件列表
    repeated string ftrace_events = 1;
    
    // 缓冲区大小 (KB)
    uint32 buffer_size_kb = 2;
    
    // 刷新间隔 (毫秒)
    uint32 flush_interval_ms = 3;
    
    // 刷新阈值 (KB)
    uint32 flush_threshold_kb = 4;
    
    // 是否解析 ksyms
    bool parse_ksyms = 5;
    
    // 时钟类型
    string clock = 6;  // "mono", "real", "boot", "perf"
    
    // 追踪周期 (毫秒)
    uint32 trace_period_ms = 7;
    
    // 调试模式
    bool debug_on = 8;
    
    // hitrace 时间 (毫秒)
    uint32 hitrace_time = 9;
}
```

### 3.4 Native Hook 配置

```protobuf
message NativeHookConfig {
    // 是否保存文件
    bool save_file = 1;
    
    // 过滤大小 (字节)
    uint32 filter_size = 2;
    
    // 共享内存页数
    uint32 smb_pages = 3;
    
    // 最大栈深度
    uint32 max_stack_depth = 4;
    
    // 目标进程名
    string process_name = 5;
    
    // 目标进程 PID
    uint32 target_pid = 6;
    
    // malloc/free 匹配间隔 (毫秒)
    uint32 malloc_free_matching_interval = 7;
    
    // malloc/free 匹配计数
    uint32 malloc_free_matching_cnt = 8;
    
    // 是否压缩字符串
    bool string_compressed = 9;
    
    // 是否启用帧指针 unwind
    bool fp_unwind = 10;
    
    // 是否 dump nmd stats
    bool dump_nmd = 11;
}
```

---

## 4. 常量定义

### 4.1 插件常量

```c
// 插件名最大长度
#define PLUGIN_MODULE_NAME_MAX 127

// 插件版本最大长度
#define PLUGIN_MODULE_VERSION_MAX 7

// 缓冲区大小
#define MAX_BUFFER_SIZE 4096

// 默认缓冲区页数
#define DEFAULT_BUFFER_PAGES 16384

// App UID 阈值
#define APP_ID_THRESH 20000000
```

### 4.2 内存计数器

```c
// 系统内存计数器
enum SysMeminfoCounters {
    PMEM_ACTIVE = 0,
    PMEM_ACTIVE_ANON,
    PMEM_ACTIVE_FILE,
    PMEM_ANON_PAGES,
    // ... 更多
}

// 虚拟内存计数器
enum VmeminfoCounters {
    VMEMINFO_NR_FREE_PAGES = 0,
    VMEMINFO_NR_ALLOC_BATCH,
    // ... 更多
}
```

---

## 5. Trace Tags

```c
// Trace 标签枚举
enum TraceTag {
    ABILITY_MANAGER = 1,
    ARKUI = 2,
    ARK = 4,
    BLUETOOTH = 8,
    COMMON_LIBRARY = 16,
    // ... 更多
};

// TraceFlag
enum TraceFlag {
    MAIN_THREAD = 1,
    ALL_THREADS = 2,
};
```

---

## 6. 相关跳转

| 主题 | 链接 |
|------|------|
| 插件系统 | [02_Plugin_System.md](./02_Plugin_System.md) |
| N-API 接口 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |
| 故障排查 | [07_Troubleshooting.md](./07_Troubleshooting.md) |

---

*最后更新: 2026-02-06*
