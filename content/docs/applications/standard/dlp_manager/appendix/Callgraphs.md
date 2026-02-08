# 附录A: 关键调用链

## 目的

本文档记录DLP Manager的关键调用链，帮助开发者理解代码执行流程。

## 调用链1: DLP文件打开流程

### 入口 → 核心逻辑

```
用户点击DLP文件
    │
    ▼
ViewAbility.onRequest(want, startId)
    │ entry/src/main/ets/Ability/ViewAbility.ets:48
    ▼
OpenDlpFileProcessor.process(want, startId, context)
    │ entry/src/main/ets/OpenDlpFile/ViewProcessor/ViewProcessor.ets:46
    ├──► checkAndSetWantParams()
    │    │ entry/src/main/ets/OpenDlpFile/data/OpenDlpFileData.ets
    │    └──► 验证Want参数
    │
    ├──► checkAndGetState()
    │    │ entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets
    │    └──► 检查文件是否正在解密
    │
    ├──► parseFile()
    │    │
    │    ├──► FileParseFactory.createFileParse()
    │    │    │ entry/src/main/ets/OpenDlpFile/handler/FileParseHandler.ets
    │    │    └──► 根据文件类型创建解析器
    │    │
    │    └──► FileParseHandler.parse()
    │         │ entry/src/main/ets/OpenDlpFile/handler/FileParseHandler.ets
    │         ├──► getFileFd() [获取文件描述符]
    │         │    │ entry/src/main/ets/common/FileUtils/utils.ets:136
    │         │
    │         ├──► getAccountTypeAndRealFileType() [解析文件头]
    │         │    │ entry/src/main/ets/common/FileUtils/utils.ets:511
    │         │    ├──► handleZipFile() [ZIP格式]
    │         │    └──► handleNonZipFile() [原始格式]
    │         │
    │         └──► FileIdHandler.getFileIdByUri() [获取文件ID]
    │
    ├──► handleAccount()
    │    │
    │    ├──► AccountHandlerFactory.createAccountHandler()
    │    │    │ entry/src/main/ets/OpenDlpFile/handler/AccountHandler.ets
    │    │
    │    └──► AccountHandler.handle()
    │         ├──► 域账号认证
    │         └──► CredConnectService.connectServiceShareAbility()
    │              │ entry/src/main/ets/rpc/CredConnectService.ets:92
    │
    ├──► decryptAndInstall()
    │    │
    │    ├──► DecryptHandler.getDecryptData()
    │    │    │ entry/src/main/ets/OpenDlpFile/handler/DecryptHandler.ets
    │    │    ├──► dlpPermission.openDlpFile() [系统服务调用]
    │    │    └──► HuksCipherUtils.decrypt() [解密数据]
    │    │         │ entry/src/main/ets/common/huks/HuksCipherUtil.ets
    │    │
    │    ├──► DecryptHandler.installSandbox()
    │    │    └──► dlpPermission.installDlpSandbox() [系统服务调用]
    │    │
    │    └──► DecryptHandler.grantUriPermission()
    │         └──► uriPermissionManager.grantUriPermission()
    │
    └──► StartSandboxHandler.startSandbox()
         │ entry/src/main/ets/OpenDlpFile/handler/StartSandboxHandler.ets
         └──► context.startAbility() [启动目标应用]
```

---

## 调用链2: DLP权限设置流程

```
用户选择"加密保护"
    │
    ▼
MainAbilityEx.onSessionCreate(want, session)
    │ entry/src/main/ets/Ability/MainAbilityEx.ets:98
    ├──► getDLPInfo()
    │
    ├──► connectDlpFileProcessAbility(want, session)
    │    │ entry/src/main/ets/Ability/MainAbilityEx.ets:60
    │    └──► context.connectServiceExtensionAbility()
    │
    ├──► checkValidWantAndAccount(session, want)
    │    │ entry/src/main/ets/Ability/MainAbilityEx.ets:248
    │    ├──► checkValidWant()
    │    │    │ entry/src/main/ets/Ability/MainAbilityEx.ets:197
    │    │    ├──► isValidPath(uri)
    │    │    │    │ entry/src/main/ets/common/FileUtils/utils.ets:507
    │    │    └──► 检查callerToken和callerBundleName
    │    │
    │    └──► getOsAccountInfo()
    │         │ entry/src/main/ets/common/FileUtils/utils.ets:148
    │
    ├──► checkDomainAccountInfo()
    │    │ entry/src/main/ets/common/FileUtils/utils.ets:153
    │    └──► 验证域账号已认证
    │
    └──► getNewWantPage(want, session)
         │ entry/src/main/ets/Ability/MainAbilityEx.ets:270
         ├──► judgeIsSandBox(want) [判断是否来自沙箱]
         │    │ entry/src/main/ets/common/FileUtils/utils.ets:212
         │
         ├──► requestIsFromSandBox() [沙箱内打开]
         │    └──► sandBoxLinkFile()
         │
         └──► requestIsNotFromSandBox() [沙箱外打开]
              ├──► FileUtils.isDLPFile() [检查文件后缀]
              ├──► getFileFd() [获取文件描述符]
              ├──► getAccountTypeAndRealFileType() [解析文件头]
              └──► dlpFilesToEncrypt() [进入加密流程]
                   │ entry/src/main/ets/Ability/MainAbilityEx.ets:363
                   ├──► findFileOpenHistoryHome() [检查打开历史]
                   └──► openDlpFile() [调用RPC打开]
                        └──► session.loadContent('pages/encryptionProtection')
                             │ entry/src/main/ets/pages/encryptionProtection.ets:87
```

---

## 调用链3: RPC服务连接流程

```
MainAbilityEx.connectDlpFileProcessAbility()
    │
    ▼
CredConnectService.constructor(context)
    │ entry/src/main/ets/rpc/CredConnectService.ets:37
    ├──► createSearchUserOptions() [创建搜索用户选项]
    ├──► createGetAccountOptions() [创建获取账号选项]
    └──► createBatchRefreshOptions() [创建批量刷新选项]
    │
    ▼
CredConnectService.connectServiceShareAbility(code)
    │ entry/src/main/ets/rpc/CredConnectService.ets:92
    ├──► 构造Want
    │    ├──► bundleName: Constants.DLP_CREDMGR_BUNDLE_NAME
    │    └──► abilityName: Constants.DLP_CREDMGR_DATA_ABILITY_NAME
    │
    └──► context.connectServiceExtensionAbility(want, options)
         │
         └──► 触发onConnect回调
              │
              ├──► optionsSearchUser.onConnect → searchUserInfo()
              ├──► optionsGetAccount.onConnect → getLocalAccountInfo()
              └──► optionsBatchRefresh.onConnect → batchRefresh()
                   │
                   └──► CredCallbackStub [处理RPC响应]
                        │ entry/src/main/ets/rpc/CredCallbackStub.ets
```

---

## 调用链4: 权限计算流程

```
MainAbilityEx.gotoPage()
    │
    ▼
getAuthPerm(accountName, dlpProperty)
    │ entry/src/main/ets/common/FileUtils/utils.ets:173
    │
    ├──► 检查是否为所有者
    │    │ if (accountName === dlpProperty.ownerAccount)
    │    └──► return FULL_CONTROL
    │
    ├──► 检查everyoneAccessList
    │    │ if (dlpProperty.everyoneAccessList?.length > 0)
    │    └──► perm = max(everyoneAccessList)
    │
    └──► 遍历authUserList
         │ for authUser in dlpProperty.authUserList
         └──► if (authUser.authAccount === accountName)
              └──► return authUser.dlpFileAccess
    │
    └──► return perm [默认NO_PERMISSION或everyone权限]
```

---

## 调用链5: 文件解析流程

```
FileParseFactory.createFileParse(openDlpFileData)
    │ entry/src/main/ets/OpenDlpFile/handler/FileParseHandler.ets
    │
    ├──► 根据文件类型选择解析器
    │    ├──► DLPZIPFileParseHandler [ZIP格式DLP文件]
    │    └──► DLPRawFileParseHandler [原始DLP文件]
    │
    ▼
FileParseHandler.parse(uri, filesDir)
    │
    ├──► getFileFd(uri) [获取文件描述符]
    │    │ entry/src/main/ets/common/FileUtils/utils.ets:136
    │    └──► fs.openSync(uri, mode)
    │
    ├──► getFileSizeByFd(fd) [获取文件大小]
    │
    └──► parseFileHeader(fd) [解析文件头]
         │
         ├──► 读取Magic [判断文件格式]
         │    ├──► DLP_ZIP_MAGIC (0x04034b50) → ZIP格式
         │    └──► DLP_RAW_MAGIC (0x087f4922) → 原始格式
         │
         ├──► getAccountTypeAndRealFileType() [获取账号类型]
         │    │ entry/src/main/ets/common/FileUtils/utils.ets:511
         │    ├──► ZIP格式: handleZipFile() → zlib.decompressFile()
         │    └──► 原始格式: handleNonZipFile() → 直接读取
         │
         └──► 提取fileId、expireTime等元信息
    │
    └──► 返回FileMetaInfo
```

---

## 调用链6: 沙箱启动流程

```
StartSandboxHandler.getInstance().startSandbox(decryptContent)
    │ entry/src/main/ets/OpenDlpFile/handler/StartSandboxHandler.ets
    │
    ├──► 获取单例实例
    │
    ├──► fillSandboxBundleInfo() [填充沙箱Bundle信息]
    │    ├──► 设置sandboxBundleName
    │    ├──► 设置appIndex
    │    └──► 设置userId
    │
    ├──► grantUriPermission() [授予URI权限]
    │    └──► uriPermissionManager.grantUriPermission()
    │
    ├──► constructWant() [构造启动Want]
    │    ├──► 设置bundleName、abilityName
    │    ├──► 设置uri（解密后文件URI）
    │    └──► 设置sandbox参数
    │
    └──► startAbility() [启动沙箱Ability]
         ├──► context.startAbility(want)
         └──► 等待Ability启动完成
              │
              └──► 目标应用启动，显示解密后的文件
```

---

## 调用链7: 状态管理流程

```
OpenDlpFileManager.getInstance()
    │ entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets:50
    │
    ├──► 首次调用创建单例
    │    ├──► new Map() [statusMap]
    │    ├──► new Map() [contentMap]
    │    └──► new AsyncLock() [lock]
    │
    ▼
操作: setStatus(uri, status)
    │
    ├──► lock.lockAsync() [获取锁]
    │    │ entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets:71
    │
    ├──► statusMap.set(uri, status) [更新状态]
    │    │ 状态: NOT_STARTED → DECRYPTING → DECRYPTED
    │
    └──► lock释放
    │
    ▼
操作: addDecryptContent(uri, content)
    │
    ├──► lock.lockAsync() [获取锁]
    │    │ entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets:118
    │
    ├──► 检查是否已存在相同tokenID
    │
    ├──► decryptContentList.add(content) [添加内容]
    │
    ├──► contentMap.set(uri, decryptContentList) [更新内容映射]
    │
    └──► statusMap.set(uri, {state: DECRYPTED}) [更新状态为已解密]
```

---

## 调用链索引

| 调用链 | 场景 | 入口文件 | 核心方法 |
|--------|------|----------|----------|
| 调用链1 | 打开DLP文件 | ViewAbility.ets:48 | process() |
| 调用链2 | 设置DLP权限 | MainAbilityEx.ets:98 | onSessionCreate() |
| 调用链3 | RPC服务连接 | CredConnectService.ets:92 | connectServiceShareAbility() |
| 调用链4 | 计算用户权限 | utils.ets:173 | getAuthPerm() |
| 调用链5 | 解析DLP文件 | FileParseHandler.ets | parse() |
| 调用链6 | 启动沙箱 | StartSandboxHandler.ets | startSandbox() |
| 调用链7 | 管理打开状态 | OpenDlpFileManager.ets | setStatus() / addDecryptContent() |

---

*本文档为代码分析结果，如有变更请同步更新*
