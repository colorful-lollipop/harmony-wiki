# 07_Troubleshooting - 常见问题与定位方法

## 1. 构建问题

### 1.1 编译失败：找不到依赖头文件

**现象**:
```
error: 'samgr_lite.h' file not found
#include "samgr_lite.h"
         ^~~~~~~~~~~~~
```

**原因**:
- 依赖组件未正确拉取
- `include_dirs` 配置错误

**定位路径**:
1. 检查 `BUILD.gn` 中的 `include_dirs` 配置
2. 确认依赖组件已编译：`hb build --target //foundation/systemabilitymgr/samgr_lite:samgr`
3. 检查 `bundle.json` 中的 `deps` 声明

**解决**:
```bash
# 确保所有依赖已拉取
repo sync -c

# 清理并重新构建
hb clean
hb build
```

### 1.2 链接失败：未定义的符号

**现象**:
```
undefined reference to `CreateSessionServer'
```

**原因**:
- SoftBus 库未链接
- 链接顺序问题

**定位路径**:
1. 检查 `BUILD.gn` 中的 `public_deps` 是否包含 `softbus_client`
2. 确认 `//foundation/communication/dsoftbus/sdk:softbus_client` 已构建

**代码位置**: `BUILD.gn:65`

### 1.3 条件编译问题

**现象**:
- Wearable 产品编译失败
- 某些函数未定义

**原因**:
- `WEARABLE_PRODUCT` 宏影响代码路径

**定位路径**:
```bash
# 检查编译命令中的宏定义
grep -r "WEARABLE_PRODUCT" source/
```

**相关代码**:
- `source/dmslite_parser.c:245` - `CanCall()` 跳过 UID 检查
- `source/dmslite_permission.c:23-42` - BMS 接口差异

## 2. 运行时问题

### 2.1 服务启动失败

**现象**:
- DMS 服务未注册到 SAMGR
- `GetFeatureApi()` 返回 NULL

**定位路径**:

1. **检查服务注册日志**:
```c
// source/dmslite.c:77-78
BOOL result = SAMGR_GetInstance()->RegisterService((Service *)&g_distributedService);
HILOGI("[dms service start %s]", result ? "success" : "failed");
```

2. **检查 Feature 注册日志**:
```c
// source/dmslite_feature.c:114-123
BOOL result = SAMGR_GetInstance()->RegisterFeature(...);
if (!result) {
    HILOGE("[dms register feature failed]");
}
```

3. **确认 init 宏调用**:
```c
// source/dmslite.c:80
SYS_SERVICE_INIT(Init);

// source/dmslite_feature.c:125
SYS_FEATURE_INIT(Init);
```

**常见原因**:
- `libdmslite.so` 未加载
- SAMGR 初始化失败
- 内存不足

### 2.2 远程启动无响应

**现象**:
- `startAbility()` 调用后无反应
- 远程设备 FA 未启动

**定位路径**:

1. **检查日志级别**:
```c
// include/dmslite_log.h:37-65
#if HILOG_COMPILE_LEVEL <= HILOG_LV_DEBUG
    #define HILOGD(fmt, ...) ...
#endif
```

2. **关键日志点**:
```c
// source/dmslite_famgr.c:87
HILOGI("[StartRemoteAbility]");

// source/dmslite_famgr.c:92
HILOGI("[StartRemoteAbility dms busy]");

// source/dmslite_session.c:197
HILOGI("[SendMessage]");

// source/dmslite_session.c:203
HILOGE("[CreateDMSSessionServer error]");
```

3. **检查设备状态**:
```c
// source/dmslite_session.c:251-258
bool IsDmsBusy() {
    if (g_curBusy && IsTimeout() && g_curSessionId >= 0) {
        CloseDMSSession();
    }
    return g_curBusy;
}
```

**常见原因**:
| 原因 | 日志特征 | 解决 |
|------|----------|------|
| 设备忙 | `[StartRemoteAbility dms busy]` | 等待或检查超时 |
| 会话创建失败 | `[CreateDMSSessionServer error]` | 检查 SoftBus |
| 组网失败 | 无设备上线日志 | 检查网络和设备发现 |
| 参数错误 | `[param error!]` | 检查 want 参数 |

### 2.3 权限检查失败

**现象**:
- 返回 `DMS_EC_CHECK_PERMISSION_FAILURE` (7)
- 或 `DMS_REC_PERMISSION_DENIED` (29360302)

**定位路径**:

1. **检查 UID 权限**:
```c
// source/dmslite_parser.c:243-254
static bool CanCall() {
    uid_t callerUid = getuid();
    if (callerUid != FOUNDATION_UID && callerUid != SHELL_UID) {
        HILOGD("[Caller uid is not allowed, uid = %u]", callerUid);
        return false;
    }
}
```

2. **检查签名比对**:
```c
// source/dmslite_permission.c:111-114
if (strcmp(permissionCheckInfo->callerSignature, calleeSignature) != 0) {
    HILOGE("[Signature unmatched]");
    return DMS_EC_CHECK_PERMISSION_FAILURE;
}
```

3. **检查 BMS 查询**:
```c
// source/dmslite_permission.c:86-88
errCode = bmsInterface->GetBundleInfo(permissionCheckInfo->calleeBundleName, ...);
if (errCode != EC_SUCCESS) {
    HILOGE("[GetBundleInfo errCode = %d]", errCode);
    return DMS_EC_GET_BUNDLEINFO_FAILURE;
}
```

**常见原因**:
| 原因 | 错误码 | 解决 |
|------|--------|------|
| UID 不在白名单 | `DMS_EC_FAILURE` | 以 foundation 或 shell 身份运行 |
| 包未安装 | `DMS_EC_GET_BUNDLEINFO_FAILURE` | 在远程设备安装目标应用 |
| 签名不匹配 | `DMS_EC_CHECK_PERMISSION_FAILURE` | 确保双方应用签名一致 |

### 2.4 TLV 解析失败

**现象**:
- 返回 `DMS_EC_PARSE_TLV_FAILURE` (3)
- 或 `DMS_REC_PARSER_TLV_FAIL` (29360301)

**定位路径**:

1. **检查解析日志**:
```c
// source/dmslite_parser.c:239
HILOGI("[errCode = %d]", errCode);

// source/dmslite_parser.c:268
HILOGI("[ProcessCommuMsg commandId %hu]", commandId);
```

2. **TLV 错误码**:
```c
// include/dmslite_tlv_common.h:39-48
typedef enum {
    DMS_TLV_SUCCESS = 0,
    DMS_TLV_ERR_NO_MEM = 1,
    DMS_TLV_ERR_PARAM = 2,
    DMS_TLV_ERR_LEN = 3,
    DMS_TLV_ERR_OUT_OF_ORDER = 4,
    DMS_TLV_ERR_BAD_NODE_NUM = 5,
    DMS_TLV_ERR_UNKNOWN_TYPE = 6,
    DMS_TLV_ERR_BAD_SOURCE = 7,
} TlvErrorCode;
```

**常见原因**:
| 原因 | 错误码 | 解决 |
|------|--------|------|
| 消息格式错误 | `DMS_TLV_ERR_PARAM` | 检查 TLV 格式 |
| 长度字段异常 | `DMS_TLV_ERR_LEN` | 检查 length 字段 |
| 节点顺序错误 | `DMS_TLV_ERR_OUT_OF_ORDER` | 确保 type 递增 |
| 节点数不足 | `DMS_TLV_ERR_BAD_NODE_NUM` | 至少 2 个节点 |

### 2.5 会话相关错误

**现象**:
- `DMS_REC_OPEN_SESSION_FAIL` (29360303)
- `DMS_REC_DEVICE_BUSY` (29360304)

**定位路径**:

1. **检查会话创建**:
```c
// source/dmslite_session.c:214
g_curSessionId = OpenSession(DMS_SESSION_NAME, DMS_SESSION_NAME, 
    deviceId, DMS_MODULE_NAME, &attr);
if (g_curSessionId < 0) {
    // 失败处理
    return EC_FAILURE;
}
```

2. **检查设备状态**:
```c
// source/dmslite_devmgr.c:24
static char g_peerDevId[NETWORK_ID_BUF_LEN] = {0};

// source/dmslite_devmgr.c:37-43
void onNodeOnline(NodeBasicInfo *info) {
    (void)strncpy_s(g_peerDevId, NETWORK_ID_BUF_LEN, 
        info->networkId, sizeof(info->networkId));
    CreateDMSSessionServer();
}
```

**常见原因**:
| 原因 | 错误码 | 解决 |
|------|--------|------|
| 设备离线 | `DMS_REC_OPEN_SESSION_FAIL` | 等待设备上线 |
| 会话忙 | `DMS_REC_DEVICE_BUSY` | 等待当前操作完成 |
| SoftBus 错误 | `DMS_REC_OPEN_SESSION_FAIL` | 检查 SoftBus 日志 |

## 3. 调试方法

### 3.1 开启详细日志

修改 `include/dmslite_log.h`:
```c
// 修改日志编译级别
#define HILOG_COMPILE_LEVEL HILOG_LV_DEBUG  // 开启 DEBUG 级别
```

或在编译时定义:
```gn
defines = [
  "HILOG_COMPILE_LEVEL=HILOG_LV_DEBUG",
]
```

### 3.2 使用 GDB 调试

```bash
# 附加到 foundation 进程
gdb -p $(pidof foundation)

# 设置断点
(gdb) break dmslite_famgr.c:StartRemoteAbility
(gdb) break dmslite_parser.c:ProcessCommuMsg

# 运行
(gdb) continue
```

### 3.3 抓包分析

由于使用 SoftBus 传输，可使用 SoftBus 提供的调试工具:

```bash
# 查看 SoftBus 日志
hilog | grep -i softbus

# 查看会话状态
cat /proc/softbus/session_list
```

### 3.4 模拟测试

使用 XTS 测试模式（跳过实际发送）:

```c
// 定义 XTS_SUITE_TEST 宏
#define XTS_SUITE_TEST

// source/dmslite_famgr.c:95-109
#ifdef XTS_SUITE_TEST
    return DMS_EC_SUCCESS;  // 测试模式直接返回成功
#else
    int32_t ret = SendDmsMessage(...);
    return ret;
#endif
```

## 4. 性能问题

### 4.1 启动延迟高

**可能原因**:
1. 首次 BMS 查询较慢
2. SoftBus 会话建立耗时
3. 网络延迟

**优化建议**:
- 预加载 Bundle 信息
- 复用 SoftBus 会话（当前设计为单会话）
- 优化超时时间设置

### 4.2 内存占用

**关键分配点**:
```c
// source/dmslite_famgr.c:52
RequestData *reqdata = (RequestData *)DMS_ALLOC(sizeof(RequestData));

// source/dmslite_session.c:80
char *message = (char *)DMS_ALLOC(dataLen);

// source/dmslite_parser.c:128
TlvNode *node = (TlvNode *)malloc(sizeof(TlvNode));
```

**检查方法**:
```bash
# 查看进程内存
procrank | grep foundation

cat /proc/$(pidof foundation)/status | grep -i vm
```

## 5. 常见问题速查表

| 问题 | 检查点 | 解决 |
|------|--------|------|
| 服务未启动 | 日志中 "dms service start" | 检查库加载、SAMGR |
| Feature 注册失败 | 日志中 "dms register feature" | 检查服务是否先注册 |
| 远程启动失败 | `IsDmsBusy()` 返回值 | 等待或检查超时 |
| 权限被拒绝 | UID、签名 | 检查调用者身份和签名 |
| 解析失败 | TLV 格式 | 检查消息格式 |
| 会话失败 | 设备组网状态 | 检查 SoftBus 和设备发现 |
| 回调未触发 | 会话关闭时机 | 检查 `InvokeCallback()` 调用点 |

## 6. 日志关键字速查

| 关键字 | 位置 | 含义 |
|--------|------|------|
| `dms service start` | dmslite.c:78 | 服务启动结果 |
| `dms register feature` | dmslite_feature.c:116,122 | Feature/API 注册结果 |
| `StartRemoteAbility` | dmslite_famgr.c:87 | 收到远程启动请求 |
| `dms busy` | dmslite_famgr.c:92 | 当前会话忙 |
| `SendMessage` | dmslite_session.c:197 | 开始发送消息 |
| `CreateDMSSessionServer error` | dmslite_session.c:203 | 会话服务器创建失败 |
| `OnBytesReceived` | dmslite_session.c:74 | 收到数据 |
| `ProcessCommuMsg` | dmslite_parser.c:277 | 处理消息 |
| `errCode` | 多处 | 操作结果码 |
| `Signature unmatched` | dmslite_permission.c:112 | 签名不匹配 |
| `Permission denied` | 多处 | 权限检查失败 |

## 7. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构设计
- [03_Public_API](03_Public_API.md) - 错误码定义
- [06_Security](06_Security.md) - 安全机制
