# 目录结构与代码地图 - Bundle Framework Lite

## 目录

- [顶层目录结构](#顶层目录结构)
- [核心文件定位](#核心文件定位)
- [代码导航图](#代码导航图)

---

## 顶层目录结构

```
/foundation/bundlemanager/bundle_framework_lite
├── frameworks/                    # BundleKit 客户端实现
│   └── bundle_lite/
│       ├── include/            # 内部头文件
│       └── src/                # 实现文件
│           └── slite/          # 轻量级设备特定实现
├── interfaces/                     # 接口定义
│   ├── kits/bundle_lite/          # 对外 API（C/C++ 头文件）
│   │   ├── js/builtin/           # JS API 实现（JSI 模块）
│   │   └── slite/                # 轻量级设备接口
│   └── inner_api/bundlemgr_lite/ # 内部 API（IPC 接口定义）
│       └── slite/                # 轻量级内部接口
├── services/bundlemgr_lite/          # BMS 服务实现
│   ├── bundle_daemon/            # Bundle Daemon（独立进程）
│   ├── tools/                    # bm 命令行工具
│   ├── include/                  # BMS 头文件
│   └── src/                      # BMS 实现文件
└── utils/bundle_lite/             # 工具库
```

**证据**: `README.md:26-39`

---

## 目录职责说明

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| **frameworks/bundle_lite/** | BundleKit 客户端实现，提供对外 API | `bundle_manager.cpp`, `bundle_info.cpp` |
| **interfaces/kits/bundle_lite/** | 对外 API 头文件定义 | `bundle_manager.h`, `ability_info.h`, `bundle_info.h` |
| **interfaces/kits/bundle_lite/js/** | JavaScript API 实现（JSI 模块） | `capability_module.cpp` |
| **interfaces/inner_api/bundlemgr_lite/** | 内部 IPC 接口定义 | `bundle_service_interface.h`, `bundle_inner_interface.h` |
| **services/bundlemgr_lite/** | BMS 服务核心实现 | `bundle_manager_service.cpp`, `bundle_installer.cpp` |
| **services/bundlemgr_lite/bundle_daemon/** | Bundle Daemon（高权限文件操作进程）| `bundle_daemon.cpp`, `bundle_daemon_handler.cpp` |
| **services/bundlemgr_lite/tools/** | bm 命令行工具 | `main.cpp`, `command_parser.cpp` |
| **utils/bundle_lite/** | 通用工具函数 | `bundle_util.cpp` |

---

## 核心文件定位

### 1. 安装/卸载相关

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| **安装 HAP** | `services/bundlemgr_lite/src/bundle_installer.cpp` | `BundleInstaller::Install()` |
| **卸载应用** | `services/bundlemgr_lite/src/bundle_installer.cpp` | `BundleInstaller::Uninstall()` |
| **HAP 解析** | `services/bundlemgr_lite/src/bundle_parser.cpp` | `BundleParser::Parse()` |
| **HAP 提取** | `services/bundlemgr_lite/src/bundle_extractor.cpp` | `BundleExtractor::Extract()` |
| **签名验证** | `services/bundlemgr_lite/src/hap_sign_verify.cpp` | `HapSignVerify::VerifySignature()` |

### 2. 信息查询相关

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| **查询 BundleInfo** | `frameworks/bundle_lite/src/bundle_manager.cpp` | `GetBundleInfo()` |
| **查询 AbilityInfo** | `frameworks/bundle_lite/src/bundle_manager.cpp` | `QueryAbilityInfo()` |
| **Bundle 信息管理** | `services/bundlemgr_lite/src/bundle_map.cpp` | `BundleMap` 类 |

### 3. IPC 通信相关

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| **BMS 服务注册** | `services/bundlemgr_lite/src/bundle_ms_host.cpp` | `BundleMsHost::Initialize()` |
| **BMS Feature 注册** | `services/bundlemgr_lite/src/bundle_ms_feature.cpp` | `BundleMsFeature::OnInitialize()` |
| **IPC 消息处理** | `services/bundlemgr_lite/src/bundle_ms_feature.cpp` | `BundleMsFeature::Invoke()` |
| **Bundle Daemon IPC** | `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon.cpp` | `BundleDaemon::Invoke()` |

### 4. 安全相关

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| **权限匹配** | `services/bundlemgr_lite/src/bundle_installer.cpp` | `CheckProvisionInfoIsValid()` |
| **路径验证** | `services/bundlemgr_lite/src/bundle_util.cpp` | `CheckRealPath()` |
| **Daemon 权限检查** | `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon.cpp` | `CheckPermission()` |

### 5. JS 绑定相关

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| **JSI 模块注册** | `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp` | `InitCapabilityModule()` |
| **HasCapability API** | `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp` | `CapabilityModule::HasCapability()` |

### 6. 命令行工具相关

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| **bm 主入口** | `services/bundlemgr_lite/tools/src/main.cpp` | `main()` |
| **命令解析** | `services/bundlemgr_lite/tools/src/command_parser.cpp` | `CommandParser::HandleCommands()` |

---

## 代码导航图

### 快速定位指南

#### 我想修改安装逻辑

```
1. 从 HAP 文件解析 → services/bundlemgr_lite/src/bundle_parser.cpp
2. 验证签名 → services/bundlemgr_lite/src/hap_sign_verify.cpp
3. 检查权限 → services/bundlemgr_lite/src/bundle_installer.cpp:264-343
4. 提取文件 → services/bundlemgr_lite/src/bundle_extractor.cpp
5. 创建目录 → services/bundlemgr_lite/bundle_daemon/src/bundle_daemon_handler.cpp
```

#### 我想添加新的查询 API

```
1. 定义 API 头文件 → interfaces/kits/bundle_lite/bundle_manager.h
2. 实现 C API → frameworks/bundle_lite/src/bundle_manager.cpp
3. 添加 IPC 命令 → interfaces/inner_api/bundlemgr_lite/bundle_inner_interface.h
4. 实现 IPC 处理器 → services/bundlemgr_lite/src/bundle_ms_feature.cpp
5. 更新消息分发表 → BundleMsFeature::BundleMsInvokeFuc[]
```

#### 我想修改 bm 工具

```
1. 主入口 → services/bundlemgr_lite/tools/src/main.cpp
2. 命令解析 → services/bundlemgr_lite/tools/src/command_parser.cpp
3. 参数验证 → command_parser.cpp:167-208
4. IPC 调用 → command_parser.cpp (通过 SAMGR)
```

#### 我想添加新的 JS API

```
1. 实现 JSI 函数 → interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp
2. 注册到模块 → InitCapabilityModule() (JSI::SetModuleAPI)
3. 调用 C API → 调用 bundle_manager.h 中的函数
4. 更新 BUILD.gn → interfaces/kits/bundle_lite/js/builtin/BUILD.gn
```

---

## 关键数据结构位置

| 数据结构 | 定义位置 | 说明 |
|---------|----------|------|
| **BundleInfo** | `interfaces/kits/bundle_lite/bundle_info.h` | 应用信息（名称、版本、图标等）|
| **AbilityInfo** | `interfaces/kits/bundle_lite/ability_info.h` | Ability 信息（名称、类型、权限等）|
| **ModuleInfo** | `interfaces/kits/bundle_lite/module_info.h` | 模块信息（名称、类型、元数据等）|
| **InstallParam** | `interfaces/kits/bundle_lite/install_param.h` | 安装参数（安装位置、是否保留数据等）|
| **SignatureInfo** | `services/bundlemgr_lite/include/hap_sign_verify.h` | 签名信息（appId、provisionBundleName 等）|

---

## 关键常量定义位置

| 常量 | 定义位置 | 值 |
|-------|----------|------|
| **BMS_SERVICE** | `interfaces/inner_api/bundlemgr_lite/bundle_service_interface.h:37` | `"bundlems"` |
| **BMS_FEATURE** | `interfaces/inner_api/bundlemgr_lite/bundle_service_interface.h:38` | `"BmsFeature"` |
| **BMS_INNER_FEATURE** | `interfaces/inner_api/bundlemgr_lite/bundle_service_interface.h:40` | `"BmsInnerFeature"` |
| **BDS_SERVICE** | `interfaces/inner_api/bundlemgr_lite/bundle_daemon_interface.h` | `"bundle_daemon"` |
| **BMS_UID** | `services/bundlemgr_lite/src/bundle_daemon_handler.cpp` | `7` |

---

## 编译输出位置

### bm 工具

**输出路径**: `out/<board>/dev_tools/bm`
**BUILD.gn**: `services/bundlemgr_lite/tools/BUILD.gn:70`

### 共享库

**libappexecfwk_kits_lite.so**: BundleKit 客户端库
**libappexecfwk_services_lite.so**: BMS 服务库（LiteOS-A）
**libbundlems.a**: BMS 服务库（LiteOS-M，静态库）

---

## 相关文档

- [项目概览](01_Overview.md) - 项目定位和功能
- [架构与数据流](03_Architecture.md) - 组件协作流程
- [对外接口文档](04_Interface.md) - API 使用方法

---

**最后更新**: 2026-02-07
