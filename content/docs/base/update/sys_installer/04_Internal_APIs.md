# 内部 API

## 目的

本文档详细说明 `sys_installer` 内部模块接口、依赖关系和稳定性分析。

## 适用范围

sys_installer 内部开发人员，需要理解模块间接口和依赖关系。

## 模块职责划分

```
┌─────────────────────────────────────────────────────────────────┐
│                        模块依赖关系图                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│  │   Client    │────>│  InnerKits  │────>│ IPC Client  │       │
│  └─────────────┘     └─────────────┘     └─────────────┘       │
│                                                   │             │
│  ┌────────────────────────────────────────────────┘             │
│  │                                                              │
│  ▼                                                              │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│  │  IPC Server │────>│   Services  │────>│  Frameworks │       │
│  │  (SA 4101/3)│     │             │     │             │       │
│  └─────────────┘     └─────────────┘     └─────────────┘       │
│                              │                                  │
│                              ▼                                  │
│                       ┌─────────────┐                          │
│                       │   Updater   │                          │
│                       │   (外部)     │                          │
│                       └─────────────┘                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 模块详细说明

### 1. IPC Client 层 (interfaces/innerkits/ipc_client/)

**职责**: 提供客户端 IPC 调用封装。

#### 1.1 SysInstallerKitsImpl

**文件**: `src/sys_installer_kits_impl.cpp` (662 行)

**接口稳定性**: ⭐⭐⭐ 稳定

**主要方法**:
```cpp
// 获取远程服务对象
sptr<ISysInstaller> GetService();

// 重置连接
void ResetService(const wptr<IRemoteObject> &remote);

// 获取 SA ID
int32_t GetSaId();

// 获取 SA 加载回调
sptr<SystemAbilityLoadCallbackStub> GetCallback();
```

**依赖**:
- `ISysInstaller` (IDL 生成)
- `ISysInstallerCallback`
- `SysInstallerLoadCallback`

#### 1.2 ModuleUpdateKitsImpl

**文件**: `src/module_update_kits_impl.cpp` (255 行)

**接口稳定性**: ⭐⭐⭐ 稳定

**主要方法**:
```cpp
// 获取远程服务对象
sptr<IModuleUpdate> GetService();

// 重置连接
void ResetService(const wptr<IRemoteObject> &remote);

// 获取 SA ID
int32_t GetSaId();
```

#### 1.3 ModuleUpdateProxy

**文件**: `src/module_update_proxy.cpp` (256 行)

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: IModuleUpdate 的 IPC Proxy 实现，负责参数序列化和远程调用。

**主要方法**:
```cpp
int32_t InstallModulePackage(const std::string &pkgPath) override;
int32_t UninstallModulePackage(const std::string &hmpName) override;
int32_t GetModulePackageInfo(...) override;
int32_t ExitModuleUpdate() override;
std::vector<HmpVersionInfo> GetHmpVersionInfo() override;
int32_t StartUpdateHmpPackage(...) override;
std::vector<HmpUpdateInfo> GetHmpUpdateResult() override;
```

### 2. IPC Server 层 (frameworks/ipc_server/)

#### 2.1 SysInstallerServer

**文件**: `src/sys_installer_server.cpp` (390 行)

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: SA 4101 服务实现，处理系统包更新请求。

**主要方法**:
```cpp
// SystemAbility 生命周期
void OnStart() override;
void OnStop() override;
void OnAddSystemAbility(int32_t systemAbilityId, const std::string &deviceId) override;

// ISysInstaller 接口实现
int32_t SysInstallerInit(...) override;
int32_t StartUpdatePackageZip(...) override;
int32_t SetUpdateCallback(...) override;
// ... 其他更新接口
```

**内部方法**:
```cpp
// 权限检查
bool CheckCallingPerm();
bool IsPermissionGranted();

// 回调管理
void CallbackEnter(int32_t code);
void CallbackExit(int32_t code, int32_t result);
```

#### 2.2 ModuleUpdateService

**文件**: `src/module_update_service.cpp` (215 行)

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: SA 4103 服务实现，处理模块更新请求。

**主要方法**:
```cpp
// SystemAbility 生命周期
void OnStart() override;
void OnStop() override;
void OnAddSystemAbility(int32_t systemAbilityId, const std::string &deviceId) override;

// IModuleUpdate 接口实现
int32_t InstallModulePackage(const std::string &pkgPath) override;
int32_t UninstallModulePackage(const std::string &hmpName) override;
int32_t GetModulePackageInfo(...) override;
int32_t ExitModuleUpdate() override;
std::vector<HmpVersionInfo> GetHmpVersionInfo() override;
int32_t StartUpdateHmpPackage(...) override;
std::vector<HmpUpdateInfo> GetHmpUpdateResult() override;
```

#### 2.3 ModuleUpdateStub

**文件**: `src/module_update_stub.cpp` (215 行)

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: IPC Stub 基类，处理 IPC 请求分发。

**主要方法**:
```cpp
// IPC 请求处理
int32_t OnRemoteRequest(uint32_t code, MessageParcel &data, 
                        MessageParcel &reply, MessageOption &option) override;

// 各命令码处理
int32_t HandleInstallModulePackage(MessageParcel &data, MessageParcel &reply);
int32_t HandleUninstallModulePackage(MessageParcel &data, MessageParcel &reply);
// ... 其他处理函数
```

### 3. Services 层 (services/)

#### 3.1 ModuleUpdate

**文件**: `module_update/src/module_update.cpp` (489 行)

**接口稳定性**: ⭐⭐ 较稳定

**职责**: 模块更新核心逻辑。

**主要方法**:
```cpp
// 初始化
int32_t Init();

// 模块包操作
int32_t InstallModulePackage(const std::string &pkgPath);
int32_t UninstallModulePackage(const std::string &hmpName);
int32_t GetModulePackageInfo(const std::string &hmpName, 
                             std::list<ModulePackageInfo> &infos);

// HMP 操作
std::vector<HmpVersionInfo> GetHmpVersionInfo();
int32_t StartUpdateHmpPackage(const std::string &path);
std::vector<HmpUpdateInfo> GetHmpUpdateResult();

// 退出
int32_t ExitModuleUpdate();
```

#### 3.2 ModuleUpdateMain

**文件**: `module_update/service/src/module_update_main.cpp` (607 行)

**接口稳定性**: ⭐⭐ 较稳定

**职责**: 模块更新服务主类，管理生产者-消费者线程。

**主要方法**:
```cpp
// 初始化
int32_t Init();

// 任务提交
int32_t SubmitTask(const ModuleUpdateTask& task);

// 任务处理
void ProcessTask(const ModuleUpdateTask& task);

// 退出
void Exit();
```

#### 3.3 ModuleFile

**文件**: `module_update/util/src/module_file.cpp` (576 行)

**接口稳定性**: ⭐⭐ 较稳定

**职责**: 模块文件操作。

**主要方法**:
```cpp
// 文件解析
int32_t ParseModulePackage(const std::string &pkgPath, ModuleFileInfo& info);

// 文件操作
int32_t ExtractFile(const std::string& src, const std::string& dst);
int32_t VerifyFile(const std::string& path, const std::string& hash);

// 目录操作
int32_t CreateModuleDir(const std::string& path);
int32_t RemoveModuleDir(const std::string& path);
```

#### 3.4 ModuleLoop

**文件**: `module_update/util/src/module_loop.cpp` (454 行)

**接口稳定性**: ⭐⭐ 较稳定

**职责**: Loop 设备管理。

**主要方法**:
```cpp
// Loop 设备操作
int32_t CreateLoopDevice(const std::string& imagePath, std::string& loopDevice);
int32_t RemoveLoopDevice(const std::string& loopDevice);

// 挂载操作
int32_t MountLoopDevice(const std::string& loopDevice, const std::string& mountPoint);
int32_t UmountLoopDevice(const std::string& mountPoint);
```

#### 3.5 ModuleDm

**文件**: `module_update/src/module_dm.cpp`

**接口稳定性**: ⭐⭐ 较稳定

**职责**: Device Mapper 设备管理。

**主要方法**:
```cpp
// DM 设备操作
int32_t CreateDmDevice(const std::string& name, const DmTable& table);
int32_t RemoveDmDevice(const std::string& name);

// 验证
int32_t VerifyDmDevice(const std::string& name);
```

### 4. Frameworks 层 (frameworks/)

#### 4.1 SysInstallerManager

**文件**: `installer_manager/src/sys_installer_manager.cpp` (308 行)

**接口稳定性**: ⭐⭐ 较稳定

**职责**: 系统安装管理器，协调更新流程。

**主要方法**:
```cpp
// 初始化
int32_t Init();

// 更新操作
int32_t StartUpdate(const std::string& taskId, const std::string& pkgPath);
int32_t CancelUpdate(const std::string& taskId);

// 状态管理
UpdateStatus GetUpdateStatus(const std::string& taskId);
int32_t SetUpdateCallback(const std::string& taskId, 
                          const sptr<ISysInstallerCallback>& callback);
```

#### 4.2 StatusManager

**文件**: `status_manager/src/status_manager.cpp`

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: 更新状态管理。

**主要方法**:
```cpp
// 状态操作
int32_t SetStatus(const std::string& taskId, UpdateStatus status);
UpdateStatus GetStatus(const std::string& taskId);

// 进度操作
int32_t SetProgress(const std::string& taskId, int32_t percent);
int32_t GetProgress(const std::string& taskId);
```

#### 4.3 ActionProcessor

**文件**: `action_processer/src/action_processer.cpp`

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: 动作处理框架。

**主要方法**:
```cpp
// 动作执行
int32_t ExecuteActions(const std::vector<IAction*>& actions);

// 回滚
int32_t RollbackActions(const std::vector<IAction*>& actions);
```

#### 4.4 PkgVerify

**文件**: `actions/verify_action/src/pkg_verify.cpp`

**接口稳定性**: ⭐⭐⭐ 稳定

**职责**: 包签名验证。

**主要方法**:
```cpp
// 验证
int32_t VerifyPackage(const std::string& pkgPath);
int32_t VerifySignature(const std::string& pkgPath, const std::string& cert);
```

## 依赖关系

### 模块依赖图

```
interfaces/innerkits/ipc_client
    ├── depends on: ipc:ipc_core
    ├── depends on: samgr:samgr_proxy
    └── depends on: safwk:system_ability_fwk

frameworks/ipc_server
    ├── depends on: interfaces/innerkits/ipc_client
    ├── depends on: frameworks/installer_manager
    ├── depends on: frameworks/status_manager
    ├── depends on: access_token:libaccesstoken_sdk
    └── depends on: safwk:system_ability_fwk

services/module_update
    ├── depends on: frameworks/action_processer
    ├── depends on: frameworks/actions/verify_action
    ├── depends on: hvb:libhvb_static_real (optional)
    └── depends on: selinux_adapter:librestorecon (optional)

services/ab_update
    └── depends on: updater:libfsmanager

services/stream_update
    └── depends on: updater:libringbuffer
```

### 循环依赖检查

**结论**: 未发现循环依赖。

依赖关系遵循单向原则：
- Client -> InnerKits -> IPC Client -> IPC Server -> Services -> Frameworks

## 接口稳定性分级

| 等级 | 标识 | 说明 | 模块 |
|------|------|------|------|
| ⭐⭐⭐ | 稳定 | 接口已固化，变更需兼容性处理 | IPC Client/Server, StatusManager |
| ⭐⭐ | 较稳定 | 接口基本稳定，可能小幅调整 | ModuleUpdate, ModuleFile |
| ⭐ | 不稳定 | 接口可能大幅调整 | 内部工具类 |

## 可替换点

### 1. 验证模块

**当前**: `PkgVerify` 使用系统证书验证

**可替换**: 可实现新的 `IAction` 接口，替换验证逻辑

**文件**: `frameworks/actions/verify_action/`

### 2. 存储后端

**当前**: 本地文件系统

**可替换**: 可实现新的 `ModuleFile` 接口，支持其他存储

**文件**: `services/module_update/util/`

### 3. 压缩算法

**当前**: ZIP + LZ4/ZLIB/BZIP2

**可替换**: 修改 `ModuleZipHelper` 支持其他格式

**文件**: `services/module_update/util/src/module_zip_helper.cpp`

## 线程安全

### 线程安全模块

| 模块 | 线程安全 | 说明 |
|------|----------|------|
| SysInstallerKitsImpl | ✅ | 使用互斥锁保护 |
| ModuleUpdateKitsImpl | ✅ | 使用互斥锁保护 |
| SysInstallerServer | ✅ | Binder 线程池处理 |
| ModuleUpdateService | ✅ | Binder 线程池 + 任务队列 |
| ModuleUpdateMain | ✅ | 生产者-消费者模式 |

### 线程模型

```
SysInstallerServer (Binder 线程池)
    │
    ├── 线程1: 处理 IPC 请求
    ├── 线程2: 处理 IPC 请求
    └── ...

ModuleUpdateService (Binder 线程池 + 工作线程)
    │
    ├── Binder 线程: 接收 IPC 请求，提交任务到队列
    │
    ├── 生产者线程: 处理任务入队
    │
    └── 消费者线程: 处理任务执行
```

## 错误传播机制

```
底层错误 (ModuleFile, ModuleLoop)
    │
    ▼ 转换为 ModuleErrorCode
ModuleUpdate
    │
    ▼ 转换为 IPC 错误码
ModuleUpdateService
    │
    ▼ 通过 IPC 返回
ModuleUpdateProxy
    │
    ▼ 转换为 API 错误码
ModuleUpdateKitsImpl
    │
    ▼ 返回给客户端
Client
```

## 相关链接

- [对外 API](03_External_APIs.md)
- [架构说明](02_Architecture.md)
- [GN Targets](05_GN_Targets.md)

---

*证据来源*:
- `interfaces/innerkits/ipc_client/src/`: IPC Client 实现
- `frameworks/ipc_server/src/`: IPC Server 实现
- `services/module_update/src/`: 模块更新实现
- `frameworks/installer_manager/src/`: 安装管理器实现
