# DSoftBus 问题定位指南

## 常见问题

### 问题 1: 设备发现失败

#### 症状

- `RefreshLNN()` 返回失败
- 设备无法被其他设备发现

#### 排查步骤

1. **检查蓝牙/WiFi 状态**

```bash
# 查看蓝牙状态
hdc shell bm -d

# 查看 WiFi 状态
hdc shell bm -w
```

2. **检查权限**

```javascript
// 应用需要声明权限
"permissions": [
  "ohos.permission.DISTRIBUTED_DATASYNC",
  "ohos.permission.DISTRIBUTED_SOFTBUS_CENTER"
]
```

3. **查看日志**

```bash
# 过滤 discovery 日志
hdc shell hilog | grep -E "Disc|discovery"
```

4. **检查 CoAP 配置**

```c
// core/discovery/coap/nstackx_coap/
// 检查 CoAP 监听端口
```

#### 常见原因

| 原因 | 解决方案 |
|-----|---------|
| 蓝牙未开启 | 开启蓝牙 |
| WiFi 未连接 | 连接 WiFi |
| 权限未申请 | 动态申请权限 |
| 防火墙阻止 | 检查网络策略 |

---

### 问题 2: 连接建立失败

#### 症状

- `JoinLNN()` 返回错误
- 认证超时

#### 排查步骤

1. **检查认证日志**

```bash
hdc shell hilog | grep -E "Auth|auth"
```

2. **检查设备绑定状态**

```javascript
// 需要先绑定设备
// 参考 Security 文档
```

3. **检查连接类型**

```c
// ConnectionAddrType
typedef enum {
    CONNECTION_ADDR_WLAN = 0,
    CONNECTION_ADDR_BR,
    CONNECTION_ADDR_BLE,
    CONNECTION_ADDR_ETH,
} ConnectionAddrType;
```

4. **检查超时配置**

```c
// 默认超时 30s
// 可通过参数调整
```

#### 常见原因

| 原因 | 错误码 | 解决方案 |
|-----|--------|---------|
| 设备未绑定 | `SOFTBUS_NETWORK_DEV_NOT_TRUST` | 完成设备绑定 |
| 认证超时 | `SOFTBUS_AUTH_TIMEOUT` | 检查网络 |
| 连接类型不支持 | `SOFTBUS_CONN_MANAGER_TYPE_NOT_SUPPORT` | 检查硬件能力 |

---

### 问题 3: 传输数据失败

#### 症状

- `SendBytes()` 返回错误
- 数据传输中断

#### 排查步骤

1. **检查会话状态**

```bash
hdc shell hilog | grep -E "Session|Trans"
```

2. **检查 QoS 配置**

```c
// QosType
typedef enum {
    QOS_TYPE_MIN_BW,
    QOS_TYPE_MAX_WAIT_TIMEOUT,
    QOS_TYPE_MIN_LATENCY,
    QOS_TYPE_RTT_LEVEL,
} QosType;
```

3. **检查通道状态**

```c
// SessionInfo
typedef struct {
    int32_t channelId;
    ChannelType channelType;
    SessionState state;
} SessionInfo;
```

#### 常见原因

| 原因 | 错误码 | 解决方案 |
|-----|--------|---------|
| 会话未建立 | `SOFTBUS_TRANS_INVALID_SESSION_ID` | 先创建会话 |
| 通道关闭 | `SOFTBUS_TRANS_PROXY_DISCONNECTED` | 重连 |
| 缓冲区满 | `SOFTBUS_CONNECTION_ERR_SENDQUEUE_FULL` | 等待重试 |

---

### 问题 4: N-API 调用崩溃

#### 症状

- JS 应用闪退
- N-API 回调异常

#### 排查步骤

1. **检查 JS 对象生命周期**

```javascript
// 避免过早释放对象
const conn = linkEnhance.createConnection();
// 使用完毕后再释放
conn.close();
```

2. **检查错误码**

```javascript
try {
    await proxyChannelManager.openProxyChannel(options);
} catch (err) {
    console.error('Error:', err.code, err.message);
}
```

3. **查看崩溃日志**

```bash
hdc shell pidof com.example.app
hdc shell cat /proc/pidof/maps
```

#### 常见原因

| 原因 | 解决方案 |
|-----|---------|
| JS 对象已释放 | 检查生命周期 |
| 参数为空 | 添加空值检查 |
| 回调重复注册 | 避免重复 on/off |

---

## 日志查看

### 日志标签

| 标签 | 模块 |
|-----|------|
| `COMM_SDK` | SDK 层 |
| `COMM_DISC` | 发现模块 |
| `COMM_CONN` | 连接模块 |
| `COMM_AUTH` | 认证模块 |
| `COMM_TRANS` | 传输模块 |
| `COMM_LNN` | 网络模块 |

### 日志命令

```bash
# 查看所有 dsoftbus 日志
hdc shell hilog | grep -E "dsoftbus|softbus|Softbus"

# 按模块过滤
hdc shell hilog | grep "COMM_SDK"

# 查看错误级别
hdc shell hilog | grep -E "E/|error|Error"
```

### HiSysEvent

**配置文件**: `hisysevent.yaml`

```yaml
SOFTBUS:
  type: occurrence
  level: info
  domain: communication
  name: softbus_event
```

---

## 调试技巧

### 1. 启用详细日志

```c
// 在代码中添加
#define CONFIG_COMM_LOG_LEVEL 3
```

### 2. 使用 trace

```c
// 添加打点
HILOGI(COMM_SDK, "Enter function");
```

### 3. 调试 N-API

```javascript
// 开发模式下启用
const dsoftbus = require('@ohos.distributedsched.linkEnhance');
// 添加断点调试
```

---

## 错误码速查

### 公共错误

| 错误码 | 说明 |
|-------|------|
| `SOFTBUS_OK` | 成功 |
| `SOFTBUS_INVALID_PARAM` | 参数无效 |
| `SOFTBUS_MEM_ERR` | 内存错误 |
| `SOFTBUS_PERMISSION_DENIED` | 权限拒绝 |
| `SOFTBUS_TIMOUT` | 超时 |

### 发现模块错误

| 错误码 | 说明 |
|-------|------|
| `SOFTBUS_NETWORK_NOT_FOUND` | 未发现设备 |
| `SOFTBUS_NETWORK_REFRESH_LNN_FAILED` | 发现失败 |
| `SOFTBUS_NETWORK_PUBLISH_LNN_FAILED` | 发布失败 |

### 认证模块错误

| 错误码 | 说明 |
|-------|------|
| `SOFTBUS_AUTH_TIMEOUT` | 认证超时 |
| `SOFTBUS_AUTH_CONN_FAIL` | 连接失败 |
| `SOFTBUS_AUTH_HICHAIN_NOT_TRUSTED` | 设备不信任 |

### 传输模块错误

| 错误码 | 说明 |
|-------|------|
| `SOFTBUS_TRANS_INVALID_SESSION_ID` | 无效会话 |
| `SOFTBUS_TRANS_SEND_LEN_BEYOND_LIMIT` | 发送超限 |
| `SOFTBUS_TRANS_FILE_PERMISSION_DENIED` | 文件权限拒绝 |

> 完整错误码参考: `interfaces/kits/common/softbus_error_code.h`

---

## 性能问题

### 设备发现耗时

| 场景 | 预期时间 | 优化建议 |
|-----|---------|---------|
| BLE 发现 | < 5s | 减少广播间隔 |
| CoAP 发现 | < 3s | 优化网络配置 |
| 首次认证 | < 10s | 预认证 |

### 数据传输性能

| 场景 | 预期带宽 | 优化建议 |
|-----|---------|---------|
| WiFi 直连 | > 50 MB/s | QoS 配置 |
| BLE | < 1 MB/s | 避免大文件 |
| BR | < 3 MB/s | 适用小数据 |

---

**相关文档**

- [项目概览](./01_Overview.md)
- [架构说明](./02_Architecture.md)
- [N-API 接口](./03_NAPI.md)
- [安全风险评审](./06_Security.md)
