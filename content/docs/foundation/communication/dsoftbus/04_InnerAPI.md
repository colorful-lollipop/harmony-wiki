# DSoftBus 内部 API 文档

## 模块概览

| 模块 | 内部接口头文件 | 稳定性 |
|------|---------------|--------|
| **Authentication** | `core/authentication/interface/auth_interface.h` | 较稳定 |
| **Bus Center** | `core/bus_center/interface/bus_center_manager.h` | 较稳定 |
| **Connection** | `core/connection/interface/softbus_conn_manager.h` | 较稳定 |
| **Discovery** | `core/discovery/interface/disc_manager.h` | 较稳定 |
| **Transmission** | `core/transmission/common/include/trans_session_manager.h` | 较稳定 |

> 代码证据: 各模块 `interface/` 目录

## Authentication 模块

### 头文件

```
core/authentication/interface/
├── auth_interface.h       # 主接口
├── auth_channel.h         # 认证通道
└── auth_session.h         # 会话接口
```

### 核心接口

| 接口函数 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `AuthInit()` | - | `int32_t` | 初始化认证模块 |
| `AuthDeinit()` | - | `int32_t` | 去初始化 |
| `AuthStartVerify()` | `const char *addr, const AuthVerifyCallback *cb` | `int32_t` | 开始认证 |
| `AuthGetSessionKey()` | `int32_t channelId, char *sessionKey, int32_t *keyLen` | `int32_t` | 获取会话密钥 |

### 依赖关系

```
Authentication
    ↓ 依赖
├── device_auth (HiChain)
├── huks (密钥管理)
└── connection (底层连接)
```

---

## Bus Center 模块

### 头文件

```
core/bus_center/interface/
├── bus_center_manager.h   # 主管理接口
├── bus_center_server.h   # 服务端接口
└── lnn_*.h              # LNN 相关接口
```

### 核心接口

| 接口函数 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `BusCenterServerInit()` | - | `int32_t` | 初始化 Bus Center |
| `JoinLNN()` | `const char *pkgName, ConnectionAddr *addr, OnJoinLNNResult cb` | `int32_t` | 加入本地网络 |
| `LeaveLNN()` | `const char *pkgName, const char *networkId, OnLeaveLNNResult cb` | `int32_t` | 离开本地网络 |
| `RegNodeDeviceStateCb()` | `const char *pkgName, INodeStateCb *cb` | `int32_t` | 注册设备状态回调 |
| `GetLocalNodeInfo()` | `NodeBasicInfo *info` | `int32_t` | 获取本机节点信息 |
| `GetPeerNodeInfo()` | `const char *networkId, NodeBasicInfo *info` | `int32_t` | 获取对端节点信息 |

### 依赖关系

```
Bus Center (LNN)
    ↓ 依赖
├── authentication (认证)
├── discovery (发现)
├── connection (连接)
└── transmission (传输)
```

---

## Connection 模块

### 头文件

```
core/connection/interface/
├── softbus_conn_manager.h   # 连接管理器
├── softbus_conn_br.h       # BR 连接
├── softbus_conn_ble.h       # BLE 连接
└── softbus_conn_tcp.h      # TCP 连接
```

### 核心接口

| 接口函数 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `ConnInit()` | - | `int32_t` | 初始化连接管理 |
| `ConnDeinit()` | - | `int32_t` | 去初始化 |
| `ConnectDevice()` | `const ConnectionAddr *addr, unsigned int timeout, const IServerChannelCallBack *cb` | `int32_t` | 连接设备 |
| `DisconnectDevice()` | `const char *networkId` | `int32_t` | 断开设备连接 |
| `SendBytes()` | `int32_t channelId, const void *data, unsigned int len` | `int32_t` | 发送字节 |
| `GetConnectionInfo()` | `int32_t channelId, ConnectionInfo *info` | `int32_t` | 获取连接信息 |

### 依赖关系

```
Connection
    ↓ 依赖
├── bluetooth (BR/BLE)
├── wifi (TCP)
└── adapter (平台适配)
```

---

## Discovery 模块

### 头文件

```
core/discovery/interface/
├── disc_manager.h         # 发现管理器
├── disc_ble.h            # BLE 发现
├── disc_coap.h           # CoAP 发现
└── disc_*.h              # 其他发现方式
```

### 核心接口

| 接口函数 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `DiscInit()` | - | `int32_t` | 初始化发现模块 |
| `PublishLNN()` | `const char *pkgName, const PublishInfo *info, const IPublishCb *cb` | `int32_t` | 发布服务 |
| `StopPublishLNN()` | `const char *pkgName, int32_t publishId` | `int32_t` | 停止发布 |
| `RefreshLNN()` | `const char *pkgName, const SubscribeInfo *info, const IRefreshCallback *cb` | `int32_t` | 发现设备 |
| `StopRefreshLNN()` | `const char *pkgName, int32_t refreshId` | `int32_t` | 停止发现 |

### 依赖关系

```
Discovery
    ↓ 依赖
├── bluetooth (BLE)
├── wifi (CoAP)
├── usb (USB)
└── adapter (平台适配)
```

---

## Transmission 模块

### 头文件

```
core/transmission/common/include/
├── trans_session_manager.h  # 会话管理
├── trans_channel.h         # 通道接口
├── socket.h                # 套接字接口
└── trans_type.h            # 类型定义
```

### 核心接口

| 接口函数 | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `TransInit()` | - | `int32_t` | 初始化传输模块 |
| `CreateSessionServer()` | `const char *pkgName, const char *sessionName, const ISessionListener *listener` | `int32_t` | 创建会话服务器 |
| `RemoveSessionServer()` | `const char *pkgName, const char *sessionName` | `int32_t` | 移除会话服务器 |
| `OpenSession()` | `const char *sessionName, const char *peerSessionName, const char *peerDeviceId, const char *groupId` | `int32_t` | 打开会话 |
| `CloseSession()` | `int32_t sessionId` | `int32_t` | 关闭会话 |
| `SendBytes()` | `int32_t sessionId, const void *data, unsigned int len` | `int32_t` | 发送字节 |
| `SendStream()` | `int32_t sessionId, const StreamData *data, const StreamData *ext` | `int32_t` | 发送流 |
| `SendFile()` | `int32_t sessionId, const char *sFileList[], const char *dFileList[], unsigned int fileCnt` | `int32_t` | 发送文件 |

### 依赖关系

```
Transmission
    ↓ 依赖
├── connection (底层连接)
├── authentication (认证)
└── bus_center (设备信息)
```

---

## 公共模块

### 头文件

```
core/common/include/
├── softbus_def.h          # 常量定义
├── softbus_utils.h        # 工具函数
└── softbus_errcode.h      # 错误码
```

### 公共类型

| 类型 | 定义文件 | 说明 |
|-----|---------|------|
| `ConnectionAddr` | `softbus_common.h` | 连接地址 |
| `DeviceInfo` | `softbus_common.h` | 设备信息 |
| `PublishInfo` | `softbus_common.h` | 发布信息 |
| `SubscribeInfo` | `softbus_common.h` | 订阅信息 |
| `SocketInfo` | `socket.h` | 套接字信息 |

---

## 稳定性标注

| 接口分类 | 位置 | 稳定性 | 说明 |
|---------|------|--------|------|
| **Public SDK** | `interfaces/kits/` | 稳定 | 官方对外接口 |
| **Inner Kit** | `interfaces/inner_kits/` | 较稳定 | 系统能力间调用 |
| **Core Internal** | `core/*/interface/` | 较稳定 | 核心模块接口 |
| **Implementation** | `core/*/src/` | 不稳定 | 实现细节，可能变更 |

---

## 适配层

### 头文件

```
adapter/
├── bus_center/            # Bus Center 适配
├── authentication/       # 认证适配
├── kv_store/              # KV 存储适配
└── transmission/          # 传输适配
```

### 适配器职责

- 屏蔽平台差异
- 统一接口抽象
- 提供测试桩

---

**相关文档**

- [项目概览](./01_Overview.md)
- [架构说明](./02_Architecture.md)
- [N-API 接口](./03_NAPI.md)
- [编译配置](./05_Build.md)
