# 编译产物

> 电池统计模块编译后生成的产物清单、安装路径及运行时加载关系。

## 目录

- [产物清单](#产物清单)
- [安装路径](#安装路径)
- [运行时加载关系](#运行时加载关系)
- [产物与源码映射](#产物与源码映射)

## 产物清单

### 动态库 (.z.so)

| 产物 | 类型 | 目标 | 说明 |
|------|------|------|------|
| `libbatterystatistics.z.so` | ohos_shared_library | frameworks/napi:BUILD.gn | N-API 模块，供 JS 调用 |
| `libbatterystats_client.z.so` | ohos_shared_library | interfaces/inner_api:BUILD.gn | Inner API 客户端库 |
| `libbatterystats_service.z.so` | ohos_shared_library | services:BUILD.gn | SA 服务主库 |

### 静态库 (.a)

| 产物 | 类型 | 目标 | 说明 |
|------|------|------|------|
| `libbatterystats_utils.a` | ohos_static_library | utils:BUILD.gn | 工具库 (stats_helper, stats_utils 等) |

### SA 配置

| 产物 | 路径 | 说明 |
|------|------|------|
| `3304.json` | sa_profile/ | SA 3304 配置文件 |

### IDL 生成代码

| 产物 | 类型 | 说明 |
|------|------|------|
| `*_proxy.cpp` | source_set | IPC 代理端代码 |
| `*_stub.cpp` | source_set | IPC 服务端代码 |
| `ibattery_stats.h` | idl_gen | IPC 接口头文件 |

**证据**: `services/BUILD.gn:37-59` 和 `services/BUILD.gn:61-95`

## 安装路径

### 默认安装路径

| 产物 | 预期路径 | 实际路径可能为 |
|------|----------|---------------|
| `libbatterystatistics.z.so` | `/system/module/libbatterystatistics.z.so` | `$OUT/lib/module/` |
| `libbatterystats_client.z.so` | `/system/lib/libbatterystats_client.z.so` | `$OUT/lib/` |
| `libbatterystats_service.z.so` | `/system/lib/libbatterystats_service.z.so` | `$OUT/lib/` |
| `3304.json` | `/system/profile/3304.json` | `$OUT/etc/sa/` |

### relative_install_dir 配置

**证据**: `frameworks/napi/BUILD.gn:53`

```gni
relative_install_dir = "module"
```

此配置使 `libbatterystatistics.z.so` 安装到 `module/` 子目录。

## 运行时加载关系

### JS 调用链

```mermaid
graph TD
    subgraph JS Runtime
        JS[JavaScript/ArkTS]
    end

    subgraph N-API Module
        NATIVE["libbatterystatistics.z.so<br/>(battery_stats_module.cpp)"]
    end

    subgraph Native Client
        CLIENT["libbatterystats_client.z.so<br/>(BatteryStatsClient)"]
    end

    subgraph IPC Layer
        PROXY["IPC Proxy<br/>(batterystats_proxy)"]
    end

    subgraph SA Process
        STUB["IPC Stub<br/>(batterystats_stub)"]
        SERVICE["libbatterystats_service.z.so<br/>(BatteryStatsService)"]
    end

    subgraph Utils
        UTILS["libbatterystats_utils.a<br/>(static link)"]
    end

    JS -->|import @ohos/battery_statistics| NATIVE
    NATIVE -->|depends| CLIENT
    CLIENT -->|IPC (Binder)| PROXY
    PROXY -->|IPC (Binder)| STUB
    STUB -->|calls| SERVICE
    SERVICE -->|links| UTILS
```

### 加载时机

| 产物 | 加载时机 | 加载方 |
|------|----------|--------|
| `libbatterystatistics.z.so` | 应用首次 `import @ohos/battery_statistics` | JS 运行时 (ACE) |
| `libbatterystats_client.z.so` | 首次调用 BatteryStatsClient | N-API 模块 |
| `libbatterystats_service.z.so` | SA 3304 启动时 | System Ability Framework |
| `libbatterystats_utils.a` | 链接时静态嵌入 | 链接器 |

### SA 生命周期

**证据**: `sa_profile/3304.json`

```json
{
    "process": "powermgr",
    "systemability": [
        {
            "name": 3304,
            "libpath": "libbatterystats_service.z.so",
            "run-on-create": false,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `run-on-create` | false | 按需启动，非系统启动时加载 |
| `distributed` | false | 非分布式 SA，仅本地使用 |
| `dump_level` | 1 | 调试 dump 级别 |

## 产物与源码映射

### libbatterystatistics.z.so

| 源文件 | 路径 | 职责 |
|--------|------|------|
| `battery_stats_module.cpp` | frameworks/napi/src/ | N-API 模块注册 |
| `battery_stats.cpp` | frameworks/napi/src/ | JS API 实现 |
| `async_callback_info.cpp` | frameworks/napi/src/ | 异步回调封装 |
| `napi_error.cpp` | frameworks/napi/src/ | 错误转换 |
| `napi_utils.cpp` | frameworks/napi/src/ | 参数校验 |

### libbatterystats_client.z.so

| 源文件 | 路径 | 职责 |
|--------|------|------|
| `battery_stats_client.cpp` | frameworks/native/src/ | IPC 客户端 |
| `battery_stats_info.cpp` | frameworks/native/src/ | 数据序列化 |

### libbatterystats_service.z.so

| 源文件 | 路径 | 职责 |
|--------|------|------|
| `battery_stats_service.cpp` | services/native/src/ | SA 主类 |
| `battery_stats_core.cpp` | services/native/src/ | 核心逻辑 |
| `battery_stats_detector.cpp` | services/native/src/ | 事件检测 |
| `battery_stats_parser.cpp` | services/native/src/ | 配置解析 |
| `battery_stats_listener.cpp` | services/native/src/ | HiSysEvent 监听 |
| `battery_stats_dumper.cpp` | services/native/src/ | 调试 dump |
| `battery_stats_subscriber.cpp` | services/native/src/ | 公共事件订阅 |
| `cpu_time_reader.cpp` | services/native/src/ | CPU 时间读取 |
| `*_entity.cpp` | services/native/src/entities/ | 各实体实现 |

### libbatterystats_utils.a

| 源文件 | 路径 | 职责 |
|--------|------|------|
| `stats_helper.cpp` | utils/native/src/ | 计时器/计数器 |
| `stats_utils.cpp` | utils/native/src/ | 工具函数 |
| `stats_hisysevent.cpp` | utils/native/src/ | HiSysEvent 封装 |
| `stats_xcollie.cpp` | utils/native/src/ | 看门狗 |

## 构建验证

### 构建命令

```bash
# 构建整个 battery_statistics part
hb build -p battery_statistics

# 仅构建 N-API 模块
gn build out/rk3568//base/powermgr/battery_statistics/frameworks/napi:batterystatistics

# 仅构建服务模块
gn build out/rk3568//base/powermgr/battery_statistics/services:batterystats_service
```

### 产物验证

```bash
# 检查产物是否存在
ls -la out/rk3568/lib/module/libbatterystatistics.z.so
ls -la out/rk3568/lib/libbatterystats_client.z.so
ls -la out/rk3568/lib/libbatterystats_service.z.so

# 检查产物链接
nm -D out/rk3568/lib/module/libbatterystatistics.z.so | grep -i "GetBatteryStats\|GetAppStats"
objdump -p out/rk3568/lib/module/libbatterystatistics.z.so | grep NEEDED
```

## 相关文档

- [GN 构建](./05_GN_Build.md)
- [N-API 参考](./03_NAPI.md)
- [架构说明](./02_Architecture.md)
- [SUMMARY](./SUMMARY.md)
