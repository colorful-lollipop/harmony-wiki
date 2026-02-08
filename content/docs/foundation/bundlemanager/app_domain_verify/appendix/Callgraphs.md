# 关键调用链

> 本文档描述 app_domain_verify 部件的关键调用链，从入口到核心逻辑。

## 1. 域名校验调用链

### 1.1 应用安装触发校验

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      应用安装 → 域名校验完整调用链                        │
└─────────────────────────────────────────────────────────────────────────┘

BundleManagerService (调用方)
        │
        │ 1. VerifyDomain(appIdentifier, bundleName, fingerprint, skillUris)
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   AppDomainVerifyMgrClient                              │
│  文件: interfaces/inner_api/client/src/app_domain_verify_mgr_client.cpp │
│                                                                        │
│  singleton<AppDomainVerifyMgrClient>                                   │
│  GetInstance() → VerifyDomain()                                        │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 2. IPC 调用
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                IAppDomainVerifyMgrService (IPC)                         │
│  文件: services/include/manager/zidl/app_domain_verify_mgr_service_*.h │
│                                                                        │
│  Proxy → Stub.dispatch(VERIFY_DOMAIN, data)                            │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 3. 分发请求
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              AppDomainVerifyMgrService::VerifyDomain()                  │
│  文件: services/src/manager/core/app_domain_verify_mgr_service.cpp      │
│  行号: ~88                                                              │
│                                                                        │
│  1. 权限检查 (IsSACall)                                                │
│  2. 参数验证                                                           │
│  3. 获取 SkillUri 列表中的域名                                          │
│  4. 调用 Agent Service 执行校验                                         │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 4. IPC 调用 Agent Service
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              IAppDomainVerifyAgentService::SingleVerify()               │
│  文件: interfaces/inner_api/client/include/sa_interface/                │
│        i_app_domain_verify_agent_service.h                              │
│                                                                        │
│  创建 VerifyTask 并添加到任务队列                                       │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 5. FFRT 任务调度
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 AppDomainVerifyTaskMgr::ExecuteTask()                   │
│  文件: frameworks/common/src/httpsession/app_domain_verify_task_mgr.cpp  │
│                                                                        │
│  FFRT Worker Thread 执行任务                                           │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 6. 创建 HTTP 任务
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                 VerifyHttpTask::CreateHttpClientTask()                  │
│  文件: frameworks/verifier/src/verify_http_task.cpp                      │
│                                                                        │
│  1. 构建 HTTP 请求 (GET /.well-known/applinking.json)                   │
│  2. 设置超时 (60s)                                                     │
│  3. 注册回调                                                           │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 7. HTTP 请求执行
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    NetStack HTTP Client                                 │
│  依赖: netstack                                                         │
│                                                                        │
│  1. DNS 解析                                                           │
│  2. TCP 连接                                                          │
│  3. TLS 握手                                                           │
│  4. 发送请求                                                           │
│  5. 接收响应                                                           │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 8. 响应回调
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              VerifyHttpTask::OnSuccess() / OnFail()                     │
│  文件: frameworks/verifier/src/verify_http_task.cpp                      │
│                                                                        │
│  OnSuccess:                                                            │
│    - 解析 JSON (DomainJsonUtil::ParseAssetLinks)                        │
│    - 验证签名 (DomainVerifier::VerifyHostWithAppIdentifier)            │
│    - 保存结果 (SaveDomainVerifyStatus)                                  │
│                                                                        │
│  OnFail:                                                               │
│    - 处理错误码                                                         │
│    - 决定重试或标记失败                                                  │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 9. 保存结果
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│           AppDomainVerifyMgrService::SaveDomainVerifyStatus()            │
│  文件: services/src/manager/core/app_domain_verify_mgr_service.cpp      │
│                                                                        │
│  1. 序列化结果                                                          │
│  2. 调用 RDB 存储                                                       │
│  3. 更新内存缓存                                                        │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 10. RDB 持久化
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│           AppDomainVerifyRdbDataManager::Insert/Update                  │
│  文件: frameworks/app_details_rdb/src/app_details_rdb_data_manager.cpp   │
│                                                                        │
│  1. 打开数据库连接                                                      │
│  2. 执行 INSERT/UPDATE                                                 │
│  3. 关闭连接                                                            │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 代码位置速查

| 步骤 | 文件 | 行号/符号 |
|-----|------|----------|
| 1 | `interfaces/inner_api/client/src/app_domain_verify_mgr_client.cpp` | `VerifyDomain()` |
| 2 | `services/include/manager/zidl/` | Proxy/Stub |
| 3 | `services/src/manager/core/app_domain_verify_mgr_service.cpp` | `VerifyDomain()` L88 |
| 4 | `interfaces/inner_api/client/include/sa_interface/i_app_domain_verify_agent_service.h` | `SingleVerify()` |
| 5 | `frameworks/common/src/httpsession/app_domain_verify_task_mgr.cpp` | `ExecuteTask()` |
| 6 | `frameworks/verifier/src/verify_http_task.cpp` | `CreateHttpClientTask()` |
| 7 | `netstack` | HTTP 库 |
| 8 | `frameworks/verifier/src/verify_http_task.cpp` | `OnSuccess()`/`OnFail()` |
| 9 | `services/src/manager/core/app_domain_verify_mgr_service.cpp` | `SaveDomainVerifyStatus()` |
| 10 | `frameworks/app_details_rdb/src/app_details_rdb_data_manager.cpp` | `Insert()`/`Update()` |

---

## 2. 隐式跳转过滤调用链

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  隐式跳转 → FilterAbilities 调用链                       │
└─────────────────────────────────────────────────────────────────────────┘

元能力管理服务 (Ability Manager Service)
        │
        │ 1. startAbility(want) - want.uri = "https://domain.com/path"
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     BundleManagerService                                  │
│                                                                        │
│  1. 初筛 ability (根据 want 的 skill 匹配)                              │
│  2. 调用 FilterAbilities 进行二次过滤                                    │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 2. FilterAbilities(want, originAbilityInfos, filteredAbilityInfos)
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   AppDomainVerifyMgrClient                              │
│  文件: interfaces/inner_api/client/src/app_domain_verify_mgr_client.cpp │
│                                                                        │
│  singleton<AppDomainVerifyMgrClient>::FilterAbilities()                  │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 3. IPC 调用
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│               IAppDomainVerifyMgrService::FilterAbilities()              │
│  文件: services/include/manager/zidl/app_domain_verify_mgr_service_stub.cpp│
│        分发 FILTER_ABILITIES (code = 3)                                 │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 4. 过滤逻辑
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│            AppDomainVerifyMgrService::FilterAbilities()                   │
│  文件: services/src/manager/core/app_domain_verify_mgr_service.cpp      │
│  行号: ~162                                                              │
│                                                                        │
│  1. 从 want 中提取域名 (Want → SkillUri → domain)                       │
│  2. 查询该域名的校验状态                                                 │
│  3. 遍历 originAbilityInfos                                             │
│  4. 过滤规则:                                                          │
│     - ability 必须在域名校验通过的包中                                   │
│     - ability 的 skill 必须匹配 want 中的 URL                            │
│  5. 返回 filteredAbilityInfos                                           │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 5. 权限检查
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   PermissionManager::IsSACall()                         │
│  文件: frameworks/common/src/permission/permission_manager.cpp          │
│                                                                        │
│  验证调用者是否为 SA (System Ability)                                   │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 6. 查询校验状态
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│           AppDomainVerifyRdbDataManager::QueryByDomain()                 │
│  文件: frameworks/app_details_rdb/src/app_details_rdb_data_manager.cpp   │
│                                                                        │
│  SELECT bundleName FROM app_verify_status WHERE domain = ?              │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. JS API 调用链

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      JS API 调用链                                       │
└─────────────────────────────────────────────────────────────────────────┘

ArkTS/JavaScript 应用
        │
        │ 1. import bundle from '@ohos.bundle.appDomainVerify'
        │ 2. bundle.queryAssociatedDomains(bundleName)
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    libappdomainverify_napi.so                            │
│  文件: interfaces/kits/js/jsi/src/native_module.cpp                      │
│                                                                        │
│  NAPI_MODULE_REGISTER → AppDomainVerifyExport()                         │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 3. N-API 绑定
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              QueryAssociatedDomains / QueryAssociatedBundleNames         │
│  文件: interfaces/kits/js/jsi/src/app_domain_verify_manager_napi.cpp     │
│                                                                        │
│  1. 解析 JS 参数 (bundleName/domain)                                    │
│  2. 转换 C++ 类型                                                       │
│  3. 调用 Inner API                                                     │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 4. 调用 Inner API
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   AppDomainVerifyMgrClient                              │
│                                                                        │
│  client.QueryAssociatedDomains()                                        │
│  或                                                                     │
│  client.QueryAssociatedBundleNames()                                    │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 5. IPC 调用
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              IAppDomainVerifyMgrService (Binder IPC)                     │
│                                                                        │
│  QUERY_ASSOCIATED_DOMAINS / QUERY_ASSOCIATED_BUNDLE_NAMES               │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 6. 查询数据库
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│           AppDomainVerifyRdbDataManager::QueryAssociated()               │
│                                                                        │
│  SELECT domain FROM app_verify_status WHERE bundleName = ?              │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 7. 返回结果
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    返回 JS 数组                                           │
│                                                                        │
│  1. 序列化结果                                                          │
│  2. 转换 JS 类型 (std::vector → JS Array)                               │
│  3. 返回调用方                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 定时刷新调用链

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      定时刷新调用链                                       │
└─────────────────────────────────────────────────────────────────────────┘

系统定时器 (每 24 小时)
        │
        │ 1. BOOT_COMPLETED 或 timedevent 触发
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   AppDomainVerifyAgentService::OnStart()                 │
│  文件: services/src/agent/core/app_domain_verify_agent_service.cpp      │
│  行号: ~185                                                              │
│                                                                        │
│  1. 初始化 FFRT 任务管理器                                               │
│  2. 注册定时回调                                                         │
│  3. 启动刷新任务                                                         │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 2. 获取失败列表
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              AppDomainVerifyMgrService::QueryAllVerifyStatus()           │
│                                                                        │
│  查询所有状态为 VERIFY_FAILED 的记录                                     │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 3. 遍历重试
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    VerifyTask (每条记录)                                 │
│                                                                        │
│  1. 检查重试次数 (< 7 次)                                               │
│  2. 指数退避等待 (1h, 2h, 4h...)                                        │
│  3. 执行 SingleVerify                                                   │
└─────────────────────────────────────────────────────────────────────────┘
        │
        │ 4. 更新状态
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              AppDomainVerifyRdbDataManager::Update()                     │
│                                                                        │
│  UPDATE app_verify_status SET status = ?, retryCount = ? WHERE ...       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. 调用链统计

### 5.1 最长调用链

| 场景 | 调用深度 | 主要耗时操作 |
|-----|---------|------------|
| 应用安装校验 | 10 | HTTP 请求 + JSON 解析 + 签名验证 |
| 隐式跳转过滤 | 6 | 数据库查询 |
| JS API 查询 | 7 | IPC + 数据库查询 |
| 定时刷新 | 8 | HTTP 请求 + 数据库更新 |

### 5.2 关键路径耗时占比

| 操作 | 预估耗时占比 |
|-----|------------|
| DNS 解析 | ~10% |
| TCP/TLS 连接 | ~20% |
| HTTP 请求/响应 | ~40% |
| JSON 解析 | ~5% |
| 签名验证 | ~5% |
| 数据库操作 | ~10% |
| 其他 | ~10% |

---

## 6. 相关文档

| 文档 | 链接 |
|-----|------|
| 架构说明 | [01_Architecture.md](../01_Architecture.md) |
| Inner API | [02_Inner_API.md](../02_Inner_API.md) |
| N-API | [03_N_API.md](../03_N_API.md) |
| 安全风险评审 | [06_Security.md](../06_Security.md) |
