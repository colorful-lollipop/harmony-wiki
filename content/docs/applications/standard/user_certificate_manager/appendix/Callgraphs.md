# 关键调用链

> 本文档提供用户证书管理部件的关键函数调用链

---

## 目的

本文档帮助开发者理解关键业务流程的函数调用路径。

## 适用范围

- 需要深入理解代码的开发者
- 调试人员
- 代码审查人员

## 相关跳转

- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计
- [04_External_API.md](wiki/04_External_API.md) - 外部 API

---

## 1. 证书安装调用链

### 1.1 用户 CA 证书安装

```
用户操作: 点击安装 CA 证书
  ↓
[Pages 层] certInstallFromStorage.ets
  ↓ 用户输入别名
[Presenter 层] CmInstallPresenter.installCert(fileUri, alias, suffix, isCa)
  ↓ 调用
[Model 层] CertMangerModel.installCertOrCred(CM_MODEL_OPT_USER_CA, alias, data, pwd, callback)
  ↓ switch 到
[Model 层] CertMangerModel.installUserCertificate(data, alias, optType, callback)
  ↓ 校验
[Model 层] 检查 data 是否为空 (CertMangerModel.ets:699-703)
  ↓ 确定格式
[Model 层] 设置 certFormat = PEM_DER 或 P7B (CertMangerModel.ets:705-708)
  ↓ 调用
[外部 API] CertManager.installUserTrustedCertificate({inData, alias, certFormat, certScope})
  ↓ 返回结果
[外部 API] 返回 {uri: string} 或抛出异常
  ↓ 处理
[Model 层] catch 块处理错误 (CertMangerModel.ets:717-729)
  - CM_ERROR_INCORRECT_FORMAT → CM_MODEL_ERROR_INCORRECT_FORMAT
  - CM_ERROR_MAX_CERT_COUNT_REACHED → CM_MODEL_ERROR_MAX_QUANTITY_REACHED
  - CM_ERROR_ALIAS_LENGTH_REACHED_LIMIT → CM_MODEL_ERROR_ALIAS_LENGTH_REACHED_LIMIT
  ↓ 回调
[Presenter 层] callback(errCode, uri)
  ↓ 更新 UI
[Pages 层] 显示安装成功或失败提示
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:287-317` (installCertOrCred)
- `certmanager/src/main/ets/model/CertMangerModel.ets:697-730` (installUserCertificate)
- `certmanager/src/main/ets/presenter/CmInstallPresenter.ets`

---

### 1.2 用户凭据安装

```
用户操作: 点击安装凭据
  ↓
[Pages 层] certPwdInput.ets
  ↓ 用户输入密码
[Presenter 层] CmInstallPresenter.installEvidence(fileUri, alias, pwd, suffix)
  ↓ 调用
[Model 层] CertMangerModel.installCertOrCred(CM_MODEL_OPT_APP_CRED, alias, data, pwd, callback)
  ↓ switch 到
[Model 层] CertMangerModel.installPublicCertificate(data, pwd, alias, callback)
  ↓ 调用
[外部 API] CertManager.installPublicCertificate(data, pwd, alias)
  ↓ 返回结果
[外部 API] 返回 {uri: string} 或抛出异常
  ↓ 处理
[Model 层] catch 块处理错误 (CertMangerModel.ets:743-757)
  - CM_ERROR_INCORRECT_FORMAT → CM_MODEL_ERROR_INCORRECT_FORMAT
  - CM_ERROR_MAX_CERT_COUNT_REACHED → CM_MODEL_ERROR_MAX_QUANTITY_REACHED
  - CM_ERROR_ALIAS_LENGTH_REACHED_LIMIT → CM_MODEL_ERROR_ALIAS_LENGTH_REACHED_LIMIT
  - CM_ERROR_PASSWORD_IS_ERR → CM_MODEL_ERROR_PASSWORD_ERR
  ↓ 回调
[Presenter 层] callback(errCode, uri)
  ↓ 更新 UI
[Pages 层] 显示安装成功或失败提示
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:287-317` (installCertOrCred)
- `certmanager/src/main/ets/model/CertMangerModel.ets:732-758` (installPublicCertificate)

---

### 1.3 系统 凭据安装

```
用户操作: 点击安装系统 凭据
  ↓
[Pages 层] certPwdInput.ets
  ↓ 用户输入密码
[Presenter 层] CmInstallPresenter.installEvidence(fileUri, alias, pwd, suffix)
  ↓ 调用
[Model 层] CertMangerModel.installCertOrCred(CM_MODEL_OPT_SYSTEM_CRED, alias, data, pwd, callback)
  ↓ switch 到
[Model 层] CertMangerModel.installSystemAppCertificate(data, pwd, alias, callback)
  ↓ 调用
[外部 API] CertManager.installSystemAppCertificate(data, pwd, alias)
  ↓ 返回结果
[外部 API] 返回 {uri: string} 或抛出异常
  ↓ 处理
[Model 层] catch 块处理错误 (CertMangerModel.ets:771-785)
  - CM_ERROR_INCORRECT_FORMAT → CM_MODEL_ERROR_INCORRECT_FORMAT
  - CM_ERROR_MAX_CERT_COUNT_REACHED → CM_MODEL_ERROR_MAX_QUANTITY_REACHED
  - CM_ERROR_ALIAS_LENGTH_REACHED_LIMIT → CM_MODEL_ERROR_ALIAS_LENGTH_REACHED_LIMIT
  - CM_ERROR_PASSWORD_IS_ERR → CM_MODEL_ERROR_PASSWORD_ERR
  ↓ 回调
[Presenter 层] callback(errCode, uri)
  ↓ 更新 UI
[Pages 层] 显示安装成功或失败提示
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:287-317` (installCertOrCred)
- `certmanager/src/main/ets/model/CertMangerModel.ets:760-786` (installSystemAppCertificate)

---

## 2. 证书查询调用链

### 2.1 用户 CA 证书列表

```
页面显示: onAboutToAppear()
  ↓
[Pages 层] certManagerFa.ets
  ↓
[Presenter 层] CmShowUserCaPresenter.loadUserCaList()
  ↓ 调用
[Model 层] CertMangerModel.getCertOrCredList(CM_MODEL_OPT_USER_CA, callback)
  ↓ switch 到
[Model 层] CertMangerModel.getAllUserTrustedCertificates(callback)
  ↓ 调用
[外部 API] CertManager.getAllUserTrustedCertificates()
  ↓ 返回结果
[外部 API] 返回 {certList: CertAbstract[]}
  ↓ 处理
[Model 层] 解析 subjectName (正则提取 CN) (CertMangerModel.ets:426-441)
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS, certList)
  ↓ 回调
[Presenter 层] callback(errCode, certList)
  ↓ 更新 UI
[Pages 层] 显示证书列表
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:90-118` (getCertOrCredList)
- `certmanager/src/main/ets/model/CertMangerModel.ets:99-103` (switch 到 USER_CA)
- `certmanager/src/main/ets/model/CertMangerModel.ets:423-453` (getAllUserTrustedCertificates)

---

### 2.2 系统 CA 证书列表

```
页面显示: onAboutToAppear()
  ↓
[Pages 层] trustedCa.ets
  ↓
[Presenter 层] CmShowSysCaPresenter.loadSysCaList()
  ↓ 调用
[Model 层] CertMangerModel.getCertOrCredList(CM_MODEL_OPT_SYSTEM_CA, callback)
  ↓ switch 到
[Model 层] CertMangerModel.getSystemTrustedCertificateList(callback)
  ↓ 调用
[外部 API] CertManager.getSystemTrustedCertificateList()
  ↓ 返回结果
[外部 API] 返回 {certList: CertAbstract[]}
  ↓ 处理
[Model 层] 解析 subjectName (正则提取 CN) (CertMangerModel.ets:322-336)
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS, certList)
  ↓ 回调
[Presenter 层] callback(errCode, certList)
  ↓ 更新 UI
[Pages 层] 显示证书列表
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:90-118` (getCertOrCredList)
- `certmanager/src/main/ets/model/CertMangerModel.ets:94-98` (switch 到 SYSTEM_CA)
- `certmanager/src/main/ets/model/CertMangerModel.ets:319-347` (getSystemTrustedCertificateList)

---

### 2.3 证书详情查询

```
用户操作: 点击证书条目
  ↓
[Pages 层] CaUserDetailPage.ets 或 CaSystemDetailPage.ets
  ↓
[Presenter 层] 获取证书 URI 并传递给 Model
  ↓ 调用
[Model 层] CertMangerModel.getCertOrCred(optType, uri, callback)
  ↓ switch 到
[Model 层] CertMangerModel.getUserTrustedCertificate(uri, callback)
  ↓ 调用
[外部 API] CertManager.getUserTrustedCertificate(uri)
  ↓ 返回结果
[外部 API] 返回 {certInfo: CertInfo}
  ↓ 处理
[Model 层] 解析 subjectName, issuerName, dateMap (CertMangerModel.ets:455-470)
  - setSubjectName(result, subjectNameMap)
  - setIssuerName(result, issuerNameMap)
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS, certInfo)
  ↓ 回调
[Presenter 层] callback(errCode, certInfo)
  ↓ 更新 UI
[Pages 层] 显示证书详情（CN、O、OU、颁发时间、有效期等）
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:120-148` (getCertOrCred)
- `certmanager/src/main/ets/model/CertMangerModel.ets:124-133` (switch 到 USER_CA)
- `certmanager/src/main/ets/model/CertMangerModel.ets:455-488` (getUserTrustedCertificate)

---

## 3. 证书删除调用链

### 3.1 用户 CA 证书删除

```
用户操作: 点击删除按钮
  ↓ 确认对话框
[Pages 层] CaUserDetailPage.ets
  ↓
[Presenter 层] 调用删除方法
  ↓ 调用
[Model 层] CertMangerModel.deleteCertOrCred(CM_MODEL_OPT_USER_CA, uri, callback)
  ↓ switch 到
[Model 层] CertMangerModel.deleteUserTrustedCertificate(uri, callback)
  ↓ 调用
[外部 API] CertManager.uninstallUserTrustedCertificate(uri)
  ↓ 返回结果
[外部 API] 返回 void 或抛出异常
  ↓ 处理
[Model 层] catch 块处理异常 (CertMangerModel.ets:490-500)
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS)
  ↓ 回调
[Presenter 层] callback(errCode)
  ↓ 更新 UI
[Pages 层] 显示删除成功提示，刷新列表
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:150-175` (deleteCertOrCred)
- `certmanager/src/main/ets/model/CertMangerModel.ets:154-158` (switch 到 USER_CA)
- `certmanager/src/main/ets/model/CertMangerModel.ets:490-501` (deleteUserTrustedCertificate)

---

## 4. 授权管理调用链

### 4.1 获取授权应用列表

```
用户操作: 点击凭据条目
  ↓
[Pages 层] CredUserDetailPage.ets
  ↓
[Presenter 层] CmAppCredAuthPresenter.loadAuthorizedAppList(uri)
  ↓ 调用
[Model 层] CertMangerModel.getAuthAppList(CM_MODEL_OPT_APP_CRED, uri, callback)
  ↓ switch 到
[Model 层] CertMangerModel.getAuthorizedAppList(uri, callback)
  ↓ 调用
[外部 API] CertManager.getAuthorizedAppList(uri)
  ↓ 返回结果
[外部 API] 返回 {appUidList: string[]}
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS, appUidList)
  ↓ 回调
[Presenter 层] callback(errCode, appUidList)
  ↓ 获取应用信息
[Presenter 层] 对每个 appUid 调用 BundleModel.getAppInfoList(appUid, callback)
  ↓ 调用
[Model 层] BundleModel.getAppInfoList(appUid, callback)
  ↓ 调用
[外部 API] bundleManager.getAppCloneIdentity(appUid)
  ↓ 调用
[外部 API] bundleResManager.getBundleResourceInfo(bundleName, bundleFlags, appIndex)
  ↓ 返回结果
[外部 API] 返回 {label, icon}
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS, {appName, appImage})
  ↓ 回调
[Presenter 层] callback(errCode, appInfoVo)
  ↓ 更新 UI
[Pages 层] 显示授权应用列表（应用名称、图标）
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:227-245` (getAuthAppList)
- `certmanager/src/main/ets/model/BundleModel.ets:24-49` (getAppInfoList)

---

### 4.2 设置应用授权

```
用户操作: 切换授权开关
  ↓
[Pages 层] AuthorizedAppManagementPage.ets
  ↓
[Presenter 层] CmAppCredAuthPresenter.setAuthorizedAppStatus(uri, appUid, status)
  ↓ 调用
[Model 层] CertMangerModel.setAppAuth(CM_MODEL_OPT_APP_CRED, uri, appUid, status, callback)
  ↓ switch 到
[Model 层] CertMangerModel.setAuthorizedAppStatus(uri, appUid, status, callback)
  ↓ 判断
[Model 层] if (status)
  ↓ 调用
  [外部 API] CertManager.grantPublicCertificate(uri, appUid)
  ↓ 返回结果
  [外部 API] 返回 {uri: string}
else
  ↓ 调用
  [外部 API] CertManager.removeGrantedPublicCertificate(uri, appUid)
  ↓ 返回结果
  [外部 API] 返回 void
  ↓ 处理
[Model 层] catch 块处理异常 (CertMangerModel.ets:677-694)
  ↓ 回调
[Model 层] callback(CM_MODEL_ERROR_SUCCESS, uri)
  ↓ 回调
[Presenter 层] callback(errCode, uri)
  ↓ 终止会话
[Presenter 层] GlobalContext.getContext().getCmContext().terminateSelfWithResult(want)
  ↓ 返回结果
[Extension] 返回给调用者
```

**证据位置**:
- `certmanager/src/main/ets/model/CertMangerModel.ets:267-285` (setAppAuth)
- `certmanager/src/main/ets/model/CertMangerModel.ets:672-695` (setAuthorizedAppStatus)
- `certmanager/src/main/ets/presenter/CmAppCredAuthPresenter.ets`

---

## 5. 用户认证调用链

### 5.1 用户认证流程

```
触发: 安装 CA 证书前需要认证
  ↓
[Pages 层] certInstallFromStorage.ets
  ↓
[Presenter 层] CmInstallPresenter.installCert(...)
  ↓ 调用
[Model 层] CertMangerModel.installCertOrCred(...)
  ↓ 调用
[Model 层] CheckUserAuthModel.auth(titleStr, callback)
  ↓ 检查支持的认证类型
[Model 层] isAuthTypeSupported(FINGERPRINT)
  ↓ 调用
[外部 API] userAuth.getAvailableStatus(FINGERPRINT, ATL1)
  ↓ 返回结果
[外部 API] 返回 void 或抛出异常
  ↓ 判断
[Model 层] if (成功)
  [Model 层] isAuthTypeSupported(PIN)
  [外部 API] userAuth.getAvailableStatus(PIN, ATL1)
  [外部 API] 返回 void 或抛出异常
else
  ↓ 调用
  [Model 层] callback(true)
  ↓ 回调
  [Presenter 层] callback(true)
  ↓ 继续安装
else
  ↓ 生成随机数
  [Model 层] getRandomData()
    [外部 API] cryptoFramework.createRandom()
    [外部 API] rand.generateRandomSync(16)
    [外部 API] 返回 {data: Uint8Array}
  ↓ 创建认证参数
  [Model 层] authParam = {challenge: randomData, authType: [FINGERPRINT], authTrustLevel: ATL1}
  [Model 层] widgetParam = {title: titleStr}
  ↓ 创建认证实例
  [外部 API] userAuth.getUserAuthInstance(authParam, widgetParam)
  [外部 API] 返回 UserAuthInstance
  ↓ 启动认证
  [外部 API] userAuthInstance.start()
  ↓ 监听结果
  [外部 API] userAuthInstance.on('result', {onResult(result)})
    ↓ 判断结果
    [Model 层] if (result === SUCCESS)
      [Model 层] callback(true)
    else if (result === CANCELED)
      [Model 层] callback(false)
    else
      [Model 层] callback(false)
  ↓ 回调
[Model 层] callback(authResult)
  ↓ 回调
[Presenter 层] callback(authResult)
  ↓ 判断
[Presenter 层] if (authResult === true)
  继续安装
else
  显示认证失败提示
```

**证据位置**:
- `certmanager/src/main/ets/model/CheckUserAuthModel.ets:61-117`
- `certmanager/src/main/ets/model/CertMangerModel.ets:287-317` (installCertOrCred)
- `certmanager/src/main/ets/presenter/CmInstallPresenter.ets`

---

## 6. 文件读取调用链

### 6.1 文件读取流程

```
用户操作: 选择文件
  ↓
[Pages 层] certManagerFa.ets
  ↓ 调用
[Presenter 层] CmFaPresenter.routeToNextInstallCert(fileUri)
  ↓ 调用
[Model 层] FileIoModel.getMediaFileSuffix(mediaUri, callback)
  ↓ 创建 URI
[Model 层] new fileUri.FileUri(mediaUri)
  ↓ 提取后缀
[Model 层] suffix = uri.name.substring(uri.name.lastIndexOf('.') + 1)
  ↓ 转换小写
[Model 层] suffixLowerCase = suffix.toLowerCase()
  ↓ 回调
[Model 层] callback(suffixLowerCase)
  ↓ 回调
[Presenter 层] callback(suffix)
  ↓ 判断格式
[Presenter 层] if (后缀 == 'cer' || 'pem' || 'crt' || 'der' || 'p7b' || 'spc')
  调用安装流程
else
  显示不支持格式提示
```

```
[Presenter 层] 调用
[Model 层] FileIoModel.getMediaFileData(mediaUri, callback)
  ↓ 打开文件
[外部 API] fs.openSync(mediaUri, READ_ONLY)
  ↓ 获取文件状态
[外部 API] fs.statSync(fd)
  ↓ 分配 buffer
[Model 层] buf = new ArrayBuffer(Number(stat.size))
  ↓ 读取文件
[外部 API] fs.readSync(fd, buf)
  ↓ 关闭文件
[外部 API] fs.closeSync(fd)
  ↓ 回调
[Model 层] callback(new Uint8Array(buf))
  ↓ 回调
[Presenter 层] callback(data)
  ↓ 调用安装
```

**证据位置**:
- `certmanager/src/main/ets/model/FileIoModel.ets:47-59` (getMediaFileSuffix)
- `certmanager/src/main/ets/model/FileIoModel.ets:21-44` (getMediaFileData)
- `certmanager/src/main/ets/presenter/CmFaPresenter.ets:64-93`

---

## 7. 防截屏调用链

```
触发: 密码输入页面显示
  ↓
[Pages 层] certPwdInput.ets
  ↓ 调用
[Model 层] PreventScreenshotsModel.getInstance().PreventScreenshots(true, session)
  ↓ 判断
[Model 层] if (session !== undefined)
  ↓ 设置隐私模式
  [外部 API] session.setWindowPrivacyMode(true)
else
  ↓ 获取窗口
  [外部 API] window.getLastWindow(context)
  ↓ 设置隐私模式
  [外部 API] windowClass.setWindowPrivacyMode(true)
```

```
触发: 密码输入页面隐藏
  ↓
[Pages 层] certPwdInput.ets
  ↓ 调用
[Model 层] PreventScreenshotsModel.getInstance().PreventScreenshots(false, session)
  ↓ 判断
[Model 层] if (session !== undefined)
  ↓ 取消隐私模式
  [外部 API] session.setWindowPrivacyMode(false)
else
  ↓ 获取窗口
  [外部 API] window.getLastWindow(context)
  ↓ 取消隐私模式
  [外部 API] windowClass.setWindowPrivacyMode(false)
```

**证据位置**:
- `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:24-57`
- `certmanager/src/main/ets/pages/certPwdInput.ets`

---

**END OF appendix/Callgraphs.md**
