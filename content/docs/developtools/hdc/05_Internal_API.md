# 内部 API

## 目的

描述 hdc 内部模块接口、依赖方向、稳定性标注，帮助理解模块间交互。

## 适用范围

本文档适用于：
- 了解模块间接口定义
- 理解依赖关系和调用方向
- 识别稳定/不稳定接口
- 添加新模块时参考接口规范

## 相关跳转

- [目录结构](./02_Directory_Structure.md) - 模块职责详情
- [架构说明](./03_Architecture.md) - 组件关系和数据流

---

## 稳定接口

### HdcSessionBase 类

**定义**：`src/common/session.h:25-245`

**职责**：Session 生命周期管理

**公共接口**：

```cpp
class HdcSessionBase {
public:
    // Session 生命周期
    virtual HSession MallocSession(bool serverOrDaemon, const ConnType connType,
                                   void *classModule, uint32_t sessionId = 0);
    virtual void FreeSession(const uint32_t sessionId);
    virtual HSession AdminSession(const uint8_t op, const uint32_t sessionId, HSession hInput);

    // 数据传输
    virtual int Send(const uint32_t sessionId, const uint32_t channelId,
             const uint16_t commandFlag, const uint8_t *data, const int dataSize);
    virtual int OnRead(HSession hSession, uint8_t *bufPtr, const int bufLen);

    // 任务管理
    virtual HTaskInfo AdminTask(const uint8_t op, HSession hSession,
                        const uint32_t channelId, HTaskInfo hInput);
    virtual bool DispatchTaskData(HSession hSession, const uint32_t channelId,
                          const uint16_t command, uint8_t *payload, int payloadSize);

    // 异步消息
    virtual void PushAsyncMessage(const uint32_t sessionId, const uint8_t method,
                          const void *data, const int dataSize);
};
```

**子类**：
- `HdcDaemon` (`src/daemon/daemon.h:32`) - Device 端实现
- `HdcServer` (`src/host/server.h`) - Host 端实现

### HdcChannelBase 类

**定义**：`src/common/channel.h:8-45`

**职责**：逻辑通道管理

**公共接口**：

```cpp
class HdcChannelBase {
public:
    HChannel AdminChannel(const uint8_t op, const uint32_t channelId, HChannel hInput);
    void FreeChannel(const uint32_t channelId);
    void SendWithCmd(const uint32_t channelId, const uint16_t commandFlag,
                     uint8_t *bufPtr, const int size);
};
```

**子类**：
- `HdcServerForClient` (`src/host/server_for_client.h`) - Server 到 Client 通道
- `HdcClientChannel` (`src/host/client.cpp`) - Client 到 Daemon 通道

### HdcTaskBase 类

**定义**：`src/common/task.h:8-46`

**职责**：任务基类

**公共接口**：

```cpp
class HdcTaskBase {
public:
    virtual bool CommandDispatch(const uint16_t command, uint8_t* payload, const int payloadSize);
    virtual void StopTask();
    virtual bool ReadyForRelease();

    void TaskFinish();
    bool SendToAnother(const uint16_t command, uint8_t *bufPtr, const int size);
    void LogMsg(MessageLevel level, const char *msg, ...);
};
```

**子类**：
- `HdcTransferBase` (`src/common/transfer.h`) - 文件传输基类
  - `HdcFile` (`src/common/file.h`) - 文件传输任务
  - `HdcApp` (`src/daemon/daemon_app.cpp`) - 应用管理任务
  - `HdcFlashd` (flashd 任务) - Flash 操作任务
- `HdcForwardBase` (`src/common/forward.h`) - 端口转发基类

### HdcTransferBase 类

**定义**：`src/common/transfer.h`

**职责**：文件传输基类

**公共接口**：

```cpp
class HdcTransferBase : public HdcTaskBase {
public:
    virtual void StopTask() override;
    virtual bool CommandDispatch(const uint16_t command, uint8_t *payload,
                             const int payloadSize) override;
};
```

### Base 命名空间

**定义**：`src/common/base.h`

**公共工具函数**：

```cpp
namespace Base {
    // 网络
    int SendToStream(uv_stream_t *handleStream, const uint8_t *buf, const int bufLen);
    int SendToPollFd(int fd, const uint8_t *buf, const int bufLen);
    void SetTcpOptions(uv_tcp_t *tcpHandle, int bufMaxSize = HDC_SOCKETPAIR_SIZE);

    // 文件 I/O
    int ReadBinFile(const char *pathName, void **buf, const size_t bufLen);
    int WriteBinFile(const char *pathName, const uint8_t *buf, const size_t bufLen, bool newFile);

    // 编码/解码
    vector<uint8_t> Base64Encode(const uint8_t *input, const int length);
    string Base64Decode(const uint8_t *input, const int length);
    string UnicodeToUtf8(const char *src, bool reverse = false);

    // 工具
    uint64_t GetRuntimeMSec();
    string GetRandomString(const uint16_t expectedLen);
    uint32_t GetSecureRandom(void);
    void SplitString(const string &origString, const string &seq, vector<string> &resultStrings);
    string &Trim(string &s, const string &w = WHITE_SPACES);
}
```

### HdcAuth 命名空间

**定义**：`src/common/auth.h`

**认证函数**：

```cpp
namespace HdcAuth {
    bool KeylistIncrement(list<void *> *listKey, uint8_t &authKeyIndex, void **out);
    void FreeKey(bool publicOrPrivate, list<void *> *listKey);
    bool GenerateKey(const char *file);
    bool AuthVerify(uint8_t *token, uint8_t *sig, int siglen);
}
```

---

## 模块依赖方向

### 依赖关系图

```
HdcDaemon (device/daemon.cpp)
    ├── extends HdcSessionBase (session.h)
    ├── uses HdcAuth (auth.cpp)                    [稳定]
    ├── uses HdcSSLBase (hdc_ssl.cpp)             [稳定]
    ├── uses HdcUSBBase (usb.cpp)                   [稳定]
    ├── uses HdcTCPBase (tcp.cpp)                   [稳定]
    ├── uses HdcUART (uart.cpp)                     [条件]
    ├── contains HdcFile (file.cpp)                   [内部任务]
    ├── contains HdcForwardBase (forward.cpp)             [内部任务]
    └── contains HdcShell (shell.cpp)                 [内部任务]

HdcServer (host/server.cpp)
    ├── extends HdcSessionBase (session.h)
    ├── uses HdcAuth (auth.cpp)                    [稳定]
    ├── uses HdcSSLBase (hdc_ssl.cpp)             [稳定]
    ├── uses HdcUSBBase (usb.cpp)                   [稳定]
    ├── uses HdcTCPBase (tcp.cpp)                   [稳定]
    ├── manages HdcServerForClient (server_for_client.cpp) [内部]
    └── manages HdcClient (client.cpp)            [内部]
```

### 依赖标注

| 接口 | 稳定性 | 说明 |
|--------|--------|------|
| `HdcSessionBase::MallocSession()` | ✅ 稳定 | 所有 Session 管理必需 |
| `HdcSessionBase::FreeSession()` | ✅ 稳定 | Session 生命周期必需 |
| `HdcSessionBase::Send()` | ✅ 稳定 | 核心数据传输接口 |
| `HdcSessionBase::OnRead()` | ✅ 稳定 | 数据接收接口 |
| `HdcSessionBase::AdminTask()` | ✅ 稳定 | 任务管理接口 |
| `HdcTaskBase::CommandDispatch()` | ✅ 稳定 | 命令分发（虚函数） |
| `HdcTaskBase::StopTask()` | ✅ 稳定 | 任务停止（虚函数） |
| `Base::*` | ✅ 稳定 | 工具函数命名空间 |
| `HdcAuth::*` | ✅ 稳定 | 认证命名空间 |
| `PayloadProtect`, `CtrlStruct` | ⚠️ 内部 | 内部数据结构 |
| `TaskCommandDispatch<T>()` | ⚠️ 内部 | 模板函数（session.h:169-190） |
| `DoTaskRemove<T>()` | ⚠️ 内部 | 模板函数（session.h:192-206） |

---

## 跨模块调用示例

### 文件传输调用链

```
Client 发送 CMD_FILE_INIT
  ↓
Server.DispatchTaskData() [session.h:116-117]
  ↓
AdminTask() [session.h:115] 创建 HdcFile 任务
  ↓
HdcFile::CommandDispatch() [task.h:14, transfer.h:command dispatch]
  ↓
HdcFile 接收文件分片
  ↓
使用 Base::SendToStream() [base.h]
  ↓
使用 Compress/Decompress [compress.cpp/decompress.cpp]
```

### 认证调用链

```
Daemon.HandDaemonAuth() [daemon.cpp:73]
  ↓
使用 HdcAuth::AuthVerify() [auth.cpp]
  ↓
使用 EVP_PKEY_verify() [OpenSSL]
  ↓
使用 SHA512() [OpenSSL]
  ↓
更新 mapAuthStatus [daemon.h:125]
```

---

## 接口版本策略

### 版本化接口

**原则**：
1. 公共头文件（`src/common/*.h`）定义接口
2. 重大变更需要更新接口版本
3. 保持向后兼容性

**版本机制**：
- `VER_PROTOCOL` (协议版本)：`src/common/define.h:80`
- `SESSION_VERSION` (Session 版本)：TODO(需确认)

### 兼容性策略

**原则**：
1. 新增功能使用新命令号
2. 不修改已有命令的行为
3. 使用 TLV 格式支持可选字段

**证据**：`src/common/tlv.cpp` - TLV 编码支持可选字段

---

## 依赖避免

### 避免循环依赖

**规则**：
- Common 层不依赖 Host/Daemon 层
- Host/Daemon 可以依赖 Common 层
- Host 和 Daemon 不互相依赖

**当前依赖关系**：
```
Common 层（src/common/）
  ├─独立（无层间依赖）
  └─被 Host/Daemon 使用

Host 层（src/host/）
  └─依赖 Common 层

Daemon 层（src/daemon/）
  └─依赖 Common 层
```

### 条件编译依赖

**Rust 模块**（`hdc_rust/`）：
- 当 `product_name != "ohos-sdk"` 时编译
- 提供 `serialize_structs` C++ 桥接库
- 与 C++ 模块互操作通过 C FFI

**证据**：`BUILD.gn:195-261`

---

## 关键结论

1. **清晰的分层架构**：Common（稳定基础）→ Host/Daemon（端实现）
2. **基于虚函数的扩展机制**：HdcTaskBase 提供统一的命令分发接口
3. **稳定接口标注**：Common 层头文件定义的公共接口是稳定的
4. **无循环依赖**：Common 层独立，Host/Daemon 单向依赖
5. **模板函数为内部**：`TaskCommandDispatch<T>()` 等是内部实现

---

## 待确认事项

**TODO(需确认)**：
1. 接口版本化策略和向后兼容性
2. Rust 模块与 C++ 模块的接口互操作细节
3. 新模块添加时的接口规范审查流程
