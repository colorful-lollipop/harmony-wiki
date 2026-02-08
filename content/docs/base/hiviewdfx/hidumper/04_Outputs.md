# 编译产物

> 目的：理解 HiDumper 构建输出的所有产物、路径、用途

## 1. 产物清单

### 1.1 可执行文件

| 产物名称 | 路径 | 大小 | 说明 |
|---------|------|-----|------|
| hidumper | `out/{target}/hiviewdfx/hidumper/frameworks/native/` | ~500KB | 主命令行工具 |

**用途**: 系统信息导出 CLI，用户直接交互入口

### 1.2 动态库 (.so)

| 产物名称 | 路径 | 类型 | 说明 |
|---------|------|-----|------|
| libhidumperclient.so | `out/{target}/hiviewdfx/hidumper/frameworks/native/` | 客户端库 | 客户端 IPC 封装 |
| libhidumper_client.so | `out/{target}/hiviewdfx/hidumper/services/` | 服务客户端库 | 服务内部 IPC 客户端 |
| libhidumperservice.so | `out/{target}/hiviewdfx/hidumper/services/` | **SA 库** | 主服务实现 (1212) |
| libhidumpermemory.so | `out/{target}/hiviewdfx/hidumper/services/` | 功能库 | 内存 dump 功能 |
| libhidumpercpuservice.so | `out/{target}/hiviewdfx/hidumper/services/` | **SA 库** | CPU 服务 (1215, 条件编译) |
| lib_dump_usage.so | `out/{target}/hiviewdfx/hidumper/interfaces/innerkits/` | SDK 库 | 内存/CPU 统计接口 |

### 1.3 配置文件

| 产物名称 | 路径 | 说明 |
|---------|------|------|
| hidumper_service.cfg | `out/{target}/hiviewdfx/hidumper/services/native/etc/` | init 启动配置 |
| infos_config.json | `out/{target}/hiviewdfx/hidumper/services/native/etc/` | 信息配置 (root variant) |
| task_enable_config.json | `out/{target}/hiviewdfx/hidumper/services/native/etc/` | 任务使能配置 |
| event_reason_config.json | `out/{target}/hiviewdfx/hidumper/services/native/etc/` | 事件原因配置 |

### 1.4 SA 配置文件

| 产物名称 | 路径 | SA ID | 说明 |
|---------|------|-------|------|
| 1212.json | `out/{target}/hiviewdfx/hidumper/sa_profile/` | 1212 | DumpManagerService 配置 |

## 2. 安装路径

### 2.1 系统安装

构建产物会安装到设备的以下路径：

| 产物 | 安装路径 |
|-----|---------|
| hidumper | `/system/bin/hidumper` |
| libhidumperservice.z.so | `/system/sa/1212/` |
| libhidumpercpuservice.z.so | `/system/sa/1215/` |
| 配置文件 | `/system/etc/` |

### 2.2 SA 配置安装

**SA ID 1212**:
```
/system/sa/1212/
└── libhidumperservice.z.so  (实际安装为 libhidumperservice.z.so)
```

**SA ID 1215**:
```
/system/sa/1215/
└── libhidumpercpuservice.z.so  (实际安装为 libhidumpercpuservice.z.so)
```

## 3. 运行时加载关系

### 3.1 客户端加载链

```
hidumper (可执行文件)
    │
    ├── libhidumperclient.so
    │       │
    │       └── libhidumper_client.so
    │               │
    │               └── libz.so, libbinder.so (系统库)
    │
    └── libbz2.so, libzlib.so (压缩库)
```

### 3.2 服务加载链

```
SystemAbilityManager
        │
        ▼
LoadSystemAbility(1212) ──▶ libhidumperservice.z.so
        │                       │
        │                       ├── libhidumper_client.so
        │                       │       │
        │                       │       └── libz.so, libbinder.so
        │                       │
        │                       ├── libhidumpermemory.so
        │                       │
        │                       ├── libdump_usage.so (按需)
        │                       │
        │                       └── utils (静态链接或 so)
```

### 3.3 按需加载

| 场景 | 加载的库 |
|-----|---------|
| 用户执行 hidumper | hidumper + libhidumperclient.so |
| 获取系统信息 | libhidumperservice.so (IPC) |
| 获取 CPU 使用率 | libhidumpercpuservice.so (IPC) |
| JS Heap 分析 | libhidumpermemory.so |

## 4. 产物与 Target 映射

| Target | 类型 | 输出产物 |
|--------|-----|---------|
| `frameworks/native:hidumper` | executable | hidumper |
| `frameworks/native:hidumperclient` | shared_library | libhidumperclient.so |
| `services:hidumperservice` | shared_library | libhidumperservice.so |
| `services:hidumpermemory` | shared_library | libhidumpermemory.so |
| `services:hidumpercpuservice` | shared_library | libhidumpercpuservice.so |
| `interfaces/innerkits:lib_dump_usage` | shared_library | lib_dump_usage.so |
| `sa_profile:hidumper_service_sa_profile` | sa_profile | 1212.json |

## 5. 产物验证

### 5.1 检查可执行文件

```bash
# 查看文件类型
file hidumper
# 输出: ELF 64-bit LSB executable, ARM aarch64

# 查看依赖
ldd hidumper
```

### 5.2 检查 SA 服务

```bash
# 查看 SA 注册
hidumper -ls
# 输出 System Ability 列表

# 手动调用 SA
hidumper -s 1212
```

## 相关文档

- [构建系统](./03_Build_System.md)
- [系统架构](./01_Architecture.md)
- [API 参考](./02_API_Reference.md)
