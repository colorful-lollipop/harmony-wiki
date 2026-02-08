# 附录：关键调用链

## 目的

本文档记录 bundle_framework_lite 的关键调用链，帮助开发者理解代码执行流程。

## 1. 系统启动调用链

### 1.1 BMS 服务启动

```
[系统启动]
    │
    ▼
 foundation 进程启动
    │
    ├──► SAMGR 初始化
    │         │
    │         ▼
    │    BundleMgrService 构造函数
    │    (services/bundlemgr_lite/src/bundle_mgr_service.cpp:1)
    │         │
    │         ▼
    │    BundleMsHost::OnInitialize()
    │    (services/bundlemgr_lite/src/bundle_ms_host.cpp:1)
    │         │
    │         ▼
    │    SAMGR_GetInstance()->RegisterService()
    │         │
    │         ▼
    │    BundleMsFeature 注册
    │    (services/bundlemgr_lite/src/bundle_ms_feature.cpp:1)
    │
    ▼
 BUNDLE_SERVICE_INITED 消息
    │
    ▼
 ManagerService::ServiceMsgProcess()
    (services/bundlemgr_lite/src/bundle_manager_service.cpp:181)
    │
    ├──► BundleDaemonClient::Initialize()
    │         │
    │         ▼
    │    连接到 Bundle Daemon
    │
    ├──► ManagerService::ScanPackages()
    │         │
    │         ├──► InstallAllSystemBundle(SYSTEM_APP_FLAG)
    │         │         │
    │         │         ▼
    │         │    InstallSystemBundle()
    │         │         │
    │         │         ▼
    │         │    BundleInstaller::Install()
    │         │
    │         ├──► ScanAppDir(THIRD_SYSTEM_BUNDLE_PATH)
    │         │
    │         └──► ScanAppDir(INSTALL_PATH)
    │
    └──► ManagerService::GetAmsInterface()
              │
              ▼
         SAMGR_GetInstance()->GetFeatureApi(AMS_SERVICE)
              │
              ▼
         amsInterface->StartKeepAliveApps()
```

## 2. 应用安装调用链

### 2.1 标准安装流程

```
应用调用 Install()
    (interfaces/kits/bundle_lite/bundle_manager.h:112)
    │
    ▼
 frameworks/bundle_lite/src/bundle_manager.cpp:Install()
    │
    ▼
 IPC 调用 (LiteIPC)
    │
    ▼
 services/bundlemgr_lite/src/bundle_ms_feature.cpp:OnFeatureMessage()
    │
    ├──► 解析 IPC 消息
    │
    ▼
 services/bundlemgr_lite/src/bundle_inner_feature.cpp:HandleInstallMessage()
    │
    ▼
 services/bundlemgr_lite/src/bundle_installer.cpp:Install()
    (services/bundlemgr_lite/src/bundle_installer.cpp:106)
    │
    ├──► 参数校验
    │         │
    │         ▼
    │    CheckInstallFileIsValid()
    │         │
    │         ├──► BundleUtil::EndWith(path, ".hap")
    │         ├──► BundleUtil::IsFile(path)
    │         └──► BundleUtil::GetFileSize(path)
    │
    ├──► 获取 HAP 类型
    │         │
    │         ▼
    │    GetHapType(path)
    │         │
    │         ├──► SYSTEM_APP_FLAG (系统应用)
    │         ├──► THIRD_SYSTEM_APP_FLAG (三方系统应用)
    │         └──► THIRD_APP_FLAG (三方应用)
    │
    ▼
 BundleInstaller::ProcessBundleInstall()
    (services/bundlemgr_lite/src/bundle_installer.cpp:168)
    │
    ├──► 签名验证
    │         │
    │         ▼
    │    HapSignVerify::VerifySignature()
    │    (services/bundlemgr_lite/src/hap_sign_verify.cpp:23)
    │         │
    │         ▼
    │    APPVERI_AppVerify()  // 调用 appverify_lite
    │
    ├──► 解析 HAP
    │         │
    │         ▼
    │    BundleParser::ParseHapProfile()
    │    (services/bundlemgr_lite/src/bundle_parser.cpp:1)
    │         │
    │         ├──► 解压 config.json
    │         │         │
    │         │         ▼
    │         │    ZipFile::ExtractFile()
    │         │
    │         ├──► 解析 JSON
    │         │         │
    │         │         ▼
    │         │    cJSON_Parse()
    │         │
    │         ├──► ParseAppInfo()    // 解析应用信息
    │         ├──► ParseModuleInfo() // 解析模块信息
    │         └──► ParseAbilityInfo() // 解析 Ability 信息
    │
    ├──► 验证 Provision 信息
    │         │
    │         ▼
    │    CheckProvisionInfoIsValid()
    │         │
    │         ├──► MatchBundleName()      // 包名匹配
    │         └──► MatchPermissions()     // 权限匹配
    │
    ├──► 检查版本和签名
    │         │
    │         ▼
    │    CheckVersionAndSignature()
    │         │
    │         ├──► ManagerService::QueryBundleInfo() // 查询旧版本
    │         ├──► 版本号比较
    │         └──► appId 比较
    │
    ├──► 解压 HAP
    │         │
    │         ▼
    │    BundleDaemonClient::ExtractHap()
    │    (services/bundlemgr_lite/src/bundle_daemon_client.cpp:1)
    │         │
    │         ▼
    │    IPC 到 Bundle Daemon
    │         │
    │         ▼
    │    bundle_daemon:BundleDaemonHandler::ExtractHap()
    │    (services/bundlemgr_lite/bundle_daemon/src/bundle_daemon_handler.cpp:41)
    │         │
    │         ▼
    │    ZipFile::ExtractToDir()
    │
    ├──► 创建数据目录
    │         │
    │         ▼
    │    BundleDaemonClient::CreateDataDirectory()
    │         │
    │         ▼
    │    bundle_daemon:CreateDataDirectory()
    │         │
    │         ▼
    │    mkdir() + chown(uid, gid)
    │
    ├──► 保存权限
    │         │
    │         ▼
    │    StorePermissions()
    │         │
    │         ▼
    │    SaveOrUpdatePermissions() // 调用 PMS
    │
    ├──► 更新 Bundle 信息
    │         │
    │         ▼
    │    ManagerService::UpdateBundleInfo()
    │         │
    │         ▼
    │    BundleMap::Update()
    │
    └──► 备份记录
              │
              ▼
         BackUpInstallRecord()
                  │
                  ▼
             写入 JSON 文件
```

### 2.2 回调通知流程

```
安装完成
    │
    ▼
 InnerTransact()
    (services/bundlemgr_lite/src/bundle_manager_service.cpp:70)
    │
    ├──► 构造 IPC 消息
    │         │
    │         ▼
    │    IpcIoInit() + WriteInt32() + WriteString()
    │
    ├──► 获取回调服务列表
    │         │
    │         ▼
    │    ManagerService::GetServiceId()
    │
    ▼
 SendRequest() // 异步 IPC
    │
    ▼
 客户端回调
    │
    ▼
 InstallerCallback(resultCode, resultMessage)
```

## 3. 应用查询调用链

### 3.1 GetBundleInfo 查询

```
应用调用 GetBundleInfo()
    (interfaces/kits/bundle_lite/bundle_manager.h:187)
    │
    ▼
 frameworks/bundle_lite/src/bundle_manager.cpp:GetBundleInfo()
    │
    ▼
 IPC 调用
    │
    ▼
 services/bundlemgr_lite/src/bundle_ms_feature.cpp:GetBundleInfo()
    │
    ▼
 ManagerService::GetBundleInfo()
    (services/bundlemgr_lite/src/bundle_manager_service.cpp:599)
    │
    ▼
 BundleMap::GetBundleInfo()
    (services/bundlemgr_lite/src/bundle_map.cpp:1)
    │
    ├──► 查找 bundleMap_
    │         │
    │         ▼
    │    std::map::find(bundleName)
    │
    ├──► 复制数据
    │         │
    │         ▼
    │    memcpy_s() // 安全复制
    │
    ▼
 返回 BundleInfo
```

### 3.2 QueryAbilityInfo 查询

```
应用调用 QueryAbilityInfo()
    (interfaces/kits/bundle_lite/bundle_manager.h:142)
    │
    ▼
 IPC 到 BMS
    │
    ▼
 services/bundlemgr_lite/src/bundle_ms_feature.cpp:QueryAbilityInfo()
    │
    ▼
 ManagerService::QueryBundleInfo()
    │
    ▼
 BundleMap::Get()
    │
    ▼
 遍历 abilityInfos 数组
    │
    ├──► 匹配 bundleName
    ├──► 匹配 abilityName
    └──► 匹配 deviceId
         │
         ▼
    返回匹配的 AbilityInfo
```

## 4. 应用卸载调用链

```
应用调用 Uninstall()
    (interfaces/kits/bundle_lite/bundle_manager.h:125)
    │
    ▼
 IPC 到 BMS
    │
    ▼
 services/bundlemgr_lite/src/bundle_installer.cpp:Uninstall()
    (services/bundlemgr_lite/src/bundle_installer.cpp:425)
    │
    ├──► 参数校验
    │         │
    │         ▼
    │    if (bundleName == nullptr)
    │
    ├──► 查询 Bundle 信息
    │         │
    │         ▼
    │    ManagerService::QueryBundleInfo()
    │
    ├──► 检查是否系统应用
    │         │
    │         ▼
    │    if (bundleInfo->isSystemApp) return ERROR
    │
    ├──► 终止应用进程
    │         │
    │         ▼
    │    amsInterface->TerminateApp(bundleName)
    │
    ├──► 删除安装目录
    │         │
    │         ▼
    │    BundleDaemonClient::RemoveInstallDirectory()
    │         │
    │         ▼
    │    bundle_daemon:RemoveInstallDirectory()
    │         │
    │         ▼
    │    rmdir() + unlink()
    │
    ├──► 删除 Bundle 信息
    │         │
    │         ▼
    │    ManagerService::RemoveBundleInfo()
    │         │
    │         ▼
    │    BundleMap::Erase()
    │
    ├──► 删除权限
    │         │
    │         ▼
    │    DeletePermissions() // 调用 PMS
    │
    ├──► 删除 UID 信息
    │         │
    │         ▼
    │    BundleUtil::DeleteUidInfoFromJson()
    │
    └──► 删除配置文件
              │
              ▼
         BundleDaemonClient::RemoveFile()
                  │
                  ▼
             unlink(jsonPath)
```

## 5. IPC 通信调用链

### 5.1 客户端到 BMS

```
客户端调用
    │
    ▼
 frameworks/bundle_lite/src/bundle_manager.cpp
    │
    ├──► 构造 Want/参数
    │
    ├──► 序列化
    │         │
    │         ▼
    │    ConvertUtils::BundleInfoToJson() / AbilityInfoToJson()
    │
    ▼
 IpcIoInit() + WriteXXX()
    │
    ▼
 SendRequest(BMS_SERVICE, BMS_FEATURE, msgId)
    │
    ▼
 SAMGR 路由
    │
    ▼
 services/bundlemgr_lite/src/bundle_ms_feature.cpp:OnFeatureMessage()
    │
    ├──► 反序列化
    │         │
    │         ▼
    │    IpcIo 读取参数
    │
    ├──► 分发处理
    │         │
    │         ▼
    │    根据 msgId 调用对应处理函数
    │
    ▼
 业务处理
```

### 5.2 BMS 到 Bundle Daemon

```
BMS 调用
    │
    ▼
 BundleDaemonClient::GetInstance()
    │
    ▼
 BundleDaemonClient::Initialize()
    │
    ├──► SAMGR_GetInstance()->GetFeatureApi(BDS_SERVICE)
    │
    └──► 保存 SvcIdentity
         │
         ▼
 具体调用 (如 ExtractHap)
         │
         ▼
    IpcIoInit() + WriteString() + WriteInt32()
         │
         ▼
    SendRequest(BDS_SERVICE, funcId)
         │
         ▼
    SAMGR 路由到 Daemon 进程
         │
         ▼
    bundle_daemon:BundleDaemonHandler::OnMessage()
         │
         ▼
    根据 funcId 分发处理
```

## 6. 签名验证调用链

```
安装流程中
    │
    ▼
 HapSignVerify::VerifySignature()
    (services/bundlemgr_lite/src/hap_sign_verify.cpp:23)
    │
    ├──► 获取调试模式
    │         │
    │         ▼
    │    ManagerService::GetInstance().IsDebugMode()
    │
    ├──► 调用验证库
    │         │
    │         ▼
    │    APPVERI_AppVerify(hapFilepath.c_str(), &verifyResult)
    │         │
    │         ▼
    │    [appverify_lite 库内部]
    │         │
    │         ├──► 解析 HAP 包
    │         ├──► 验证证书链
    │         ├──► 验证签名
    │         └──► 提取 Provision 信息
    │
    ├──► 转换错误码
    │         │
    │         ▼
    │    SwitchErrorCode(ret)
    │
    └──► 提取信息到 SignatureInfo
              │
              ├──► signatureInfo.appId
              ├──► signatureInfo.provisionBundleName
              └──► signatureInfo.restrictedPermissions
```

## 7. 系统能力查询调用链

```
JS 调用 capability.has()
    │
    ▼
 interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp:HasCapability()
    │
    ├──► 参数校验
    │         │
    │         ▼
    │    if (argsSize < 1) return false
    │    JSI::ValueToString(args[0])
    │
    ▼
 HasSystemCapability(str)
    │
    ▼
 IPC 到 BMS
    │
    ▼
 ManagerService::HasSystemCapability()
    (services/bundlemgr_lite/src/bundle_manager_service.cpp:607)
    │
    ▼
 SAMGR_GetInstance()->HasSystemCapability(sysCapName)
    │
    ▼
 返回布尔值
```

---

**相关链接**:
- [架构说明](../02_Architecture.md)
- [对外 API](../03_Public_API.md)
- [内部 API](../04_Internal_API.md)
