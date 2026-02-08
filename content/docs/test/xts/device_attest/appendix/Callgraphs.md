# 附录：关键调用链

本文档提供 `device_attest` 模块的关键调用链，帮助开发者理解代码执行路径。

## 调用链图例

- `→` 函数调用
- `⇒` 跨模块/IPC 调用
- `↩` 返回

---

## 调用链 1：JS 获取认证结果（完整链路）

```
JS 应用
  └── deviceAttest.getAttestStatus()
      └── N-API: GetAttestResultInfo() [devattest_napi.cpp:141]
          ├── 参数校验
          │   └── napi_typeof() 检查 callback 类型
          ├── 创建异步工作
          │   └── napi_create_async_work()
          ├── 执行阶段（工作线程）
          │   └── Execute() [devattest_napi.cpp:95]
          │       └── DevAttestClient::GetAttestStatus()
          │           ⇒ IPC 调用（见调用链 2）
          │           ↩ AttestResultInfo
          └── 完成阶段（主线程）
              └── Complete() [devattest_napi.cpp:111]
                  ├── GenerateReturnValue()
                  │   └── GenerateDevAttestHandle()
                  │       └── napi_create_object()
                  │       └── napi_set_named_property() × 4
                  └── Promise resolve / Callback 调用
                      └── napi_resolve_deferred() / napi_call_function()
```

**文件路径**:
- `interfaces/kits/napi/src/devattest_napi.cpp`

---

## 调用链 2：C++ SDK → IPC → Service

```
DevAttestClient::GetAttestStatus() [devattest_client.cpp:110]
  ├── GetDeviceProfileService() [devattest_client.cpp:35]
  │   ├── SystemAbilityManagerClient::GetSystemAbilityManager()
  │   ├── samgrProxy->CheckSystemAbility(SA_ID_DEVICE_ATTEST_SERVICE)
  │   │   └── 如果服务已启动：返回 IRemoteObject
  │   └── 如果服务未启动
  │       └── LoadDevAttestProfile() [devattest_client.cpp:68]
  │           ├── new DevAttestProfileLoadCallback()
  │           ├── samgr->LoadSystemAbility(SA_ID_DEVICE_ATTEST_SERVICE, callback)
  │           └── proxyConVar_.wait_for(timeout=10s) [devattest_client.cpp:88]
  ├── attestClientInterface->GetAttestStatus(attestResultInfo)
  │   ⇒ DevAttestServiceProxy::GetAttestStatus() [devattest_service_proxy.cpp]
  │       ├── MessageParcel data, reply
  │       ├── data.WriteInterfaceToken(GetDescriptor())
  │       ├── remote->SendRequest(GET_AUTH_RESULT, data, reply, option)
  │       │   ⇒ Binder IPC → devattest_service 进程
  │       │       └── DevAttestServiceStub::OnRemoteRequest() [devattest_service_stub.cpp:34]
  │       │           ├── data.ReadInterfaceToken() != GetDescriptor()
  │       │           │   └── 不匹配 → 返回 DEVATTEST_SERVICE_FAILED
  │       │           ├── DelayUnloadTask() 重置卸载定时器
  │       │           └── requestFuncMap_[GET_AUTH_RESULT]()
  │       │               └── GetAttestStatusInner() [devattest_service_stub.cpp:54]
  │       │                   ├── Permission::IsSystem() 权限检查 [permission.cpp:36]
  │       │                   │   ├── IPCSkeleton::GetCallingTokenID()
  │       │                   │   ├── AccessTokenKit::GetTokenTypeFlag()
  │       │                   │   └── TokenIdKit::IsSystemAppByFullTokenID()
  │       │                   ├── 非系统应用 → 返回 DEVATTEST_ERR_JS_IS_NOT_SYSTEM_APP
  │       │                   └── DevAttestService::GetAttestStatus() [devattest_service.cpp:161]
  │       │                       ├── malloc(resultArray)
  │       │                       └── QueryAttest() [attest_entry.c:36]
  │       │                           ⇒ Core 层调用（见调用链 3）
  │       │                           ↩ ret
  │       │                       ├── CopyAttestResult()
  │       │                       └── free() 资源
  │       │                   └── attestResultInfo.Marshalling(reply)
  │       │           ↩ IPC 返回
  │       └── reply.ReadInt32()
  │       └── AttestResultInfo::Unmarshalling(reply)
  └── LoadSystemAbilityFail() 清除缓存
```

**文件路径**:
- `interfaces/innerkits/native_cpp/src/devattest_client.cpp`
- `interfaces/innerkits/native_cpp/src/devattest_service_proxy.cpp`
- `services/devattest_ability/src/devattest_service_stub.cpp`
- `services/devattest_ability/src/devattest_service.cpp`
- `common/permission/src/permission.cpp`

---

## 调用链 3：Core 层查询认证状态

```
QueryAttest() [attest_entry.c:36]
  └── QueryAttestStatus() [attest_service.c:534]
      ├── pthread_mutex_lock(&g_mtxAttest)
      └── QueryAttestStatusSwitch() [attest_service.c:505]
          ├── GetAuthResultCode() [attest_service.c:43]
          │   ├── pthread_mutex_lock(&g_authStatusMutex)
          │   ├── AttestReadAuthResultCode() [adapter]
          │   └── pthread_mutex_unlock(&g_authStatusMutex)
          └── switch(authResultCode)
              ├── case AUTH_UNKNOWN (2): → SetAttestStatusDefault()
              │   └── 返回全部 0
              ├── case AUTH_FAILED (1): → SetAttestStatusFailed()
              │   └── 返回全部 2
              └── case AUTH_SUCCESS (0): → SetAttestStatusSucc()
                  ├── GetAuthStatus() [attest_service_auth.c]
                  │   └── 读取本地存储的认证状态
                  ├── DecodeAuthStatus()
                  │   └── Base64Decode + Decrypt
                  ├── ReadTicketFromDevice() [attest_security_ticket.c]
                  │   └── 从安全分区读取 ticket
                  └── CopyResultArray()
                      └── 填充 resultArray[ATTEST_RESULT_*]
      └── pthread_mutex_unlock(&g_mtxAttest)
```

**文件路径**:
- `services/core/attest_entry.c`
- `services/core/attest/attest_service.c`
- `services/core/security/attest_security_ticket.c`

---

## 调用链 4：认证主流程（首次启动）

```
设备启动 → 网络连接成功
  └── DevAttestTask::CreateThread() [devattest_task.cpp]
      └── AttestTask() [attest_entry.c:24]
          └── ProcAttest() [attest_service.c:357]
              ├── pthread_mutex_lock(&g_mtxAttest)
              ├── IsFullLoad() [attest_service.c:83] 流量控制
              │   └── 检查每小时请求次数
              ├── InitNetworkServerInfo() [attest_network.c]
              └── ProcAttestImpl() [attest_service.c:328]
                  ├── InitSysData() [attest_service_device.c]
                  │   ├── 读取设备参数（manufacturer, brand, model...）
                  │   └── 读取 OS 参数（version, patch level...）
                  ├── IsAuthStatusChg() 检查是否需要重新认证
                  └── AttestStartup() [attest_service.c:274] 认证主流程
                      ├── ResetDevice() [attest_service.c:121]
                      │   ├── GetChallenge() [attest_service_challenge.c]
                      │   ├── GenResetMsg() [attest_service_reset.c]
                      │   ├── SendResetMsg() [attest_network.c]
                      │   │   └── HTTPS POST 到云端
                      │   └── ParseResetResult()
                      ├── AuthDevice() [attest_service.c:161]
                      │   ├── GetChallenge()
                      │   ├── GenAuthMsg() [attest_service_auth.c]
                      │   │   ├── 收集设备信息
                      │   │   ├── 计算签名
                      │   │   └── JSON 序列化
                      │   ├── SendAuthMsg() [attest_network.c]
                      │   │   └── HTTPS POST 到云端
                      │   └── ParseAuthResultResp()
                      │       └── 解析云端响应，获取 ticket
                      ├── FlushAttestData() [attest_service.c:256]
                      │   ├── FlushAuthResult() 保存到文件
                      │   └── FlushAttestStatusPara() 更新系统参数
                      └── ActiveToken() [attest_service.c:206]
                          ├── FlushToken() 保存 token
                          ├── GetChallenge()
                          ├── GenActiveMsg() [attest_service_active.c]
                          ├── SendActiveMsg() [attest_network.c]
                          └── ParseActiveResult()
              └── pthread_mutex_unlock(&g_mtxAttest)
```

**文件路径**:
- `services/devattest_ability/src/devattest_task.cpp`
- `services/core/attest/attest_service.c`
- `services/core/network/attest_network.c`

---

## 调用链 5：安全加密流程

```
加密数据
  └── Encrypt() [attest_security.c:567]
      ├── 参数校验
      ├── EncryptAesCbc() [attest_security.c:511]
      │   ├── mbedtls_cipher_set_padding_mode(PKCS7)
      │   ├── cipherCtx.add_padding() 添加填充
      │   ├── mbedtls_aes_setkey_enc() 设置密钥
      │   └── mbedtls_aes_crypt_cbc() AES-CBC 加密
      ├── mbedtls_base64_encode() Base64 编码
      └── memcpy_s() 复制结果

解密数据
  └── Decrypt() [attest_security.c:606]
      ├── 参数校验
      ├── mbedtls_base64_decode() Base64 解码
      ├── DecryptAesCbc() [attest_security.c:374]
      │   ├── mbedtls_aes_setkey_dec() 设置密钥
      │   └── mbedtls_aes_crypt_cbc() AES-CBC 解密
      └── memcpy_s() 复制结果
      └── memset_s() 清零临时缓冲区

HUKS 加密
  └── EncryptHks() [attest_security.c:474]
      ├── HksInitParamSet() 初始化参数集
      ├── HksKeyExist() 检查密钥是否存在
      │   └── 不存在 → HksGenerateKey() 生成密钥
      └── EncryptHksImpl()
          ├── HksEncrypt() HUKS 加密
          └── mbedtls_base64_encode() Base64 编码

密钥派生
  └── GetAesKey() [attest_security.c:339]
      ├── GetProductInfo()
      │   ├── AttestGetManufacturekey() [adapter]
      │   └── AttestGetProductId() [adapter]
      ├── GetPsk() [attest_security.c:174]
      │   ├── mbedtls_base64_decode(g_pskKey)
      │   ├── mbedtls_base64_decode(g_encryptedPsk)
      │   └── psk[i] = base64Psk[i] ^ base64PskKey[i]
      └── mbedtls_hkdf() HKDF-SHA256 派生密钥
          └── 输入: salt + psk + productInfo
          └── 输出: aesKey
```

**文件路径**:
- `services/core/security/attest_security.c`

---

## 调用链 6：服务生命周期

```
系统启动
  └── init 进程读取 devattest_service.cfg
      └── 启动 devattest_service 进程
          └── main()
              └── DevAttestService::OnStart() [devattest_service.cpp:51]
                  ├── 检查 state_ 避免重复启动
                  ├── Init() [devattest_service.cpp:78]
                  │   ├── EventRunner::Create(ATTEST_UNLOAD_TASK_ID)
                  │   └── Publish(this) 注册到 SA 管理器
                  ├── state_ = STATE_RUNNING
                  └── 根据启动原因处理
                      ├── OnDemandReasonId::BOOT / NETWORK_AVAILABLE
                      │   └── DevAttestTask::CreateThread()
                      │       └── 执行认证流程（见调用链 4）
                      └── OnDemandReasonId::INTERFACE_CALL
                          └── AddDevAttestSystemAbilityListener(COMM_NET_CONN_MANAGER_SA_ID)
                              └── 监听网络状态变化

IPC 调用触发
  └── DevAttestService::OnStart(REASON_INTERFACE_CALL)
      ├── Init() 初始化
      └── 服务已运行，继续处理请求
      └── DelayUnloadTask() 启动 10 分钟卸载定时器
          └── unloadHandler_->PostTask(task, ATTEST_UNLOAD_TASK_ID, 600000)
              └── 10分钟后执行
                  ├── UnregisterNetConnCallback()
                  ├── AttestDestroyTimerTask()
                  └── samgrProxy->UnloadSystemAbility(SA_ID_DEVICE_ATTEST_SERVICE)

服务停止
  └── DevAttestService::OnStop() [devattest_service.cpp:99]
      ├── state_ = STATE_NOT_START
      └── registerToSa_ = false
```

**文件路径**:
- `services/devattest_ability/src/devattest_service.cpp`
- `services/etc/init/devattest_service.cfg`

---

## 相关链接

- [架构说明](../02_Architecture.md) - 系统架构文档
- [内部 API 文档](../04_Inner_API.md) - 接口说明
- [常见问题](../07_Troubleshooting.md) - 问题定位

---

**证据来源**：
- N-API：`interfaces/kits/napi/src/devattest_napi.cpp`
- IPC：`services/devattest_ability/src/devattest_service_stub.cpp`
- Core：`services/core/attest/attest_service.c`
- Security：`services/core/security/attest_security.c`
- Service：`services/devattest_ability/src/devattest_service.cpp`
