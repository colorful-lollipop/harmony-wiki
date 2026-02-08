# 目录结构

## 目的

本文档详细说明 `sys_installer` 的目录组织结构，帮助开发者快速定位代码。

## 适用范围

OpenHarmony sys_installer 部件源代码树。

## 顶层目录

```
base/update/sys_installer/
├── BUILD.gn                    # 根构建文件
├── bundle.json                 # 部件配置
├── sys_installer_default_cfg.gni # 默认构建配置
├── LICENSE                     # Apache 2.0 许可证
├── README.md / README.en.md    # 中英文 readme
├── CODEOWNERS                  # 代码所有者
├── OAT.xml                     # OSS 审计配置
├── common/                     # 公共代码
├── frameworks/                 # 框架层
├── interfaces/                 # 接口层
├── services/                   # 业务实现层
├── tools/                      # 工具
└── test/                       # 测试 (本文档不覆盖)
```

## 详细目录结构

### 1. common/ - 公共代码

```
common/
└── include/
    └── sys_installer_common.h    # 公共定义、错误码、日志路径
```

**关键符号**:
- 错误码定义
- 日志文件路径常量
- 公共宏定义

### 2. frameworks/ - 框架层

框架层提供系统安装的基础框架能力。

#### 2.1 action_processer/ - 动作处理器

```
frameworks/action_processer/
├── BUILD.gn
├── include/
│   └── action_processer.h        # 动作处理框架
└── src/
    └── action_processer.cpp      # 动作处理器实现
```

**职责**: 提供可扩展的动作处理框架，支持链式动作执行。

#### 2.2 actions/verify_action/ - 包验证动作

```
frameworks/actions/verify_action/
├── BUILD.gn
├── include/
│   ├── iaction.h                 # 动作接口基类
│   └── pkg_verify.h              # 包验证动作
└── src/
    └── pkg_verify.cpp            # 包签名验证实现
```

**关键类**:
- `PkgVerify`: 包签名验证动作

#### 2.3 installer_manager/ - 安装管理器

```
frameworks/installer_manager/
├── BUILD.gn
├── include/
│   ├── sys_installer_manager.h           # 主安装管理器
│   ├── sys_installer_manager_helper.h    # 管理器辅助类
│   ├── stream_installer_manager.h        # 流式更新管理器
│   └── stream_installer_manager_helper.h # 流式辅助类
└── src/
    ├── sys_installer_manager.cpp
    ├── sys_installer_manager_helper.cpp
    ├── stream_installer_manager.cpp
    └── stream_installer_manager_helper.cpp
```

**关键类**:
- `SysInstallerManager`: 系统安装管理器主类
- `StreamInstallerManager`: 流式更新管理器

#### 2.4 ipc_server/ - SA 服务端框架

```
frameworks/ipc_server/
├── BUILD.gn
├── etc/
│   ├── BUILD.gn
│   ├── sys_installer.cfg         # SA 4101 配置
│   ├── sys_installer.para        # 系统参数
│   ├── sys_installer.para.dac    # 参数权限
│   ├── module_update_sa.cfg      # SA 4103 配置
│   ├── check_module_update.cfg   # 启动检查配置
│   └── sys_installer_sa.rc       # init rc 脚本
├── include/
│   ├── sys_installer_server.h    # SA 4101 服务类
│   ├── module_update_service.h   # SA 4103 服务类
│   └── module_update_stub.h      # IPC Stub 实现
├── sa_profile/
│   ├── BUILD.gn
│   ├── 4101.json                 # SA 4101 定义
│   └── 4103.json                 # SA 4103 定义
└── src/
    ├── sys_installer_server.cpp
    ├── module_update_service.cpp
    └── module_update_stub.cpp
```

**关键类**:
- `SysInstallerServer`: SA 4101 服务实现
- `ModuleUpdateService`: SA 4103 服务实现
- `ModuleUpdateStub`: IPC Stub 基类

**SA 配置文件**:
- `4101.json`: SYS_INSTALLER_DISTRIBUTED_SERVICE_ID
- `4103.json`: MODULE_UPDATE_SERVICE_ID

#### 2.5 status_manager/ - 状态管理器

```
frameworks/status_manager/
├── BUILD.gn
├── include/
│   ├── status_manager.h          # 更新状态管理
│   └── stream_status_manager.h   # 流式更新状态
└── src/
    ├── status_manager.cpp
    └── stream_status_manager.cpp
```

### 3. interfaces/ - 接口层

#### 3.1 inner_api/ - 内部 API 定义

```
interfaces/inner_api/
└── include/
    ├── sys_installer_sa_ipc_interface_code.h  # IPC 命令码
    ├── imodule_update.h                       # IModuleUpdate 接口
    └── isys_installer_callback_func.h         # 回调接口
```

**关键定义**:
- `ModuleUpdateInterfaceCode`: 模块更新 IPC 命令码 (1-8)
- `SysInstallerCallbackInterfaceCode`: 回调 IPC 命令码
- `IModuleUpdate`: 模块更新接口定义

#### 3.2 innerkits/ - 对外 Kit 实现

```
interfaces/innerkits/
└── ipc_client/
    ├── BUILD.gn
    ├── ISysInstaller.idl           # IDL 接口定义
    ├── ISysInstallerCallback.idl   # 回调 IDL
    ├── Types.idl                   # 公共类型 IDL
    ├── include/
    │   ├── sys_installer_kits.h            # 主 Kit 接口
    │   ├── sys_installer_kits_impl.h       # Kit 实现
    │   ├── module_update_kits.h            # 模块更新 Kit
    │   ├── module_update_kits_impl.h       # 模块更新实现
    │   ├── module_update_proxy.h           # IPC Proxy
    │   ├── sys_installer_callback.h        # 回调包装
    │   ├── sys_installer_load_callback.h   # SA 加载回调
    │   ├── module_update_load_callback.h   # 模块 SA 加载回调
    │   ├── buffer_info_parcel.h            # Parcel 辅助
    │   └── sys_installer_task_const.h      # 任务常量
    └── src/
        ├── sys_installer_kits_impl.cpp     # 主 Kit 实现 (662 行)
        ├── module_update_kits_impl.cpp     # 模块 Kit 实现 (255 行)
        ├── module_update_proxy.cpp         # Proxy 实现 (256 行)
        ├── sys_installer_callback.cpp
        ├── sys_installer_load_callback.cpp
        ├── module_update_load_callback.cpp
        ├── buffer_info_parcel.cpp
        └── sys_installer_client.cpp        # 旧版客户端
```

**关键类**:
- `SysInstallerKits`: 系统安装器主 Kit 接口
- `ModuleUpdateKits`: 模块更新 Kit 接口
- `ModuleUpdateProxy`: IPC Proxy 实现

### 4. services/ - 业务实现层

#### 4.1 ab_update/ - AB 更新

```
services/ab_update/
├── BUILD.gn
├── include/
│   └── ab_update.h               # AB 更新接口
└── src/
    └── ab_update.cpp             # AB 更新实现
```

**职责**: 处理 A/B 分区更新逻辑。

#### 4.2 stream_update/ - 流式更新

```
services/stream_update/
├── BUILD.gn
├── include/
│   └── stream_update.h           # 流式更新接口
└── src/
    └── stream_update.cpp         # 流式更新实现
```

**职责**: 处理网络流式增量更新。

#### 4.3 module_update/ - 模块更新

```
services/module_update/
├── BUILD.gn
├── include/
│   ├── module_update.h           # 主模块更新类
│   ├── module_update_task.h      # 任务管理
│   ├── module_dm.h               # Device Mapper 操作
│   ├── module_loop.h             # Loop 设备管理
│   └── module_file_repository.h  # 文件仓库
├── service/
│   ├── BUILD.gn
│   ├── include/
│   │   ├── module_update_main.h      # 服务主类
│   │   ├── module_update_producer.h  # 生产者线程
│   │   ├── module_update_consumer.h  # 消费者线程
│   │   └── module_update_queue.h     # 任务队列
│   └── src/
│       ├── module_update_main.cpp    # 服务实现 (607 行)
│       ├── module_update_producer.cpp
│       ├── module_update_consumer.cpp
│       ├── module_update_queue.cpp
│       └── main.cpp                  # 服务入口
├── src/
│   ├── BUILD.gn
│   ├── module_dm.cpp             # DM 实现
│   ├── module_file_repository.cpp
│   ├── module_loop.cpp           # Loop 设备 (454 行)
│   ├── module_update.cpp         # 核心实现 (489 行)
│   ├── module_update_task.cpp
│   └── main.cpp                  # 检查入口
└── util/
    ├── include/
    │   ├── module_constants.h      # 常量定义
    │   ├── module_error_code.h     # 错误码
    │   ├── module_file.h           # 模块文件结构
    │   ├── module_hvb_ops.h        # HVB 操作
    │   ├── module_hvb_utils.h      # HVB 工具
    │   ├── module_ipc_helper.h     # IPC 序列化
    │   ├── module_update_verify.h  # 验证
    │   ├── module_utils.h          # 通用工具
    │   └── module_zip_helper.h     # ZIP 处理
    └── src/
        ├── module_file.cpp         # 文件操作 (576 行)
        ├── module_hvb_ops.cpp
        ├── module_hvb_utils.cpp
        ├── module_ipc_helper.cpp
        ├── module_update_verify.cpp  # 验证实现 (249 行)
        ├── module_utils.cpp        # 工具 (498 行)
        └── module_zip_helper.cpp   # ZIP (222 行)
```

**关键类**:
- `ModuleUpdate`: 模块更新主类
- `ModuleUpdateMain`: 服务主类
- `ModuleFile`: 模块文件管理
- `ModuleLoop`: Loop 设备管理
- `ModuleDm`: Device Mapper 操作

**关键路径** (来自 `module_constants.h`):
- `/data/module_update_package/`: 更新包目录
- `/data/module_update/active/`: 激活模块目录
- `/data/module_update/backup/`: 备份模块目录
- `/system/module_update/`: 预装模块目录

### 5. tools/ - 工具

```
tools/
├── module_update_tool/
│   ├── BUILD.gn
│   └── main.cpp                  # CLI 工具 (219 行)
└── zipalign/
    ├── BUILD.gn
    └── ...                       # Java ZIP 对齐工具
```

## 文件统计

| 模块 | 源文件数 | 主要代码行数 |
|------|----------|--------------|
| frameworks/installer_manager | 8 | ~1,500 |
| frameworks/ipc_server | 6 | ~1,200 |
| interfaces/innerkits | 21 | ~3,000 |
| services/module_update | 32 | ~5,000 |
| 其他 | 22 | ~2,000 |
| **总计** | **89** | **~12,700** |

## 快速定位指南

| 需求 | 文件路径 |
|------|----------|
| 查看 SA 4101 服务实现 | `frameworks/ipc_server/src/sys_installer_server.cpp` |
| 查看 SA 4103 服务实现 | `frameworks/ipc_server/src/module_update_service.cpp` |
| 查看 Kit 接口 | `interfaces/innerkits/ipc_client/include/sys_installer_kits.h` |
| 查看模块更新核心 | `services/module_update/src/module_update.cpp` |
| 查看包验证 | `frameworks/actions/verify_action/src/pkg_verify.cpp` |
| 查看 HVB 验证 | `services/module_update/util/src/module_hvb_utils.cpp` |
| 查看权限检查 | `frameworks/ipc_server/src/sys_installer_server.cpp:329-350` |
| 查看 SA 配置 | `frameworks/ipc_server/sa_profile/4101.json` |

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](02_Architecture.md)
- [GN Targets](05_GN_Targets.md)

---

*证据来源*: 实际目录结构扫描，详见 `wiki/_work/NOTES.md`
