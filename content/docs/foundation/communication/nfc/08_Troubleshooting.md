# 常见问题与问题排查

## 文档信息

| 项目 | 内容 |
|------|------|
| **目的** | 提供 NFC 组件常见问题排查指南 |
| **适用范围** | 开发者、测试工程师、技术支持 |
| **相关文档** | [架构设计](01_Architecture.md)、[构建产物](07_Build_Artifacts.md) |

---

## 1. 问题排查工具

### 1.1 日志查看

```bash
# 查看 NFC 相关日志
hilog | grep -i nfc

# 查看 NFC 服务日志
hilog | grep Nfc_Core

# 查看 N-API 层日志
hilog | grep Nfc_EtsFwk

# 实时查看
hilog -g nfc

# 查看指定级别以上日志
hilog -l INFO | grep nfc
```

### 1.2 服务状态检查

```bash
# 检查 NFC 服务进程
ps -ef | grep nfc

# 检查 SA 状态
ls -l /system/profile/1140.json

# 检查服务是否注册
samgr_tool -l | grep 1140
```

### 1.3 库文件检查

```bash
# 检查库文件是否存在
ls -l /system/lib/libnfc*.so
ls -l /system/lib/module/nfc/

# 检查库依赖
ldd /system/lib/libnfc_service.z.so

# 检查 SELinux 标签
ls -Z /system/lib/libnfc_service.z.so
```

### 1.4 权限检查

```bash
# 检查 NFC 权限
acm getperm ohos.permission.NFC
acm getperm ohos.permission.NFC_TAG
acm getperm ohos.permission.NFC_CARD_EMULATION

# 检查应用权限
bm dump -n com.example.app | grep permission
```

---

## 2. NFC 无法开启

### 2.1 现象
- 调用 `enableNfc()` 返回错误
- NFC 状态始终为 `STATE_OFF`
- 设置界面 NFC 开关无法打开

### 2.2 排查步骤

**步骤 1: 检查硬件支持**
```bash
# 检查 NFC 是否被标记为不支持
param get const.nfc.not_support
# 期望输出: false
```

**步骤 2: 检查服务状态**
```bash
# 检查 NFC 服务进程是否存在
ps -ef | grep nfc_service

# 检查 SA 是否已注册
samgr_tool -l | grep 1140
```

**步骤 3: 查看日志**
```bash
hilog | grep -E "(NfcService|TurnOn|DoTurnOn)"
```

**常见错误日志**:
```
# NCI 初始化失败
E/Nfc_Core: NciNativeSelector: load libnci_native_default.z.so fail

# 权限错误
E/Nfc_Core: NfcPermissionChecker: permission denied

# 固件下载超时
E/Nfc_Core: NfcService: firmware download timeout
```

### 2.3 解决方案

| 问题 | 解决方案 |
|------|----------|
| NCI 库缺失 | 检查 `libnci_native_default.z.so` 是否存在 |
| HAL 服务未就绪 | 检查 `const.nfc.hal_service.ready` 参数是否为 true |
| 权限错误 | 确认应用具有 `ohos.permission.NFC` 权限 |
| 固件超时 | 检查 NFC 芯片连接，可能需要重启设备 |

---

## 3. 标签无法读取

### 3.1 现象
- 标签靠近设备无反应
- `getTagInfo()` 返回 null
- 前台分发未触发

### 3.2 排查步骤

**步骤 1: 检查 NFC 状态**
```javascript
import controller from '@ohos.nfc.controller';

// 检查 NFC 是否开启
let state = controller.getNfcState();
console.log('NFC State:', state);  // 应为 STATE_ON (3)
```

**步骤 2: 检查权限**
```bash
# 检查应用是否具有 NFC_TAG 权限
acm getperm ohos.permission.NFC_TAG

# 检查日志中的权限错误
hilog | grep -i "permission denied"
```

**步骤 3: 检查标签分发日志**
```bash
hilog | grep -E "(TagDispatcher|OnTagDiscovered|RegForegroundDispatch)"
```

**步骤 4: 检查 NCI 层日志**
```bash
hilog | grep -E "(NciTagProxy|OnTagDiscovered|tag_nci_adapter)"
```

### 3.3 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| `ERR_TAG_STATE_NFC_CLOSED` | NFC 未开启 | 先调用 `enableNfc()` |
| `ERR_NO_PERMISSION` | 缺少权限 | 申请 `ohos.permission.NFC_TAG` |
| `ERR_TAG_APP_NOT_FOREGROUND` | 应用不在前台 | 确保应用在读取标签时在前台 |
| `ERR_TAG_APP_NOT_REGISTERED` | 未注册前台分发 | 调用 `registerForegroundDispatch()` |

---

## 4. HCE 卡模拟失败

### 4.1 现象
- `startHce()` 返回错误
- APDU 命令未收到回调
- 读卡器无法识别设备

### 4.2 排查步骤

**步骤 1: 检查权限**
```bash
hilog | grep -E "(HceSession|StartHce|permission)"
```

**步骤 2: 检查 AID 路由**
```bash
hilog | grep -E "(AddAidRouting|CeService|routing)"
```

**步骤 3: 检查 APDU 流程**
```bash
hilog | grep -E "(OnCeApduData|SendRawFrame|HceCmdCallback)"
```

### 4.3 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| `ERR_HCE_PARAMETERS` | Bundle name 不匹配 | 确保 element.bundleName 与调用者一致 |
| `ERR_NO_PERMISSION` | 缺少权限 | 申请 `ohos.permission.NFC_CARD_EMULATION` |
| `ERR_HCE_STATE_NFC_CLOSED` | NFC 未开启 | 先开启 NFC |
| `ERR_NOT_SYSTEM_APP` | 非系统应用 | HCE 功能仅限系统应用使用 |

---

## 5. N-API 调用错误

### 5.1 现象
- JS 调用 NFC API 抛出异常
- Promise 被拒绝
- 回调未触发

### 5.2 排查步骤

**步骤 1: 检查模块加载**
```javascript
// 检查模块是否正确导入
try {
    import controller from '@ohos.nfc.controller';
    console.log('Module loaded:', controller);
} catch (e) {
    console.error('Module load failed:', e);
}
```

**步骤 2: 检查参数**
```javascript
// 确保参数类型正确
controller.enableNfc().then(() => {
    console.log('Success');
}).catch((err) => {
    console.error('Error code:', err.code);
    console.error('Error message:', err.message);
});
```

**步骤 3: 查看 N-API 日志**
```bash
hilog | grep Nfc_EtsFwk
hilog | grep "BUSI_ERR"
```

### 5.3 常见错误码

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| 201 | 权限不足 | 检查应用权限配置 |
| 202 | 非系统应用 | 该 API 仅限系统应用使用 |
| 401 | 参数错误 | 检查参数类型和数量 |
| 801 | 能力不支持 | 设备不支持 NFC |
| 3100100+ | NFC 状态错误 | 检查 NFC 是否开启 |
| 3100200+ | 标签 I/O 错误 | 检查标签连接 |
| 3100300+ | 卡模拟错误 | 检查 HCE 配置 |

---

## 6. 性能问题

### 6.1 NFC 开启慢

**可能原因**:
- 固件下载超时
- NCI 初始化缓慢

**排查**:
```bash
# 查看初始化时间
hilog | grep -E "(Initialize|WAIT_MS_INIT|DoTurnOn)"
```

**优化建议**:
- 检查 `WAIT_MS_INIT` 配置 (默认 90 秒)
- 确保 NFC HAL 服务正常启动

### 6.2 标签响应慢

**可能原因**:
- APDU 超时设置过短
- 标签距离/位置问题

**排查**:
```bash
hilog | grep -E "(Transceive|timeout|SetTimeout)"
```

---

## 7. 崩溃问题

### 7.1 服务崩溃

**排查**:
```bash
# 查看崩溃日志
hilog | grep -E "(Fatal|Crash|Abort)"

# 检查 tombstone
ls -l /data/tombstones/
```

**常见崩溃原因**:
- NCI 回调空指针
- IPC 连接断开
- 内存分配失败

### 7.2 N-API 崩溃

**排查**:
```bash
# 查看 JS 运行时日志
hilog | grep -E "(napi|Napi|exception)"
```

---

## 8. 调试技巧

### 8.1 开启 Debug 日志

```bash
# 临时开启 Debug 日志
param set persist.nfc.debug 1

# 重启 NFC 服务
killall nfc_service
# 等待自动重启
```

### 8.2 抓取完整日志

```bash
# 抓取启动到第一次操作的完整日志
hilog -r -g nfc > /data/nfc_debug.log &
# 执行 NFC 操作
kill %1
```

### 8.3 使用 gdb 调试

```bash
# 附加到 NFC 服务进程
gdb /system/lib/libnfc_service.z.so $(pidof nfc_service)

# 设置断点
(gdb) break NfcService::Initialize
(gdb) continue
```

---

## 9. 参考信息

### 9.1 日志标签

| 标签 | 来源 | 说明 |
|------|------|------|
| Nfc_Core | 服务层 | 核心服务日志 |
| Nfc_EtsFwk | ETS 框架 | Taihe 框架日志 |
| Nfc_NAPI | N-API | N-API 层日志 |

### 9.2 关键参数

| 参数名 | 说明 |
|--------|------|
| `const.nfc.not_support` | NFC 是否被标记为不支持 |
| `const.nfc.hal_service.ready` | NFC HAL 服务是否就绪 |
| `const.nfc.state` | NFC 状态 (on/off) |
| `persist.nfc.debug` | Debug 日志开关 |

### 9.3 常用命令速查

```bash
# 重启 NFC 服务
killall nfc_service

# 查看 NFC 状态
param get const.nfc.state

# 手动触发 NFC 开关
param set const.nfc.state on

# 检查 SA 状态
samgr_tool -l | grep 1140

# 查看库依赖
ldd /system/lib/libnfc_service.z.so
```

