# 编译产物

## 目的

本文档说明 bundle_framework_lite 的编译产物，包括输出文件、安装路径和运行时加载关系。

## 产物清单

### 1. 共享库 (.so)

| 产物名 | 来源 Target | 说明 |
|--------|------------|------|
| `libbundle.so` | `frameworks/bundle_lite:bundle` | BundleKit 客户端库 |
| `libbundlems.so` | `services/bundlemgr_lite:bundlems` | BMS 服务库 |
| `libcapability_api.so` | `js/builtin:capability_api` | JS API 库 |

### 2. 静态库 (.a)

| 产物名 | 来源 Target | 说明 |
|--------|------------|------|
| `libbundle.a` | `frameworks/bundle_lite:bundle` | BundleKit 静态库 (LiteOS-M) |
| `libbundlems.a` | `services/bundlemgr_lite:bundlems` | BMS 静态库 (LiteOS-M) |

### 3. 可执行文件

| 产物名 | 来源 Target | 说明 |
|--------|------------|------|
| `bundle_daemon` | `bundle_daemon:bundle_daemon` | Bundle 守护进程 |
| `bm` | `tools:bm` | 包管理命令行工具 |

### 4. 配置文件

| 产物名 | 来源 | 说明 |
|--------|------|------|
| `bundle.json` | 根目录 | 组件配置 |
| `bundle_framework_lite.gni` | 根目录 | GN 配置 |

## 输出路径

### 标准输出目录结构

```
out/{board}/{product}/
├── dev_tools/
│   └── bin/
│       └── bm                    # bm 工具
├── lib/
│   ├── libbundle.so              # BundleKit 库
│   ├── libbundlems.so            # BMS 服务库
│   └── libcapability_api.so      # JS API 库
├── bin/
│   └── bundle_daemon             # 守护进程
└── obj/
    └── foundation/bundlemanager/bundle_framework_lite/
        ├── frameworks/bundle_lite/
        ├── services/bundlemgr_lite/
        └── ...
```

### 具体路径示例 (hispark_taurus)

**证据**: `README_zh.md:51`

```
out/hispark_taurus/ipcamera_hispark_taurus/
├── dev_tools/bin/bm              # bm 工具
├── lib/libbundle.so              # BundleKit
├── lib/libbundlems.so            # BMS
└── bin/bundle_daemon             # 守护进程
```

## 安装路径

### 运行时安装路径

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `libbundle.so` | `/lib/` 或 `/system/lib/` | 系统库目录 |
| `libbundlems.so` | `/lib/` 或 `/system/lib/` | 系统库目录 |
| `bundle_daemon` | `/bin/` 或 `/system/bin/` | 系统可执行文件目录 |
| `bm` | `/bin/` 或开发工具目录 | 命令行工具 |

### 应用安装目录

**证据**: `services/bundlemgr_lite/include/bundle_common.h:62-71`

```cpp
const char INSTALL_PATH[] = "/data/app";                    // 三方应用安装路径
const char DATA_PATH[] = "/data/data";                      // 应用数据路径
const char SYSTEM_BUNDLE_PATH[] = "/system/app";            // 系统应用路径
const char THIRD_SYSTEM_BUNDLE_PATH[] = "/system/vendor";   // 三方系统应用路径
const char EXTEANAL_INSTALL_PATH[] = "/sdcard/app";         // 外部存储安装路径
const char EXTEANAL_DATA_PATH[] = "/sdcard/data";           // 外部存储数据路径
const char JSON_PATH[] = "/data/accounts/account_0/applications/";  // JSON 配置路径
const char SHARED_LIB_PATH[] = "/data/shared_lib";          // 共享库路径
```

## 运行时加载关系

### 进程加载图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         进程加载关系                                     │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                        foundation 进程                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      启动时加载                                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │  │  libsamgr.  │  │  libbundle. │  │      libbundlems.       │  │   │
│  │  │     so      │──│     so      │──│          so             │  │   │
│  │  │ (SAMGR 服务)│  │(BundleKit)  │  │    (BMS 服务实现)        │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │   │
│  │         │                │                      │               │   │
│  │         │                │                      ▼               │   │
│  │         │                │               ┌─────────────┐        │   │
│  │         │                │               │ bundle_ms_  │        │   │
│  │         │                │               │    host     │        │   │
│  │         │                │               │ (服务注册)   │        │   │
│  │         │                │               └─────────────┘        │   │
│  └─────────┼────────────────┼──────────────────────────────────────┘   │
│            │                │                                          │
│            │                ▼                                          │
│            │         ┌─────────────┐                                   │
│            │         │ 应用调用 API │                                   │
│            │         │(libbundle.so)│                                  │
│            │         └─────────────┘                                   │
│            │                                                           │
│            ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     运行时 IPC 调用                               │   │
│  │  ┌─────────────┐      IPC       ┌─────────────────────────────┐  │   │
│  │  │   客户端     │ ◄────────────► │   BMS Feature (服务实现)     │  │   │
│  │  │(libbundle.so)│                │  - bundle_ms_feature         │  │   │
│  │  └─────────────┘                │  - bundle_inner_feature      │  │   │
│  │                                 └─────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ IPC (跨进程)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      bundle_daemon 进程                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      启动时加载                                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │  │  libsamgr.  │  │  libhilog.  │  │      bundle_daemon      │  │   │
│  │  │     so      │  │     so      │  │      (可执行文件)        │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │   │
│  │                                                      │           │   │
│  │                      运行时 IPC 调用                  │           │   │
│  │  ┌───────────────────────────────────────────────────▼───────┐  │   │
│  │  │              BundleDaemonHandler (请求处理)                │  │   │
│  │  │  - ExtractHap                                              │  │   │
│  │  │  - CreateDataDirectory                                     │  │   │
│  │  │  - RemoveFile                                              │  │   │
│  │  └───────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### 库依赖关系

#### libbundle.so 依赖

```
libbundle.so
├── libwant.so (aafwk_lite)
├── libhilog_shared.so (hilog_lite)
├── libpms_client.so (permission_lite)
├── libcjson_shared.so (cJSON)
└── libbounds_checking_function.so
```

#### libbundlems.so 依赖

```
libbundlems.so
├── libbundle.so (bundle_framework_lite)
├── libverify.so (appverify_lite)
├── libhilog_shared.so (hilog_lite)
├── libglobal_resmgr.so (resource_management_lite)
├── libsamgr.so (samgr_lite)
├── libcjson_shared.so (cJSON)
└── libzlib_shared.so (zlib)
```

#### bundle_daemon 依赖

```
bundle_daemon
├── libhilog_shared.so (hilog_lite)
├── libsamgr.so (samgr_lite)
└── libzlib_shared.so (zlib)
```

#### bm 依赖

```
bm
├── libbundle.so (bundle_framework_lite)
└── libhilog_shared.so (hilog_lite)
```

## 运行时数据流

### 应用安装时的库调用链

```
应用调用 Install()
    └── libbundle.so:Install()
         └── IPC 到 BMS
              └── libbundlems.so:BundleMsFeature::OnFeatureMessage()
                   └── BundleInstaller::Install()
                        ├── libverify.so:APPVERI_AppVerify()  [签名验证]
                        ├── libbundlems.so:BundleParser::ParseHapProfile()  [解析]
                        └── IPC 到 Daemon
                             └── bundle_daemon:BundleDaemonHandler::ExtractHap()
                                  └── libzlib_shared.so:unzip()  [解压]
```

### 应用查询时的库调用链

```
应用调用 GetBundleInfo()
    └── libbundle.so:GetBundleInfo()
         └── IPC 到 BMS
              └── libbundlems.so:BundleMsFeature::GetBundleInfo()
                   └── BundleMap::GetBundleInfo()  [内存查询]
```

## 文件权限

### 安装后文件权限

**证据**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon_handler.cpp`

| 路径类型 | 所有者 | 权限 | 说明 |
|----------|--------|------|------|
| `/data/app/{bundleName}/` | app_uid:app_gid | 755 | 应用代码目录 |
| `/data/data/{bundleName}/` | app_uid:app_gid | 755 | 应用数据目录 |
| `/system/app/` | root:root | 755 | 系统应用目录（只读） |
| `/data/accounts/account_0/applications/` | system:system | 755 | Bundle 配置目录 |

### UID/GID 分配

**证据**: `services/bundlemgr_lite/include/bundle_common.h:54-60`

| 类型 | UID 范围 | 说明 |
|------|----------|------|
| 系统应用 | 100 - 999 | BASE_SYS_UID 起始 |
| 三方系统应用 | 1000 - 9999 | BASE_SYS_VEN_UID 起始 |
| 三方应用 | 10000+ | BASE_APP_UID 起始 |

## 运行时配置

### 配置文件路径

| 配置 | 路径 | 说明 |
|------|------|------|
| Bundle 列表 | `/data/accounts/account_0/applications/{bundleName}.json` | 每个应用的配置 |
| UID 映射 | `/data/accounts/account_0/applications/uid_gid_map.json` | UID/GID 映射表 |
| 三方系统应用 | `/data/accounts/account_0/applications/third_system_bundle.json` | 三方系统应用列表 |
| 卸载记录 | `/data/accounts/account_0/applications/uninstall_third_system_bundle.json` | 卸载记录 |

### Bundle JSON 格式

**证据**: `services/bundlemgr_lite/src/bundle_util.cpp`

```json
{
  "bundleName": "com.example.app",
  "appId": "com.example.app_xxx",
  "versionCode": 1000000,
  "versionName": "1.0.0",
  "codePath": "/data/app/com.example.app",
  "dataPath": "/data/data/com.example.app",
  "uid": 10000,
  "gid": 10000,
  "isSystemApp": false,
  "moduleInfos": [...],
  "abilityInfos": [...]
}
```

## 内存占用

### ROM 占用

**证据**: `bundle.json:26`

```json
"rom": "300KB"
```

| 组件 | 估算大小 |
|------|----------|
| libbundle.so | ~50KB |
| libbundlems.so | ~150KB |
| bundle_daemon | ~50KB |
| bm | ~30KB |
| 配置文件 | ~20KB |
| **总计** | **~300KB** |

### RAM 占用

**证据**: `bundle.json:27`

```json
"ram": ">2MB"
```

| 组件 | 估算大小 |
|------|----------|
| BMS 服务 | ~1MB |
| Bundle 信息缓存 | ~0.5MB |
| 运行时堆栈 | ~0.5MB |
| **总计** | **~2MB** |

---

**相关链接**:
- [GN 构建目标](05_GN_Targets.md)
- [项目概览](00_Overview.md)
- [安全风险分析](07_Security_Analysis.md)
