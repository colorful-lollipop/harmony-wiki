# 目录结构与模块职责

---

## 目的

本文档说明 DBMS 项目的目录结构、模块职责和主要文件组织。

---

## 适用范围

- ✅ 顶层目录结构
- ✅ 各模块职责划分
- ✅ 核心文件清单（排除测试目录）
- ❌ 详细的实现逻辑（请参考架构文档）

---

## 顶层目录结构

```
distributed_bundle_framework/
├── interfaces/                    # 对外接口层
│   ├── inner_api/              # 内部 API（Proxy、接口定义）
│   └── kits/                   # 应用层接口
│       ├── ani/                # ArkTS 新接口
│       └── js/                 # JavaScript N-API 接口
├── services/dbms/                # DBMS 服务实现
│   ├── include/                # 服务头文件
│   ├── src/                    # 服务实现
│   └── sa_profile/             # SA 配置
├── wiki/                        # Wiki 文档
│   ├── README.md
│   ├── SUMMARY.md
│   └── ...
├── BUILD.gn                      # 根构建文件
├── bundle.json                   # 组件元数据
├── dbms.gni                     # GN 配置文件
├── README_zh.md                 # 项目说明（中文）
└── LICENSE                       # 许可证
```

---

## 模块职责

### 1. interfaces/kits/js - JavaScript 应用接口

**职责**：为 JavaScript 应用提供 N-API 接口，支持跨设备查询组件信息。

**子模块**：

#### 1.1 distributedBundle（旧版）
- **路径**: `interfaces/kits/js/distributedBundle/`
- **输出库**: `libdistributedbundlemanager.z.so`
- **安装路径**: `system/lib/module/bundle/`
- **导出模块名**: `distributedBundle`
- **导出方法**:
  - `getRemoteAbilityInfo(elementName, locale, callback?)` - 单个查询
  - `getRemoteAbilityInfos(elementNames, locale, callback?)` - 批量查询

**核心文件**:
- `native_module.cpp:49-64` - 模块注册
- `distributed_bundle.cpp:230-275` - 方法实现
- `distributed_helper.cpp` - 业务逻辑

**证据**: `interfaces/kits/js/distributedBundle/native_module.cpp:54`

#### 1.2 distributebundlemgr（新版）
- **路径**: `interfaces/kits/js/distributebundlemgr/`
- **输出库**: `libdistributedbundle.z.so`
- **安装路径**: `system/lib/module/`
- **导出模块名**: `bundle.distributedBundleManager`
- **导出方法**:
  - `getRemoteAbilityInfo(elementName, locale, callback?)` - 支持单/批量查询

**核心文件**:
- `native_module.cpp:49-56` - 模块注册
- `distributed_bundle_mgr.cpp:324-494` - 方法实现

**证据**: `interfaces/kits/js/distributebundlemgr/native_module.cpp:53`

### 2. interfaces/kits/ani - ArkTS 新接口

**职责**：为 ArkTS 应用提供编译时接口。

**子模块**：

#### 2.1 distributed_bundle_manager
- **路径**: `interfaces/kits/ani/distributed_bundle_manager/`
- **输出库**: `libani_distributed_bundle_manager.z.so`
- **输出字节码**: `distributed_bundle_manager.abc`, `remote_ability_info.abc`
- **安装路径**: `system/lib/`, `system/framework/`

**核心文件**:
- `ani_distributed_bundle_manager.cpp` - ANI 接口实现
- `ani_distributed_bundle_manager_common.cpp` - 通用实现
- `ets/` - ArkTS 源码

**证据**: `interfaces/kits/ani/distributed_bundle_manager/BUILD.gn:17-63`

### 3. interfaces/inner_api - 内部 API 框架

**职责**：提供 IPC 代理和服务接口定义，内部模块间通信。

**子模块**：

#### 3.1 dbms_fwk
- **路径**: `interfaces/inner_api/`
- **输出库**: `libdbms_fwk.z.so`
- **安装路径**: `system/lib/`
- **innerapi_tags**: ["platformsdk"]

**核心文件**:
- `include/distributed_bms_interface.h:31-42` - IDistributedBms 接口定义
- `include/distributed_bms_proxy.h:27-32` - Proxy 类定义
- `include/distributed_bms_acl_info.h:16-23` - ACL 信息结构
- `include/distributed_bundle_ipc_interface_code.h` - IPC 命令码定义
- `src/distributed_bms_proxy.cpp` - Proxy 实现

**证据**: `interfaces/inner_api/BUILD.gn:21-64`

### 4. services/dbms - DBMS 系统服务

**职责**：实现分布式包管理系统服务的核心逻辑。

**子模块**：

#### 4.1 libdbms（系统服务）
- **路径**: `services/dbms/`
- **输出库**: `libdbms.z.so`
- **SA ID**: 402 (DISTRIBUTED_BUNDLE_MGR_SERVICE_SYS_ABILITY_ID)
- **进程名**: d-bms
- **shlib_type**: "sa"

**核心文件**:

| 文件 | 行数 | 说明 |
|------|------|------|
| include/distributed_bms.h | 171 | 服务类定义 |
| include/distributed_bms_host.h | 47 | IPC Stub 类定义 |
| include/dbms_device_manager.h | 42 | 设备管理器接口 |
| include/distributed_data_storage.h | TODO | 数据存储接口 |
| src/distributed_bms.cpp | 673 | 服务实现（权限验证、业务逻辑）|
| src/distributed_bms_host.cpp | TODO | IPC 处理实现 |
| src/dbms_device_manager.cpp | 134 | 设备管理器实现 |
| src/distributed_data_storage.cpp | TODO | 数据存储实现 |
| src/account_manager_helper.cpp | TODO | 账号助手 |

**证据**: `services/dbms/BUILD.gn:27-105`

#### 4.2 sa_profile - SA 配置
- **路径**: `services/dbms/sa_profile/`
- **输出**: SA 配置文件和启动脚本
- **SA ID**: 402

**核心文件**:
- `402.json:4-28` - SA 能力配置
- `distributedbms.cfg:11-31` - 启动配置

**证据**: `services/dbms/sa_profile/402.json:5`, `services/dbms/sa_profile/distributedbms.cfg:11`

---

## 文件统计（排除测试）

### 源文件统计

| 类型 | 数量 |
|------|------|
| C/C++ 源文件 (.cpp, .cc, .c) | 11 |
| 头文件 (.h) | 9 |
| 构建文件 (BUILD.gn, .gni) | 8 |

**完整清单**:
```
interfaces/kits/js/distributedBundle/native_module.cpp
interfaces/kits/js/distributedBundle/distributed_bundle.cpp
interfaces/kits/js/distributedBundle/distributed_bundle_unsupported.cpp
interfaces/kits/js/distributedBundle/distributed_helper.cpp
interfaces/kits/js/distributebundlemgr/native_module.cpp
interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp
interfaces/kits/ani/distributed_bundle_manager/ani_distributed_bundle_manager.cpp
interfaces/kits/ani/distributed_bundle_manager/ani_distributed_bundle_manager_common.cpp
interfaces/kits/ani/distributed_bundle_manager/ani_distributed_bundle_manager_unsupported.cpp
interfaces/inner_api/src/distributed_bms_proxy.cpp
interfaces/inner_api/src/distributed_bms_acl_info.cpp
services/dbms/src/distributed_bms.cpp
services/dbms/src/distributed_bms_host.cpp
services/dbms/src/dbms_device_manager.cpp
services/dbms/src/distributed_data_storage.cpp
services/dbms/src/account_manager_helper.cpp
services/dbms/src/event_report.cpp
services/dbms/src/image_compress.cpp
```

---

## 模块依赖关系

### 依赖方向（避免循环）

```
JS/ANI 应用层
    │
    ▼
N-API 绑定层
    │
    ▼
inner_api (dbms_fwk) ←──── IPC ───→ services/dbms
    │                               │
    │                               ▼
    └─────────→ 外部系统服务
                    ┌─────────────┬─────────────┐
                    │             │             │
              Bundle Manager      Device Manager   其他依赖
              SA = 401            SA
```

### 模块职责边界

| 模块 | 职责 | 边界 |
|------|------|------|
| JS/ANI | 参数解析、类型转换、异步调度 | 不涉及业务逻辑，只做 N-API 绑定 |
| inner_api | IPC 代理、接口定义 | 不实现业务逻辑，只做 IPC 转发 |
| services/dbms | 业务逻辑、权限验证、数据存储 | 核心业务实现 |
| 外部 SA | Bundle 查询、设备管理 | 跨系统服务 |

---

## 配置文件

### 根目录配置

| 文件 | 用途 |
|------|------|
| BUILD.gn | 根构建入口，定义顶层 group targets |
| bundle.json | 组件元数据（依赖、系统能力、资源占用）|
| dbms.gni | GN 路径和 feature flags 定义 |
| README_zh.md | 项目中文说明 |

### 模块配置

| 目录 | 配置文件 | 用途 |
|------|----------|------|
| services/dbms | sa_profile/402.json, sa_profile/distributedbms.cfg | SA 配置和启动脚本 |
| interfaces/ | 各子目录的 BUILD.gn | 模块构建配置 |

---

## 关键目录说明

### interfaces/kits
存放应用层接口，分为三种类型：
1. **js/** - JavaScript N-API 接口（推荐使用）
2. **ani/** - ArkTS 编译时接口（新特性）
3. **js/子模块** - 支持旧版和新版 API

### interfaces/inner_api
存放内部 API 框架，包括：
1. **Proxy 类** - 客户端 IPC 调用
2. **接口定义** - IDistributedBms
3. **数据结构** - Parcelable、ACL 信息

### services/dbms
存放系统服务实现，包括：
1. **服务主类** - DistributedBms
2. **IPC Stub** - DistributedBmsHost
3. **设备管理** - DbmsDeviceManager
4. **数据存储** - DistributedDataStorage
5. **SA 配置** - sa_profile/

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| JS 模块注册 | interfaces/kits/js/distributedBundle/native_module.cpp:49-64 |
| SA ID = 402 | services/dbms/sa_profile/402.json:5 |
| Proxy 类定义 | interfaces/inner_api/include/distributed_bms_proxy.h:27-32 |
| 服务主类 | services/dbms/include/distributed_bms.h:33-35 |
