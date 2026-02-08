# 03_构建系统

> hievent_lite GN 构建配置、Targets 清单与编译产物说明

## 1. 构建配置概览

### 1.1 构建系统

| 项目 | 说明 |
|------|------|
| **构建工具** | GN (Generate Ninja) + Ninja |
| **构建入口** | `BUILD.gn` |
| **配置文件** | `bundle.json` (组件描述) |

### 1.2 适配系统

| 系统类型 | 配置值 | 说明 |
|----------|--------|------|
| **mini** | `"mini"` | 轻量级设备 (LiteOS-M) |

**证据**: `bundle.json:17`

---

## 2. Targets 清单

### 2.1 根目录 BUILD.gn

**文件位置**: `BUILD.gn`

```
Target: hievent_lite_static
├── 类型: static_library
├── 输出: libhievent_lite_static.a
├── Sources:
│   ├── frameworks/hiview_event.c
│   └── frameworks/hiview_output_event.c
├── Defines:
│   ├── FAULT_EVENT_FILE_SIZE=1024
│   ├── UE_EVENT_FILE_SIZE=1024
│   ├── STAT_EVENT_FILE_SIZE=1024
│   ├── EVENT_CACHE_SIZE=256
│   └── HIVIEW_HIEVENT_FILE_BUF_SIZE=128
├── Include Dirs:
│   ├── //base/hiviewdfx/hievent_lite/interfaces/native/innerkits
│   ├── //base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite
│   ├── //base/hiviewdfx/hiview_lite
│   ├── //commonlibrary/utils_lite/include
│   ├── //third_party/bounds_checking_function/include
│   └── //foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr
├── Public Configs:
│   └── //base/hiviewdfx/hiview_lite:hiview_lite_config
└── Deps:
    └── //base/hiviewdfx/hiview_lite
```

```
Target: hievent_lite
├── 类型: group (逻辑分组)
├── 条件: hievent_lite_customize_implementation
│   ├── true:
│   │   └── public_configs: //base/hiviewdfx/hiview_lite:hiview_lite_config
│   └── false:
│       └── public_deps: :hievent_lite_static
└── 说明: 对外聚合 target，按实现方式选择配置
```

**证据**: `BUILD.gn:23-53`

### 2.2 command/BUILD.gn

**文件位置**: `command/BUILD.gn`

```
Target: hievent_lite_command
├── 类型: static_library
├── 输出: libhievent_lite_command.a
├── Sources:
│   └── command/hievent_lite_command.c
├── Cflags:
│   └── -Wall
└── Include Dirs:
    ├── //base/hiviewdfx/utils/lite
    └── //commonlibrary/utils_lite/include
```

---

## 3. 依赖关系

### 3.1 外部依赖

| 依赖组件 | 类型 | 用途 | 代码位置 |
|----------|------|------|----------|
| **hiview_lite** | `deps` | 基础服务、内存管理、文件系统 | `BUILD.gn:44` |
| **hilog_lite** | `include_dirs` | 日志输出 | `BUILD.gn:37-38` |
| **samgr_lite** | `include_dirs` | 系统能力管理 | `BUILD.gn:41` |
| **utils_lite** | `include_dirs` | 通用工具函数 | `BUILD.gn:39` |
| **bounds_checking_function** | `include_dirs` | 安全字符串函数 | `BUILD.gn:40` |

### 3.2 内部依赖

```
hievent_lite (group)
    │
    └── false ──▶ hievent_lite_static
                     │
                     └── hiview_lite (//base/hiviewdfx/hiview_lite)
```

### 3.3 依赖方向

```
                    ┌─────────────────┐
                    │  utils_lite     │  (通用工具)
                    └────────┬────────┘
                             │ include_dirs
                             ▼
┌──────────────────────────────────────────────────────────────┐
│  hievent_lite                                                │
│  ┌──────────────────┐  ┌──────────────────┐                 │
│  │ hiview_event.c   │  │ hiview_output_   │                 │
│  │                  │  │ event.c           │                 │
│  └────────┬─────────┘  └────────┬─────────┘                 │
│           │                     │                          │
│           └──────────┬──────────┘                          │
│                      │ deps                                   │
│                      ▼                                       │
│           ┌─────────────────────┐                            │
│           │   hiview_lite      │  (核心依赖)                  │
│           │   (//base/.../     │                            │
│           │    hiview_lite)    │                            │
│           └─────────┬───────────┘                            │
│                     │                                         │
│                     ▼                                         │
│           ┌─────────────────────┐                            │
│           │   samgr_lite       │  (系统能力)                  │
│           └─────────────────────┘                            │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. 编译产物

### 4.1 产物清单

| Target | 产物类型 | 产物名称 | 预计路径 |
|--------|----------|----------|----------|
| **hievent_lite_static** | 静态库 | `libhievent_lite_static.a` | `out/mini/.../obj/base/hiviewdfx/hievent_lite/` |
| **hievent_lite_command** | 静态库 | `libhievent_lite_command.a` | `out/mini/.../obj/base/hiviewdfx/hievent_lite/command/` |

### 4.2 产物大小

| 产物 | ROM | 说明 |
|------|-----|------|
| libhievent_lite_static.a | ~20 KB | 核心事件功能 |
| libhievent_lite_command.a | ~2 KB | 命令行工具 |
| **总计** | **~26 KB** | `bundle.json:27` |

### 4.3 运行时加载关系

```
静态链接到 hiview_lite
        │
        ▼
┌─────────────────────────────────────┐
│                                     │
│  用户进程 (链接 libhievent_lite.a)  │
│                                     │
│  调用流程:                          │
│  HiEventCreate()                    │
│      │                              │
│      ▼                              │
│  hiview_event.c (内联/静态链接)      │
│      │                              │
│      ▼                              │
│  hiview_output_event.c (内联/静态链接)│
│      │                              │
│      ▼                              │
│  hiview_lite (运行时依赖)            │
│      │                              │
│      ▼                              │
│  Flash 文件 / UART 输出              │
└─────────────────────────────────────┘
```

---

## 5. 配置参数

### 5.1 可配置项 (declare_args)

**证据**: `BUILD.gn:14-21`

| 参数名 | 类型 | 默认值 | 单位 | 说明 |
|--------|------|--------|------|------|
| `hievent_lite_fault_file_size` | int | 1024 | 字节 | 故障事件文件大小 |
| `hievent_lite_ue_file_size` | int | 1024 | 字节 | 用户事件文件大小 |
| `hievent_lite_stat_file_size` | int | 1024 | 字节 | 统计事件文件大小 |
| `hievent_lite_cache_size` | int | 256 | 字节 | 事件缓存大小 |
| `hievent_lite_file_buffer_size` | int | 128 | 字节 | 文件缓冲区大小 |
| `hievent_lite_customize_implementation` | bool | false | - | 自定义实现开关 |

### 5.2 编译宏定义

| 宏名 | 值来源 | 说明 |
|------|--------|------|
| `FAULT_EVENT_FILE_SIZE` | `hievent_lite_fault_file_size` | 故障文件大小 |
| `UE_EVENT_FILE_SIZE` | `hievent_lite_ue_file_size` | 用户文件大小 |
| `STAT_EVENT_FILE_SIZE` | `hievent_lite_stat_file_size` | 统计文件大小 |
| `EVENT_CACHE_SIZE` | `hievent_lite_cache_size` | 缓存大小 |
| `HIVIEW_HIEVENT_FILE_BUF_SIZE` | `hievent_lite_file_buffer_size` | 缓冲区大小 |

**证据**: `BUILD.gn:28-34`

### 5.3 配置示例

```gn
# product_name/product_name/libs/BUILD.gn 或类似位置

# 自定义事件文件大小
hievent_lite_fault_file_size = 2048   # 2KB 故障文件
hievent_lite_ue_file_size = 4096       # 4KB 用户事件文件
hievent_lite_stat_file_size = 8192     # 8KB 统计文件
hievent_lite_cache_size = 512          # 512B 缓存
```

---

## 6. 组件配置

### 6.1 bundle.json 定义

**证据**: `bundle.json:13-41`

```json
{
    "component": {
        "name": "hievent_lite",
        "subsystem": "hiviewdfx",
        "adapted_system_type": ["mini"],
        "features": [
            "hievent_lite_fault_file_size",
            "hievent_lite_ue_file_size",
            "hievent_lite_stat_file_size",
            "hievent_lite_cache_size",
            "hievent_lite_file_buffer_size",
            "hievent_lite_customize_implementation"
        ],
        "rom": "26KB",
        "ram": "~10KB",
        "deps": {
            "components": ["hiview_lite", "samgr_lite"],
            "third_party": []
        },
        "build": {
            "sub_component": [
                "//base/hiviewdfx/hievent_lite:hievent_lite"
            ]
        }
    }
}
```

### 6.2 子组件路径

| 组件 | GN 路径 |
|------|---------|
| hievent_lite | `//base/hiviewdfx/hievent_lite:hievent_lite` |

---

## 7. 构建命令

### 7.1 标准构建

```bash
# 在 OpenHarmony 根目录执行
hb set
hb build -f

# 或直接使用 GN
gn gen out/mini
ninja -C out/mini
```

### 7.2 只构建 hievent_lite

```bash
# 方式1: 使用 GN 路径
ninja -C out/mini //base/hiviewdfx/hievent_lite:hievent_lite

# 方式2: 使用 gn
gn gen out/mini --args="hievent_lite_fault_file_size=2048"
```

### 7.3 查看构建产物

```bash
# 查找静态库
find out/mini -name "libhievent_lite*.a"

# 查看产物大小
ls -lh out/mini/obj/base/hiviewdfx/hievent_lite/*.a
```

---

## 8. 资源占用

### 8.1 ROM 占用 (26 KB)

| 模块 | 估算大小 | 说明 |
|------|----------|------|
| hiview_event.c | ~8 KB | 事件创建、编码、宏 |
| hiview_output_event.c | ~12 KB | 缓存、文件、输出 |
| hiview_event.h | ~1 KB | 头文件 |
| 命令行工具 | ~2 KB | hievent_lite_command |
| 其他开销 | ~3 KB | 对齐、符号表 |

### 8.2 RAM 占用 (~10 KB)

| 用途 | 大小 | 说明 |
|------|------|------|
| 事件缓存 (×3) | 256 × 3 = 768 B | EVENT_CACHE_SIZE × 3 |
| 文件缓冲区 | 128 B | HIVIEW_HIEVENT_FILE_BUF_SIZE |
| 事件对象 | 动态分配 | HiEventCreate 分配 |
| 运行时开销 | ~9 KB | 互斥锁、消息队列等 |

---

**跳转**: [02_API_Reference.md](02_API_Reference.md) | [04_Security_Review.md](04_Security_Review.md) | [SUMMARY.md](SUMMARY.md)
