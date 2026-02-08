# T2Stack 关键调用链

> 本文档记录 t2stack 模块的关键调用链，用于理解代码执行路径。

## 目录

- [1. Fillp 调用链](#1-fillp-调用链)
- [2. DFile 调用链](#2-dfile-调用链)
- [3. NStackX 调用链](#3-nstackx-调用链)

---

## 1. Fillp 调用链

### 1.1 初始化调用链

```
FtInit()
    ↓
FillpApiRegLibSysFunc()      // 注册系统回调
    ↓
FillpApiRegAppCallbackFunc() // 注册应用回调
    ↓
FillpStackInit()             // 内部初始化
    ├── AllocPcbPool()       // 分配 PCB 池
    ├── InitTimingWheel()    // 初始化时间轮
    └── StartTimerThread()   // 启动定时器线程
```

**代码证据**：`fillp/include/fillpinc.h:1033-1073` - FtInit 相关接口

### 1.2 连接建立调用链

**服务端**：
```
FtSocket()
    ↓
FtBind()
    ↓
FtListen()
    ↓
FtAccept()
    └── OnConnectionInd()    // 连接指示回调
```

**客户端**：
```
FtSocket()
    ↓
FtConnect()
    ├── SendConnectReq()      // 发送连接请求
    ├── WaitForConnectCnf()  // 等待连接确认
    └── OnConnectCnf()       // 连接确认回调
```

**代码证据**：`fillp/include/fillpinc.h:126-264` - Socket 相关接口

### 1.3 数据发送调用链

```
FtSendFrame(data, size, frameInfo)
    ↓
FillpSend(data, size)
    ├── FillpPcbOutput()    // PCB 输出
    ├── FlowControlCheck()   // 流控检查
    │   └── CongestionAvoid() // 拥塞避免
    └── Packetize()          // 分片
        ├── Encrypt()        // 加密
        └── SendToLower()    // 发送到底层
            └── SpungeSend() // UDP 发送
```

**代码证据**：`fillp/include/fillpinc.h:93-94` - FtSendFrame 接口

### 1.4 数据接收调用链

```
FtRecv()
    ↓
FillpRecv(buffer, len)
    ├── WaitForData()        // 等待数据
    ├── Reassemble()         // 重组
    │   ├── Decrypt()        // 解密
    │   └── Reorder()        // 排序
    └── NotifyApp()          // 通知应用
```

---

## 2. DFile 调用链

### 2.1 会话创建调用链

**服务端**：
```
NSTACKX_DFileServer()
    ↓
DFileSessionCreate()
    ├── AllocSession()       // 分配会话
    ├── InitSession()       // 初始化会话
    ├── RegisterReceiver()   // 注册回调
    └── StartListener()      // 启动监听
```

**客户端**：
```
NSTACKX_DFileClient()
    ↓
DFileSessionCreate()
    ├── AllocSession()
    ├── ConnectToServer()   // 连接服务端
    └── Handshake()         // 握手
        ├── ExchangeCap()   // 能力交换
        └── VerifyKey()     // 密钥验证
```

**代码证据**：`nstackx_core/dfile/interface/nstackx_dfile.h:312-340` - 会话创建接口

### 2.2 文件发送调用链

```
NSTACKX_DFileSendFiles()
    ↓
DFileSendPrepare()
    ├── ParseFileList()     // 解析文件列表
    ├── CalcFileHash()      // 计算文件哈希
    └── InitTransfer()      // 初始化传输

DFileSendData()
    ├── ReadFileBlock()     // 读取文件块
    ├── EncryptBlock()      // 加密数据块
    └── SendToTransport()   // 发送到传输层
        └── SoftBusSend()   // 软总线发送
```

**代码证据**：`nstackx_core/dfile/interface/nstackx_dfile.h:381-382` - 文件发送接口

### 2.3 文件接收调用链

```
SoftBusRecv()
    ↓
DFileOnDataInd()
    ├── DecryptBlock()      // 解密数据块
    ├── VerifyHash()        // 验证哈希
    └── WriteToDisk()       // 写入磁盘
        ├── CheckStoragePath() // 检查存储路径
        └── RenameHook()      // 重命名回调
```

**代码证据**：`nstackx_core/dfile/interface/nstackx_dfile.h:419-425` - 接收配置接口

---

## 3. NStackX 调用链

### 3.1 初始化调用链

```
NSTACKX_Init()
    ↓
NSTACKX_InitInner()
    ├── InitCoapStack()     // 初始化 CoAP 协议栈
    ├── InitDeviceTable()   // 初始化设备表
    ├── InitTimer()         // 初始化定时器
    └── StartDiscoverThread() // 启动发现线程
```

**代码证据**：`nstackx_ctrl/interface/nstackx.h:48` - Init 接口

### 3.2 设备发现调用链

```
NSTACKX_StartDeviceFind()
    ↓
StartBroadcast()
    ├── PrepareBroadcastMsg() // 准备广播消息
    │   ├── SerializeDeviceInfo() // 序列化设备信息
    │   └── AddCapabilityBitmap() // 添加能力位图
    └── SendBroadcast()      // 发送广播
        └── CoapSendMsg()    // CoAP 发送
            └── UdpSend()    // UDP 发送

OnBroadcastRecv()
    ├── ParseDiscoveryReq()  // 解析发现请求
    ├── UpdateRemoteDevice() // 更新远程设备
    └── NotifyUpperLayer()   // 通知上层
```

**代码证据**：`nstackx_ctrl/interface/nstackx.h:73` - StartDeviceFind 接口

### 3.3 设备列表获取调用链

```
NSTACKX_GetDeviceList()
    ↓
GetDeviceListFromCache()
    ├── LockDeviceTable()    // 锁定设备表
    ├── CopyDeviceInfo()     // 复制设备信息
    └── UnlockDeviceTable()  // 解锁设备表
```

---

## 关键入口→核心逻辑映射

| 入口函数 | 核心模块 | 主要处理函数 |
|----------|----------|--------------|
| `FtInit()` | Fillp Core | `FillpStackInit()` |
| `FtSocket()` | Fillp PCB | `AllocPcb()` |
| `FtSendFrame()` | Fillp TX | `FillpSend()` |
| `FtRecv()` | Fillp RX | `FillpRecv()` |
| `NSTACKX_DFileServer()` | DFile Session | `DFileSessionCreate()` |
| `NSTACKX_DFileSendFiles()` | DFile TX | `DFileSendPrepare()` |
| `NSTACKX_StartDeviceFind()` | NStackX Disc | `StartBroadcast()` |
| `NSTACKX_GetDeviceList()` | NStackX Device | `GetDeviceListFromCache()` |

---

## 相关文档

- [架构说明](../01_Architecture.md) - 组件交互图
- [模块结构](../02_Module_Structure.md) - 模块职责
- [API 参考](../03_CAPI_Reference.md) - 完整 API 文档

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
