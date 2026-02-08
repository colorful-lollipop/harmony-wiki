# 关键调用链

## 目的

本文档详细说明 `sys_installer` 的关键调用链，帮助理解代码执行流程。

## 适用范围

开发人员、调试人员、安全审计人员。

## 调用链 1: 系统包更新

### 流程图

```
Client (Updater App)
    │
    ▼
SysInstallerKits::StartUpdatePackageZip(taskId, pkgPath)
    │ interfaces/innerkits/ipc_client/src/sys_installer_kits_impl.cpp:100
    ▼
SysInstallerKitsImpl::GetService()
    │ 获取远程服务对象
    ▼
ISysInstaller::StartUpdatePackageZip (IPC 调用)
    │ Binder IPC
    ▼
SysInstallerServer::OnRemoteRequest()
    │ frameworks/ipc_server/src/sys_installer_server.cpp:200
    ▼
SysInstallerServer::StartUpdatePackageZip()
    │ frameworks/ipc_server/src/sys_installer_server.cpp:250
    ├── CheckCallingPerm() (权限检查)
    │   ├── GetCallingUid()
    │   └── VerifyAccessToken()
    ▼
SysInstallerManager::StartUpdate()
    │ frameworks/installer_manager/src/sys_installer_manager.cpp:100
    ▼
ActionProcessor::ExecuteActions()
    │ frameworks/action_processer/src/action_processer.cpp:50
    ├── PkgVerify::Execute() (包验证)
    │   ├── 签名验证
    │   └── 哈希验证
    ▼
ABUpdate::Execute() (AB 更新)
    │ services/ab_update/src/ab_update.cpp:80
    ├── 写入分区
    └── 更新启动标志
```

### 关键代码路径

```cpp
// 1. 客户端调用
// interfaces/innerkits/ipc_client/src/sys_installer_kits_impl.cpp:100
int32_t SysInstallerKitsImpl::StartUpdatePackageZip(
    const std::string &taskId, 
    const std::string &pkgPath) {
    auto proxy = GetService();
    if (proxy == nullptr) {
        return ERR_INVALID_VALUE;
    }
    return proxy->StartUpdatePackageZip(taskId, pkgPath);
}

// 2. 服务端处理
// frameworks/ipc_server/src/sys_installer_server.cpp:250
int32_t SysInstallerServer::StartUpdatePackageZip(
    const std::string &taskId, 
    const std::string &pkgPath) {
    if (!CheckCallingPerm()) {
        return ERR_PERMISSION_DENIED;
    }
    return installerManager_->StartUpdate(taskId, pkgPath);
}

// 3. 权限检查
// frameworks/ipc_server/src/sys_installer_server.cpp:329-350
bool SysInstallerServer::CheckCallingPerm() {
    int32_t callingUid = OHOS::IPCSkeleton::GetCallingUid();
    if (callingUid == 0) {
        return true;
    }
    return callingUid == USER_UPDATE_AUTHORITY && IsPermissionGranted();
}
```

## 调用链 2: 模块安装

### 流程图

```
Client (BMS)
    │
    ▼
ModuleUpdateKits::InstallModulePackage(pkgPath)
    │ interfaces/innerkits/ipc_client/src/module_update_kits_impl.cpp:80
    ▼
ModuleUpdateKitsImpl::GetService()
    │
    ▼
IModuleUpdate::InstallModulePackage (IPC 调用)
    │
    ▼
ModuleUpdateStub::OnRemoteRequest()
    │ frameworks/ipc_server/src/module_update_stub.cpp:100
    ├── 反序列化参数
    └── 分发到具体处理函数
    ▼
ModuleUpdateService::InstallModulePackage()
    │ frameworks/ipc_server/src/module_update_service.cpp:120
    ├── CheckCallingPerm() (权限检查)
    └── VerifyModulePackageSign() (签名验证)
    ▼
ModuleUpdate::InstallModulePackage()
    │ services/module_update/src/module_update.cpp:200
    ├── ModuleZipHelper::Extract() (解压)
    ├── ModuleUpdateVerify::Verify() (验证)
    │   ├── HVB 验证 (如启用)
    │   └── 签名验证
    └── ModuleFile::Install() (安装)
        ├── 创建 Loop 设备
        ├── 挂载镜像
        └── 复制文件
```

### 关键代码路径

```cpp
// 1. 服务端 Stub 处理
// frameworks/ipc_server/src/module_update_stub.cpp:100
int32_t ModuleUpdateStub::OnRemoteRequest(
    uint32_t code, 
    MessageParcel &data,
    MessageParcel &reply, 
    MessageOption &option) {
    switch (code) {
        case INSTALL_MODULE_PACKAGE:
            return HandleInstallModulePackage(data, reply);
        // ...
    }
}

// 2. 签名验证
// frameworks/ipc_server/src/module_update_service.cpp:120
int32_t ModuleUpdateService::InstallModulePackage(
    const std::string &pkgPath) {
    if (VerifyModulePackageSign(pkgPath) != 0) {
        LOG(ERROR) << "Verify sign failed " << pkgPath;
        return ERR_VERIFY_FAIL;
    }
    return moduleUpdate_->InstallModulePackage(pkgPath);
}

// 3. 模块安装
// services/module_update/src/module_update.cpp:200
int32_t ModuleUpdate::InstallModulePackage(const std::string &pkgPath) {
    // 解压
    int32_t ret = zipHelper_->Extract(pkgPath, tempDir);
    if (ret != ERR_OK) {
        return ret;
    }
    
    // 验证
    ret = verifier_->Verify(tempDir);
    if (ret != ERR_OK) {
        return ret;
    }
    
    // 安装
    return fileMgr_->Install(tempDir, targetDir);
}
```

## 调用链 3: VAB 更新

### 流程图

```
Client
    │
    ▼
SysInstallerKits::CreateVabSnapshotCowImg(cowSize, pkgPath)
    │
    ▼
SysInstallerServer::CreateVabSnapshotCowImg()
    │ frameworks/ipc_server/src/sys_installer_server.cpp:400
    ▼
SysInstallerManager::CreateVabSnapshot()
    │
    ▼
// 创建 COW (Copy-On-Write) 快照
DmCreateDevice("cow", table)
    │
    ▼
SysInstallerKits::StartUpdateVabPackageZip(taskId, pkgPath)
    │
    ▼
SysInstallerServer::StartUpdateVabPackageZip()
    │
    ▼
// 写入更新到 COW 区域
WriteToCowPartition(pkgPath)
    │
    ▼
SysInstallerKits::StartVabMerge(taskId)
    │
    ▼
SysInstallerServer::StartVabMerge()
    │
    ▼
// 标记合并 pending
Bootloader 在下次启动时完成合并
```

## 调用链 4: IPC 连接建立

### 流程图

```
Client
    │
    ▼
SysInstallerKitsImpl::GetService()
    │ interfaces/innerkits/ipc_client/src/sys_installer_kits_impl.cpp:50
    ├── 检查缓存的 proxy
    └── 如果无效，重新获取
    ▼
SystemAbilityLoadCallbackStub::OnLoadSystemAbilitySuccess()
    │ 等待 SA 加载完成
    ▼
samgr::GetSystemAbility(SYS_INSTALLER_DISTRIBUTED_SERVICE_ID)
    │
    ▼
SysInstallerLoadCallback::OnLoadSystemAbilitySuccess()
    │ interfaces/innerkits/ipc_client/src/sys_installer_load_callback.cpp:40
    ├── 获取 IRemoteObject
    └── 创建 ISysInstaller Proxy
    ▼
返回 ISysInstaller Proxy 给客户端
```

### 关键代码路径

```cpp
// 获取服务
// interfaces/innerkits/ipc_client/src/sys_installer_kits_impl.cpp:50
sptr<ISysInstaller> SysInstallerKitsImpl::GetService() {
    if (proxy_ != nullptr) {
        return proxy_;
    }
    
    sptr<ISystemAbilityManager> samgr = 
        SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    if (samgr == nullptr) {
        return nullptr;
    }
    
    sptr<IRemoteObject> object = samgr->GetSystemAbility(
        SYS_INSTALLER_DISTRIBUTED_SERVICE_ID);
    if (object == nullptr) {
        // SA 未启动，触发按需加载
        samgr->LoadSystemAbility(SYS_INSTALLER_DISTRIBUTED_SERVICE_ID, 
                                  loadCallback_);
        return nullptr;
    }
    
    proxy_ = iface_cast<ISysInstaller>(object);
    return proxy_;
}
```

## 调用链 5: 回调通知

### 流程图

```
SysInstallerServer (SA 4101)
    │
    ▼
更新进度变化
    │
    ▼
ISysInstallerCallback::OnUpgradeProgress(status, percent)
    │ IPC 回调
    ▼
SysInstallerCallback::OnUpgradeProgress()
    │ interfaces/innerkits/ipc_client/src/sys_installer_callback.cpp:40
    ▼
客户端注册的回调函数
    │
    ▼
Updater App UI 更新
```

## 调用链 6: 模块更新服务启动

### 流程图

```
触发条件:
- persist.samgr.moduleupdate.start=true
- persist.moduleupdate.bms.scan=revert

init / samgr
    │
    ▼
启动 module_update_sa 进程
    │
    ▼
main()
    │ services/module_update/service/main.cpp
    ▼
ModuleUpdateService::OnStart()
    │ frameworks/ipc_server/src/module_update_service.cpp:60
    ├── 初始化 ModuleUpdateMain
    └── 注册到 samgr
    ▼
ModuleUpdateMain::Init()
    │ services/module_update/service/src/module_update_main.cpp:100
    ├── 创建任务队列
    ├── 启动 Producer 线程
    └── 启动 Consumer 线程
    ▼
等待 IPC 调用
```

## 相关链接

- [架构说明](../02_Architecture.md)
- [对外 API](../03_External_APIs.md)
- [内部 API](../04_Internal_APIs.md)

---

*证据来源*:
- `interfaces/innerkits/ipc_client/src/`: IPC Client 实现
- `frameworks/ipc_server/src/`: IPC Server 实现
- `services/module_update/src/`: 模块更新实现
