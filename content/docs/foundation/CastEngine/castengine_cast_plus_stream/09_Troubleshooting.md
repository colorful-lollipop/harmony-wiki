# 09_Troubleshooting - 常见问题与定位

> 本文档提供 Cast+ Stream 模块的常见问题、调试方法和日志分析指南。

---

## 1. 常见问题

### 1.1 连接问题

#### 问题: 设备发现成功但无法连接

**现象**: 能搜索到设备，但点击连接后失败

**可能原因**:
1. 网络不通（防火墙、不同网段）
2. 权限未授予
3. 设备端未启动服务

**排查步骤**:
```bash
# 1. 检查网络连通性
ping {device_ip}

# 2. 检查权限
hdc shell aa dump -a | grep cast

# 3. 检查服务状态
hdc shell ps -A | grep cast

# 4. 查看日志
hdc shell hilog | grep -E "Cast-Session|ChannelManager"
```

**关键日志**:
```
# 正常流程
I Cast-SessionImpl: AddDevice success
I Cast-SessionImpl: ProcessConnect start
I Cast-SessionImpl: CreateChannel success
I RtspController: Start RTSP session

# 异常日志
E Cast-SessionImpl: AddDevice failed, permission denied
E ChannelManager: CreateChannel failed, network unreachable
E RtspController: RTSP handshake timeout
```

#### 问题: RTSP 握手失败

**现象**: 连接建立但 M1-M6 握手失败

**可能原因**:
1. 协议版本不匹配
2. 参数协商失败
3. 加密算法不支持

**排查步骤**:
```bash
# 查看 RTSP 详细日志
hdc shell hilog -b D -D 0xD002b2b
hdc shell hilog | grep -E "RtspController|RTSP"
```

**关键日志**:
```
# 正常握手
D RtspController: Send M1: OPTIONS
D RtspController: Recv M2: 200 OK
D RtspController: Send M3: GET_PARAMETER
D RtspController: Recv M4: 200 OK
...

# 异常日志
E RtspController: Unsupported RTSP method: XXX
E RtspController: Parameter negotiation failed
E RtspController: Encryption algorithm not supported
```

### 1.2 播放问题

#### 问题: 镜像投屏黑屏

**现象**: 连接成功但 Sink 端黑屏

**可能原因**:
1. Surface 未设置
2. 编码失败
3. 视频通道未建立

**排查步骤**:
```bash
# 1. 检查 Surface 设置
hdc shell hilog | grep -i surface

# 2. 检查视频通道
hdc shell hilog | grep -E "VIDEO|Video"

# 3. 检查编码器
hdc shell hilog | grep -E "codec|encoder"
```

**关键日志**:
```
# 正常流程
I MirrorPlayer: SetSurface success
I MirrorPlayer: Video encoder started
I ChannelManager: VIDEO channel created
I ChannelManager: Video frame sent

# 异常日志
E MirrorPlayer: SetSurface failed, invalid producer
E MirrorPlayer: Video encoder init failed
E ChannelManager: VIDEO channel creation failed
```

#### 问题: 流媒体播放卡顿

**现象**: 播放不流畅，频繁缓冲

**可能原因**:
1. 网络带宽不足
2. 缓存设置不当
3. 解码性能不足

**排查步骤**:
```bash
# 1. 检查网络质量
hdc shell ping -c 10 {device_ip}

# 2. 查看缓存状态
hdc shell hilog | grep -i cache

# 3. 查看解码性能
hdc shell hilog | grep -E "decode|render"
```

### 1.3 权限问题

#### 问题: 权限检查失败

**现象**: 调用接口返回 `ERR_UNKNOWN_TRANSACTION`

**可能原因**:
1. 应用未申请权限
2. 权限检查实现问题

**排查步骤**:
```bash
# 1. 检查应用权限
hdc shell aa dump -p {bundle_name}

# 2. 查看权限日志
hdc shell hilog | grep -i permission
```

**关键日志**:
```
E Cast-Permission: Permission denied: ohos.permission.ACCESS_CAST_ENGINE_MIRROR
E CastSessionImplStub: DoCreateMirrorPlayer permission check failed
```

**解决方案**:
在应用 `config.json` 中添加权限声明:
```json
{
  "module": {
    "reqPermissions": [
      {
        "name": "ohos.permission.ACCESS_CAST_ENGINE_MIRROR"
      },
      {
        "name": "ohos.permission.ACCESS_CAST_ENGINE_STREAM"
      }
    ]
  }
}
```

---

## 2. 调试方法

### 2.1 日志级别设置

```bash
# 开启所有 CastEngine 调试日志
hdc shell hilog -b D -D 0xD004601
hdc shell hilog -b D -D 0xD002B00
hdc shell hilog -b D -D 0xD00ff00
hdc shell hilog -b D -D 0xD002b2b
hdc shell hilog -b D -D 0xD003900
hdc shell hilog -b D -D 0xD0015c0

# 关闭日志
hdc shell hilog -b W -D 0xD004601
```

### 2.2 日志过滤

```bash
# 按标签过滤
hdc shell hilog | grep "Cast-SessionImpl"

# 按关键字过滤
hdc shell hilog | grep -E "error|failed|timeout"

# 实时查看
hdc shell hilog -g | grep "Cast"
```

### 2.3 抓包分析

```bash
# 抓取 SoftBus 流量
hdc shell tcpdump -i any -w /data/cast.pcap

# 抓取特定端口
hdc shell tcpdump -i any port 4455 -w /data/cast_tcp.pcap
```

### 2.4 性能分析

```bash
# CPU 使用率
hdc shell top -p $(hdc shell pidof cast_engine_service)

# 内存使用
hdc shell dumpsys meminfo cast_engine_service

# 线程状态
hdc shell ps -T -p $(hdc shell pidof cast_engine_service)
```

---

## 3. 日志分析

### 3.1 正常流程日志

#### 会话建立
```
[正常流程日志模板]

1. 添加设备
I Cast-SessionImpl: AddDevice deviceId={device_id}
I Cast-SessionImpl: ProcessConnect start

2. 创建通道
I ChannelManager: CreateChannel linkType={type} moduleType={type}
I ChannelManager: Connection established

3. RTSP 握手
D RtspController: Send M1: OPTIONS * RTSP/1.0
D RtspController: Recv M2: 200 OK
D RtspController: Send M3: GET_PARAMETER
D RtspController: Recv M4: 200 OK
...
D RtspController: Session established

4. 状态变更
I Cast-SessionImpl: State changed: DISCONNECTED -> CONNECTING
I Cast-SessionImpl: State changed: CONNECTING -> CONNECTED
I Cast-SessionImpl: OnDeviceState state=CONNECTED
```

#### 播放控制
```
[播放控制日志模板]

1. 开始播放
I Cast-SessionImpl: Play deviceId={device_id}
D RtspController: Send PLAY request
D RtspController: Recv 200 OK
I Cast-SessionImpl: State changed: PAUSED -> PLAYING

2. 暂停
I Cast-SessionImpl: Pause deviceId={device_id}
D RtspController: Send PAUSE request
I Cast-SessionImpl: State changed: PLAYING -> PAUSED
```

### 3.2 异常日志模式

#### 连接超时
```
E Cast-SessionImpl: ProcessConnect timeout
E ChannelManager: Connection timeout
E RtspController: RTSP handshake timeout
```

**可能原因**:
- 网络不通
- 对端未响应
- 防火墙阻断

#### 参数协商失败
```
E RtspController: Parameter negotiation failed
E RtspController: Unsupported video codec: XXX
E RtspController: Unsupported audio codec: XXX
```

**可能原因**:
- 编解码器不支持
- 参数范围不匹配

#### 通道错误
```
E ChannelManager: Channel error, errorCode={code}
E SoftBusConnection: SoftBus error: {error_msg}
E TcpConnection: TCP error: {error_msg}
```

**可能原因**:
- 网络断开
- 对端关闭连接
- 传输错误

---

## 4. 调试技巧

### 4.1 状态机调试

```bash
# 查看当前会话状态
hdc shell hilog | grep "State changed"

# 查看状态转换历史
hdc shell hilog | grep -E "Enter|Exit" | grep State
```

### 4.2 通道调试

```bash
# 查看通道创建/销毁
hdc shell hilog | grep -E "CreateChannel|DestroyChannel"

# 查看通道数据
hdc shell hilog | grep "OnDataReceived"
```

### 4.3 RTSP 调试

```bash
# 查看 RTSP 消息
hdc shell hilog | grep -E "Send|Recv"

# 查看 RTSP 状态
hdc shell hilog | grep "RtspEngineState"
```

---

## 5. 常见问题速查

### 5.1 错误码速查

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| `ERR_NONE` (0) | 成功 | - |
| `ERR_NULL_OBJECT` (-1) | 空对象 | 检查参数 |
| `ERR_INVALID_DATA` (-2) | 无效数据 | 检查数据格式 |
| `ERR_UNKNOWN_TRANSACTION` (-3) | 权限不足 | 申请权限 |
| `IPC_STUB_ERR` (-5) | IPC 错误 | 检查服务状态 |

### 5.2 状态速查

| 状态 | 说明 |
|------|------|
| `DEFAULT` | 初始状态 |
| `DISCONNECTED` | 未连接 |
| `CONNECTING` | 连接中 |
| `CONNECTED` | 已连接 |
| `PLAYING` | 播放中 |
| `PAUSED` | 暂停 |
| `DISCONNECTING` | 断开中 |

### 5.3 模块类型速查

| 类型 | 值 | 说明 |
|------|-----|------|
| `AUTH` | 0 | 认证通道 |
| `RTSP` | 1 | RTSP 控制通道 |
| `VIDEO` | 2 | 视频通道 |
| `AUDIO` | 3 | 音频通道 |
| `STREAM` | 6 | 流媒体通道 |

---

## 6. 联系与支持

### 6.1 相关仓库

- [castengine_cast_framework](https://gitee.com/openharmony-sig/castengine_cast_framework) - 框架层
- [castengine_wifi_display](https://gitee.com/openharmony-sig/castengine_wifi_display) - WiFi Display
- [castengine_dlna](https://gitee.com/openharmony-sig/castengine_dlna) - DLNA

### 6.2 日志收集

遇到问题时，请收集以下信息:

```bash
# 1. 系统日志
hdc shell hilog > cast_log.txt

# 2. 进程信息
hdc shell ps -A | grep cast > cast_ps.txt

# 3. 内存信息
hdc shell dumpsys meminfo > meminfo.txt

# 4. 网络信息
hdc shell ifconfig > network.txt
```

---

## 7. 相关文档

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [08_Security_Review.md](./08_Security_Review.md) - 安全评审
