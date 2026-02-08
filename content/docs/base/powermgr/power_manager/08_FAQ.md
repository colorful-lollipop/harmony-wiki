# 常见问题 (FAQ)

本文档收集 Power Manager 模块的常见问题及解决方案。

---

## 1. 构建问题

### Q1: 编译报错 "undefined reference to PowerMgrClient"

**问题**:
```
error: undefined reference to 'OHOS::PowerMgr::PowerMgrClient::GetInstance()'
```

**原因**: 缺少依赖库

**解决方案**:
```bash
# 添加依赖
ninja -C out //base/powermgr/power_manager/interfaces/inner_api:powermgr_client
```

**相关文件**:
- `interfaces/inner_api/BUILD.gn`

---

### Q2: N-API 模块编译失败

**问题**:
```
error: 'napi.h' file not found
```

**原因**: 缺少 napi 组件

**解决方案**:
```bash
# 在 bundle.json 中添加组件依赖
# 或者使用 hb set 选择包含 napi 的产品
```

---

### Q3: GN 配置不生效

**问题**:
```
gn args out/
# 修改 powermgr.gni 后重新生成
```

**原因**: GN 配置缓存

**解决方案**:
```bash
# 清理后重新生成
rm -rf out/
gn gen out/
ninja -C out
```

---

## 2. 运行问题

### Q4: PowerMgrService 启动失败

**问题**:
```
hdc shell sa_ps | grep powermgr
# 无输出，服务未运行
```

**排查步骤**:

1. 查看日志:
```bash
hdc shell hilog | grep -E "PowerMgr|ERROR"
```

2. 检查 SA 配置:
```bash
hdc shell cat /system/profile/3301.json
```

3. 检查依赖服务:
```bash
hdc shell sa_ps | grep -E "samgr|safwk"
```

**常见原因**:
- SA 配置文件缺失
- 依赖服务未启动
- 权限不足

---

### Q5: API 调用返回错误码 4603001

**问题**:
```typescript
power.shutdown('user').catch((err) => {
    console.error(err.code); // 4603001
});
```

**错误说明**: IPC 通信失败

**排查步骤**:

1. 检查服务状态:
```bash
hdc shell sa_ps | grep powermgr
```

2. 检查 IPC 连接:
```bash
hdc shell dumpsys power
```

3. 常见原因:
- 服务未启动
- IPC 通信中断
- 权限不足

---

### Q6: RunningLock 无法保持

**问题**:
```typescript
const lock = await runningLock.create('test', RunningLockType.BACKGROUND);
lock.hold(0);
// 设备仍然休眠
```

**原因分析**:

1. 锁类型不支持:
```typescript
const supported = runningLock.isSupported(RunningLockType.BACKGROUND);
console.log(supported); // false
```

2. 锁已释放:
```typescript
lock.unhold();  // 忘记调用
```

3. 超时设置:
```typescript
lock.hold(30000);  // 30秒后自动释放
```

**解决方案**:

1. 检查设备支持:
```bash
hdc shell power-shell runninglock -d
```

2. 检查锁状态:
```bash
hdc shell dumpsys power | grep RunningLock
```

---

## 3. 调试技巧

### Q7: 如何调试 Power Manager

**方法 1: 查看日志**

```bash
# 过滤 Power 相关日志
hdc shell hilog | grep -E "PowerMgr|COMP_FWK"

# 详细日志
hdc shell hilog -D on
```

**方法 2: dumpsys**

```bash
# 查看完整状态
hdc shell dumpsys power

# 查看 RunningLock
hdc shell dumpsys power | grep -A 20 "RunningLock"

# 查看电源状态
hdc shell dumpsys power | grep -A 5 "PowerState"
```

**方法 3: Shell 调试**

```bash
# 进入 shell
hdc shell power-shell

# 查看帮助
power-shell help

# 查看状态
power-shell state

# 查看 RunningLock
power-shell runninglock -d
```

---

### Q8: 如何添加自定义日志

**C++ 代码**:

```cpp
#include "power_log.h"

POWER_HILOGD(COMP_FWK, "Debug message");
POWER_HILOGI(COMP_FWK, "Info message");
POWER_HILOGW(COMP_FWK, "Warning message");
POWER_HILOGE(COMP_FWK, "Error message");
```

**日志级别**:
- `D`: Debug
- `I`: Info
- `W`: Warning
- `E`: Error

**组件标签**:
- `COMP_FWK`: 框架
- `FEATURE_RUNNING_LOCK`: 运行锁
- `FEATURE_SHUTDOWN`: 关机
- `FEATURE_POWER`: 电源

---

## 4. 常见错误码

### API 错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 401 | `ERR_PARAM_INVALID` | 参数无效 |
| 4603001 | `ERR_IPC` | IPC 通信失败 |
| 4603002 | `ERR_SERVICE_NOT_READY` | 服务未就绪 |

**代码位置**: `interfaces/inner_api/native/include/power_errors.h`

### 状态机错误码

| 值 | 说明 |
|----|------|
| 0 | 成功 |
| -1 | 失败 |
| 1 | 状态错误 |

---

## 5. 性能问题

### Q9: API 响应慢

**排查方法**:

1. 检查是否是 IPC 调用:
```bash
# 计时 API 调用
hdc shell time dumpsys power
```

2. 检查线程阻塞:
```bash
# 查看线程堆栈
hdc shell "cat /proc/$(pgrep PowerMgrService)/stack"
```

3. 常见原因:
- HDI 调用阻塞
- 状态机锁竞争
- 回调处理超时

---

### Q10: 内存占用高

**排查方法**:

1. 查看内存使用:
```bash
hdc shell "cat /proc/$(pgrep PowerMgrService)/status | grep VmRSS"
```

2. 检查 RunningLock 泄漏:
```bash
hdc shell dumpsys power | grep "RunningLock count"
```

3. 常见原因:
- 回调未注销
- RunningLock 未释放
- 日志缓冲区过大

---

## 6. 功能问题

### Q11: 设备无法休眠

**排查步骤**:

1. 检查运行锁:
```bash
hdc shell dumpsys power | grep -A 10 "RunningLock"
```

2. 检查 WakeLock:
```bash
hdc shell dumpsys power | grep -i "wake"
```

3. 常见原因:
- 应用持有 RunningLock
- 屏幕保持唤醒
- 电源模式为省电模式

---

### Q12: 电源模式设置不生效

**排查步骤**:

1. 检查当前模式:
```typescript
const mode = power.getPowerMode();
console.log('当前模式:', mode);
```

2. 检查权限:
```bash
# 需要 ohos.permission.POWER_MANAGER 权限
```

3. 检查设备支持:
```bash
hdc shell dumpsys power | grep -i "mode"
```

---

## 7. 移植问题

### Q13: 如何添加新的电源状态

**步骤**:

1. 在 `power_state_machine.cpp` 添加状态:
```cpp
enum class PowerState : uint32_t {
    // ... 现有状态
    NEW_STATE,  // 新增
};
```

2. 在状态机添加处理:
```cpp
void PowerStateMachine::HandleEvent(PowerStateEvent event)
{
    switch (event) {
        case PowerStateEvent::NEW_EVENT:
            TransitionTo(PowerState::NEW_STATE);
            break;
    }
}
```

3. 更新 API 文档

---

### Q14: 如何添加新的运行锁类型

**步骤**:

1. 在 `running_lock.h` 添加类型:
```cpp
enum class RunningLockType : uint32_t {
    // ... 现有类型
    NEW_TYPE,  // 新增
};
```

2. 在 `running_lock_mgr.cpp` 实现:
```cpp
bool RunningLockMgr::IsSupported(RunningLockType type)
{
    switch (type) {
        case RunningLockType::NEW_TYPE:
            return CheckHardwareSupport(NEW_TYPE);
    }
}
```

3. 添加 FFRT 任务支持

---

## 8. 相关资源

### 文档链接

- [OpenHarmony Power Management](https://gitee.com/openharmony/docs)
- [N-API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-power.md)
- [系统能力文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-syscap.md)

### 工具链接

- [hdc 工具](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/tools/hdc-guide.md)
- [HiTool](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/debug/hitrace.md)

---

## 9. 反馈与支持

### 问题反馈

如遇到本文档未覆盖的问题，请:

1. 查看日志: `hdc shell hilog | grep PowerMgr`
2. 检查状态: `hdc shell dumpsys power`
3. 提交 Issue 到: https://gitee.com/openharmony/powermgr_power_manager/issues
