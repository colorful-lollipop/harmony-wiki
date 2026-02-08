# 编译产物

## 目的

本文档列出 DLP 权限管理服务的所有编译产物，包括 .so/.a/.hap 文件、安装路径和运行时加载关系。

## 适用范围

- 目标读者：平台开发者、系统集成者
- 覆盖内容：产物清单、安装路径、运行时加载

---

## 产物清单

### 共享库 (.so)

| 产物名 | 来源 Target | 安装路径 | 用途 |
|---------|-------------|----------|------|
| `libdlppermission_napi.so` | libdlppermission_napi | `module/` | DLP 权限 N-API (JS 绑定） |
| `libdlpsetdlpfeature_napi.so` | libdlpsetdlpfeature_napi | `module/` | 特性开关 N-API |
| `identifysensitivecontent_napi.so` | identifysensitivecontent_napi | `module/security/` | 敏感内容识别 N-API |
| `ohdlp_permission.so` | ohdlp_permission | (默认) | C API / NDK 库 |
| `libdlp_permission_sdk.so` | libdlp_permission_sdk | (内部） | 主 SDK |
| `libdlp_permission_common_interface.so` | libdlp_permission_common_interface | (内部） | 通用接口库 |
| `libdlp_setconfig_sdk.so` | libdlp_setconfig_sdk | (内部） | 配置 SDK |
| `libdlp_fuse.so` | libdlp_fuse | (内部） | FUSE 文件系统库 |
| `libdlpparse.so` | libdlpparse | (内部） | DLP 文件解析库（公共） |
| `libdlpparse_inner.so` | libdlpparse_inner | (内部） | DLP 文件解析库（内部） |
| `libdlp_permission_service.z.so` | dlp_permission_service | - | System Ability 服务 |

**代码证据**：
- `libdlppermission_napi.so`: BUILD.gn:17-31, relative_install_dir = "module"
- `libdlp_permission_service.z.so`: BUILD.gn:26, .z.so 后缀

### 静态库 (.a)

| 产物名 | 来源 Target | 用途 |
|---------|-------------|------|
| `libdlp_hex_string.a` | dlp_hex_string_static | 十六进制字符串工具 |
| `libdlp_permission_serializer.a` | dlp_permission_serializer_static | 序列化器 |

**代码证据**：
- `dlp_hex_string_static`: services/dlp_permission/sa/BUILD.gn
- `dlp_permission_serializer_static`: services/dlp_permission/sa/BUILD.gn

### 配置文件

| 文件名 | 来源 Target | 安装路径 | 用途 |
|--------|-------------|----------|------|
| `3521.json` | dlp_permission_sa_profile_standard | System Ability profile | SA 配置 (SA ID: 3521) |
| `dlp_permission.para` | param_files | `etc/param/` | 服务参数配置 |
| `dlp_permission.para.dac` | param_files | `etc/param/` | DAC 配置 |
| `dlp_config.json` | param_files | (根目录） | 支持的文件类型配置 |
| `clone_app_permission.json` | clone_app_permission_config | `dlp_permission/` | 克隆应用权限配置 |

**代码证据**：
- `3521.json`: services/dlp_permission/sa/sa_profile/3521.json
- `dlp_permission.para`: services/dlp_permission/sa/etc/BUILD.gn

---

## 安装路径

### 系统分区布局

```
/system/
├── lib/
│   ├── module/
│   │   ├── libdlppermission_napi.so
│   │   ├── libdlpsetdlpfeature_napi.so
│   │   └── security/
│   │       └── identifysensitivecontent_napi.so
│   ├── ohdlp_permission.so
│   └── libdlp_permission_service.z.so
├── etc/
│   └── param/
│       ├── dlp_permission.para
│       └── dlp_permission.para.dac
├── dlp_permission/
│   └── clone_app_permission.json
└── dlp_config.json
```

### N-API 模块安装

| N-API 模块 | .so 文件 | 模块名 | JS import |
|-----------|----------|----------|----------|
| dlpPermission | libdlppermission_napi.so | "dlpPermission" | `import dlpPermission from '@ohos.dlpPermissionService';` |
| dlpSetDlpFeature | libdlpsetdlpfeature_napi.so | "dlpSetDlpFeature" | `import dlpSetDlpFeature from '@ohos.dlpPermissionService';` |
| security.identifySensitiveContent | identifysensitivecontent_napi.so | "security.identifySensitiveContent" | `import identifySensitiveContent from '@ohos.dlpPermissionService';` |

**代码证据**：
- 模块名：napi_dlp_permission_manager.cpp:49, napi_dlp_feature.cpp:198, napi_identify_sensitive_content.cpp:301

---

## 运行时加载关系

### System Ability 加载

**启动流程**：

```
Init 进程 (/system/bin/init)
    ↓ 读取 dlp_permission.para
    ↓ 启动 dlp_permission_service 进程
    ↓ 加载 libdlp_permission_service.z.so
    ↓ 读取 3521.json (SA profile)
    ↓ 注册到 SystemAbilityManager (SA ID: 3521)
    ↓ 服务启动 (OnStart)
    ↓ Publish(this)
```

**代码证据**：
- SA 配置：services/dlp_permission/sa/sa_profile/3521.json
- 服务注册：dlp_permission_service.cpp:122, `REGISTER_SYSTEM_ABILITY_BY_ID()`

### N-API 模块加载

**JS 引擎加载流程**：

```
JS 引擎 (Ark Runtime)
    ↓ 加载 libdlppermission_napi.so
    ↓ 调用 DlpPermissionModuleRegister() (constructor)
    ↓ napi_module_register(&_module)
    ↓ Init() 函数
    ↓ 导出 JS 方法/类
    ↓ JS import dlpPermission
```

**代码证据**：
- 模块注册：napi_dlp_permission_manager.cpp:57-60
- 模块名：napi_dlp_permission_manager.cpp:49

### FUSE 文件系统加载

**FUSE daemon 启动流程**：

```
沙箱应用启动
    ↓ 安装 DLP 沙箱 (InstallDlpSandbox)
    ↓ 调用 libdlp_fuse.so
    ↓ DlpFuseStart() / fuse_daemon 启动
    ↓ fuse_loop() / fuse_loop_mt()
    ↓ FUSE mount
    ↓ 文件系统请求通过 FUSE 传递
    ↓ DLP 文件解析/解密
```

**代码证据**：
- FUSE 启动：fuse_daemon.cpp
- FUSE mount：dlp_fuse_helper.cpp

---

## 运行时依赖

### 进程依赖

| 进程名 | 用途 | 依赖 |
|---------|------|------|
| `dlp_permission_service` | DLP 权限管理服务进程 | SystemAbilityManager, AccessTokenKit, HUKS, BundleManager |
| 沙箱应用 | DLP 沙箱隔离应用 | dlp_permission_service (SA 3521), libdlp_fuse.so |
| 三方应用 | 使用 DLP 服务 | dlp_permission_service (SA 3521), N-API 模块 |

### 库依赖

| 库 | 依赖的外部库 |
|-----|-------------|
| libdlppermission_napi.so | ace_napi, access_token, ipc, ability_runtime |
| libdlp_permission_service.z.so | safwk, samgr, ipc, huks, ability_runtime, bundle_framework |
| libdlp_fuse.so | libfuse, libdlpparse_inner.so, openssl |

**代码证据**：
- 外部依赖：bundle.json:32-66
- N-API 依赖：interfaces/kits/dlp_permission/BUILD.gn:60-82

---

## 链接关系

### 符号导出

#### libdlppermission_napi.so

**导出的 N-API 符号**：
- `DlpPermissionModuleRegister` - 模块注册函数
- `DlpFeatureModuleRegister` - 特性模块注册函数（通过 libdlpsetdlpfeature_napi.so）

**代码证据**：
- `DlpPermissionModuleRegister`: napi_dlp_permission_manager.cpp:57
- `DlpFeatureModuleRegister`: napi_dlp_feature.cpp:197

#### libdlp_permission_service.z.so

**导出的 System Ability 符号**：
- `DlpPermissionService` 类 (通过 REGISTER_SYSTEM_ABILITY_BY_ID 宏）

**代码证据**：
- `REGISTER_SYSTEM_ABILITY_BY_ID(DlpPermissionService, SA_ID_DLP_PERMISSION_SERVICE, true)`: dlp_permission_service.cpp:122

#### libdlp_fuse.so

**导出的 FUSE 符号**：
- `DlpFuseStart()` - FUSE daemon 启动函数
- `DlpCreateLinkFile()` - 创建 link 文件
- `DlpStopFuse()` - 停止 FUSE

**代码证据**：
- `DlpFuseStart`: fuse_daemon.cpp
- `DlpCreateLinkFile`: dlp_link_file.cpp

---

## 运行时加载顺序

### 系统启动顺序

```
1. Init 进程启动
2. 加载 SA 配置 (3521.json)
3. 启动 dlp_permission_service 进程
4. 加载 libdlp_permission_service.z.so
5. DlpPermissionService 构造函数
6. OnStart() 被调用
7. Publish(this) 注册到 SAMGR
8. 服务可用 (状态：RUNNING)
```

**代码证据**：
- 服务启动：dlp_permission_service.cpp:129-154

### 应用启动顺序

```
1. 三方应用启动
2. Ark Runtime 加载
3. 加载 N-API 模块 (libdlppermission_napi.so)
4. JS 执行 import dlpPermission
5. DlpPermissionModuleRegister() 被调用
6. 导出的 JS 方法可用
```

**代码证据**：
- N-API 注册：napi_dlp_permission_manager.cpp:44-60

---

## 相关跳转链接

- [GN Targets](05_GN_Targets.md) - 查看构建配置
- [目录结构与模块职责](01_Directory_Structure.md) - 查看代码组织

---

最后更新时间：2026-02-06
