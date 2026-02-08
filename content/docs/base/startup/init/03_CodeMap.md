# 目录结构与代码地图

本文档帮助开发者快速定位 init 模块的核心代码，理解各目录职责和关键文件位置。

## 顶层目录概览

```
base/startup/init/
├── device_info/           # [M] 设备信息服务 (SA 服务)
├── initsync/             # [M] 启动同步服务
├── interfaces/            # [E] 对外接口层
│   ├── innerkits/        # [E1] 内部组件接口 (供其他子系统使用)
│   └── kits/             # [E2] 外部 SDK 接口 (N-API)
├── remount/              # [M] 重新挂载服务
├── scripts/              # [D] 启动脚本
├── services/             # [C] 核心服务实现
│   ├── begetctl/        # [C1] 命令行工具
│   ├── init/            # [C2] init 主进程 (核心)
│   ├── log/             # [C3] 日志服务
│   ├── loopevent/       # [C4] 事件循环框架
│   ├── modules/         # [C5] 功能模块
│   ├── param/           # [C6] 参数服务
│   ├── sandbox/         # [C7] 沙箱服务
│   └── utils/           # [C8] 工具函数
├── ueventd/              # [M] uevent 守护进程
├── watchdog/             # [M] 看门狗服务
├── simulator/            # [D] 模拟器 (测试用)
└── BUILD.gn             # [B] 构建入口
```

**标记说明**：
- `[C]` Core：核心实现
- `[E]` External：对外接口
- `[M]` Module：子模块/服务
- `[D]` Development：开发辅助
- `[B]` Build：构建配置

---

## 核心代码定位

### C2 - init 主进程（最重要）

**小型系统实现**：
| 文件 | 职责 |
|------|------|
| `services/init/lite/init.c:32-110` | **核心入口**：SystemInit() → SystemConfig() → SystemRun() |
| `services/init/lite/init_jobs.c` | Jobs 解析与执行 |
| `services/init/lite/init_service.c` | 服务生命周期管理 |
| `services/init/lite/init_cmds.c` | 命令执行 (exec/loadcfg) |
| `services/init/lite/init_signal_handler.c` | 信号处理 (SIGCHLD/SIGTERM) |
| `services/init/lite/init_reboot.c` | 系统重启 |

**标准系统实现**：
| 文件 | 职责 |
|------|------|
| `services/init/standard/main_early.c` | 标准系统主入口 |
| `services/init/standard/init.c` | 主初始化逻辑 |
| `services/init/standard/init_cmds.c` | **命令集（34KB！）** |
| `services/init/standard/init_cmdexecutor.c` | 命令执行器 |
| `services/init/standard/init_service.c` | 服务管理 |
| `services/init/standard/init_firststage.c` | 第一阶段初始化 |

**证据**（`services/init/lite/init.c:64-96` - 核心启动流程）：
```c
void SystemConfig(const char *uptime)
{
    InitServiceSpace();
    // 加载参数
    ReadConfig();  // 读取 init.cfg

    // 三阶段启动
    DoJob("pre-init");  // 文件系统挂载等
    DoJob("init");      // 服务启动
    DoJob("post-init"); // 后置操作
    ReleaseAllJobs();
}
```

---

## 核心头文件定位

### 初始化相关

| 头文件 | 职责 |
|--------|------|
| `services/init/include/init.h` | 核心函数声明（SystemInit/SystemConfig/SystemRun） |
| `services/init/include/init_service.h` | 服务结构与函数 |
| `services/init/include/init_jobs_internal.h` | Jobs 内部结构 |
| `services/init/include/init_cmds.h` | 命令表定义 |

### 接口相关

| 头文件 | 职责 |
|--------|------|
| `interfaces/innerkits/include/init_socket.h` | Socket 初始化 |
| `interfaces/innerkits/include/init_file.h` | 文件操作 |
| `interfaces/innerkits/include/init_reboot.h` | 重启接口 |
| `interfaces/innerkits/include/hookmgr.h` | 钩子管理 |
| `interfaces/innerkits/include/service_control.h` | 服务控制 |
| `interfaces/innerkits/include/loop_event.h` | 事件循环 |

---

## 功能模块定位

### C5 - 功能模块 (services/modules/)

| 模块 | 文件 | 职责 |
|------|------|------|
| bootchart | `services/modules/bootchart/` | 启动时间分析 |
| bootevent | `services/modules/bootevent/` | 启动事件管理 |
| crashhandler | `services/modules/crashhandler/` | 崩溃处理 |
| init_eng | `services/modules/init_eng/` | 初始化引擎 |
| init_hook | `services/modules/init_hook/` | 钩子回调 |
| reboot | `services/modules/reboot/` | 重启模块 |
| seccomp | `services/modules/seccomp/` | **安全：系统调用过滤** |
| selinux | `services/modules/selinux/` | **安全：SELinux 集成** |
| sysevent | `services/modules/sysevent/` | 系统事件 |

### C6 - 参数服务 (services/param/)

| 文件 | 职责 |
|------|------|
| `services/param/include/param_manager.h` | 参数管理 |
| `services/param/include/param_persist.h` | 持久化参数 |
| `services/param/include/trigger_manager.h` | 触发器管理 |
| `services/param/include/param_security.h` | 参数安全 |

---

## 代码导航图

### 我想修改...

| 修改目标 | 目标文件 |
|---------|---------|
| **添加新的 Jobs 命令** | `services/init/lite/init_cmds.c` + `services/init/standard/init_cmds.c` |
| **修改服务启动逻辑** | `services/init/lite/init_service.c` |
| **添加新的 init.cfg 解析** | `services/init/lite/init_jobs.c` |
| **修改信号处理** | `services/init/lite/init_signal_handler.c` |
| **添加 SELinux 支持** | `services/modules/selinux/selinux_adp.h` |
| **添加 Seccomp 策略** | `services/modules/seccomp/` |
| **修改系统参数读写** | `services/param/` |

### 我想理解...

| 理解目标 | 关键文件 |
|---------|---------|
| **init 何时被调用** | `services/init/lite/init.c:64-96` |
| **Jobs 如何执行** | `services/init/lite/init_jobs.c:126-145` |
| **服务如何启动** | `services/init/lite/init_service.c:81-101` |
| **进程崩溃如何处理** | `services/init/lite/init_signal_handler.c:33-45` |
| **配置文件格式** | README.md `init.cfg` 示例 |
| **N-API 入口** | `interfaces/kits/` |

---

## 子模块定位

### D - ueventd (设备事件)

```
ueventd/
├── ueventd.c          # 主循环
├── ueventd.h
├── uevent_parser.c    # 事件解析
└── uevent_watcher.c   # 事件监控
```

### M - watchdog (看门狗)

```
watchdog/
├── watchdog.c          # 看门狗主逻辑
└── watchdog.h
```

### M - device_info (设备信息服务)

```
device_info/
├── device_info.cpp    # SA 服务实现
└── device_info.h
```

---

## 接口层概览

### E1 - Inner API (内部接口)

Inner kits 供其他 OpenHarmony 子系统调用：

| 接口 | 头文件 | 说明 |
|------|--------|------|
| 服务控制 | `service_control.h` | 启动/停止服务 |
| 参数读写 | `syspara/parameter.h` | GetParameter/SetParameter |
| 事件循环 | `loop_event.h` | LE_Loop/LE_Timer |
| 钩子管理 | `hookmgr.h` | 注册回调 |

### E2 - N-API (JS 接口)

| 模块 | 主要文件 | JS API 示例 |
|------|---------|------------|
| systemparameter | `native_parameters_js.cpp` | `parameter.get()` |
| deviceInfo | `native_deviceinfo_js.cpp` | `deviceInfo.getInfo()` |

---

## 构建配置定位

### 关键 GN 文件

| 文件 | 职责 |
|------|------|
| `begetd.gni` | **特性开关定义**（30+ 个 feature） |
| `BUILD.gn` | **构建入口**：init_fwk_group / init_service_group |
| `services/init/lite/BUILD.gn` | 小型系统编译配置 |
| `services/init/standard/BUILD.gn` | 标准系统编译配置 |

### 关键产物

| 产物 | 位置 | 说明 |
|------|------|------|
| `init` | system/bin/ | init 主进程 |
| `begetctl` | system/bin/ | 命令行工具 |
| `libbegetutil.so` | system/lib/ | N-API 库 |
| `ueventd` | system/bin/ | 设备事件守护 |
| `watchdog` | system/bin/ | 看门狗服务 |

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [N-API 接口](03_NAPI.md)
- [Inner API](04_InnerAPI.md)
- [构建配置](05_Build.md)
- [攻击面分析](05_AttackSurface.md)
- [安全风险评估](06_SecurityReview.md)
