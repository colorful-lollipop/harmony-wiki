# 编译产物

## 目的

本文档列出 Resource Schedule Service 的所有编译输出产物，包括库文件、配置文件和安装路径。

## 适用范围

- 系统集成工程师
- 调试和部署人员

---

## 共享库 (.so)

### 服务库

| 产物名 | 类型 | 路径 | 说明 |
|--------|------|------|------|
| `libresschedsvc.z.so` | SA | `/system/lib64/` | 资源调度主服务 |
| `libresschedexesvc.z.so` | SA | `/system/lib64/` | 资源调度执行器服务 |

### 客户端库

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `libressched_client.z.so` | `/system/lib64/` | RSS 客户端库 |
| `libresschedexeclient.z.so` | `/system/lib64/` | Executor 客户端库 |
| `libressched_common_utils.z.so` | `/system/lib64/` | 公共工具库 |
| `libprocess_group.z.so` | `/system/lib64/` | 进程组管理库 |

### 插件库

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `libcgroup_sched.z.so` | `/system/lib64/` | Cgroup 调度插件 |
| `libsocperf_plugin.z.so` | `/system/lib64/` | SocPerf 插件 |
| `libframe_aware_plugin.z.so` | `/system/lib64/` | 帧感知插件 |
| `libdevice_standby_plugin.z.so` | `/system/lib64/` | 设备待机插件 |
| `libsocperf_executor_plugin.z.so` | `/system/lib64/` | SocPerf 执行器插件 |

### N-API 库

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `systemload.so` | `/system/lib64/module/resourceschedule/` | SystemLoad JS API |
| `libbackgroundprocessmanager_napi.z.so` | `/system/lib64/` | BackgroundProcessManager JS API |

### C 接口库

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `libbackground_process_manager.z.so` | `/system/lib64/ndk/` | BackgroundProcessManager C API |

### 静态库 (.a)

| 产物名 | 说明 |
|--------|------|
| `libresschedsvc_static.a` | resschedsvc 静态库（测试用） |
| `libresschedexesvc_static.a` | resschedexesvc 静态库（测试用） |
| `libsocperf_plugin_static.a` | SocPerf 插件静态库 |

---

## 配置文件

### SA 配置

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `1901.json` | `/system/profile/` | ressched SA 配置 |
| `1918.json` | `/system/profile/` | ressched_executor SA 配置 |

**1901.json 内容**:
```json
{
    "name": 1901,
    "path": "/system/lib64/libresschedsvc.z.so"
}
```

### 资源调度配置

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `res_sched_config.xml` | `/system/etc/ressched/` | 插件主配置 |
| `res_sched_plugin_switch.xml` | `/system/etc/ressched/` | 插件开关配置 |

### Cgroup 配置

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `cgroup_action_config.json` | `/system/etc/cgroup_sched/` | Cgroup 策略配置 |

### Init 配置

| 产物名 | 路径 | 说明 |
|--------|------|------|
| `resource_schedule_service.cfg` | `/system/etc/init/` | 主服务启动配置 |
| `resource_schedule_executor.cfg` | `/system/etc/init/` | 执行器启动配置 |

---

## 运行时加载关系

### 进程启动时

```
resource_schedule_service 进程 (SA 1901)
├── 加载 libresschedsvc.z.so
│   ├── 依赖 libressched_client.z.so
│   ├── 依赖 libresschedexeclient.z.so
│   ├── 依赖 libressched_common_utils.z.so
│   └── 依赖 libprocess_group.z.so
│
└── 运行时加载插件 (根据配置)
    ├── libsocperf_plugin.z.so (可选)
    ├── libframe_aware_plugin.z.so (可选)
    ├── libdevice_standby_plugin.z.so (可选)
    └── libcgroup_sched.z.so (通常启用)

resource_schedule_executor 进程 (SA 1918)
├── 加载 libresschedexesvc.z.so
│   ├── 依赖 libresschedexeclient.z.so
│   └── 依赖 libprocess_group.z.so
│
└── 运行时加载插件
    └── libsocperf_executor_plugin.z.so
```

### JS 应用调用时

```
JS 应用
├── 加载 systemload.so
│   └── 依赖 libressched_client.z.so
│       └── IPC 调用 SA 1901
│
└── 加载 libbackgroundprocessmanager_napi.z.so
    └── 依赖 libbackground_process_manager.z.so
        └── 依赖 libressched_client.z.so
            └── IPC 调用 SA 1901
```

---

## 产物依赖树

```
/system/lib64/
├── libresschedsvc.z.so (SA 1901)
│   ├── libressched_client.z.so
│   │   └── libipc_core.z.so
│   ├── libresschedexeclient.z.so
│   ├── libressched_common_utils.z.so
│   │   ├── libc_utils.z.so
│   │   └── libffrt.so (条件)
│   └── libprocess_group.z.so
│
├── libresschedexesvc.z.so (SA 1918)
│   ├── libresschedexeclient.z.so
│   └── libprocess_group.z.so
│
└── 插件目录/
    ├── libcgroup_sched.z.so
    ├── libsocperf_plugin.z.so
    ├── libframe_aware_plugin.z.so
    └── libdevice_standby_plugin.z.so
```

---

## 安装路径汇总

| 路径 | 内容 |
|------|------|
| `/system/lib64/` | 主要共享库 |
| `/system/lib64/module/resourceschedule/` | N-API 模块 |
| `/system/lib64/ndk/` | NDK 库 |
| `/system/profile/` | SA 配置文件 |
| `/system/etc/ressched/` | 资源调度配置 |
| `/system/etc/cgroup_sched/` | Cgroup 配置 |
| `/system/etc/init/` | Init 启动配置 |

---

## 代码证据

### Init 配置文件

```json
// 文件: ressched/etc/init/resource_schedule_service.cfg

{
    "services": [{
        "name": "resource_schedule_service",
        "path": [
            "/system/bin/sa_main",
            "/system/lib64/libresschedsvc.z.so"
        ],
        "uid": "system",
        "gid": "system",
        "secon": "u:r:resource_schedule_service:s0",
        "start-mode": "boot"
    }]
}
```

### GN 安装配置

```gn
# 文件: ressched/interfaces/kits/js/napi/systemload/BUILD.gn

ohos_shared_library("systemload") {
    # ...
    relative_install_dir = "module/resourceschedule"
    subsystem_name = "resourceschedule"
    part_name = "resource_schedule_service"
}
```

---

## 相关链接

- [GN 构建](05_GN_Targets.md) - 构建配置详情
- [目录结构](02_Directory_Structure.md) - 代码组织
- [问题排查](08_Troubleshooting.md) - 调试方法
