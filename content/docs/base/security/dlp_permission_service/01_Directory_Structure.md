# 目录结构与模块职责

## 目的

本文档描述 `dlp_permission_service` 的顶层目录结构和各模块的职责，帮助开发者快速定位代码。

## 适用范围

- 目标读者：DLP 权限管理服务的开发者、维护者
- 覆盖内容：顶层目录结构、模块职责、代码组织原则

---

## 顶层目录结构

```
dlp_permission_service/
├── BUILD.gn                              # 根构建入口
├── bundle.json                           # 组件配置（依赖、系统能力、构建目标）
├── dlp_permission_service.gni           # GN 配置（根目录路径、feature flags）
├── identify_sensitive_content.gni       # 敏感内容识别 feature flag
├── config/                               # 构建配置（覆盖率、fortify 标志）
├── figures/                             # 文档图片资源
├── frameworks/                          # 框架层（基础功能实现）
│   ├── access_config/                   # 访问权限配置
│   ├── common/                          # 框架公共代码
│   ├── dlp_permission/                  # DLP 权限管理框架
│   └── test/                            # 框架层测试（忽略）
├── interfaces/                          # 接口层
│   ├── inner_api/                       # 内部接口（服务间）
│   │   ├── dlp_fuse/                    # link 文件及 FUSE 文件系统
│   │   ├── dlp_parse/                   # DLP 文件解析
│   │   ├── dlp_permission/              # DLP 权限管理内部接口
│   │   └── dlp_set_config/              # DLP 配置设置
│   └── kits/                            # 外部接口（N-API、C API）
│       ├── c/                           # C 接口（NDK）
│       ├── dlp_permission/              # N-API 接口（JS 绑定）
│       ├── identify_sensitive_content/  # 敏感内容识别 N-API
│       └── napi_common/                 # N-API 公共代码
├── services/                            # 服务层
│   └── dlp_permission/sa/               # DLP 权限管理 SA
│       ├── adapt/                       # 数据适配（账户、应用、文件等）
│       ├── callback/                    # 监听回调实现
│       ├── etc/                         # 配置文件
│       ├── sa_common/                   # 服务层公共代码
│       ├── sa_main/                     # DLP 权限管理服务主代码
│       └── sa_profile/                  # SA 配置定义
└── test/                                # 测试（忽略）
```

---

## 模块职责详解

### 1. 根目录

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | 根构建入口，定义 `dlp_permission_build_module` 组目标 |
| `bundle.json` | 组件配置：依赖列表、系统能力、构建目标、inner kits |
| `dlp_permission_service.gni` | GN 配置：定义根目录路径 `dlp_root_dir` 和 feature flags |
| `identify_sensitive_content.gni` | 敏感内容识别 feature flag 配置 |

---

### 2. frameworks/ - 框架层

框架层提供基础功能实现，不直接对外暴露。

#### 2.1 frameworks/access_config/

**职责**：访问权限配置文件管理

**证据**：
- `clone_app_permission_config` target (BUILD.gn:20)
- `clone_app_permission.json` 配置文件

#### 2.2 frameworks/common/

**职责**：框架层公共代码和数据结构

**关键文件** (frameworks/common/include/)：
- `permission_policy.h` - 权限策略定义（DLPFileAccess 枚举、ActionFlags）
- `cert_parcel.h` - 证书 Parcel 结构
- `retention_sandbox_info.h` - 保留沙箱信息
- `visited_dlp_file_info.h` - 访问记录信息

#### 2.3 frameworks/dlp_permission/

**职责**：DLP 权限管理框架实现

**包含**：核心权限管理逻辑、数据结构定义

---

### 3. interfaces/ - 接口层

接口层分为内部接口（服务间）和外部接口（N-API、C API）。

#### 3.1 interfaces/inner_api/ - 内部接口

##### 3.1.1 inner_api/dlp_fuse/

**职责**：DLP link 文件和 FUSE 文件系统实现

**功能**：
- 创建 link 文件指向 DLP 文件
- FUSE daemon 管理文件访问
- 支持 FUSE 文件系统操作

**关键文件**：
- `dlp_fuse_fd.h` - FUSE 文件描述符
- `dlp_fuse_helper.cpp` - FUSE 辅助函数
- `dlp_link_file.cpp` - link 文件管理
- `fuse_daemon.cpp` - FUSE daemon

**输出**: `libdlp_fuse.so` (BUILD.gn:21)

##### 3.1.2 inner_api/dlp_parse/

**职责**：DLP 文件解析和操作

**功能**：
- DLP 文件格式解析
- 加密/解密操作
- ZIP 文件处理
- 原始文件操作

**关键文件**：
- `dlp_file.h` / `dlp_file.cpp` - DLP 文件类
- `dlp_crypt.h` / `dlp_crypt.cpp` - 加密解密
- `dlp_zip_file.h` / `dlp_zip_file.cpp` - ZIP 文件处理
- `dlp_file_manager.h` / `dlp_file_manager.cpp` - 文件管理器

**输出**:
- `libdlpparse.so` - 公共解析库 (bundle.json:101-114)
- `libdlpparse_inner.so` - 内部解析库 (bundle.json:125-138)

##### 3.1.3 inner_api/dlp_permission/

**职责**：DLP 权限管理内部接口

**功能**：
- IDL 接口定义
- IPC stub/proxy 生成
- 客户端连接管理

**关键文件**：
- `IDlpPermissionService.idl` - IPC 服务接口定义 (35+ 方法)
- `DlpPermissionTypes.idl` - 类型定义
- `src/dlp_permission_client.cpp` - 客户端单例
- `src/dlp_permission_async_stub.cpp` - 异步回调 stub
- `include/dlp_permission_kit.h` - SDK 接口头文件

**输出**:
- `libdlp_permission_sdk.so` - 主 SDK (bundle.json:73-90)
- `libdlp_permission_common_interface.so` - 通用接口 (BUILD.gn:22)

##### 3.1.4 inner_api/dlp_set_config/

**职责**：DLP 配置设置 SDK

**功能**：
- 设置 DLP 配置
- 获取 DLP 配置

**输出**: `libdlp_setconfig_sdk.so` (bundle.json:92-99)

#### 3.2 interfaces/kits/ - 外部接口

##### 3.2.1 kits/c/

**职责**：C API / NDK 接口

**输出**: `ohdlp_permission.so` (BUILD.gn:25)

##### 3.2.2 kits/dlp_permission/napi/

**职责**：JavaScript N-API 绑定

**关键文件**：
- `napi_dlp_permission_manager.cpp` - 模块注册入口
- `napi_dlp_permission.cpp` - 主 API 实现 (2213 行)
- `napi_dlp_feature.cpp` - 特性开关接口
- `napi_dlp_connection_plugin.cpp` - 连接插件接口
- `napi_common.cpp` - 公共工具（参数校验、异步上下文）
- `napi_error_msg.cpp` - 错误消息映射

**输出**:
- `libdlppermission_napi.so` - DLP 权限 N-API (BUILD.gn:17-31)
- `libdlpsetdlpfeature_napi.so` - 特性开关 N-API

##### 3.2.3 kits/identify_sensitive_content/napi/

**职责**：敏感内容识别 N-API

**关键文件**：
- `napi_identify_sensitive_content.cpp` - 内容扫描接口

**输出**: `identifysensitivecontent_napi.so` (BUILD.gn:31)

##### 3.2.4 kits/napi_common/

**职责**：N-API 公共工具

**关键文件**：
- `napi_common.cpp` - 参数校验、异步工作管理
- `napi_error_msg.cpp` - 错误码到消息映射

---

### 4. services/ - 服务层

服务层实现 System Ability，提供 IPC 服务。

#### 4.1 services/dlp_permission/sa/

##### 4.1.1 sa_main/

**职责**：DLP 权限管理服务主实现

**关键文件**：
- `dlp_permission_service.h` / `dlp_permission_service.cpp` - 服务主类
- `dlp_credential.h` / `dlp_credential.cpp` - 凭证管理

**服务类继承**：
```
SystemAbility (safwk)
    ↑
DlpPermissionServiceStub (IDL-generated)
    ↑
DlpPermissionService (实现)
```

**生命周期**：
- `OnStart()` - 注册监听器、发布服务
- `OnStop()` - 清理资源

##### 4.1.2 sa_common/

**职责**：服务层公共代码

**关键文件**：
- `permission_manager_adapter.h` / `.cpp` - 权限校验适配器
- `dlp_sandbox_info.h` - 沙箱信息结构

##### 4.1.3 adapt/

**职责**：数据适配层

**子模块**：
- `adapt_utils/account_adapt/` - 账户适配
- `adapt_utils/alg_adapt/` - 算法适配（加密、密钥）
- `adapt_utils/app_observer/` - 应用状态观察者
- `adapt_utils/critical_handler/` - 关键处理器
- `adapt_utils/file_manager/` - 文件管理适配

##### 4.1.4 callback/

**职责**：回调监听实现

**子模块**：
- `dlp_sandbox_change_callback/` - 沙箱变化回调
- `open_dlp_file_callback/` - 打开文件回调

##### 4.1.5 etc/

**职责**：服务配置文件

**文件**：
- `dlp_permission.para` - 服务参数配置
- `dlp_permission.para.dac` - DAC 配置
- `dlp_config.json` - 支持的文件类型配置

##### 4.1.6 sa_profile/

**职责**：SA 配置定义

**文件**：
- `3521.json` - SA ID 3521 的配置 (进程名、库路径、运行策略)

**输出**: `dlp_permission_sa_profile_standard` target (BUILD.gn:27)

---

## 代码组织原则

### 依赖方向

```
N-API 层 → Inner API 层 → Service 层
```

- **N-API 层** (interfaces/kits/napi_*)：对外 JS 接口
- **Inner API 层** (interfaces/inner_api/*)：内部 SDK 和接口
- **Service 层** (services/*)：System Ability 实现

### 避免循环依赖

当前架构避免了循环依赖：
- Service 层不依赖 N-API 层
- Inner API 层单向依赖 Service 层（通过 IPC）

### 测试隔离

测试代码与业务代码分离：
- `frameworks/*/test/` - 框架层测试
- `test/` - 组件级测试（单元测试、fuzz 测试）

---

## 相关跳转链接

- [架构说明](02_Architecture.md) - 深入理解模块间关系
- [对外 N-API](03_NAPI.md) - 查看 N-API 接口详情
- [内部 API](04_Internal_API.md) - 查看内部接口定义
- [GN Targets](05_GN_Targets.md) - 查看构建目标配置

---

最后更新时间：2026-02-06
