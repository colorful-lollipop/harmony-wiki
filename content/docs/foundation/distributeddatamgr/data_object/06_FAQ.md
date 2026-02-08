# 常见问题

> 构建、运行、调试过程中的常见问题与解决方案

## 构建问题

### Q1: 编译时提示 "napi_module_register 未定义"

**错误信息**:
```
error: undefined reference to 'napi_module_register'
```

**原因**: 未链接 napi 库

**解决方案**:
```bash
# 检查 BUILD.gn 中的 external_deps 是否包含 napi
# interfaces/jskits/BUILD.gn 应包含:
external_deps = [
    "napi:ace_napi",
    # ... 其他依赖
]
```

**相关文件**: `interfaces/jskits/BUILD.gn:120`

---

### Q2: 编译时提示 "access_token 权限校验失败"

**错误信息**:
```
error: 'AccessTokenKit' is not found
```

**原因**: 未链接 access_token 库

**解决方案**:
```bash
# 检查 BUILD.gn 中的 external_deps 是否包含 access_token
external_deps = [
    "access_token:libaccesstoken_sdk",
    # ... 其他依赖
]
```

**相关文件**:
- `interfaces/innerkits/BUILD.gn:61`
- `interfaces/jskits/BUILD.gn:112`

---

### Q3: 编译 JS 模块时提示 "es2abc 未找到"

**错误信息**:
```
error: cannot find es2abc tool
```

**原因**: es2abc 工具未配置

**解决方案**:
```bash
# 确保已安装 es2abc 工具
which es2abc

# 或检查环境变量
echo $ES2ABC_PATH
```

---

### Q4: 链接时提示 "cannot find -ldistributeddb"

**错误信息**:
```
ld: cannot find -ldistributeddb
```

**原因**: kv_store 组件未编译

**解决方案**:
```bash
# 先编译 kv_store 组件
./build.sh --product name --build-target //foundation/distributeddatamgr/kv_store:distributeddb

# 或全量编译
./build.sh --product name --build-target data_object
```

---

### Q5: 版本脚本错误

**错误信息**:
```
ld: cannot open version script: libnative_dataobject.versionscript
```

**原因**: 版本脚本文件缺失

**检查**:
```bash
# 确认文件存在
ls -la interfaces/innerkits/libnative_dataobject.versionscript

# 如果不存在，检查 BUILD.gn 中的配置
version_script = "libnative_dataobject.versionscript"
```

**相关文件**: `interfaces/innerkits/BUILD.gn:93`

---

## 运行问题

### Q6: 应用启动时提示 "permission denied"

**错误信息**:
```
E/ObjectStore: Permission verification failed
```

**原因**: 缺少 `ohos.permission.DISTRIBUTED_DATASYNC` 权限

**解决方案**:
```json
// 在 module.json5 中添加权限声明
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.DISTRIBUTED_DATASYNC",
        "reason": "Need for distributed data synchronization",
        "usedScene": {
          "abilities": ["MainAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

**代码中动态申请**:
```javascript
import Ability from '@ohos.ability.featureAbility'
import bundle from '@ohos.bundle'

// 动态申请权限
const permissions = ['ohos.permission.DISTRIBUTED_DATASYNC'];
Ability.featureAbility.requestPermissionsFromUser(permissions, (result) => {
    if (result.authResults[0] === 0) {
        console.log('权限申请成功');
    }
});
```

---

### Q7: 数据同步不生效

**现象**:
- 修改数据后其他设备未收到变更
- `on('change')` 回调未触发

**排查步骤**:
```javascript
// 1. 检查 sessionId 是否一致
console.log('本设备 sessionId:', obj['__sessionId']);

// 2. 检查设备是否在同一可信组网
// 使用 DeviceManager 检查在线设备

// 3. 检查权限
const result = await checkPermission('ohos.permission.DISTRIBUTED_DATASYNC');
console.log('权限状态:', result);

// 4. 检查网络状态
obj.on('status', (data) => {
    console.log('设备状态:', data.status);  // online/offline
});
```

**常见原因**:
| 原因 | 解决方法 |
|------|----------|
| sessionId 不一致 | 确保使用相同 sessionId |
| 设备不在线 | 检查网络连接 |
| 权限缺失 | 申请权限 |
| bundleName 不同 | 确保应用 bundleName 一致 |

---

### Q8: setSessionId 返回 false

**错误码**: API 返回 `false`

**排查步骤**:
```javascript
// 1. 检查 sessionId 格式
const sessionId = distributedObject.genSessionId();
console.log('生成的 sessionId:', sessionId);

// 2. 检查是否已加入其他会话
const currentSession = obj['__sessionId'];
console.log('当前会话:', currentSession);

// 3. 检查是否在同一可信组网
// 需要设备管理器 API 检查

// 4. 检查权限
try {
    obj.setSessionId(sessionId);
} catch (e) {
    console.error('设置失败:', e.code, e.message);
}
```

**可能原因**:
- sessionId 格式错误
- 设备不在同一组网
- 权限被撤销

---

### Q9: save 操作失败

**错误码**: 返回非 0 值或回调报错

**排查步骤**:
```javascript
// 1. 检查是否已设置 sessionId
if (obj['__sessionId'] == null || obj['__sessionId'] === '') {
    console.error('未加入会话，无法保存');
    return;
}

// 2. 检查 deviceId 是否有效
const deviceId = 'local';  // 或从 DeviceManager 获取
obj.save(deviceId, (err, result) => {
    if (err) {
        console.error('保存失败:', err.code, err.message);
        return;
    }
    console.log('保存成功:', result);
});
```

**相关证据**: `distributed_data_object.js:74-80` (save 方法)

---

### Q10: 资产绑定失败

**错误码**: 15400002 或 15400003

**排查步骤**:
```javascript
// 1. 检查是否已设置 sessionId（不能设置后再绑定资产）
if (obj['__sessionId'] != null && obj['__sessionId'] !== '') {
    throw {
        code: 15400003,
        message: 'SessionId has been set, and asset cannot be set.'
    };
}

// 2. 检查 URI 是否有效
const uri = 'file:///data/avatar.jpg';
try {
    await obj.setAsset('avatar', uri);
} catch (e) {
    console.error('资产设置失败:', e.code, e.message);
}

// 3. 检查 URI 格式
if (!uri.startsWith('file://')) {
    throw {
        code: 15400002,
        message: 'The asset uri must start with file://'
    };
}
```

**相关证据**: `distributed_data_object.js:502-523` (setAsset 方法)

---

## 调试问题

### Q11: 如何查看日志

**日志标签**: `ObjectStore`

**日志级别**: `HILOG_ENABLE` 宏控制

**查看日志**:
```bash
# 使用 hilog 查看
hilog | grep ObjectStore

# 或使用 pidof 获取进程 PID 后过滤
hilog | grep <pid>
```

**代码中的日志**:
```cpp
// LOG_INFO - 信息日志
LOG_INFO("create object success, sessionId: %{public}s", sessionId.c_str());

// LOG_ERROR - 错误日志
LOG_ERROR("create object failed, status: %{public}d", status);

// LOG_WARN - 警告日志
LOG_WARN("permission denied, bundleName: %{public}s", bundleName.c_str());
```

---

### Q12: 如何调试 N-API 层

**方法 1: 使用 gdb**

```bash
# 启动调试
gdb -p <app_pid>

# 设置断点
break js_distributedobject.cpp:149

# 继续执行
continue

# 查看变量
print version
print sessionId

# 单步调试
next / step
```

**方法 2: 使用 lldb (macOS)**

```bash
# 启动调试
lldb -p <app_pid>

# 设置断点
breakpoint set --name JSCreateObjectSync

# 运行
process continue
```

---

### Q13: 如何追踪数据流

**方法 1: 使用 HiTrace**

```cpp
#include "hitrace.h"

// 开始追踪
DataObjectHiTrace trace("DistributedObject::PutString");

// 追踪特定操作
HiTraceId = HitraceBegin(TRACE_OBJECT, HITRACE_FLAG_DEFAULT);
// ... 执行操作 ...
HitraceEnd(HiTraceId);
```

**方法 2: 使用日志追踪**

```bash
# 启用详细日志
hilog -D all | grep -E "(ObjectStore|DistributedObject|Session)"

# 或保存日志到文件
hilog > objectstore.log 2>&1 &
```

---

### Q14: IPC 调用追踪

**方法 1: Binder 调试**

```bash
# 查看 Binder 调用
binderfs /dev/binderfs/
cat /sys/kernel/debug/binder/transactions
```

**方法 2: 使用 ipc_debug**

```bash
# 启用 IPC 调试
echo 1 > /sys/module/ipc_core/parameters/debug
dmesg | grep binder
```

---

### Q15: 性能问题排查

**方法 1: 使用 systrace**

```bash
# 录制 systrace
python3 $OHOS_SDK_PATH/prebuilts/python3/linux-x64/3.8.5/bin/systrace.py \
    --time=10 \
    -o objectstore_trace.html \
    sched freq idle am wm gfx view binder_driver
```

**方法 2: 使用 perf**

```bash
# 性能采样
perf record -g -p <pid> -- sleep 10

# 生成报告
perf report
```

**关注指标**:
- CPU 使用率
- 内存占用
- I/O 操作
- 同步延迟

---

## 定位路径速查

| 问题类型 | 日志标签 | 追踪工具 |
|----------|----------|----------|
| N-API 调用 | `ObjectStore` | gdb/lldb |
| IPC 通信 | `Binder` | binder_debug |
| 数据同步 | `DistributedDB` | dsoftbus_log |
| 权限问题 | `AccessToken` | accesstoken_log |
| 性能问题 | `sched` | systrace/perf |

---

## 相关资源

| 资源 | 链接 |
|------|------|
| 日志系统 | HiLog 开发指南 |
| 调试工具 | GDB/LLDB 使用指南 |
| 性能分析 | systrace/perf |
| IPC 调试 | Binder 调试指南 |

---

## 相关章节

- [概览](./00_Overview.md) → 项目约束与限制
- [架构](./01_Architecture.md) → 组件交互与数据流
- [N-API 接口](./02_N-API.md) → API 使用示例
- [构建配置](./04_Build.md) → 构建命令与产物
- [安全评审](./05_Security.md) → 权限与安全考量
