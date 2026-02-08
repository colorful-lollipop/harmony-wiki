# 08_Internals - 内部实现细节

## 核心类职责

### 1. dtbschedmgr 服务

```
DistributedSchedService (单例)
├── 职责: 主服务入口，处理 IPC 请求
├── 文件: services/dtbschedmgr/src/distributed_sched_service.cpp:180
└── 方法:
    ├── OnStart() - 服务启动
    ├── OnStop() - 服务停止
    ├── StartRemoteAbilityInner() - 远程启动
    ├── ContinueMissionInner() - 续接任务
    └── ConnectRemoteAbilityInner() - 远程绑定

DistributedSchedStub
├── 职责: IPC 请求分发
├── 文件: services/dtbschedmgr/src/distributed_sched_stub.cpp:27
└── 方法:
    ├── OnRemoteRequest() - 请求路由
    └── InitRemoteFuncsInner() - 注册处理函数

DSchedContinue
├── 职责: 续接会话管理
├── 文件: services/dtbschedmgr/src/continue/dsched_continue.cpp:165
└── 方法:
    ├── ExecuteContinue() - 执行续接
    ├── PackStartData() - 打包启动数据
    └── ProcessContinueEvent() - 处理续接事件

DSchedTransportSoftbusAdapter (单例)
├── 职责: SoftBus 传输适配
├── 文件: services/dtbschedmgr/src/softbus_adapter/transport/dsched_transport_softbus_adapter.cpp
└── 方法:
    ├── ConnectDevice() - 连接设备
    ├── SendData() - 发送数据
    └── OnBytes() - 接收数据回调
```

### 2. dtbabilitymgr 服务

```
DistributedAbilityManagerService (单例)
├── 职责: 能力管理服务
├── 文件: services/dtbabilitymgr/src/distributed_ability_manager_service.cpp:59
└── 方法:
    ├── Register() - 注册续接能力
    ├── Unregister() - 注销
    └── StartDeviceManager() - 启动设备选择器

DeviceSelectionNotifierProxy/Stub
├── 职责: 设备选择回调
├── 文件: services/dtbabilitymgr/src/continuation_manager/device_selection_notifier_*.cpp
└── 方法:
    └── OnDeviceConnect() - 设备连接通知
```

### 3. dtbcollabmgr 服务

```
AbilityConnectionManager (单例)
├── 职责: 连接会话管理
├── 文件: services/dtbcollabmgr/src/ability_connection_manager/ability_connection_manager.cpp:85
└── 方法:
    ├── ConnectSession() - 创建会话
    ├── Disconnect() - 断开连接
    └── CheckSessionPermission() - 权限检查

ChannelManager (单例)
├── 职责: 数据传输通道
├── 文件: services/dtbcollabmgr/src/channel_manager/channel_manager.cpp
└── 方法:
    ├── SendBytes() - 发送字节数据
    ├── SendMessage() - 发送消息
    └── OnBytesReceived() - 接收回调

AbilityConnectionSession
├── 职责: 单一会话状态
├── 文件: services/dtbcollabmgr/src/ability_connection_manager/ability_connection_session.cpp
└── 属性:
    ├── sessionId_
    ├── peerInfo_
    └── connectState_
```

---

## 关键数据结构

### DistributedWant

**文件**: `services/dtbschedmgr/src/distributedWant/distributed_want.h`

```cpp
class DistributedWant {
private:
    std::shared_ptr<AAFwk::WantParams> wantParams_;  // 参数
    std::shared_ptr<ElementName> element_;          // 组件名
    int32_t flags_;                                  // 标志
    std::string uri_;                                // URI
    std::string action_;                             // Action
    std::vector<std::string> entities_;             // Entities
    int32_t bundleUid_;
    int32_t bundlePid_;
};
```

### CallerInfo

**文件**: `services/dtbschedmgr/include/caller_info.h`

```cpp
struct CallerInfo {
    int32_t uid = -1;
    int32_t pid = -1;
    std::string sourceDeviceId;
    std::string targetDeviceId;
    std::string callerAppId;
    std::vector<std::string> callerBundleNames;
    uint64_t callerAccessToken = 0;
    uint32_t dmsVersion = 0;
    int32_t callerSdkVersion = 0;
    
    bool WriteToParcel(Parcel& parcel) const;
    bool ReadFromParcel(Parcel& parcel);
};
```

### SessionDataHeader (SoftBus 协议头)

**文件**: `services/dtbschedmgr/include/softbus_adapter/transport/dsched_softbus_session.h`

```cpp
struct SessionDataHeader {
    uint16_t version;           // 协议版本
    uint8_t fragFlag;           // 分片标志 (FRAG_START/MID/END)
    uint32_t dataType;          // 数据类型
    uint32_t seqNum;            // 序列号
    uint32_t totalLen;          // 总长度
    uint16_t subSeq;            // 子序列号
    uint32_t dataLen;           // 当前数据长度
};

// 常量定义
constexpr uint32_t BINARY_HEADER_FRAG_LEN = 49;       // 头部长度
constexpr uint32_t BINARY_DATA_MAX_TOTAL_LEN = 100MB; // 最大总长度
constexpr uint32_t BINARY_DATA_MAX_LEN = 4MB;         // 单包最大长度
```

---

## 资源生命周期

### 续接会话生命周期

```
创建
├── StartContinuation() / ContinueMission()
├── 创建 DSchedContinue 对象
└── softbusSessionId_ = -1

连接
├── ConnectDevice() → OnBind()
├── softbusSessionId_ 被赋值
└── 状态: CONNECTING

数据传输
├── PackStartData() / PackData()
├── SendData() 通过 SoftBus
└── 状态: DATA_TRANSFERRING

完成/错误
├── OnContinueComplete()
├── 关闭 SoftBus 会话
└── 销毁 DSchedContinue 对象
```

### 连接会话生命周期

```
创建
├── CreateAbilityConnectionSession()
├── 生成 sessionId
└── 状态: INIT

连接
├── Connect() → AcceptConnect()
├── 建立 SoftBus 通道
└── 状态: CONNECTED

数据传输
├── SendMessage() / SendData()
├── 通过 ChannelManager
└── 状态: DATA_FLOW

断开
├── Disconnect()
├── 关闭通道
└── 状态: DISCONNECTED

销毁
├── DestroyAbilityConnectionSession()
├── 清理资源
└── 从管理器移除
```

---

## 内部 API 契约

### 稳定接口

| 接口 | 文件 | 稳定性 |
|-----|------|-------|
| `DistributedAbilityManagerClient` | `interfaces/innerkits/common/include/distributed_ability_manager_client.h` | 稳定 |
| `IDistributedSched` | `services/dtbschedmgr/include/distributed_sched_interface.h` | 稳定 |
| `IAbilityConnectionManager` | `services/dtbcollabmgr/include/ability_connection_manager/ability_connection_manager_interface.h` | 稳定 |

### 内部实现细节（可能变更）

| 接口/类 | 文件 | 说明 |
|---------|------|------|
| `DSchedContinue` | `services/dtbschedmgr/src/continue/dsched_continue.cpp` | 续接实现细节 |
| `DSchedSoftbusSession` | `services/dtbschedmgr/src/softbus_adapter/transport/dsched_softbus_session.cpp` | 会话协议 |
| `SessionDataHeader` | `dsched_softbus_session.h` | 协议头格式 |

---

## 关键算法

### 数据分片算法

**文件**: `services/dtbschedmgr/src/softbus_adapter/transport/dsched_softbus_session.cpp:283-339`

```cpp
// 发送端分片
int32_t DSchedSoftbusSession::UnPackSendData(...) {
    while (offset < headerPara.totalLen) {
        // 计算当前分片大小
        uint32_t currentLen = std::min(remainLen, maxSendSize);
        
        // 构建 TLV 头部
        SessionDataHeader header;
        header.fragFlag = (offset == 0) ? FRAG_START : 
                         (offset + currentLen >= totalLen) ? FRAG_END : FRAG_MID;
        header.dataLen = currentLen;
        // ...
        
        // 发送分片
        SendPacket(header, data + offset, currentLen);
        offset += currentLen;
    }
}

// 接收端重组
void DSchedSoftbusSession::AssembleFrag(...) {
    if (headerPara.fragFlag == FRAG_START || headerPara.fragFlag == FRAG_START_END) {
        // 新的开始
        totalLen_ = headerPara.totalLen;
        recvBuf_ = std::make_shared<DSchedDataBuffer>(totalLen_);
        memcpy(recvBuf_->Data(), data, dataLen);
    } else {
        // 继续追加
        memcpy(recvBuf_->Data() + offset_, data, dataLen);
    }
    
    if (headerPara.fragFlag == FRAG_END || headerPara.fragFlag == FRAG_START_END) {
        // 完成，通知上层
        OnDataReady(recvBuf_);
    }
}
```

### TLV 编解码

**编码**: `WriteTlvToBuffer()`  
**解码**: `ReadTlvToHeader()`

```cpp
// TLV 格式: Type(1B) + Length(4B) + Value(Length)
struct TLVItem {
    uint8_t type;
    uint32_t length;
    uint8_t value[MAX_TLV_LEN];
};
```

---

## 性能考虑

### 内存使用

| 组件 | 内存特点 |
|-----|---------|
| DSchedContinue | 每续接会话约 10-50KB |
| DSchedSoftbusSession | 每连接约 100KB + 缓冲区 |
| ChannelManager | 全局单例，约 500KB |

### 优化策略

1. **对象池**: DSchedContinue 使用智能指针管理
2. **零拷贝**: 大数据使用 shared_ptr 共享
3. **超时清理**: 会话超时自动释放
4. **流控**: SoftBus 层实现背压

---

## 调试与诊断

### 日志标签

| 标签 | 位置 |
|-----|------|
| `DistributedSchedStub` | IPC 层 |
| `DSchedContinue` | 续接流程 |
| `DSchedTransport` | 网络传输 |
| `AbilityConnectionManager` | 协作管理 |

### DFX 事件

**文件**: `hisysevent.yaml`

```yaml
domain: DISTRIBUTED_SCHEDULE
events:
  DMS_INIT_BEHAVIOR:
    - START_REMOTE_ABILITY
    - CONTINUE_MISSION
  DMS_BEHAVIOR:
    - CONNECT_REMOTE_ABILITY
    - DISCONNECT_REMOTE_ABILITY
```

---

## 相关链接

- 上一章: [07_Build.md](07_Build.md) - 构建配置
- 返回: [SUMMARY.md](SUMMARY.md) - 全站导航
