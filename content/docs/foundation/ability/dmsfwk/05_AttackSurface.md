# 05_AttackSurface - 攻击面分析

## 攻击面总览

```
                    ┌──────────────────────────────────────────┐
                    │            外部攻击者                     │
                    └─────────────────┬────────────────────────┘
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          │                           │                           │
          ▼                           ▼                           ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│    JS/N-API     │     │  IPC/SoftBus    │     │   配置文件      │
│   接口层        │     │   网络层        │     │   攻击          │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                      dmsfwk 系统服务层                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ dtbschedmgr │  │dtbabilitymgr│  │      dtbcollabmgr       │  │
│  │   (SA 1401) │  │  (SA 1404)  │  │   (协作管理服务)         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. N-API 攻击面

### 1.1 输入入口

| 接口 | 入口点 | 参数类型 | 风险等级 |
|-----|-------|---------|---------|
| `register()` | `js_continuation_manager.cpp:126` | JSON Object | 中 |
| `startDeviceManager()` | `js_continuation_manager.cpp:84` | Token + Options | 中 |
| `createAbilityConnectionSession()` | `js_ability_connection_manager.cpp` | BundleName/ModuleName | 高 |
| `sendMessage()` | `js_ability_connection_manager.cpp` | String/Data | 高 |
| `sendData()` | `js_ability_connection_manager.cpp` | ArrayBuffer | 高 |

### 1.2 参数注入风险

**BundleName 注入**: `services/dtbcollabmgr/src/ability_connection_manager/ability_connection_manager.cpp:465`
```cpp
// 风险点：未验证 bundleName 格式
if (peerInfo.bundleName.empty()) {
    HILOGE("bundleName is empty!");
    return INVALID_PARAMETERS_ERR;
}
// TODO: 缺乏对 bundleName 格式的正则校验
```

**SurfaceId 注入风险**: 
- 攻击者可传入恶意构造的 SurfaceId
- 影响：可能导致 Surface 劫持或信息泄露

### 1.3 数据大小限制

| 数据类型 | 最大限制 | 限制位置 |
|---------|---------|---------|
| Message | 无显式限制 | TODO |
| Data | `BINARY_DATA_MAX_LEN` (4MB) | `dsched_softbus_session.h` |
| File | `MAX_FILE_COUNT` (500) | `channel_manager.h` |

---

## 2. IPC 攻击面

### 2.1 IPC 接口入口

| 接口 | Code | 权限检查 | 位置 |
|-----|------|---------|------|
| `StartRemoteAbility` | 1 | VerifyAccessToken | `distributed_sched_stub.cpp:271` |
| `ContinueMission` | 16 | IsFoundationCall | `distributed_sched_stub.cpp:578` |
| `ConnectRemoteAbility` | 6 | IsFoundationCall | `distributed_sched_stub.cpp:905` |
| `RegisterDSchedEventListener` | 81 | CheckCallingUid | `distributed_sched_stub.cpp:1231` |

### 2.2 IPC 反序列化风险

**DistributedWant 反序列化**: `services/dtbschedmgr/src/distributedWant/distributed_want.cpp:1070`
```cpp
bool DistributedWant::ReadFromParcel(Parcel& parcel)
{
    // 读取 Flags
    int32_t flags = parcel.ReadInt32();
    // 读取 Element
    if (!element_->ReadFromParcel(parcel)) {  // 风险点
        return false;
    }
    // 读取 Params
    wantParams_ = std::make_shared<DistributedWantParams>();
    if (!wantParams_->ReadFromParcel(parcel)) {  // 风险点
        return false;
    }
}
```

**风险**: 恶意构造的 Parcel 可能导致越界读取或拒绝服务

### 2.3 权限绕过风险

**IsFoundationCall 检查**: `services/dtbschedmgr/src/distributed_sched_permission.cpp:618`
```cpp
bool DistributedSchedPermission::IsFoundationCall() const
{
    uint32_t accessToken = IPCSkeleton::GetCallingTokenID();
    AccessToken::NativeTokenInfo nativeTokenInfo;
    int32_t result = AccessToken::AccessTokenKit::GetNativeTokenInfo(accessToken, nativeTokenInfo);
    if (result == ERR_OK && nativeTokenInfo.processName == FOUNDATION_PROCESS_NAME) {
        return true;
    }
    return false;
}
```

**分析**: 
- 仅检查进程名，不检查签名
- 风险：如果攻击者能在 foundation 进程中执行代码，可绕过权限检查

---

## 3. SoftBus 网络攻击面

### 3.1 网络入口点

| 回调 | 位置 | 处理函数 | 风险 |
|-----|------|---------|------|
| `OnBind` | `dsched_transport_softbus_adapter.cpp:427` | 连接建立 | 中 |
| `OnBytes` | `dsched_transport_softbus_adapter.cpp:534` | **数据接收** | **高** |
| `OnShutdown` | `dsched_transport_softbus_adapter.cpp:453` | 连接断开 | 低 |
| `OnBytesRecv` | `channel_manager.cpp:1343` | 通道数据 | **高** |

### 3.2 数据包解析风险

**SessionDataHeader 解析**: `services/dtbschedmgr/src/softbus_adapter/transport/dsched_softbus_session.cpp:110`
```cpp
void DSchedSoftbusSession::OnBytesReceived(std::shared_ptr<DSchedDataBuffer> buffer)
{
    // 校验 1：最小长度检查
    if (buffer->Size() < BINARY_HEADER_FRAG_LEN) {
        HILOGE("pack recv data error, size too small");
        return;
    }
    
    // 解析 TLV Header
    PackRecvData(buffer);
}
```

**TLV 解析风险**: `dsched_softbus_session.cpp:162-207`
```cpp
int32_t DSchedSoftbusSession::ReadTlvToHeader(...)
{
    // 风险：基于输入数据计算偏移量
    while (index < totalLen) {
        tlvItem.type = buffer[index++];
        tlvItem.length = buffer[index++];
        // ... 继续解析
    }
}
```

**风险**: 
- 如果 `totalLen` 被恶意篡改，可能导致越界读取
- 缺乏深度校验，可能解析恶意构造的数据

### 3.3 分片重组风险

**数据分片重组**: `dsched_softbus_session.cpp:239-287`
```cpp
void DSchedSoftbusSession::AssembleFrag(...)
{
    if (isWaiting_) {
        // 等待后续分片
        totalLen_ = headerPara.totalLen;
        // 风险：未验证 totalLen 的合理性
    }
}
```

**风险**:
- 如果 `totalLen` 被设置为极大值，可能导致内存分配失败 (DoS)
- 分片重叠攻击：攻击者发送重叠的分片覆盖数据

---

## 4. 外部输入清单

### 4.1 用户输入

| 输入源 | 位置 | 验证情况 |
|-------|------|---------|
| deviceId | 多处 | 空检查，但无格式验证 |
| bundleName | 多处 | 空检查 |
| abilityName | 多处 | 空检查 |
| Want/Operation | Parcel | 类型检查 |

### 4.2 网络输入

| 输入源 | 位置 | 验证情况 |
|-------|------|---------|
| SoftBus 数据包 | `dsched_softbus_session.cpp` | 长度、类型检查 |
| JSON 命令 | `dsched_continue_event.cpp` | 类型验证 |
| 文件传输 | `channel_manager.cpp` | 大小、数量限制 |

### 4.3 文件输入

| 输入源 | 位置 | 风险 |
|-------|------|------|
| 配置文件 | `distributed_sched_utils.cpp` | JSON 解析，长度检查 |
| Trust profile | `etc/profile/` | 系统文件，权限控制 |

---

## 5. 敏感操作清单

### 5.1 特权操作

| 操作 | 位置 | 权限要求 | 风险 |
|-----|------|---------|------|
| StartRemoteAbility | `distributed_sched_service.cpp` | `DISTRIBUTED_DATASYNC` | 高 |
| ContinueMission | `dsched_continue.cpp` | `IsFoundationCall` | 高 |
| ConnectRemoteAbility | `ability_connection_manager.cpp` | `DISTRIBUTED_DATASYNC` + 4项权限 | 中 |
| 设备选择回调 | `device_selection_notifier` | IPC 检查 | 中 |

### 5.2 系统服务调用

| 被调用服务 | 调用位置 | 用途 |
|-----------|---------|------|
| `device_auth` | `distributed_sched_adapter.cpp` | 设备认证 |
| `dsoftbus` | `dsched_transport_softbus_adapter.cpp` | 网络通信 |
| `access_token` | 多处 | 权限验证 |
| `ability_runtime` | 多处 | Ability 管理 |

---

## 6. 信任边界

### 6.1 信任边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                         不可信区域                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │   外部应用   │  │   远程设备   │  │       攻击者网络         │  │
│  └──────┬──────┘  └──────┬──────┘  └────────────┬────────────┘  │
│         │                │                       │               │
│         ▼                ▼                       ▼               │
│  ═══════════════════════════════════════════════════════════════│
│                        [信任边界 1]                              │
│                         N-API / IPC                             │
│  ═══════════════════════════════════════════════════════════════│
│         │                │                       │               │
│         ▼                ▼                       ▼               │
│  ┌────────────────────────────────────────────────────────────┐│
│  │              dmsfwk 系统服务 (dtbschedmgr)                  ││
│  │                    [受信区域 1]                             ││
│  └──────────────────────────┬─────────────────────────────────┘│
│                             │                                   │
│  ═══════════════════════════════════════════════════════════════│
│                        [信任边界 2]                              │
│                      SoftBus 网络层                             │
│  ═══════════════════════════════════════════════════════════════│
│                             │                                   │
│                             ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐│
│  │                    远程设备服务                             ││
│  │                    [受信区域 2]                             ││
│  └────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 跨越信任边界的数据流

| 边界 | 数据流 | 验证机制 |
|-----|-------|---------|
| N-API → Service | JS 参数 | N-API 类型检查 + 业务校验 |
| IPC → Service | Parcel | IPC 机制 + Token 校验 |
| SoftBus → Service | 网络包 | 设备认证 + 长度校验 |
| Service → Remote | 跨设备数据 | 设备互信 + 加密传输 |

---

## 7. 攻击向量汇总

| 攻击向量 | 入口点 | 影响 | 可能性 |
|---------|-------|------|-------|
| **N-API 参数注入** | JS API | 权限绕过/DoS | 中 |
| **IPC Parcel 伪造** | IPC 接口 | 权限绕过/数据篡改 | 低 |
| **SoftBus 数据包伪造** | 网络层 | 数据泄露/DoS | 中 |
| **JSON 解析攻击** | 网络命令 | DoS/RCE | 低 |
| **分片重组攻击** | 网络层 | 内存耗尽/数据篡改 | 中 |
| **配置文件篡改** | 文件系统 | 配置注入 | 低 |

---

## 相关链接

- 上一章: [04_Interface.md](04_Interface.md) - 接口文档
- 下一章: [06_SecurityReview.md](06_SecurityReview.md) - 安全风险评估
