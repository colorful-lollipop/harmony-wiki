# 攻击面分析

**Distributed Data Manager Service (SA ID: 1301)**

---

## 目录

- [执行摘要](#执行摘要)
- [攻击面总览](#攻击面总览)
- [外部输入清单](#外部输入清单)
- [敏感操作清单](#敏感操作清单)
- [信任边界图](#信任边界图)
- [Feature 级攻击面](#feature-级攻击面)
- [输入处理分析](#输入处理分析)
- [权限绕过风险](#权限绕过风险)
- [网络攻击面](#网络攻击面)

---

## 执行摘要

### 关键发现

| 指标 | 数值 | 风险等级 |
|-----|------|---------|
| **SA ID** | 1301 | - |
| **进程名** | distributeddata | - |
| **Feature 数量** | 6 个 | - |
| **RPC 接口总数** | 125 个 | 🔴 高 |
| **高危 RPC** | ~35 个 | 🔴 高 |
| **中危 RPC** | ~55 个 | 🟡 中 |
| **输入验证点** | 待统计 | - |
| **权限检查点** | ~20 个 | - |

### 核心风险

1. **大规模 IPC 暴露**: 125 个 RPC 方法提供广泛攻击面
2. **跨设备信任边界**: 数据同步跨越设备安全边界
3. **复杂权限模型**: 多类权限 (DISTRIBUTED_DATASYNC, CLOUDDATA_CONFIG 等)
4. **数据持久化**: 处理敏感用户数据的加密/解密

---

## 攻击面总览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           攻击者视角                                         │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ 恶意应用 A   │  │ 恶意应用 B   │  │ 恶意设备 C   │  │ 网络中间人   │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                 │                 │                 │           │
│         └─────────────────┴─────────────────┴─────────────────┘           │
│                                   │                                       │
│                           ┌───────▼───────┐                               │
│                           │   IPC 接口    │                               │
│                           │  (125 方法)   │                               │
│                           └───────┬───────┘                               │
│                                   │                                       │
│         ┌─────────────────────────┼─────────────────────────┐             │
│         │                         │                         │             │
│  ┌──────▼──────┐          ┌───────▼────────┐       ┌───────▼────────┐   │
│  │  RDB (27)   │          │   KVDB (22)    │       │   Cloud (21)   │   │
│  └─────────────┘          └────────────────┘       └────────────────┘   │
│                                                                             │
│  ┌──────────────┐          ┌────────────────┐       ┌────────────────┐   │
│  │ Object (11)  │          │ DataShare (27) │       │   UDMF (17)    │   │
│  └──────────────┘          └────────────────┘       └────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 外部输入清单

### IPC 输入入口

所有客户端请求通过 `FeatureStubImpl::OnRemoteRequest()` 进入:

```cpp
// services/distributeddataservice/app/src/feature_stub_impl.h:31
int OnRemoteRequest(uint32_t code, MessageParcel &data, MessageParcel &reply,
    MessageOption &option) override;
```

**数据流**:
```
Client IPC → FeatureStubImpl → Feature::OnRemoteRequest() → 具体处理
```

### 输入类型分析

| 输入类型 | 来源 | 风险 | 验证点 |
|---------|------|------|-------|
| **MessageParcel** | IPC 调用 | 🔴 高 | 每个 RPC 方法 |
| **BundleName** | IPC 参数 | 🟡 中 | `BundleChecker` |
| **StoreId** | IPC 参数 | 🟡 中 | 长度/格式检查 |
| **SQL/Query** | 查询参数 | 🔴 高 | SQL 注入风险 |
| **File Path** | 配置/备份 | 🔴 高 | 路径遍历风险 |
| **Device ID** | 同步参数 | 🟡 中 | 格式验证 |
| **User ID** | 账户参数 | 🟡 中 | 范围检查 |

### 关键输入验证点

#### 1. Bundle 验证

```cpp
// services/distributeddataservice/app/src/checker/bundle_checker.cpp
bool BundleChecker::CheckBundleName(const std::string &bundleName) {
    // 验证 bundleName 格式
    // 检查信任/不信任列表
    // 通过 AccessTokenKit 验证 token
}
```

#### 2. StoreMetaData 验证

```cpp
// services/distributeddataservice/framework/include/metadata/store_meta_data.h
struct StoreMetaData {
    std::string appId;        // 应用 ID
    std::string bundleName;   // Bundle 名
    std::string storeId;      // 存储 ID
    int32_t user;             // 用户 ID
    std::string deviceId;     // 设备 ID
    // ...
};
```

#### 3. SQL 注入风险点

```cpp
// RDB 查询接口接收 SQL 语句或查询条件
// services/distributeddataservice/service/rdb/rdb_service_impl.cpp
// 需要验证 Predicate 和 Query 参数
```

---

## 敏感操作清单

### 权限检查矩阵

| Feature | 操作 | 所需权限 | 检查位置 |
|---------|-----|---------|---------|
| **所有** | 基础同步 | `ohos.permission.DISTRIBUTED_DATASYNC` | `PermissionValidator::CheckSyncPermission()` |
| **Cloud** | 云配置 | `ohos.permission.CLOUDDATA_CONFIG` | `PermissionValidator::IsCloudConfigPermit()` |
| **UDMF** | UTD 管理 | `ohos.permission.MANAGE_DYNAMIC_UTD_TYPE` | `VerifyPermission()` |
| **DataShare** | 静默访问 | 特殊授权 | `PermitDelegate::VerifyPermission()` |

### 高危操作列表

#### 🔴 数据删除

| Feature | 方法 | 路径 | 风险 |
|---------|-----|------|------|
| RDB | `OnDelete` | `rdb_service_stub.cpp` | 删除整个数据库 |
| KVDB | `OnDelete` | `kvdb_service_stub.cpp` | 删除 KV 存储 |
| KVDB | `OnRemoveDeviceData` | `kvdb_service_stub.cpp` | 删除设备数据 |
| Cloud | `OnClean` | `cloud_service_stub.cpp` | 清理云数据 |
| UDMF | `OnDeleteData` | `udmf_service_stub.cpp` | 删除 UDMF 数据 |
| DataShare | `OnDeleteEx` | `data_share_service_stub.cpp` | 删除共享数据 |

#### 🔴 权限变更

| Feature | 方法 | 路径 | 风险 |
|---------|-----|------|------|
| Cloud | `OnChangePrivilege` | `cloud_service_stub.cpp` | 修改数据权限 |
| Cloud | `OnShare` / `OnUnshare` | `cloud_service_stub.cpp` | 分享/取消分享 |
| UDMF | `OnAddPrivilege` | `udmf_service_stub.cpp` | 添加权限 |
| UDMF | `OnSetAppShareOption` | `udmf_service_stub.cpp` | 设置分享选项 |
| DataShare | `OnSetSilentSwitch` | `data_share_service_stub.cpp` | 设置静默开关 |

#### 🔴 密码/密钥获取

| Feature | 方法 | 路径 | 风险 |
|---------|-----|------|------|
| RDB | `OnGetPassword` | `rdb_service_stub.cpp` | 获取数据库密码 |
| KVDB | `OnGetBackupPassword` | `kvdb_service_stub.cpp` | 获取备份密码 |

#### 🔴 配置修改

| Feature | 方法 | 路径 | 风险 |
|---------|-----|------|------|
| Cloud | `OnSetGlobalCloudStrategy` | `cloud_service_stub.cpp` | 全局策略修改 |
| Cloud | `OnEnableCloud` / `OnDisableCloud` | `cloud_service_stub.cpp` | 云开关 |
| RDB | `OnSetConfig` | `rdb_service_stub.cpp` | RDB 配置 |
| KVDB | `OnSetConfig` | `kvdb_service_stub.cpp` | KVDB 配置 |

### 系统服务调用

**Adapter 层调用外部系统服务**:

| Adapter | 系统服务 | API | 风险 |
|---------|---------|-----|------|
| Account | OhosAccountKits | `QueryOhosAccountInfo()` | 账户信息泄露 |
| Account | OsAccountManager | `QueryActiveOsAccountIds()` | 用户信息枚举 |
| DeviceManager | DeviceManager | `GetTrustedDeviceList()` | 设备信息泄露 |
| DeviceManager | DeviceManager | `IsSameAccount()` | 账户关联信息 |
| Network | NetConnClient | `GetDefaultNet()` | 网络信息 |
| ScreenLock | ScreenLockManager | `IsScreenLocked()` | 锁屏状态 |
| DFX | HiSysEvent | `OH_HiSysEvent_Write()` | 日志注入 |

---

## 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           信任边界分析                                       │
└─────────────────────────────────────────────────────────────────────────────┘

【安全域层次】

Level 5: 内核态 (Kernel)
    │
    │ 系统调用
    ▼
Level 4: 系统服务 (System Services)
    │
    │ IPC (SA ID: 1301)
    ▼
Level 3: 本服务 (DDS)
    │ ┌──────────────────────────────────────────┐
    │ │  App Layer → Service Layer → Framework   │
    │ │  (processing trust boundary)             │
    │ └──────────────────────────────────────────┘
    │
    │ SoftBus / 网络
    ▼
Level 2: 远端设备 (Remote Devices)
    │
    │ 跨设备同步
    ▼
Level 1: 云服务 (Cloud Services)


【数据流信任边界】

┌─────────────────────────────────────────────────────────────────────────────┐
│                              应用进程 (不可信)                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  Bundle A                    │  Bundle B (不同账户)                     ││
│  └──────────────┬───────────────┴──────────────────┬──────────────────────┘│
└─────────────────┼──────────────────────────────────┼───────────────────────┘
                  │ IPC                              │ IPC
                  ▼                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DDS 服务进程 (SA 1301)                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  【边界1: IPC 验证】                                                    ││
│  │  - AccessTokenKit 验证                                                  ││
│  │  - BundleChecker 验证                                                   ││
│  │  - PermissionValidator 权限检查                                         ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  【边界2: 数据隔离】                                                    ││
│  │  (user, app, database) 三元组隔离                                       ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                        │
│                                    ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  【边界3: 同步决策】                                                    ││
│  │  - 设备认证 (DeviceManager)                                             ││
│  │  - 账户匹配检查                                                         ││
│  │  - 访问控制检查                                                         ││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
                  │
                  │ SoftBus (加密通道)
                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          远端设备 (同级信任)                                  │
│  - 设备间通过 DSoftBus 建立加密通道                                          │
│  - 同步前进行设备认证和账户匹配                                               │
└─────────────────────────────────────────────────────────────────────────────┘
                  │
                  │ HTTPS/TLS
                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          云端服务 (外部信任域)                                │
│  - 需要云账号认证                                                            │
│  - 数据传输加密                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Feature 级攻击面

### RDB Feature 攻击面

**关键文件**: `services/distributeddataservice/service/rdb/`

```
攻击入口:
├─ OnRemoteDoSync/OnRemoteDoAsync (同步操作)
│   ├─ 风险: 强制同步大量数据 → DoS
│   ├─ 风险: 同步到未授权设备
│   └─ 检查: CheckPermission(), 设备认证
│
├─ OnRemoteDoRemoteQuery (远程查询)
│   ├─ 风险: SQL 注入 (如使用原生 SQL)
│   ├─ 风险: 数据泄露到其他应用
│   └─ 检查: Predicate 验证, 权限检查
│
├─ OnDelete (删除数据库)
│   ├─ 风险: 恶意删除数据
│   └─ 检查: 需要 DISTRIBUTED_DATASYNC 权限
│
├─ OnGetPassword (获取密码)
│   ├─ 风险: 密码泄露
│   └─ 检查: 需要特殊权限验证
│
└─ OnRemoteSetDistributedTables
    ├─ 风险: 修改分布式表配置
    └─ 检查: 表名白名单验证
```

### KVDB Feature 攻击面

**关键文件**: `services/distributeddataservice/service/kvdb/`

```
攻击入口:
├─ OnSync/OnCloudSync
│   ├─ 风险: 强制高频同步 → DoS
│   ├─ 风险: 同步恶意数据
│   └─ 检查: 流控 (FlowControlManager)
│
├─ OnPutSwitch (写入开关数据)
│   ├─ 风险: 篡改系统开关
│   └─ 检查: 需要系统权限
│
├─ OnRemoveDeviceData
│   ├─ 风险: 删除其他设备数据
│   └─ 检查: 设备所有权验证
│
└─ OnSetConfig
    ├─ 风险: 修改服务配置
    └─ 检查: 配置范围限制
```

### Cloud Feature 攻击面

**关键文件**: `services/distributeddataservice/service/cloud/`

```
攻击入口:
├─ OnEnableCloud/OnDisableCloud
│   ├─ 风险: 禁用用户云同步
│   └─ 检查: 需要 CLOUDDATA_CONFIG 权限
│
├─ OnShare/OnUnshare/OnChangePrivilege
│   ├─ 风险: 分享数据到未授权用户
│   ├─ 风险: 提升自身权限
│   └─ 检查: 分享权限验证, 邀请确认
│
├─ OnCloudSync
│   ├─ 风险: 强制高频云同步 → DoS/费用攻击
│   └─ 检查: 流控, 用户配额
│
├─ OnClean
│   ├─ 风险: 清除用户云数据
│   └─ 检查: 需要系统权限
│
└─ OnSetGlobalCloudStrategy
    ├─ 风险: 修改全局策略影响所有用户
    └─ 检查: 需要系统权限
```

### DataShare Feature 攻击面

**关键文件**: `services/distributeddataservice/service/data_share/`

```
攻击入口:
├─ OnSetSilentSwitch (静默开关)
│   ├─ 风险: 启用静默访问绕过正常权限检查
│   └─ 检查: 需要系统权限 PermitDelegate
│
├─ OnPublishProxyData/OnDeleteProxyData
│   ├─ 风险: 发布虚假数据
│   ├─ 风险: 删除他人数据
│   └─ 检查: 提供者身份验证
│
├─ OnInsertEx/OnUpdateEx/OnDeleteEx
│   ├─ 风险: 修改共享数据
│   └─ 检查: URI 权限检查
│
└─ OnNotifyObserver
    ├─ 风险: 伪造数据变更通知
    └─ 检查: 变更来源验证
```

### UDMF Feature 攻击面

**关键文件**: `services/distributeddataservice/service/udmf/`

```
攻击入口:
├─ OnSetData/OnUpdateData
│   ├─ 风险: 写入超大数据 → 存储耗尽
│   ├─ 风险: 写入恶意序列化数据
│   └─ 检查: 数据大小限制, 类型验证
│
├─ OnAddPrivilege
│   ├─ 风险: 为自己添加数据访问权限
│   └─ 检查: 数据所有权验证
│
├─ OnSync
│   ├─ 风险: 同步恶意数据到其他设备
│   └─ 检查: 设备认证, 权限检查
│
└─ URI 处理
    ├─ 风险: URI 解析漏洞
    ├─ 风险: 目录遍历通过 file:// URI
    └─ 检查: URI 规范化, 权限验证
```

---

## 输入处理分析

### IPC 参数解析风险

```cpp
// 典型 RPC 方法处理流程
int FeatureXxxStub::OnXxx(MessageParcel &data, MessageParcel &reply) {
    // 1. 读取 IPC 参数
    std::string bundleName = data.ReadString();
    std::string storeId = data.ReadString();
    int32_t userId = data.ReadInt32();
    
    // 2. 参数验证 (关键安全检查点)
    if (!ValidateBundleName(bundleName)) {
        return E_INVALID_ARGS;
    }
    
    // 3. 权限检查
    if (!CheckPermission(tokenId, requiredPermission)) {
        return E_NO_PERMISSION;
    }
    
    // 4. 业务处理
    // ...
}
```

### 潜在漏洞模式

#### 1. 整数溢出

```cpp
// 风险: 读取长度字段后未验证
size_t dataLen = data.ReadUint32();
char *buffer = new char[dataLen];  // 可能溢出
```

#### 2. 字符串截断

```cpp
// 风险: 未验证字符串长度
std::string path = data.ReadString();
// path 可能过长或包含特殊字符
```

#### 3. 数组越界

```cpp
// 风险: 使用 IPC code 作为数组索引未验证范围
return (this->*HANDLERS[code])(data, reply);
```

### 安全编码建议

1. **严格验证 IPC 参数**
   - 长度检查
   - 范围检查
   - 格式检查

2. **防御性编程**
   - 所有外部输入视为不可信
   - 失败安全 (fail-safe)

3. **权限最小化**
   - 每个 RPC 方法声明所需权限
   - 动态权限检查

---

## 权限绕过风险

### 已知权限检查机制

| 机制 | 实现 | 风险 |
|-----|------|------|
| **Token 验证** | `AccessTokenKit::GetTokenTypeFlag()` | Token 伪造 |
| **权限验证** | `AccessTokenKit::VerifyAccessToken()` | 权限提升 |
| **Bundle 验证** | `BundleChecker::CheckBundleName()` | Bundle 欺骗 |
| **设备验证** | `DeviceManager::IsSameAccount()` | 设备伪装 |
| **跨账户验证** | `VerifyAcrossAccountsPermission()` | 账户隔离绕过 |

### 潜在绕过场景

#### 场景 1: Bundle 名欺骗

```
攻击者注册与合法应用相似的 Bundle 名
→ BundleChecker 可能误判
→ 获得非法数据访问权限
```

**检查点**: `services/distributeddataservice/app/src/checker/bundle_checker.cpp`

#### 场景 2: Token 重用

```
攻击者获取合法应用的 token
→ 在其他上下文中重用
→ 绕过身份验证
```

**检查点**: `AccessTokenKit::VerifyAccessToken()` 调用处

#### 场景 3: 设备伪装

```
攻击者模拟可信设备 ID
→ 通过设备认证
→ 接收不应获得的数据同步
```

**检查点**: `DeviceManagerDelegate` 设备验证

---

## 网络攻击面

### SoftBus 通信

**通道建立**:
```
Device A (DDS) ←──SoftBus──→ Device B (DDS)
               (加密通道)
```

**潜在风险**:
1. **中间人攻击**: 如 SoftBus 证书验证存在缺陷
2. **重放攻击**: 同步消息被截获重放
3. **DoS**: 大量同步请求耗尽资源

### 云端通信

**通道**:
```
DDS → HTTPS → Cloud Server
```

**潜在风险**:
1. **证书固定绕过**: 如未正确验证服务器证书
2. **API 滥用**: 高频云同步导致费用问题
3. **数据泄露**: 云传输加密缺陷

---

## 风险评估矩阵

| 威胁 | 可能性 | 影响 | 风险等级 | 优先级 |
|-----|-------|------|---------|-------|
| IPC 参数注入 | 中 | 高 | 🔴 高 | P1 |
| 权限绕过 | 低 | 高 | 🟡 中 | P2 |
| 数据泄露 | 中 | 高 | 🔴 高 | P1 |
| DoS (资源耗尽) | 高 | 中 | 🟡 中 | P2 |
| 跨设备数据污染 | 中 | 中 | 🟡 中 | P2 |
| 云同步滥用 | 中 | 低 | 🟢 低 | P3 |

---

## 相关文档

- [IPC 接口清单](04_Interface.md) - 完整 RPC 方法列表
- [安全评审](03_Security.md) - 详细风险评估
- [架构设计](01_Architecture.md) - 系统架构理解

---

**证据来源**:
- Stub 定义: `services/distributeddataservice/service/*/`*_service_stub.h
- 权限检查: `services/distributeddataservice/service/permission/`
- Bundle 检查: `services/distributeddataservice/app/src/checker/`
