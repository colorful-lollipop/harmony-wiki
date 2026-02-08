# 问题排查指南

## 构建问题

### 问题 1：头文件找不到

**错误信息**：
```
fatal error: 'mc_controller_manager.h' file not found
```

**原因**：include_dirs 配置缺失。

**解决方案**：
```bash
# 检查 BUILD.gn include_dirs
hb build //foundation/distributedhardware/mechbody_controller/services --gn-args verbose=true
```

**检查文件**：`services/BUILD.gn:98-110`

```gn
include_dirs = [
  "include",
  "include/controller",
  # ... 确认路径配置正确
]
```

---

### 问题 2：外部依赖未找到

**错误信息**：
```
ninja: error: '//third_party/xxx:yyy' is not a known target
```

**原因**：依赖的子系统未构建。

**解决方案**：
```bash
# 同步依赖子系统
hb set
hb build
```

**相关依赖**：
- `bluetooth` - 蓝牙框架
- `camera_framework` - 相机框架
- `ipc` - IPC 框架
- `safwk` - SystemAbility 框架

---

### 问题 3：Sanitizer 编译失败

**错误信息**：
```
undefined reference to '__ubsan_handle_*'
```

**原因**：sanitizer 库未链接。

**解决方案**：确保 `BUILD.gn` 中的 sanitize 配置正确：

```gn
sanitize = {
  boundary_sanitize = true
  cfi = true
  integer_overflow = true
  ubsan = true
}
```

---

## 运行问题

### 问题 4：SA 8550 启动失败

**错误信息**：
```
E0100 00:00:00.000000  mechbody_service: Failed to register SA 8550
```

**原因**：
- SA 配置文件错误
- dlopen 加载失败
- 权限不足

**排查步骤**：

1. 检查 SA 配置：`sa_profile/8550.json`
```json
{
  "process": "mechbody",
  "libpath": "libmechbody_service.z.so"
}
```

2. 检查日志：
```bash
hilog | grep -E "mechbody|MechBody"
```

3. 检查权限：`etc/init/mechbody.cfg`

4. 检查 SELinux：
```bash
sesearch --source mechbody --target system --class process
```

---

### 问题 5：N-API 模块加载失败

**错误信息**：
```
Failed to load N-API module: libmechanicmanager_napi.so
```

**原因**：
- so 文件未安装到正确路径
- 依赖库缺失
- dlopen 失败

**排查步骤**：

1. 检查安装路径：
```bash
ls -la /system/app/NXXXXX/distributedhardware/
```

2. 检查依赖：
```bash
ldd /system/app/NXXXXX/distributedhardware/libmechanicmanager_napi.so
```

3. 检查 dlmopen 错误：
```bash
dmesg | grep mechbody
```

---

### 问题 6：权限拒绝

**错误信息**：
```
E MechBody: PERMISSION_DENIED
```

**原因**：应用未声明或获取所需权限。

**解决方案**：

1. 在 `module.json5` 中声明权限：
```json
"requestPermissions": [
  {
    "name": "ohos.permission.CONNECT_MECHANIC_HARDWARE",
    "reason": "Need to connect mechanical devices",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  }
]
```

2. 运行时申请权限（仅第三方应用）：
```typescript
import { abilityAccessCtrl, bundleManager } from '@kit.AbilityKit';

let atManager = abilityAccessCtrl.createAtManager();
let requestInfo = bundleManager.BundleFlag.getDefaultProc()
try {
  await atManager.requestPermissionsFromUser(context, [
    'ohos.permission.CONNECT_MECHANIC_HARDWARE'
  ]);
} catch (error) {
  console.error('Permission request failed:', error);
}
```

---

### 问题 7：设备发现失败

**错误信息**：
```
getAttachedMechDevices returns empty
```

**原因**：
- 蓝牙未开启
- 设备不在范围内
- BLE 扫描未启动

**排查步骤**：

1. 检查蓝牙状态：
```typescript
import { bluetooth } from '@kit.ConnectivityKit';

let state = bluetooth.getState();
if (state === bluetooth.BluetoothState.STATE_OFF) {
  console.log('Bluetooth is off');
}
```

2. 检查设备是否配对：
```bash
# 设备端确认
hdc shell btmac  // 查看本机 MAC
```

3. 检查蓝牙日志：
```bash
hilog | grep -i bluetooth
```

---

### 问题 8：API 调用超时

**错误信息**：
```
rotate: TIMEOUT
```

**原因**：
- 设备无响应
- 蓝牙连接断开
- 命令队列阻塞

**排查步骤**：

1. 检查设备连接状态：
```typescript
mechManager.on('attachStateChange', (state) => {
  console.log('State:', state);
});
```

2. 检查命令队列：
```bash
# 查看内核日志
hdc shell dmesg | grep mechbody
```

3. 重试操作：
```typescript
try {
  await mechManager.rotate(mechId, params, duration);
} catch (e) {
  if (e.code === TIMEOUT) {
    // 重试
    await mechManager.stopMoving(mechId);
    await mechManager.rotate(mechId, params, duration);
  }
}
```

---

## 调试方法

### 日志查看

**服务日志**：
```bash
hilog | grep -E "mechbody|MechBody|MECH"
```

**详细日志**：
```bash
hilog -D all | grep -E "mechbody"
```

**HiSysEvent**：
```bash
hisysevent dump -n mechbody_controller
```

### 调试命令

**查看 SA 状态**：
```bash
hdc shell sa list
```

**查看进程信息**：
```bash
hdc shell ps -A | grep mechbody
```

**查看 dlopen 加载**：
```bash
hdc shell cat /proc/$(pgrep mechbody)/maps | grep mech
```

### GDB 调试

**附加到进程**：
```bash
hdc gdb
(gdb) attach $(pgrep mechbody)
```

**设置断点**：
```bash
(gdb) break MechBodyControllerService::OnStart
(gdb) continue
```

---

## 常见错误码

| 错误码 | 值 | 说明 | 解决方案 |
|--------|-----|------|----------|
| PARAMETER_CHECK_FAILED | -1 | 参数校验失败 | 检查 API 参数类型和范围 |
| PERMISSION_DENIED | 202 | 权限拒绝 | 申请 ohos.permission.CONNECT_MECHANIC_HARDWARE |
| DEVICE_NOT_CONNECTED | -1 | 设备未连接 | 先调用 getAttachedMechDevices |
| DEVICE_NOT_SUPPORTED | -1 | 设备不支持 | 检查设备兼容性 |
| MECH_ID_INVALID | -1 | 无效设备 ID | 使用 getAttachedMechDevices 获取有效 ID |
| TIMEOUT | -1 | 操作超时 | 检查蓝牙连接，重试 |
| SYSTEM_ERROR | 100 | 系统错误 | 查看日志，联系开发者 |

---

## 性能问题

### 问题 9：API 响应慢

**可能原因**：
- 蓝牙通信延迟
- 命令队列阻塞
- 内存压力

**排查方法**：

1. 添加性能日志：
```typescript
let start = Date.now();
await mechManager.rotate(mechId, params, duration);
console.log('Duration:', Date.now() - start);
```

2. 检查内存：
```bash
hdc shell cat /proc/$(pgrep mechbody)/status | grep -i vm
```

3. 优化建议：
- 减少不必要的 API 调用
- 使用批量命令
- 调整超时参数

---

### 问题 10：内存泄漏

**可能原因**：
- 回调未注销
- N-API handle scope 未释放
- 事件订阅未清理

**排查方法**：

1. 检查回调注销：
```typescript
// 退出时注销回调
mechManager.off('attachStateChange');
```

2. 检查资源释放：
```bash
hdc shell dumpsys meminfo $(pgrep mechbody)
```

3. 使用 ASAN 构建测试：
```bash
hb build --gn-args "mechbody_controller_feature_product=true" --sanitize address
```

---

## 相关文档

- [N-API 参考](03_NAPI_Reference.md) → API 错误码
- [架构说明](02_Architecture.md) → 系统交互
- [安全评审](06_Security_Review.md) → 权限问题
