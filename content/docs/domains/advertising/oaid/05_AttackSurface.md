# OAID 攻击面分析

## 攻击面概览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            OAID 攻击面全景                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐                                                        │
│  │   外部攻击者     │                                                        │
│  └────────┬────────┘                                                        │
│           │                                                                 │
│           ▼                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      攻击入口 (Attack Surface)                       │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  ① N-API 层        ② IPC 接口         ③ 文件系统         ④ 配置    │   │
│  │  ┌─────────┐      ┌─────────┐        ┌─────────┐       ┌─────────┐ │   │
│  │  │ JS 参数 │      │ IPC 消息│        │ JSON文件│       │ 启动配置│ │   │
│  │  │ 解析    │      │ 处理    │        │ 解析    │       │ 读取    │ │   │
│  │  └─────────┘      └─────────┘        └─────────┘       └─────────┘ │   │
│  │       │                │                  │                │        │   │
│  │       ▼                ▼                  ▼                ▼        │   │
│  │  ┌─────────┐      ┌─────────┐        ┌─────────┐       ┌─────────┐ │   │
│  │  │类型混淆 │      │权限绕过 │        │路径遍历 │       │配置注入│ │   │
│  │  │DoS      │      │UID伪造 │        │JSON注入 │       │        │ │   │
│  │  └─────────┘      └─────────┘        └─────────┘       └─────────┘ │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 外部输入清单

### 1. N-API 参数输入

| 输入点 | 文件 | 行号 | 输入类型 | 说明 |
|--------|------|------|---------|------|
| `ParseParameters` | `oaid.cpp` | 117-130 | JS 回调函数 | `getOAID(callback)` 的参数 |
| `argc` 计数 | `oaid.cpp` | 120 | 整数 | 参数数量检查 |
| `valuetype` 检查 | `oaid.cpp` | 123-125 | 类型枚举 | 参数类型验证 |

**代码证据**:
```cpp
// interfaces/kits/js/napi/oaid/src/oaid.cpp:117-130
napi_value ParseParameters(
    const napi_env &env, const napi_value (&argv)[OAID_MAX_PARA], const size_t &argc, napi_ref &callback)
{
    NAPI_ASSERT(env, argc >= OAID_MAX_PARA - 1, "Wrong number of arguments.");
    
    napi_valuetype valuetype = napi_undefined;
    if (argc >= OAID_MAX_PARA) {
        NAPI_CALL(env, napi_typeof(env, argv[0], &valuetype));
        NAPI_ASSERT(env, valuetype == napi_function, "Wrong argument type, function expected.");
    }
}
```

**风险分析**:
- 参数数量检查 `argc >= OAID_MAX_PARA - 1` 正确
- 类型检查 `valuetype == napi_function` 正确
- **低风险**: N-API 框架已做边界保护

---

### 2. IPC 参数输入

| 输入点 | 文件 | 行号 | 输入类型 | 说明 |
|--------|------|------|---------|------|
| `OnRemoteRequest` | `oaid_service_stub.cpp` | 162-193 | IPC MessageParcel | IPC 请求主入口 |
| `code` 参数 | `oaid_service_stub.cpp` | 162 | uint32_t | IPC 命令码 |
| `data` Parcel | `oaid_service_stub.cpp` | 162 | MessageParcel | IPC 数据包 |
| `bundleName` | `oaid_service_stub.cpp` | 170 | string | 调用者包名 |
| `uid` | `oaid_service_stub.cpp` | 169 | pid_t | 调用者 UID |

**代码证据**:
```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:162-193
int32_t OAIDServiceStub::OnRemoteRequest(
    uint32_t code, MessageParcel &data, MessageParcel &reply, MessageOption &option)
{
    pid_t uid = IPCSkeleton::GetCallingUid();
    DelayedSingleton<BundleMgrHelper>::GetInstance()->GetBundleNameByUid(static_cast<int>(uid), bundleName);
    
    // 权限检查（在 InterfaceToken 验证前！）
    if (code == GET_OAID && !CheckPermission(OAID_TRACKING_CONSENT_PERMISSION)) {
        return IPCObjectStub::OnRemoteRequest(code, data, reply, option);
    }
    
    // InterfaceToken 验证
    std::u16string myDescripter = OAIDServiceStub::GetDescriptor();
    std::u16string remoteDescripter = data.ReadInterfaceToken();
    if (myDescripter != remoteDescripter) {
        return ERR_SYSYTEM_ERROR;
    }
    return SendCode(code, data, reply);
}
```

**风险分析**:
- ✅ `GetCallingUid()` / `GetCallingTokenID()` 由 IPC 框架提供，可信
- ✅ `GetBundleNameByUid()` 从系统服务获取，可信
- ⚠️ **顺序问题**: 权限检查在 `InterfaceToken` 验证之前（行171 vs 行186-191）
- **中风险**: 命令码 `code` 未在 `SendCode` 中验证范围，但枚举类型限制

---

### 3. 文件系统输入

#### 3.1 配置文件读取

| 输入点 | 文件 | 行号 | 输入类型 | 路径 |
|--------|------|------|---------|------|
| `LoadAndCheckOaidTrustList` | `oaid_service_stub.cpp` | 93-141 | JSON 文件 | `/etc/advertising/oaid/oaid_service_config*.json` |
| `checkProviderBundleName` | `oaid_service_stub.cpp` | 234-273 | JSON 文件 | 同上 |
| `getWantInfo` | `connect_ads_stub.cpp` | 216-266 | JSON 文件 | 同上 |

**代码证据**:
```cpp
// services/oaid_manager/src/oaid_service_stub.cpp:95-104
bool LoadAndCheckOaidTrustList(const std::string &bundleName)
{
    char pathBuff[PATH_MAX] = {0};
    GetOneCfgFile(OAID_TRUSTLIST_EXTENSION_CONFIG_PATH.c_str(), pathBuff, PATH_MAX);
    char realPath[PATH_MAX] = {0};
    if (realpath(pathBuff, realPath) == nullptr) {
        GetOneCfgFile(OAID_TRUSTLIST_CONFIG_PATH.c_str(), pathBuff, PATH_MAX);
        if (realpath(pathBuff, realPath) == nullptr) {
            return false;
        }
    }
    std::ifstream inFile(realPath, std::ios::in);
    std::string fileContent((std::istreambuf_iterator<char>(inFile)), std::istreambuf_iterator<char>());
    cJSON *root = cJSON_Parse(fileContent.c_str());
    // ...
}
```

**风险分析**:
- ✅ 使用 `realpath()` 规范化路径
- ⚠️ **路径遍历风险**: 未验证 `realPath` 是否在 `/etc/advertising/oaid/` 目录内
- ⚠️ **JSON 注入**: `cJSON_Parse` 无大小限制，大文件可导致内存耗尽
- ⚠️ **字符串读取**: 使用 `istreambuf_iterator` 读取整个文件到内存

#### 3.2 更新检查文件

| 输入点 | 文件 | 行号 | 输入类型 | 路径 |
|--------|------|------|---------|------|
| `GainOAID` | `oaid_service.cpp` | 241-260 | JSON 文件 | `/data/service/el1/public/database/oaid_service_manager/update_check.json` |

**代码证据**:
```cpp
// services/oaid_manager/src/oaid_service.cpp:241-260
std::string OAIDService::GainOAID()
{
    if (OAIDFileOperator::IsFileExsit(OAID_UPDATE)) {
        OAIDFileOperator::OpenAndReadFile(OAID_UPDATE, oaidKvStoreStr);
        OAIDFileOperator::ClearFile(OAID_UPDATE);
        cJSON *root = cJSON_Parse(oaidKvStoreStr.c_str());
        if (root != nullptr && !cJSON_IsInvalid(root)) {
            cJSON *oaidObj = cJSON_GetObjectItem(root, "oaid");
            if (cJSON_IsString(oaidObj)) {
                oaid = oaidObj->valuestring;
            }
        }
        cJSON_Delete(root);
    }
}
```

**风险分析**:
- ⚠️ **TOCTOU**: `IsFileExsit` 和 `OpenAndReadFile` 之间存在竞态窗口
- ⚠️ **JSON 注入**: 同上，无大小限制
- ⚠️ **路径检查**: `OAID_UPDATE` 为硬编码路径，但 `ClearFile` 使用相对路径

---

### 4. 数据库输入

| 输入点 | 文件 | 行号 | 输入类型 | 说明 |
|--------|------|------|---------|------|
| `ReadValueFromKvStore` | `oaid_service.cpp` | 193-214 | KVStore | OAID 存储值 |
| `ReadValueFromUnderAgeKvStore` | `oaid_service.cpp` | 387-403 | KVStore | 未成年信息 |

**风险分析**:
- ✅ 数据库存储加密 (`options.encrypt = true`)
- ✅ 安全级别 S1
- ✅ 互斥锁保护
- **低风险**: 数据受 KVStore 框架保护

---

## 敏感操作清单

### 1. 权限检查操作

| 操作 | 文件 | 行号 | 说明 |
|------|------|------|------|
| `CheckPermission` | `oaid_service_stub.cpp` | 45-80 | 服务端权限验证 |
| `CheckSystemApp` | `oaid_service_stub.cpp` | 82-91 | 系统应用检查 |
| `CheckPermission` (Client) | `oaid_service_client.cpp` | 185-203 | 客户端权限验证 |

**权限检查流程**:
```
调用者
    │
    ├──► Client::CheckPermission() ──► VerifyAccessToken()
    │
    └──► (IPC)
         │
         └──► Stub::OnRemoteRequest()
              │
              ├──► CheckPermission() ──► VerifyAccessToken()
              │
              └──► [ResetOAID only]
                   ├──► LoadAndCheckOaidTrustList()
                   └──► CheckSystemApp()
```

**潜在风险**:
- Client 和 Server 双重检查可能不一致
- 代理调用场景下 `firstCallToken` 处理复杂

### 2. 文件操作

| 操作 | 文件 | 行号 | 说明 |
|------|------|------|------|
| `OpenAndReadFile` | `oaid_file_operator.cpp` | 40-54 | 文件读取 |
| `ClearFile` | `oaid_file_operator.cpp` | 56-71 | 文件删除 |
| `IsFileExsit` | `oaid_file_operator.cpp` | 28-38 | 文件存在检查 |

**代码证据**:
```cpp
// utils/native/src/oaid_file_operator.cpp:56-71
bool OAIDFileOperator::ClearFile(const std::string &fileName)
{
    struct stat statbuf {};
    if (lstat(fileName.c_str(), &statbuf) != 0) {
        return false;
    }
    if (S_ISREG(statbuf.st_mode)) {
        if (access(fileName.c_str(), F_OK) != 0) {
            return true;
        }
        remove(fileName.c_str());  // TOCTOU!
    }
    return true;
}
```

**风险**: `lstat` → `access` → `remove` 三步操作存在 TOCTOU 竞争条件

### 3. 数据库操作

| 操作 | 文件 | 行号 | 说明 |
|------|------|------|------|
| `Get` | `oaid_service.cpp` | 204 | KVStore 读取 |
| `Put` | `oaid_service.cpp` | 227 | KVStore 写入 |
| `InitKvStore` | `oaid_service.cpp` | 327-375 | 数据库初始化 |

### 4. 系统调用

| 调用 | 文件 | 行号 | 说明 |
|------|------|------|------|
| `RAND_bytes` | `oaid_service.cpp` | 57, 71 | OpenSSL 随机数生成 |
| `realpath` | `oaid_service_stub.cpp` | 98, 100, 239, 241 | 路径规范化 |
| `access` | `oaid_file_operator.cpp` | 34, 64 | 文件访问检查 |
| `lstat` | `oaid_file_operator.cpp` | 59 | 文件状态获取 |
| `remove` | `oaid_file_operator.cpp` | 68 | 文件删除 |

### 5. IPC 调用

| 操作 | 文件 | 行号 | 说明 |
|------|------|------|------|
| `SendRequest` | `connect_ads_stub.cpp` | 177 | 向 Ads Service 发送消息 |
| `ConnectServiceExtensionAbility` | `connect_ads_stub.cpp` | 362 | 连接扩展服务 |

---

## 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界分析                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         非信任域                                     │   │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │   │
│  │  │  恶意应用    │    │  普通应用    │    │  系统应用    │             │   │
│  │  │  (沙盒内)    │    │  (沙盒内)    │    │  (受限权限)  │             │   │
│  │  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘             │   │
│  │         │                  │                  │                     │   │
│  │         └──────────────────┴──────────────────┘                     │   │
│  │                            │                                        │   │
│  │                            ▼                                        │   │
│  │                   ┌─────────────────┐                               │   │
│  │                   │   N-API 层      │                               │   │
│  │                   │  (参数解析)      │                               │   │
│  │                   └────────┬────────┘                               │   │
│  └────────────────────────────┼────────────────────────────────────────┘   │
│                               │                                             │
│  ═══════════════════════════════════════════════════════════════════════  │
│                               │              边界 1: JS → C++              │
│  ═══════════════════════════════════════════════════════════════════════  │
│                               │                                             │
│  ┌────────────────────────────┼────────────────────────────────────────┐   │
│  │                         半信任域                                     │   │
│  │                            │                                        │   │
│  │                   ┌────────▼────────┐                               │   │
│  │                   │  Client SDK     │                               │   │
│  │                   │ (权限检查1)     │                               │   │
│  │                   └────────┬────────┘                               │   │
│  │                            │                                        │   │
│  │                            ▼                                        │   │
│  │                   ┌─────────────────┐                               │   │
│  │                   │    IPC 通道      │                               │   │
│  │                   └────────┬────────┘                               │   │
│  └────────────────────────────┼────────────────────────────────────────┘   │
│                               │                                             │
│  ═══════════════════════════════════════════════════════════════════════  │
│                               │              边界 2: IPC                   │
│  ═══════════════════════════════════════════════════════════════════════  │
│                               │                                             │
│  ┌────────────────────────────┼────────────────────────────────────────┐   │
│  │                         信任域                                       │   │
│  │                            │                                        │   │
│  │                   ┌────────▼────────┐                               │   │
│  │                   │  Service Stub   │                               │   │
│  │                   │ (权限检查2)     │                               │   │
│  │                   └────────┬────────┘                               │   │
│  │                            │                                        │   │
│  │                   ┌────────▼────────┐                               │   │
│  │                   │  OAIDService    │                               │   │
│  │                   │ (核心业务逻辑)  │                               │   │
│  │                   └────────┬────────┘                               │   │
│  │                            │                                        │   │
│  │         ┌──────────────────┼──────────────────┐                    │   │
│  │         ▼                  ▼                  ▼                    │   │
│  │  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐              │   │
│  │  │  KVStore    │   │  文件系统    │   │ Ads Service │              │   │
│  │  │ (加密存储)  │   │ (配置读取)  │   │ (扩展连接)  │              │   │
│  │  └─────────────┘   └─────────────┘   └─────────────┘              │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 边界跨越点安全控制

| 边界 | 位置 | 安全控制 |
|------|------|---------|
| **边界 1** (JS → C++) | `oaid.cpp:117-130` | N-API 参数类型/数量检查 |
| **边界 2** (IPC) | `oaid_service_stub.cpp:162-193` | InterfaceToken + 权限校验 |

---

## 攻击路径分析

### 攻击路径 1: 权限绕过（低概率）

```
攻击者目标: 获取 OAID 而不申请权限
    │
    ├──► 尝试直接调用 IPC (绕过 N-API)
    │      │
    │      ├──► Stub::OnRemoteRequest
    │      │        │
    │      │        ├──► CheckPermission
    │      │        │        ├──► GetCallingTokenID
    │      │        │        └──► VerifyAccessToken
    │      │        │
    │      │        └──► ❌ 权限检查失败
    │      │
    │      └──► ❌ 攻击失败
    │
    └──► 需要 TOKEN_HAP + 已授权权限
```

### 攻击路径 2: JSON 拒绝服务（可行）

```
攻击者目标: 导致 OAID 服务崩溃或内存耗尽
    │
    ├──► 条件: 能够写入配置文件 (需要 root)
    │
    ├──► 写入超大 JSON 到 /etc/advertising/oaid/oaid_service_config.json
    │      │
    │      ├──► Service 启动时读取配置
    │      ├──► cJSON_Parse 解析超大文件
    │      ├──► 内存耗尽 (OOM)
    │      │
    │      └──► ✅ 攻击成功 (需要 root 权限)
    │
    └──► 缓解: 配置文件通常需要 root 才能修改
```

### 攻击路径 3: TOCTOU 竞争（低影响）

```
攻击者目标: 阻止 update_check.json 被清除
    │
    ├──► 时间线:
    │      T1: IsFileExsit(OAID_UPDATE) → true
    │      T2: [攻击者替换/锁定文件]
    │      T3: OpenAndReadFile 失败 或 ClearFile 失败
    │
    ├──► 影响: 可能导致 OAID 重复更新或处理失败
    │
    └──► ✅ 攻击可行但影响有限
```

---

## 攻击面汇总表

| 攻击面 | 风险等级 | 攻击类型 | 可行性 | 影响 |
|--------|---------|---------|--------|------|
| N-API 参数 | 低 | 参数注入 | 低 | 低 |
| IPC 消息 | 中 | 权限绕过 | 低 | 中 |
| 配置文件 | 中 | JSON DoS | 中 (需 root) | 中 |
| 文件系统 | 中 | 路径遍历 | 低 | 中 |
| KVStore | 低 | 数据篡改 | 低 | 高 (但加密) |
| IPC 通信 | 低 | 中间人 | 极低 | 高 (框架保护) |

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构分析](02_Architecture.md)
- [接口文档](04_Interface.md)
- [安全风险评估](06_SecurityReview.md)
