# 目录结构与模块职责

## 目的

本文档详细说明 bundle_framework_lite 的目录结构，帮助开发者快速定位代码和理解模块职责。

## 顶层目录结构

**证据**: 根目录 `README_zh.md:28-40`

```
/foundation/bundlemanager/bundle_framework_lite
├── frameworks/bundle_lite          # BundleKit 客户端代码
├── interfaces                      # 接口定义
│   ├── kits/bundle_lite           # 对外 API
│   └── inner_api/bundlemgr_lite   # 内部 API
├── services/bundlemgr_lite         # BMS 服务实现
├── utils/bundle_lite               # 工具代码
├── figures/                        # 架构图
├── bundle.json                     # 组件配置
└── bundle_framework_lite.gni       # GN 配置
```

## 详细目录说明

### 1. frameworks/bundle_lite - 客户端框架

**职责**: BundleKit 与 Bundle Manager Service 通信的客户端代码

```
frameworks/bundle_lite/
├── BUILD.gn
├── include/                        # 头文件
│   ├── ability_info_utils.h       # AbilityInfo 工具
│   ├── bundle_callback.h          # 回调机制
│   ├── bundle_callback_utils.h    # 回调工具
│   ├── bundle_info_utils.h        # BundleInfo 工具
│   ├── bundle_self_callback.h     # 自回调
│   ├── bundlems_slite_client.h    # Slite 客户端
│   ├── convert_utils.h            # JSON 转换
│   ├── element_name_utils.h       # ElementName 工具
│   └── module_info_utils.h        # ModuleInfo 工具
└── src/                           # 实现文件
    ├── ability_info.cpp           # AbilityInfo 操作
    ├── ability_info_utils.cpp     # AbilityInfo 工具实现
    ├── bundle_callback.cpp        # 回调实现
    ├── bundle_callback_utils.cpp  # 回调工具实现
    ├── bundle_info.cpp            # BundleInfo 操作
    ├── bundle_info_utils.cpp      # BundleInfo 工具实现
    ├── bundle_manager.cpp         # BundleManager 主类
    ├── bundle_self_callback.cpp   # 自回调实现
    ├── bundle_status_callback.cpp # 状态回调
    ├── convert_utils.cpp          # JSON 转换实现
    ├── element_name.cpp           # ElementName 操作
    ├── module_info.cpp            # ModuleInfo 操作
    ├── module_info_utils.cpp      # ModuleInfo 工具实现
    ├── token_generate.cpp         # Token 生成
    └── slite/                     # 轻量级设备实现
        ├── bundle_manager.cpp
        ├── bundle_manager_inner.cpp
        └── bundlems_slite_client.cpp
```

**关键类**:
- `BundleManager` (`src/bundle_manager.cpp:1`) - 主管理类
- `BundleInfoUtils` (`include/bundle_info_utils.h:1`) - BundleInfo 工具
- `ConvertUtils` (`include/convert_utils.h:1`) - IPC 序列化

### 2. interfaces/kits/bundle_lite - 对外 API

**职责**: 为开发者提供的公共接口

```
interfaces/kits/bundle_lite/
├── ability_info.h                 # AbilityInfo 结构定义
├── appexecfwk_errors.h            # 错误码定义
├── bundle_info.h                  # BundleInfo 结构定义
├── bundle_manager.h               # C API 接口
├── bundle_status_callback.h       # 状态回调定义
├── element_name.h                 # ElementName 结构定义
├── install_param.h                # 安装参数定义
├── module_info.h                  # ModuleInfo 结构定义
├── js/                            # JS API
│   └── builtin/
│       ├── BUILD.gn
│       ├── include/
│       │   └── capability_module.h
│       └── src/
│           └── capability_module.cpp   # JS API 实现
└── slite/                         # 轻量级接口
    └── bundle_manager.h
```

**关键接口** (`bundle_manager.h`):
```cpp
// 安装/卸载
bool Install(const char *hapPath, const InstallParam *installParam, InstallerCallback installerCallback);
bool Uninstall(const char *bundleName, const InstallParam *installParam, InstallerCallback installerCallback);

// 查询
uint8_t QueryAbilityInfo(const Want *want, AbilityInfo *abilityInfo);
uint8_t GetBundleInfo(const char *bundleName, int32_t flags, BundleInfo *bundleInfo);
uint8_t GetBundleInfos(const int flags, BundleInfo **bundleInfos, int32_t *len);
```

### 3. interfaces/inner_api/bundlemgr_lite - 内部 API

**职责**: BundleKit 核心实现和 BMS 为其他子系统提供的接口

```
interfaces/inner_api/bundlemgr_lite/
├── bundle_daemon_interface.h      # Bundle Daemon IPC 接口
├── bundle_inner_interface.h       # 内部服务 IPC 接口
├── bundle_service_interface.h     # 服务接口定义
└── slite/                         # 轻量级内部接口
    ├── bundle_install_msg.h
    └── bundle_manager_inner.h
```

**关键定义**:
- `BmsCmd` 枚举 (`bundle_inner_interface.h:35-52`) - IPC 命令码
- `BdsCmd` 枚举 (`bundle_daemon_interface.h:28-40`) - Daemon 命令码

### 4. services/bundlemgr_lite - 服务实现

**职责**: Bundle Manager Service 的核心实现

```
services/bundlemgr_lite/
├── BUILD.gn
├── include/                       # 头文件
│   ├── bundle_common.h            # 公共定义
│   ├── bundle_daemon_client.h     # Daemon 客户端
│   ├── bundle_extractor.h         # HAP 提取器
│   ├── bundle_info_creator.h      # BundleInfo 创建
│   ├── bundle_inner_feature.h     # 内部特性
│   ├── bundle_installer.h         # 安装器
│   ├── bundle_manager_service.h   # 服务管理
│   ├── bundle_map.h               # Bundle 信息存储
│   ├── bundle_message_id.h        # 消息 ID
│   ├── bundle_ms_feature.h        # MS 特性
│   ├── bundle_ms_host.h           # 服务宿主
│   ├── bundle_parser.h            # HAP 解析器
│   ├── bundle_res_transform.h     # 资源转换
│   ├── bundle_util.h              # 工具函数
│   ├── extractor_util.h           # 提取工具
│   ├── gt_bundle_extractor.h      # GT 提取器
│   ├── gt_bundle_installer.h      # GT 安装器
│   ├── gt_bundle_manager_service.h # GT 服务管理
│   ├── gt_bundle_parser.h         # GT 解析器
│   ├── gt_extractor_util.h        # GT 提取工具
│   ├── hap_sign_verify.h          # 签名验证
│   └── zip_file.h                 # ZIP 处理
├── src/                           # 实现文件
│   ├── bundle_daemon_client.cpp   # Daemon 客户端实现
│   ├── bundle_extractor.cpp       # 提取器实现
│   ├── bundle_info_creator.cpp    # BundleInfo 创建实现
│   ├── bundle_inner_feature.cpp   # 内部特性实现
│   ├── bundle_installer.cpp       # 安装器实现
│   ├── bundle_manager_service.cpp # 服务管理实现
│   ├── bundle_map.cpp             # Bundle 存储实现
│   ├── bundle_ms_feature.cpp      # MS 特性实现
│   ├── bundle_ms_host.cpp         # 服务宿主实现
│   ├── bundle_parser.cpp          # 解析器实现
│   ├── bundle_res_transform.cpp   # 资源转换实现
│   ├── bundle_util.cpp            # 工具函数实现
│   ├── extractor_util.cpp         # 提取工具实现
│   ├── gt_bundle_extractor.cpp    # GT 提取器实现
│   ├── gt_bundle_installer.cpp    # GT 安装器实现
│   ├── gt_bundle_manager_service.cpp # GT 服务管理实现
│   ├── gt_bundle_parser.cpp       # GT 解析器实现
│   ├── gt_extractor_util.cpp      # GT 提取工具实现
│   ├── gt_bundle_parser.cpp       # GT 解析器实现
│   ├── hap_sign_verify.cpp        # 签名验证实现
│   └── zip_file.cpp               # ZIP 处理实现
├── bundle_daemon/                 # Bundle Daemon
│   ├── BUILD.gn
│   ├── include/
│   │   ├── bundle_daemon.h
│   │   ├── bundle_daemon_handler.h
│   │   ├── bundle_daemon_log.h
│   │   ├── bundle_file_utils.h
│   │   └── bundlems_client.h
│   └── src/
│       ├── bundle_daemon.cpp
│       ├── bundle_daemon_handler.cpp
│       ├── bundle_file_utils.cpp
│       ├── bundlems_client.cpp
│       └── main.cpp
└── tools/                         # bm 工具
    ├── BUILD.gn
    ├── include/
    │   └── command_parser.h
    └── src/
        ├── command_parser.cpp
        └── main.cpp
```

**核心类**:
- `ManagerService` (`include/bundle_manager_service.h:33`) - 服务管理单例
- `BundleInstaller` (`include/bundle_installer.h:36`) - 安装器
- `BundleMap` (`include/bundle_map.h:28`) - Bundle 信息存储
- `BundleDaemonClient` (`include/bundle_daemon_client.h:28`) - Daemon 客户端

### 5. utils/bundle_lite - 工具代码

**职责**: 包管理服务实现中使用的工具代码

```
utils/bundle_lite/
├── aafwk_event_error_code.h       # AAFWK 错误码
├── aafwk_event_error_id.h         # AAFWK 错误 ID
├── adapter.h                      # 内存适配器
├── bundle_log.h                   # 日志宏
├── mutex_lock.h                   # 互斥锁
├── nocopyable.h                   # 不可复制基类
├── token_generate.h               # Token 生成
├── uptr.h                         # 智能指针
├── utils.h                        # 通用工具
└── utils_list.h                   # 列表工具
```

**关键工具**:
- `AdapterMalloc/AdapterFree` (`adapter.h:32-36`) - 内存分配适配
- `HILOG_XXX` (`bundle_log.h:1`) - 日志宏
- `Utils::Strdup` (`utils.h:1`) - 字符串复制

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层                                   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              interfaces/kits/bundle_lite                        │
│                    (对外 API)                                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              frameworks/bundle_lite                             │
│                 (客户端框架)                                     │
└───────────────────────────┬─────────────────────────────────────┘
                            │ IPC
┌───────────────────────────▼─────────────────────────────────────┐
│              services/bundlemgr_lite                            │
│              (Bundle Manager Service)                           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │bundle_ms_│ │ bundle_  │ │ bundle_  │ │ bundle_  │           │
│  │  feature │ │ installer│ │  parser  │ │daemon_   │           │
│  └──────────┘ └──────────┘ └──────────┘ │  client  │           │
│                                         └────┬─────┘           │
└────────────────────────────────────────────────┼────────────────┘
                                                 │ IPC
┌────────────────────────────────────────────────▼────────────────┐
│              services/bundlemgr_lite/bundle_daemon              │
│                    (Bundle Daemon)                               │
└─────────────────────────────────────────────────────────────────┘
```

## 代码统计（非测试代码）

| 目录 | 头文件(.h) | 源文件(.cpp) | 代码行数（估算） |
|------|-----------|-------------|-----------------|
| frameworks/bundle_lite | 8 | 16 | ~3000 |
| interfaces/kits/bundle_lite | 9 | 1 | ~800 |
| interfaces/inner_api/bundlemgr_lite | 5 | 0 | ~300 |
| services/bundlemgr_lite | 26 | 24 | ~15000 |
| utils/bundle_lite | 10 | 0 | ~500 |
| **总计** | **58** | **41** | **~19600** |

---

**相关链接**:
- [项目概览](00_Overview.md)
- [架构说明](02_Architecture.md)
- [内部 API](04_Internal_API.md)
