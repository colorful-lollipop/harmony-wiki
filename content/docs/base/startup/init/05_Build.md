# 构建与编译

## GN 构建系统

Init 模块使用 **GN (Generate Ninja)** 作为构建系统。

### 核心配置文件

| 文件 | 路径 | 说明 |
|------|------|------|
| **begetd.gni** | 根目录 | 核心构建配置，定义特性开关 |
| **BUILD.gn** | 根目录 | 根构建入口，定义 target groups |

### begetd.gni 关键配置

```python
# 特性开关示例
init_feature_ab_partition = false              # AB 分区支持
init_feature_seccomp_privilege = false         # Seccomp 特权
init_feature_group_type = true                 # 组类型支持
init_feature_use_hook_mgr = true               # 钩子管理器
init_startup_feature_erofs_overlay = false     # EROFS 覆盖层
init_appspawn_client_module = true             # AppSpawn 客户端
init_paramwatcher_hicollie_enable = true       # HiCollie 监视器
```

---

## 主要 Targets

### 框架组 (init_fwk_group)

| Target | 路径 | 类型 | 产物 |
|--------|------|------|------|
| `init` | `services/init/standard` | executable | `init` |
| `init_early` | `services/init/standard` | executable | `init_early` |
| `init` (lite) | `services/init/lite` | executable | `init` |
| `begetctl_cmd` | `services/begetctl` | executable | `begetctl` |
| `startup_init` | `services` | group | - |
| `modulesgroup` | `services/modules` | group | - |
| `parameter` | `services/param` | group | - |
| `loopeventgroup` | `services/loopevent` | group | - |
| `innergroup` | `interfaces/innerkits` | group | - |
| `kitsgroup` | `interfaces/kits` | group | - |

### 服务组 (init_service_group)

| Target | 路径 | 类型 | 产物 |
|--------|------|------|------|
| `startup_ueventd` | `ueventd` | executable | `ueventd` |
| `watchdog` | `watchdog` | executable | `watchdog` |
| `overlayremount` | `remount` | executable | `overlay_remount` |

---

## 库 Targets

### 内部库 (innerkits)

| Target | 路径 | 类型 | 说明 |
|--------|------|------|------|
| `libfsmanager_static` | `interfaces/innerkits/fs_manager` | static_library | 文件系统管理 |
| `libbegetutil` | `interfaces/innerkits` | shared_library | 工具库 |
| `libbegetutil_static` | `interfaces/innerkits` | static_library | 工具库 (静态) |
| `begetutil_headers` | `interfaces/innerkits` | headers | 头文件导出 |
| `libbeget_proxy` | `interfaces/innerkits` | shared_library | 代理库 |
| `seccomp` | `interfaces/innerkits/seccomp` | headers | Seccomp 策略头 |
| `libinit_module_engine` | `interfaces/innerkits/init_module_engine` | shared_library | 模块引擎 |
| `libsystemparam` | `interfaces/innerkits` | shared_library | 系统参数库 |

---

## 依赖关系

### 主要 deps

| Target | deps | 说明 |
|--------|------|------|
| `init` | loopevent, param, modules | 核心依赖 |
| `begetctl` | utils, beget_ext | 工具依赖 |
| `startup_ueventd` | ueventd, utils | uevent 依赖 |
| `loopeventgroup` | utils | 事件循环依赖 |

### configs

| Target | configs | 说明 |
|--------|---------|------|
| `init` | init_config, hilog | 日志配置 |
| `libbegetutil` | utils_config | 工具配置 |

---

## 编译产物

### 可执行文件

| 产物 | 目标设备 | 安装路径 | 说明 |
|------|----------|----------|------|
| `init` | standard | `/system/bin/` | 主 init 进程 |
| `init_early` | standard | `/system/bin/` | 早期 init (ramdisk) |
| `init` | lite | `/system/bin/` | 轻量系统 init |
| `begetctl` | all | `/system/bin/` | 控制工具 |
| `ueventd` | standard | `/system/bin/` | 设备事件守护 |
| `watchdog` | standard | `/system/bin/` | 看门狗服务 |

### 库文件

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| `libbegetutil.z.so` | shared | `/system/lib/` | 工具库 |
| `libbeget_proxy.z.so` | shared | `/system/lib/` | 代理库 |
| `libfsmanager.z.so` | shared | `/system/lib/` | 文件系统管理 |
| `libinit_module_engine.z.so` | shared | `/system/lib/` | 模块引擎 |
| `libsystemparam.z.so` | shared | `/system/lib/` | 系统参数 |

---

## 编译命令

### 全量编译

```bash
# 编译 init 模块
hb build -p init

# 单独编译
gn gen out/xxx
ninja -C out/xxx init
```

### 增量编译

```bash
ninja -C out/xxx //base/startup/init:services/init
```

---

## 构建特性开关

### 在 ohos.build 中配置

```json
{
    "subsystem": "startup",
    "components": [
        {
            "component": "init",
            "features": [
                "init_feature_seccomp_privilege = true",
                "init_feature_use_hook_mgr = true",
                "init_startup_feature_erofs_overlay = true"
            ]
        }
    ]
}
```

---

## 详细 Targets

### services/init/

| Target | BUILD.gn 路径 | 类型 | 源文件 |
|--------|--------------|------|--------|
| `init` | `services/init/standard` | executable | `main.c`, `init_service_manager.c`... |
| `init_early` | `services/init/standard` | executable | `init_firststage.c`... |

### services/modules/

| Target | BUILD.gn 路径 | 类型 | 职责 |
|--------|--------------|------|------|
| `bootevent` | `services/modules/bootevent` | static_library | 启动事件 |
| `crashhandler` | `services/modules/crashhandler` | static_library | 崩溃处理 |
| `seccomp` | `services/modules/seccomp` | static_library | Seccomp 策略 |
| `selinux` | `services/modules/selinux` | static_library | SELinux 适配 |
| `reboot` | `services/modules/reboot` | static_library | 重启功能 |
| `init_hook` | `services/modules/init_hook` | static_library | 钩子管理 |
| `init_eng` | `services/modules/init_eng` | static_library | 工程模式 |
| `sysevent` | `services/modules/sysevent` | static_library | 系统事件 |
| `udid` | `services/modules/udid` | static_library | 设备 ID |
| `encaps` | `services/modules/encaps` | static_library | 封装模块 |
| `init_context` | `services/modules/init_context` | static_library | 上下文 |
| `trace` | `services/modules/trace` | static_library | 跟踪 |
| `bootchart` | `services/modules/bootchart` | static_library | 启动图表 |
| `module_update` | `services/modules/module_update` | static_library | 模块更新 |
| `crashhandler` | `services/modules/crashhandler` | static_library | 崩溃处理 |

### services/param/

| Target | BUILD.gn 路径 | 类型 | 职责 |
|--------|--------------|------|------|
| `param_base` | `services/param/base` | static_library | 参数基础 |
| `param_liteos` | `services/param/liteos` | static_library | LiteOS 参数 |
| `param_linux` | `services/param/linux` | static_library | Linux 参数 |
| `param_watcher` | `services/param/watcher` | static_library | 参数监视 |
| `param_manager` | `services/param/manager` | static_library | 参数管理 |
| `param_trigger` | `services/param/trigger` | static_library | 参数触发 |

### interfaces/

| Target | BUILD.gn 路径 | 类型 | 职责 |
|--------|--------------|------|------|
| `libbegetutil` | `interfaces/innerkits` | shared_library | 工具库 |
| `libbegetutil_static` | `interfaces/innerkits` | static_library | 静态工具库 |
| `libbeget_proxy` | `interfaces/innerkits` | shared_library | 代理库 |
| `libfsmanager_static` | `interfaces/innerkits/fs_manager` | static_library | 文件系统 |
| `libinit_module_engine` | `interfaces/innerkits/init_module_engine` | shared_library | 模块引擎 |
| `seccomp` | `interfaces/innerkits/seccomp` | headers | Seccomp 头 |

---

## begetd.gni 完整配置

```python
# 基础配置
init_innerkits_path = "//base/startup/init/interfaces/innerkits"

# 特性开关
enable_ohos_startup_init_feature_watcher = true
enable_ohos_startup_init_feature_deviceinfo = true
init_feature_ab_partition = false
init_feature_begetctl_liteos = false
init_lite_use_posix_file_api = false
init_feature_enable_lite_process_priority = false
init_feature_use_hook_mgr = true
init_lite_memory_size = 10240
init_startup_feature_decode_group_file = false
startup_init_with_param_base = false
init_appspawn_client_module = true
init_begetutil_extra_modules = ""
init_extra_static_modules = ""
init_use_encaps = false
init_startup_feature_erofs_overlay = false
init_startup_feature_system_call_switch = false
init_paramwatcher_hicollie_enable = true
init_feature_seccomp_privilege = false
init_feature_group_type = true
init_get_disk_sn = false
init_feature_custom_sandbox = false
init_feature_support_asan = true
init_feature_support_saspawn = false
```

---

## 相关跳转

- [概览](01_Overview.md)
- [架构](02_Architecture.md)
- [安全机制](06_Security.md)
