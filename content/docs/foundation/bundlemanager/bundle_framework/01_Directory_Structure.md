# 目录结构与模块职责

## 顶层目录概览

```
bundle_framework/
├── common/                    # 公共组件（日志、工具）
├── etc/                       # 配置文件
├── figures/                   # 架构图等资源
├── interfaces/                # API 接口层
├── sa_profile/                # SA 配置文件
├── services/                  # 服务实现
└── test/                      # 测试代码（忽略）
```

---

## 接口层 (interfaces/)

### 内部 API (inner_api/)

**用途**: 供 OpenHarmony 其他子系统使用的 C++ 接口。

| 目录 | 职责 | 关键头文件 |
|------|------|------------|
| `appexecfwk_base/` | 基础数据结构与常量 | `ability_info.h`, `application_info.h`, `bundle_info.h`, `appexecfwk_errors.h`, `bundle_constants.h` |
| `appexecfwk_core/` | 核心 IPC 接口与代理 | `bundle_mgr_interface.h`, `bundle_installer_interface.h`, `bundle_mgr_proxy.h` |
| `bundlemgr_extension/` | BundleManager 扩展接口 | 扩展能力接口 |
| `bundlemgr_graphics/` | 图形相关包接口 | 图形能力接口 |

**证据来源**: `interfaces/inner_api/`

### 应用开发套件 (kits/)

**用途**: 供第三方应用开发者使用的 API。

| 目录 | 类型 | 说明 |
|------|------|------|
| `js/` | N-API | JavaScript/TypeScript API（14 个模块） |
| `native/` | NDK | C/C++ 原生接口 |
| `ani/` | ANI | ArkTS Native Interface |
| `cj/` | CJ | Cangjie 语言绑定 |

#### JS N-API 模块详解

| 目录 | JS 命名空间 | 导出方法数 | 主要功能 |
|------|-------------|------------|----------|
| `bundle_manager/` | `bundle.bundleManager` | 98 | **主包管理器 API**（推荐使用） |
| `bundlemgr/` | `bundle` | 24 | 传统包管理器 API（兼容） |
| `installer/` | `bundle.installer` | 13 | 安装器操作 |
| `launcher_bundle_manager/` | `bundle.launcherBundleManager` | 8 | 启动器包管理 |
| `launchermgr/` | `bundle.innerBundleManager` | 5 | 内部启动器服务 |
| `app_control/` | `bundle.appControl` | 16 | 应用控制/处置规则 |
| `default_app/` | `bundle.defaultAppManager` | 10 | 默认应用管理 |
| `overlay/` | `bundle.overlay` | 6 | 叠加包管理 |
| `free_install/` | `bundle.freeInstall` | 6 | 自由安装/按需安装 |
| `bundle_resource/` | `bundle.bundleResourceManager` | 8 | 包资源管理 |
| `bundle_monitor/` | `bundle.bundleMonitor` | 2 | 包变更监控 |
| `shortcut_manager/` | `bundle.shortcutManager` | 8 | 快捷方式管理 |
| `package/` | `package` | 1 | 包工具 |
| `zip/` | `zlib` | 15+ | ZIP 压缩工具 |
| `common/` | - | - | 公共工具（错误处理、参数解析等） |

**证据来源**: `interfaces/kits/js/` 各模块的 `native_module.cpp`

---

## 服务层 (services/bundlemgr/)

### 核心服务文件

| 文件 | 职责 |
|------|------|
| `bundle_mgr_service.cpp` | **主服务入口** - BundleMgrService (SA 401) |
| `bundle_mgr_host_impl.cpp` | IBundleMgr 接口实现 |
| `bundle_data_mgr.cpp` | 包数据中央管理器 |
| `bundle_installer.cpp` | 安装编排 |
| `bundle_installer_host.cpp` | IBundleInstaller 实现 |
| `base_bundle_installer.cpp` | 基础安装器（通用逻辑） |
| `bundle_install_checker.cpp` | 安装前校验 |
| `bundle_parser.cpp` | HAP 包解析 |
| `bundle_profile.cpp` | 配置文件处理 |
| `bundle_permission_mgr.cpp` | 权限管理 |
| `inner_bundle_info.cpp` | 内部包表示 |

**证据来源**: `services/bundlemgr/src/`

### 功能子模块

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `aging/` | 包老化/资源清理 | `bundle_aging_mgr.cpp` |
| `aot/` | AOT 编译管理 | `aot_executor.cpp` |
| `app_control/` | 应用跳转拦截 | 应用运行控制规则 |
| `app_provision_info/` |  provisioning 配置文件 | Provision 信息解析 |
| `app_service_fwk/` | 应用服务框架 | 服务框架安装 |
| `bms_extension/` | BMS 扩展客户端 | 扩展客户端实现 |
| `bundle_backup/` | 备份/恢复功能 | BackupManager |
| `bundle_resource/` | 资源管理 | ResourceHelpers |
| `bundlemgr_ext/` | BundleManager 扩展 | 扩展框架 |
| `clone/` | 应用克隆支持 | CloneInstaller |
| `default_app/` | 默认应用管理 | DefaultAppResolver |
| `distributed_manager/` | 分布式包管理 | 分布式数据同步 |
| `driver/` | 驱动安装 | DriverInstallExt |
| `exception/` | 异常处理 | ExceptionHandlers |
| `extend_resource/` | 扩展资源 | ResourceExtensions |
| `first_install_data_mgr/` | 首次安装数据 | 首次安装跟踪 |
| `free_install/` | 自由安装（按需） | `bundle_connect_ability_mgr.cpp` |
| `idle_condition_mgr/` | 空闲条件处理 | 空闲状态管理 |
| `installd/` | Installd 客户端（IPC 到 SA 511） | `installd_client.cpp` |
| `ipc/` | IPC 参数结构 | IPC 参数类 |
| `navigation/` | 导航支持 | 导航处理 |
| `on_demand_install/` | 按需安装 | 动态安装 |
| `overlay/` | 叠加安装 | `bundle_overlay_manager.cpp` |
| `plugin/` | 插件支持 | PluginInstaller |
| `quick_fix/` | 快速修复（补丁） | `quick_fix_mgr.cpp` |
| `rdb/` | 关系数据库封装 | `rdb_data_manager.cpp` |
| `rpcid_decode/` | RPC ID 解码 | RPC ID 解析器 |
| `sandbox_app/` | 沙箱应用支持 | `bundle_sandbox_installer.cpp` |
| `shared/` | 共享包管理 | SharedBundleHandling |
| `uninstall_data_mgr/` | 卸载数据清理 | CleanupTracking |
| `user_auth/` | 用户认证 | Auth 集成 |
| `utd/` | 统一类型描述符 | UTD 管理 |
| `verify/` | 验证（签名） | 代码签名验证 |

---

## SA 配置文件 (sa_profile/)

| 文件 | SA ID | 库 | 职责 |
|------|-------|-----|------|
| `401.json` | 401 | libbms.z.so | BundleMgrService - 包管理主服务 |
| `511.json` | 511 | libinstalls.z.so | InstalldService - 特权文件操作 |

**证据来源**: `sa_profile/401.json:5-10`, `sa_profile/511.json:5-10`

---

## 配置文件 (etc/)

| 文件 | 职责 |
|------|------|
| `bms.para` | Bundle Manager 系统参数 |
| `bms.para.dac` | DAC（自主访问控制）参数 |

---

## 公共组件 (common/)

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `log/` | 日志框架（HiLog 封装） | `app_log_wrapper.h` |
| `utils/` | 公共工具函数 | `bundle_file_util.h`, `bundle_memory_guard.h` |

**证据来源**: `common/`

---

## 关键文件清单

### 架构文档

| 文件 | 说明 |
|------|------|
| `README_zh.md` | 项目概览（中文） |
| `CLAUDE.md` | 详细开发者指南 |
| `figures/appexecfwk.png` | 架构图 |
| `appexecfwk.gni` | 构建配置（feature flags） |
| `bundle.json` | 组件元数据和依赖 |
| `hisysevent.yaml` | HiSysEvent 事件定义 |
| `bundle_hisysevent.yaml` | Bundle 特定事件定义 |

---

## 模块依赖方向

```
                    ┌─────────────────┐
                    │  BundleMgrService │ (SA 401)
                    │  bundle_mgr_service.cpp
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  BundleDataMgr  │ │  BundleInstaller │ │  BundlePermission │
│ bundle_data_mgr │ │ bundle_installer │ │     Mgr         │
└─────────────────┘ └─────────────────┘ └─────────────────┘
         │                   │                   │
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│     RDB         │ │  InstalldClient │ │  AccessToken    │
│ (持久化存储)     │ │  (IPC to 511)   │ │  (权限校验)      │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

---

## 延伸阅读

- [架构说明](02_Architecture.md)
- [N-API 参考](03_N-API_Reference.md)
- [GN 构建](05_GN_Build.md)
