# T2Stack C API 参考

> **重要说明**：t2stack 是纯 C/C++ 库，**不提供 N-API（JS 接口）**。本文档记录内部 C API 接口，供 C/C++ 开发者使用。

## 目录

- [1. Fillp 流传输接口](#1-fillp-流传输接口)
- [2. DFile 文件传输接口](#2-dfile-文件传输接口)
- [3. NStackX 设备发现接口](#3-nstackx-设备发现接口)
- [4. 错误码定义](#4-错误码定义)

---

## 1. Fillp 流传输接口

### 1.1 初始化与销毁

#### FtInit

```c
/**
 * @brief 初始化 Fillp 栈
 *
 * @return FILLP_INT
 *         FILLP_SUCCESS (0): 成功
 *         负值: 失败（错误码）
 *
 * @note
 * - 两个线程不能同时调用 FtInit()
 * - FtInit() 成功后不能多次调用
 * - 建议由单线程调用一次
 *
 * @code
 * FILLP_INT ret = FtInit();
 * if (ret != FILLP_SUCCESS) {
 *     // 处理错误
 * }
 * @endcode
 */
extern FILLP_INT DLL_API FtInit(void);
```

**代码位置**：`fillp/include/fillpinc.h:1059`

#### FtDestroy

```c
/**
 * @brief 销毁 Fillp 栈
 *
 * @note
 * - 调用前必须关闭所有通过 FtSocket() 创建的 socket
 * - 否则 FtDestroy() 会阻塞
 * - 两个线程不能同时调用
 *
 * @code
 * FtClose(fd1);
 * FtClose(fd2);
 * FtDestroy();  // 等待所有资源释放
 * @endcode
 */
extern void DLL_API FtDestroy(void);
```

**代码位置**：`fillp/include/fillpinc.h:1073`

### 1.2 Socket 操作

#### FtSocket

```c
/**
 * @brief 创建 Fillp socket
 *
 * @param [in] domain     协议族 (AF_INET/AF_INET6)
 * @param [in] type       套接字类型 (SOCK_STREAM)
 * @param [in] protocol   协议 (IPPROTO_FILLP)
 *
 * @return FILLP_INT
 *         非负值: socket 描述符
 *         负值: 失败
 *
 * @code
 * FILLP_INT fd = FtSocket(AF_INET, SOCK_STREAM, IPPROTO_FILLP);
 * if (fd < 0) {
 *     // 创建失败
 * }
 * @endcode
 */
extern FILLP_INT DLL_API FtSocket(IN FILLP_INT domain, IN FILLP_INT type, IN FILLP_INT protocol);
```

**代码位置**：`fillp/include/fillpinc.h:126`

#### FtBind

```c
/**
 * @brief 绑定 socket 到地址
 *
 * @param [in] fd        socket 描述符
 * @param [in] name      地址结构体
 * @param [in] nameLen   地址结构体长度
 *
 * @return FILLP_INT
 *         0: 成功
 *         负值: 失败
 *
 * @note
 * - 不支持绑定到 INADDR_ANY 地址
 * - 绑定失败后必须调用 FtClose() 关闭 socket
 *
 * @code
 * struct sockaddr_in addr;
 * addr.sin_family = AF_INET;
 * addr.sin_port = htons(8080);
 * addr.sin_addr.s_addr = inet_addr("192.168.1.100");
 *
 * FILLP_INT ret = FtBind(fd, (struct sockaddr *)&addr, sizeof(addr));
 * @endcode
 */
extern FILLP_INT DLL_API FtBind(FILLP_INT fd, FILLP_CONST struct sockaddr *name, FILLP_UINT32 nameLen);
```

**代码位置**：`fillp/include/fillpinc.h:112`

#### FtListen

```c
/**
 * @brief 监听连接请求
 *
 * @param [in] fd        socket 描述符
 * @param [in] backLog  待处理连接队列的最大长度
 *
 * @return FILLP_INT
 *         0: 成功
 *         负值: 失败
 *
 * @note
 * - backLog 值必须在 0 到最大连接数之间
 * - 超出范围会使用默认值
 *
 * @code
 * FILLP_INT ret = FtListen(fd, 10);  // 最多 10 个等待连接
 * @endcode
 */
extern FILLP_INT DLL_API FtListen(FILLP_INT fd, FILLP_INT backLog);
```

**代码位置**：`fillp/include/fillpinc.h:264`

#### FtAccept

```c
/**
 * @brief 接受连接
 *
 * @param [in] fd       socket 描述符
 * @param [out] addr   对端地址（可选，传入 NULL 表示不需要）
 * @param [out] addrLen 地址长度（可选）
 *
 * @return FILLP_INT
 *         非负值: 客户端 socket 描述符
 *         负值: 失败
 *
 * @code
 * struct sockaddr_in clientAddr;
 * socklen_t addrLen = sizeof(clientAddr);
 *
 * FILLP_INT clientFd = FtAccept(fd, (struct sockaddr *)&clientAddr, &addrLen);
 * if (clientFd < 0) {
 *     // 接受失败
 * }
 * @endcode
 */
extern FILLP_INT DLL_API FtAccept(FILLP_INT fd, struct sockaddr *addr, socklen_t *addrLen);
```

**代码位置**：`fillp/include/fillpinc.h:250`

#### FtConnect

```c
/**
 * @brief 连接到服务端
 *
 * @param [in] fd    socket 描述符
 * @param [in] name  服务器地址
 * @param [in] nameLen 地址长度
 *
 * @return FILLP_INT
 *         0: 成功
 *         负值: 失败（可能是非阻塞模式下的 EAGAIN）
 *
 * @code
 * struct sockaddr_in serverAddr;
 * serverAddr.sin_family = AF_INET;
 * serverAddr.sin_port = htons(8080);
 * serverAddr.sin_addr.s_addr = inet_addr("192.168.1.100");
 *
 * FILLP_INT ret = FtConnect(fd, (struct sockaddr *)&serverAddr, sizeof(serverAddr));
 * @endcode
 */
extern FILLP_INT DLL_API FtConnect(FILLP_INT fd, FILLP_CONST FILLP_SOCKADDR *name, socklen_t nameLen);
```

**代码位置**：`fillp/include/fillpinc.h:142`

#### FtClose

```c
/**
 * @brief 关闭 socket
 *
 * @param [in] fd socket 描述符
 *
 * @return FILLP_INT
 *         0: 成功
 *         负值: 失败
 *
 * @note
 * - 调用后不能再对 socket 进行任何操作
 *
 * @code
 * FILLP_INT ret = FtClose(fd);
 * @endcode
 */
extern FILLP_INT DLL_API FtClose(FILLP_INT fd);
```

**代码位置**：`fillp/include/fillpinc.h:213`

### 1.3 数据收发

#### FtSendFrame

```c
/**
 * @brief 发送视频帧
 *
 * @param [in] fd    socket 描述符
 * @param [in] data 数据指针
 * @param [in] size 数据大小
 * @param [in] flag 标志位
 * @param [in] frame 帧信息（I帧/P帧等）
 *
 * @return FILLP_INT
 *         0: 成功
 *         负值: 失败
 *
 * @code
 * struct FrameInfo frameInfo = {
 *     .frameType = FRAME_TYPE_I,  // I 帧
 *     .timestamp = 0,
 * };
 *
 * FILLP_INT ret = FtSendFrame(fd, data, size, 0, &frameInfo);
 * @endcode
 */
extern FILLP_INT DLL_API FtSendFrame(FILLP_INT fd, FILLP_CONST void *data, size_t size, FILLP_INT flag,
    FILLP_CONST struct FrameInfo *frame);
```

**代码位置**：`fillp/include/fillpinc.h:93-94`

#### FtRecv

```c
/**
 * @brief 接收数据
 *
 * @param [in] fd   socket 描述符
 * @param [out] mem 接收缓冲区
 * @param [in] len 缓冲区长度
 * @param [in] flag 标志位
 *
 * @return FILLP_INT
 *         非负值: 接收的字节数
 *         负值: 失败
 *
 * @code
 * char buffer[4096];
 * FILLP_INT bytes = FtRecv(fd, buffer, sizeof(buffer), 0);
 * if (bytes > 0) {
 *     // 处理接收到的数据
 * }
 * @endcode
 */
extern FILLP_INT DLL_API FtRecv(FILLP_INT fd, void *mem, size_t len, FILLP_INT flag);
```

**代码位置**：`fillp/include/fillpinc.h:164`

### 1.4 Epoll 接口

#### FtEpollWait

```c
/**
 * @brief 等待 I/O 事件
 *
 * @param [in] epFd      epoll 文件描述符
 * @param [out] events   事件数组
 * @param [in] maxEvents 最大事件数
 * @param [in] timeout   超时时间（毫秒）
 *
 * @return FILLP_INT
 *         非负值: 就绪的事件数
 *         负值: 失败
 *
 * @note
 * - timeout = -1: 阻塞等待
 * - timeout = 0: 非阻塞
 *
 * @code
 * struct SpungeEpollEvent events[64];
 * FILLP_INT nfds = FtEpollWait(epFd, events, 64, -1);
 * for (int i = 0; i < nfds; i++) {
 *     // 处理每个事件
 * }
 * @endcode
 */
extern FILLP_INT DLL_API FtEpollWait(FILLP_INT epFd, struct SpungeEpollEvent *events,
    FILLP_INT maxEvents, FILLP_INT timeout);
```

**代码位置**：`fillp/include/fillpinc.h:332-333`

---

## 2. DFile 文件传输接口

### 2.1 会话管理

#### NSTACKX_DFileServer

```c
/**
 * @brief 创建 DFile 服务端会话
 *
 * @param [in] localAddr     本地地址（IP 和端口，主机序）
 * @param [in] addrLen       地址长度
 * @param [in] key           加密密钥（16 字节）
 * @param [in] keyLen        密钥长度（16）
 * @param [in] msgReceiver   消息回调函数
 *
 * @return int32_t
 *         正值: session ID（成功）
 *         负值: 失败
 *
 * @code
 * struct sockaddr_in localAddr;
 * localAddr.sin_family = AF_INET;
 * localAddr.sin_port = 0;  // 系统自动分配端口
 * localAddr.sin_addr.s_addr = inet_addr("0.0.0.0");
 *
 * uint8_t key[16] = {0};
 * int32_t sessionId = NSTACKX_DFileServer(&localAddr, sizeof(localAddr),
 *                                          key, 16, MyMsgReceiver);
 * @endcode
 */
NSTACKX_EXPORT int32_t NSTACKX_DFileServer(struct sockaddr_in *localAddr, socklen_t addrLen, const uint8_t *key,
                                            uint32_t keyLen, DFileMsgReceiver msgReceiver);
```

**代码位置**：`nstackx_core/dfile/interface/nstackx_dfile.h:312-313`

#### NSTACKX_DFileClient

```c
/**
 * @brief 创建 DFile 客户端会话
 *
 * @param [in] srvAddr     远端地址
 * @param [in] addrLen    地址长度
 * @param [in] key        加密密钥（可为 NULL 表示不加密）
 * @param [in] keyLen     密钥长度（16 或 0）
 * @param [in] msgReceiver 消息回调函数
 *
 * @return int32_t
 *         正值: session ID
 *         负值: 失败
 *
 * @code
 * struct sockaddr_in srvAddr;
 * srvAddr.sin_family = AF_INET;
 * srvAddr.sin_port = htons(8888);
 * srvAddr.sin_addr.s_addr = inet_addr("192.168.1.100");
 *
 * int32_t sessionId = NSTACKX_DFileClient(&srvAddr, sizeof(srvAddr),
 *                                          NULL, 0, MyMsgReceiver);
 * @endcode
 */
NSTACKX_EXPORT int32_t NSTACKX_DFileClient(struct sockaddr_in *srvAddr, socklen_t addrLen, const uint8_t *key,
                                           uint32_t keyLen, DFileMsgReceiver msgReceiver);
```

**代码位置**：`nstackx_core/dfile/interface/nstackx_dfile.h:339-340`

#### NSTACKX_DFileClose

```c
/**
 * @brief 关闭 DFile 会话
 *
 * @param [in] sessionId 会话 ID
 *
 * @code
 * NSTACKX_DFileClose(sessionId);
 * @endcode
 */
NSTACKX_EXPORT void NSTACKX_DFileClose(int32_t sessionId);
```

**代码位置**：`nstackx_core/dfile/interface/nstackx_dfile.h:372`

### 2.2 文件操作

#### NSTACKX_DFileSendFiles

```c
/**
 * @brief 发送文件列表
 *
 * @param [in] sessionId 会话 ID
 * @param [in] files    文件名列表
 * @param [in] fileNum 文件数量
 * @param [in] userData 用户上下文数据
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 *
 * @code
 * const char *files[] = {
 *     "/sdcard/video1.mp4",
 *     "/sdcard/video2.mp4"
 * };
 * int32_t ret = NSTACKX_DFileSendFiles(sessionId, files, 2, "user_data");
 * @endcode
 */
NSTACKX_EXPORT int32_t NSTACKX_DFileSendFiles(int32_t sessionId, const char *files[], uint32_t fileNum,
                                               const char *userData);
```

**代码位置**：`nstackx_core/dfile/interface/nstackx_dfile.h:381-382`

#### NSTACKX_DFileSetStoragePath

```c
/**
 * @brief 设置接收文件存储路径
 *
 * @param [in] sessionId 会话 ID
 * @param [in] path     存储根路径
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 *
 * @code
 * NSTACKX_DFileSetStoragePath(sessionId, "/data/received_files/");
 * @endcode
 */
NSTACKX_EXPORT int32_t NSTACKX_DFileSetStoragePath(int32_t sessionId, const char *path);
```

**代码位置**：`nstackx_core/dfile/interface/nstackx_dfile.h:419`

#### NSTACKX_DFileSetRenameHook

```c
/**
 * @brief 设置文件名冲突处理回调
 *
 * @param [in] sessionId   会话 ID
 * @param [in] onRenameFile 重命名回调函数
 *
 * @code
 * void MyRenameHook(DFileRenamePara *renamePara) {
 *     // 生成新文件名
 *     snprintf(renamePara->newFileName, sizeof(renamePara->newFileName),
 *              "%s_%llu", renamePara->initFileName, GetTickCount());
 * }
 *
 * NSTACKX_DFileSetRenameHook(sessionId, MyRenameHook);
 * @endcode
 */
NSTACKX_EXPORT int32_t NSTACKX_DFileSetRenameHook(int32_t sessionId, OnDFileRenameFile onRenameFile);
```

**代码位置**：`nstackx_core/dfile/interface/nstackx_dfile.h:425`

### 2.3 消息回调类型

```c
/**
 * @brief DFile 消息回调类型
 *
 * @param [in] sessionId 会话 ID
 * @param [in] msgType  消息类型
 * @param [in] msg      消息数据
 */
typedef void (*DFileMsgReceiver)(int32_t sessionId, DFileMsgType msgType, const DFileMsg *msg);
```

**消息类型** (`nstackx_dfile.h:65-82`)：

| 消息类型 | 说明 |
|----------|------|
| `DFILE_ON_CONNECT_SUCCESS` | 连接成功 |
| `DFILE_ON_CONNECT_FAIL` | 连接失败 |
| `DFILE_ON_FILE_LIST_RECEIVED` | 收到文件列表 |
| `DFILE_ON_FILE_RECEIVE_SUCCESS` | 文件接收成功 |
| `DFILE_ON_FILE_RECEIVE_FAIL` | 文件接收失败 |
| `DFILE_ON_FILE_SEND_SUCCESS` | 文件发送成功 |
| `DFILE_ON_FILE_SEND_FAIL` | 文件发送失败 |
| `DFILE_ON_FATAL_ERROR` | 致命错误 |
| `DFILE_ON_TRANS_IN_PROGRESS` | 传输进度更新 |

---

## 3. NStackX 设备发现接口

### 3.1 初始化与反初始化

#### NSTACKX_Init

```c
/**
 * @brief 初始化 NStackX
 *
 * @param [in] parameter 初始化参数
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 *
 * @code
 * NSTACKX_Parameter param = {0};
 * param.businessType = NSTACKX_BUSINESS_TYPE_DFILE;
 * param.moduleId = 0;
 *
 * int32_t ret = NSTACKX_Init(&param);
 * @endcode
 */
DFINDER_EXPORT int32_t NSTACKX_Init(const NSTACKX_Parameter *parameter);
```

**代码位置**：`nstackx_ctrl/interface/nstackx.h:48`

#### NSTACKX_Deinit

```c
/**
 * @brief 反初始化 NStackX
 */
DFINDER_EXPORT void NSTACKX_Deinit(void);
```

**代码位置**：`nstackx_ctrl/interface/nstackx.h:58`

### 3.2 设备注册

#### NSTACKX_RegisterDevice

```c
/**
 * @brief 注册本地设备信息
 *
 * @param [in] localDeviceInfo 本地设备信息
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 */
DFINDER_EXPORT int32_t NSTACKX_RegisterDevice(const NSTACKX_LocalDeviceInfo *localDeviceInfo);
```

**代码位置**：`nstackx_ctrl/interface/nstackx.h:29`

### 3.3 设备发现

#### NSTACKX_StartDeviceFind

```c
/**
 * @brief 开始设备发现
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 */
DFINDER_EXPORT int32_t NSTACKX_StartDeviceFind(void);
```

**代码位置**：`nstackx_ctrl/interface/nstackx.h:73`

#### NSTACKX_StopDeviceFind

```c
/**
 * @brief 停止设备发现
 */
DFINDER_EXPORT int32_t NSTACKX_StopDeviceFind(void);
```

**代码位置**：`nstackx_ctrl/interface/nstackx.h:85`

### 3.4 设备列表

#### NSTACKX_GetDeviceList

```c
/**
 * @brief 获取缓存的设备列表
 *
 * @param [out] deviceList    设备列表缓冲区
 * @param [in,out] deviceCountPtr 缓冲区大小（输入）/实际数量（输出）
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 *
 * @code
 * NSTACKX_DeviceInfo devices[10];
 * uint32_t count = 10;
 *
 * int32_t ret = NSTACKX_GetDeviceList(devices, &count);
 * if (ret == 0) {
 *     for (uint32_t i = 0; i < count; i++) {
 *         // 处理每个设备
 *     }
 * }
 * @endcode
 */
DFINDER_EXPORT int32_t NSTACKX_GetDeviceList(NSTACKX_DeviceInfo *deviceList, uint32_t *deviceCountPtr);
```

**代码位置**：`nstackx_ctrl/interface/nstackx.h:194`

---

## 4. 错误码定义

### Fillp 错误码

Fillp 使用负值表示错误，具体错误码定义请参考 `fillpinc.h`。

### DFile 错误码

DFile 使用 `nstackx_error.h` 中定义的错误码：

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 负值 | 失败（具体含义见 nstackx_error.h） |

### NStackX 错误码

NStackX 使用与 DFile 相同的错误码体系。

---

## 相关文档

- [项目概览](./00_Overview.md) - 核心能力介绍
- [架构说明](./01_Architecture.md) - 组件交互和线程模型
- [模块结构](./02_Module_Structure.md) - 目录结构详解
- [构建系统](./05_Build_System.md) - GN 构建配置

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
*API 证据来源：头文件代码分析*
