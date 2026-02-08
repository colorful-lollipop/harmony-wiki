# 编译产物

## 目的与适用范围

本文档详细描述分布式屏幕的编译产物清单、安装路径和运行时加载关系。

---

## 产物清单

### 共享库 (.so)

| 产物名 | 对应Target | 类型 | 说明 |
|--------|-----------|------|------|
| `libdistributed_screen_utils.z.so` | `distributed_screen_utils` | 基础库 | 公共工具函数 |
| `libdistributed_screen_source_sdk.z.so` | `distributed_screen_source_sdk` | SDK | Source端Native接口 |
| `libdistributed_screen_sink_sdk.z.so` | `distributed_screen_sink_sdk` | SDK | Sink端Native接口 |
| `libdistributed_screen_source.z.so` | `distributed_screen_source` | SA服务 | Source端System Ability |
| `libdistributed_screen_sink.z.so` | `distributed_screen_sink` | SA服务 | Sink端System Ability |
| `libdistributed_screen_sourcetrans.z.so` | `distributed_screen_sourcetrans` | 内部库 | Source传输组件 |
| `libdistributed_screen_sinktrans.z.so` | `distributed_screen_sinktrans` | 内部库 | Sink传输组件 |
| `libdistributed_screen_client.z.so` | `distributed_screen_client` | 内部库 | 屏幕客户端 |
| `libdistributed_screen_handler.z.so` | `distributed_screen_handler` | 内部库 | 硬件处理器 |

### 配置文件

| 产物名 | 对应Target | 安装路径 | 说明 |
|--------|-----------|----------|------|
| `4807.json` | `dscreen_sa_profile` | SA配置目录 | Source端SA配置 |
| `4808.json` | `dscreen_sa_profile` | SA配置目录 | Sink端SA配置 |
| `dscreen.cfg` | `dscreen.cfg` | `/etc/init/` | 进程启动配置 |

---

## 安装路径

### 共享库安装路径

```
/system/lib/
├── libdistributed_screen_utils.z.so
├── libdistributed_screen_source_sdk.z.so
├── libdistributed_screen_sink_sdk.z.so
├── libdistributed_screen_source.z.so
├── libdistributed_screen_sink.z.so
├── libdistributed_screen_sourcetrans.z.so
├── libdistributed_screen_sinktrans.z.so
├── libdistributed_screen_client.z.so
└── libdistributed_screen_handler.z.so
```

**证据**: 标准OpenHarmony共享库安装路径

### SA配置文件安装路径

```
/system/profile/
├── 4807.json
└── 4808.json
```

### Init配置文件安装路径

```
/etc/init/
└── dscreen.cfg
```

**证据**: `sa_profile/BUILD.gn:25-30`

```gn
ohos_prebuilt_etc("dscreen.cfg") {
  relative_install_dir = "init"      # 安装到 /etc/init/
  source = "dscreen.cfg"
  part_name = "distributed_screen"
  subsystem_name = "distributedhardware"
}
```

---

## 运行时加载关系

### SA服务加载流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SA服务运行时加载关系                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. 系统启动                                                                 │
│      │                                                                       │
│      ▼                                                                       │
│   2. Init进程读取 /etc/init/dscreen.cfg                                      │
│      │                                                                       │
│      ▼                                                                       │
│   3. 启动 dscreen 进程                                                       │
│      │                                                                       │
│      ├──────────────────────────────────────────────────────────────┐       │
│      │                                                              │       │
│      ▼                                                              ▼       │
│   4. SAMGR加载SA 4807                                         SAMGR加载SA 4808│
│      │                                                              │       │
│      ▼                                                              ▼       │
│   5. 加载 libdistributed_screen_source.z.so                 加载 libdistributed_screen_sink.z.so│
│      │                                                              │       │
│      ▼                                                              ▼       │
│   6. 执行 DScreenSourceService::OnStart()                   执行 DScreenSinkService::OnStart()│
│      │                                                              │       │
│      ▼                                                              ▼       │
│   7. 依赖加载:                                                     依赖加载: │
│      - libdistributed_screen_sourcetrans.z.so                      - libdistributed_screen_sinktrans.z.so│
│      - libdistributed_screen_utils.z.so                            - libdistributed_screen_client.z.so│
│      - ...                                                         - libdistributed_screen_utils.z.so│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### SDK库加载流程

```
应用进程
    │
    │ dlopen / 动态链接
    ▼
┌─────────────────────────────────────────────────────────────┐
│                SDK库加载 (根据角色选择)                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   主控端应用                    被控端应用                    │
│        │                             │                      │
│        ▼                             ▼                      │
│   ┌─────────────┐              ┌─────────────┐             │
│   │ source_sdk  │              │ sink_sdk    │             │
│   └──────┬──────┘              └──────┬──────┘             │
│          │                           │                     │
│          ▼                           ▼                     │
│   ┌─────────────┐              ┌─────────────┐             │
│   │ utils       │              │ utils       │             │
│   └─────────────┘              └─────────────┘             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 依赖关系详解

### Source服务端依赖

**产物**: `libdistributed_screen_source.z.so`

**直接依赖**:
```
libdistributed_screen_source.z.so
├── libdistributed_screen_utils.z.so (deps)
├── libdistributed_screen_sourcetrans.z.so (deps)
├── libaccesstoken_sdk.z.so (external_deps)
├── libtokenid_sdk.z.so (external_deps)
├── libipc_core.z.so (external_deps)
├── libhilog.z.so (external_deps)
├── libsafwk.z.so (external_deps)
└── ... (29个外部依赖)
```

**证据**: `services/screenservice/sourceservice/BUILD.gn:67-109`

### Sink服务端依赖

**产物**: `libdistributed_screen_sink.z.so`

**直接依赖**:
```
libdistributed_screen_sink.z.so
├── libdistributed_screen_utils.z.so (deps)
├── libdistributed_screen_client.z.so (deps)
├── libdistributed_screen_sinktrans.z.so (deps)
├── libaccesstoken_sdk.z.so (external_deps)
├── libipc_core.z.so (external_deps)
└── ... (21个外部依赖)
```

**证据**: `services/screenservice/sinkservice/BUILD.gn:62-99`

---

## 运行时数据目录

### Dump数据目录 (root变体)

**路径**: `/data/data/dscreen/`

**用途**: 
- 存储屏幕数据dump文件（调试用）
- 仅在root版本中启用（`DUMP_DSCREEN_FILE`宏）

**证据**: `common/include/dscreen_constants.h:94`

```cpp
const std::string DUMP_FILE_PATH = "/data/data/dscreen";
```

### Dump文件大小限制

**证据**: `common/include/dscreen_constants.h:150`

```cpp
constexpr uint32_t DUMP_FILE_MAX_SIZE = 295 * 1024 * 1024;  // 295MB
```

---

## 进程模型

### dscreen进程

**进程名**: `dscreen`

**证据**: `sa_profile/4807.json:2`, `common/include/dscreen_constants.h:164`

```cpp
const std::string DSCREEN_PROCESS_NAME = "dscreen";
```

**包含的服务**:
- Source端服务 (SA 4807)
- Sink端服务 (SA 4808)

**说明**: 两个SA共享同一个进程，但拥有独立的线程。

### 进程启动配置 (dscreen.cfg)

```ini
# 示例配置（实际以代码为准）
service dscreen /system/bin/sa_main /system/profile/4807.json
    class distributedhardware
    user system
    group system
    seclabel u:r:dscreen:s0
```

---

## 头文件安装

### SDK头文件

根据 `bundle.json:74-89`，SDK头文件安装配置：

```json
{
  "inner_kits": [
    {
      "type": "so",
      "name": "//.../screen_sink:distributed_screen_sink_sdk",
      "header": {
        "header_base": "//.../screen_sink/include",
        "header_files": [ "idscreen_sink.h" ]
      }
    },
    {
      "type": "so",
      "name": "//.../screen_source:distributed_screen_source_sdk",
      "header": {
        "header_base": "//.../screen_source/include",
        "header_files": [ "idscreen_source.h" ]
      }
    }
  ]
}
```

**安装路径**:
```
/usr/include/
├── idscreen_source.h
└── idscreen_sink.h
```

---

## 产物大小估计

基于bundle.json中的配置:

| 指标 | 值 |
|------|-----|
| ROM占用 | 5120 KB (5MB) |
| RAM占用 | 33580 KB (33MB) |

**证据**: `bundle.json:22-23`

```json
"rom": "5120KB",
"ram": "33580KB",
```

---

## 相关跳转

- [GN构建系统](05_Build_System.md) - 构建配置详解
- [项目概览](00_Overview.md) - 项目基本信息
- [安全风险](07_Security.md) - 安全分析