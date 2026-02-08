# 编译产物

## 目的

本文档详细说明 `sys_installer` 的编译产物、安装路径和运行时加载关系。

## 适用范围

系统集成人员、发布工程师、运维人员。

## 产物清单

### 共享库 (.so)

| 产物名称 | 来源 Target | 大小(估计) | 用途 |
|----------|-------------|------------|------|
| libsys_installer.z.so | sys_installer | ~500KB | SA 4101 服务主库 |
| libmodule_update_service.z.so | module_update_service | ~400KB | SA 4103 服务主库 |
| libsysinstaller_shared.z.so | libsysinstaller_shared | ~300KB | 客户端共享库 |
| libmodule_update_shared.z.so | libmodule_update_shared | ~200KB | 模块更新客户端库 |
| libmodule_update_utils.z.so | module_update_utils | ~600KB | 模块更新工具库 |

### 静态库 (.a)

| 产物名称 | 来源 Target | 用途 |
|----------|-------------|------|
| libsysinstallerkits.a | libsysinstallerkits | 客户端静态链接 |
| libmodule_update_client_static.a | module_update_client_static | 模块更新客户端静态库 |
| libinstallermanager.a | libinstallermanager | 安装管理器 |
| libstatusmanager.a | libstatusmanager | 状态管理器 |
| libactionprocesser.a | libactionprocesser | 动作处理器 |
| libverifyaction.a | libverifyaction | 包验证动作 |
| libabupdate.a | libabupdate | AB 更新 |
| libstreamupdate.a | libstreamupdate | 流式更新 |
| libmodule_update_service_static.a | libmodule_update_service_static | 模块更新服务静态库 |
| check_module_update_static.a | check_module_update_static | 检查模块更新静态库 |
| module_update_static.a | module_update_static | 模块更新静态库 |

### 可执行文件

| 产物名称 | 来源 Target | 大小(估计) | 用途 |
|----------|-------------|------------|------|
| sys_installer_client | sys_installer_client | ~100KB | 测试客户端 |
| module_update_client | module_update_client | ~100KB | 模块更新客户端 |
| check_module_update_init | check_module_update | ~150KB | 启动时模块检查 |
| module_update_tool | module_update_tool | ~100KB | 模块更新 CLI 工具 |

### 配置文件

| 产物名称 | 来源 | 用途 |
|----------|------|------|
| sys_installer.cfg | etc/BUILD.gn | SA 4101 init 配置 |
| sys_installer.para | etc/BUILD.gn | 系统参数 |
| sys_installer.para.dac | etc/BUILD.gn | 参数权限 |
| module_update_sa.cfg | etc/BUILD.gn | SA 4103 init 配置 |
| check_module_update.cfg | etc/BUILD.gn | 启动检查配置 |
| sys_installer_sa.rc | etc/BUILD.gn | init rc 脚本 |
| 4101.json | sa_profile/BUILD.gn | SA 4101 定义 |
| 4103.json | sa_profile/BUILD.gn | SA 4103 定义 |

## 安装路径

### 库文件

```
/system/lib/
├── libsys_installer.z.so
├── libmodule_update_service.z.so
├── libsysinstaller_shared.z.so
├── libmodule_update_shared.z.so
└── libmodule_update_utils.z.so
```

### 可执行文件

```
/system/bin/
├── sys_installer_client
├── module_update_client
├── check_module_update_init
└── module_update_tool
```

### 配置文件

```
/system/etc/init/
├── sys_installer.cfg
├── module_update_sa.cfg
├── check_module_update.cfg
└── sys_installer_sa.rc

/system/etc/param/
├── sys_installer.para
└── sys_installer.para.dac

/system/profile/
├── 4101.json
└── 4103.json
```

### 运行时目录

```
/data/module_update_package/    # 更新包目录
/data/module_update/active/     # 激活模块目录
/data/module_update/backup/     # 备份模块目录
/data/updater/log/              # 日志目录
```

## 运行时加载关系

### SA 4101 加载链

```
init 进程
    │
    ├── 读取 /system/etc/init/sys_installer.cfg
    │
    └── 按需启动 sys_installer_sa 进程
            │
            ├── 加载 libsys_installer.z.so
            │
            ├── 链接依赖库:
            │   ├── libsysinstaller_shared.z.so
            │   ├── libmodule_update_utils.z.so (间接)
            │   └── 系统库 (libipc_core.z.so, etc.)
            │
            └── 注册到 samgr (System Ability Manager)
                    │
                    └── 等待 IPC 调用
```

### SA 4103 加载链

```
触发条件 (start-on-demand):
- persist.samgr.moduleupdate.start=true
- persist.moduleupdate.bms.scan=revert

init 进程 / samgr
    │
    ├── 检测触发条件
    │
    └── 启动 module_update_sa 进程
            │
            ├── 加载 libmodule_update_service.z.so
            │
            ├── 链接依赖库:
            │   ├── libmodule_update_utils.z.so
            │   └── 系统库
            │
            └── 注册到 samgr
                    │
                    └── 等待 IPC 调用

停止条件 (stop-on-demand):
- bootevent.boot.completed=true
```

### 客户端加载链

```
客户端应用 (如 Updater)
    │
    ├── 链接 libsysinstaller_shared.z.so
    │
    ├── 调用 SysInstallerKits::GetInstance()
    │
    ├── 通过 samgr 获取 SA 4101 代理
    │   └── 如果 SA 未启动，触发按需加载
    │
    └── 发起 IPC 调用
```

## 依赖关系图

### 运行时依赖

```
libsys_installer.z.so
    ├── libsysinstaller_shared.z.so
    ├── libipc_core.z.so (系统)
    ├── libsamgr_proxy.z.so (系统)
    ├── libaccesstoken_sdk.z.so (系统)
    └── libhilog.z.so (系统)

libmodule_update_service.z.so
    ├── libmodule_update_utils.z.so
    ├── libipc_core.z.so (系统)
    ├── libsamgr_proxy.z.so (系统)
    ├── libaccesstoken_sdk.z.so (系统)
    └── libhvb_static_real.z.so (可选)

libsysinstaller_shared.z.so
    ├── libipc_core.z.so (系统)
    ├── libsamgr_proxy.z.so (系统)
    └── libhilog.z.so (系统)

libmodule_update_utils.z.so
    ├── liblz4.z.so (系统)
    ├── libz.so (系统)
    ├── libbz2.so (系统)
    ├── libcrypto.z.so (系统)
    └── libhilog.z.so (系统)
```

## 构建输出目录结构

```
out/
└── standard/
    ├── system/
    │   ├── lib/
    │   │   ├── libsys_installer.z.so
    │   │   ├── libmodule_update_service.z.so
    │   │   ├── libsysinstaller_shared.z.so
    │   │   ├── libmodule_update_shared.z.so
    │   │   └── libmodule_update_utils.z.so
    │   ├── bin/
    │   │   ├── sys_installer_client
    │   │   ├── module_update_client
    │   │   ├── check_module_update_init
    │   │   └── module_update_tool
    │   └── etc/
    │       ├── init/
    │       │   ├── sys_installer.cfg
    │       │   ├── module_update_sa.cfg
    │       │   └── sys_installer_sa.rc
    │       └── param/
    │           ├── sys_installer.para
    │           └── sys_installer.para.dac
    └── system_profile/
        ├── 4101.json
        └── 4103.json
```

## 产物验证

### 文件权限

| 产物类型 | 权限 | 所有者 |
|----------|------|--------|
| .so 文件 | 644 | root:root |
| 可执行文件 | 755 | root:root |
| 配置文件 | 644 | root:root |
| 运行时目录 | 755 | system:system |

### SELinux 标签

```
/system/lib/libsys_installer.z.so    u:object_r:system_lib_file:s0
/system/bin/sys_installer_client     u:object_r:sys_installer_exec:s0
/system/etc/init/sys_installer.cfg   u:object_r:system_configs_file:s0
```

## 版本信息

### 库版本

```bash
# 查看库版本
readelf -V libsys_installer.z.so

# 预期输出
Version symbols section '.gnu.version' contains 100 entries:
 Addr: 0000000000000000  Offset: 0x000000  Link: 3 (.dynsym)
  000:   0 (*local*)       1 (*global*)       2 (IPC::...)        3 (OHOS::...)
```

### 构建信息

```cpp
// 代码中嵌入的版本信息
#define SYS_INSTALLER_VERSION "3.2.0"
#define SYS_INSTALLER_BUILD_TIME __DATE__ " " __TIME__
```

## 相关链接

- [GN Targets](05_GN_Targets.md)
- [目录结构](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)

---

*证据来源*:
- `bundle.json`: 产物定义
- `frameworks/ipc_server/etc/`: 配置文件
- `frameworks/ipc_server/sa_profile/`: SA 定义
- `BUILD.gn`: 构建配置
