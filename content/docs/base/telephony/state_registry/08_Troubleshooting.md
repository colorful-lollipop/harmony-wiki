# 故障排查指南

## 常见构建问题

### 问题 1：编译找不到头文件

**错误信息**：
```
fatal error: 'telephony_observer_client.h' file not found
```

**原因分析**：
`include_dirs` 配置不完整或路径错误。

**解决方案**：
```bash
# 验证头文件存在
ls -la frameworks/native/observer/include/
ls -la services/include/

# 检查 BUILD.gn 中的 include_dirs
cat BUILD.gn | grep -A 10 "include_dirs"
```

**证据来源**：`BUILD.gn` 中 `include_dirs` 配置。

### 问题 2：依赖库版本不匹配

**错误信息**：
```
error: undefined reference to 'TelephonyObserverClient::GetInstance()'
```

**原因分析**：
`core_service` API 库版本与代码不匹配。

**解决方案**：
```bash
# 更新依赖
./build.sh --product-name <product> --clean

# 或仅更新 telephony 相关
hb build -p <product> -f //base/telephony/core_service
hb build -p <product> -f //base/telephony/state_registry
```

### 问题 3：Sanitizer 检查失败

**错误信息**：
```
error: CFI: control flow integrity check failed
```

**原因分析**：
启用了 CFI 检查但存在非法函数指针调用。

**解决方案**：
```gn
# 临时禁用 CFI 进行调试（仅限调试版本）
ohos_shared_library("tel_state_registry") {
  sanitize = {
    cfi = false
    cfi_cross_dso = false
    debug = false
  }
}
```

**证据来源**：`BUILD.gn` 中 `sanitize` 配置。

## 常见运行时问题

### 问题 1：观察者注册无响应

**症状**：
调用 `observer.on()` 后，状态变化时没有收到回调。

**排查步骤**：

1. **检查回调参数**：
```typescript
// 错误示例：回调函数签名不正确
observer.on('callStateChange', (err) => {});

// 正确示例：需要接收 data 参数
observer.on('callStateChange', (err, data) => {
    if (err) {
        console.error(`错误: ${err.code} - ${err.message}`);
        return;
    }
    console.log(`状态: ${data.state}, 号码: ${data.number}`);
});
```

2. **检查权限**：
```typescript
// 检查是否已申请必要权限
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

async function checkPermission(permission: string): Promise<boolean> {
    const atManager = abilityAccessCtrl.createAtManager();
    const bundleInfo = await bundleManager.getBundleInfoForSelf(
        bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );
    return atManager.checkAccessTokenSync(
        bundleInfo.appInfo.accessTokenId,
        permission
    ) === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
}
```

3. **检查服务状态**：
```bash
# 查看 StateRegistry SA 状态
hilog | grep -i "StateRegistry"

# 期望输出示例：
# 01-01 00:00:00.000 1234 5678 I StateRegistry: StateRegistryService started
# 01-01 00:00:01.000 1234 5678 I StateRegistry: RegisterObserver success, type=callStateChange
```

4. **检查 SIM 卡状态**：
```typescript
import sim from '@ohos.telephony.sim';

async function checkSimState() {
    const simState = await sim.getSimState(0);
    console.log(`SIM 状态: ${simState}`);
    if (simState !== sim.SimState.SIM_STATE_READY) {
        console.warn('SIM 卡未就绪，无法监听状态变化');
    }
}
```

### 问题 2：回调触发过于频繁

**症状**：
`signalInfoChange` 事件每秒触发数十次，导致应用性能下降。

**解决方案**：
```typescript
// 实现防抖/节流机制
class ObserverThrottle {
    private lastCall = 0;
    private readonly interval = 1000; // 1秒
    
    onSignalChange(handler: (data: SignalInformation) => void) {
        return observer.on('signalInfoChange', { slotId: 0 }, (err, data) => {
            const now = Date.now();
            if (now - this.lastCall < this.interval) {
                return; // 节流
            }
            this.lastCall = now;
            handler(data[0]);
        });
    }
}
```

### 问题 3：多卡槽事件混淆

**症状**：
双卡设备上，两个卡槽的事件混合在一起，无法区分。

**解决方案**：
```typescript
// 明确指定 slotId
const slot1Observer = observer.on('networkStateChange', { slotId: 0 }, handler1);
const slot2Observer = observer.on('networkStateChange', { slotId: 1 }, handler2);

// 检查事件来源
observer.on('networkStateChange', (err, data) => {
    const slotId = data.slotId; // 确认来源
    console.log(`卡槽 ${slotId} 网络状态变化`);
});
```

## 调试方法

### 1. 使用 hilog 查看日志

```bash
# 过滤 StateRegistry 相关日志
hilog | grep -E "StateRegistry|Observer|Telephony"

# 查看详细调试日志
hilog -D all | grep -i "state_registry"
```

### 2. 使用 hdc 调试

```bash
# 连接设备
hdc list targets

# 进入 shell
hdc shell

# 查看进程
ps -A | grep telephony

# 查看加载的模块
cat /proc/<pid>/maps | grep -i state_registry
```

### 3. SA Dump

```bash
# 获取 StateRegistry SA 状态
hdc shell
sa_conn <sa_id>
dump -a <sa_id>
```

### 4. GDB 调试（仅限调试版本）

```bash
# 附加到进程
gdbclient <pid>

# 设置断点
break TelephonyStateRegistryService::RegisterObserver

# 运行并触发
continue

# 查看调用栈
backtrace
```

## 性能问题排查

### 回调延迟检测

```typescript
// 测量回调延迟
const start = Date.now();
observer.on('networkStateChange', (err, data) => {
    const latency = Date.now() - start;
    if (latency > 1000) {
        console.warn(`回调延迟过高: ${latency}ms`);
    }
});
```

### 内存占用检查

```bash
# 查看进程内存占用
hdc shell
cat /proc/<pid>/status | grep -E "VmRSS|VmSize|VmData"
```

## 相关文档

- [JS API](03_JS_API.md)
- [GN 构建](05_GN_Build.md)
- [安全评审](07_Security_Review.md)
