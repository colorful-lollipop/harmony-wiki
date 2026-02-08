# 常见问题与故障排查

> **目的**: 常见构建、运行和调试问题的定位路径
> **适用范围**: 所有人,特别是遇到问题的人
> **阅读时间**: 20分钟

---

## 构建相关问题

### 问题 1: 编译失败 - 找不到头文件

**症状**:
```
fatal error: 'power_mgr.h' file not found
```

**原因**:
- include_dirs 配置错误
- 系统类型检测失败

**定位路径**:
1. 检查 `powermgr.gni`:
   ```bash
   grep system_type powermgr.gni
   ```
2. 检查 `config.gni`:
   ```bash
   cat config.gni
   ```
3. 验证 `ohos_kernel_type`:
   ```bash
   # 查看 OHOS 构建配置
   find out -name args.gn -exec cat {} \;
   grep ohos_kernel_type
   ```

**解决方案**:
```bash
# 明确指定内核类型
hb build -f powermgr/powermgr_lite --ohos_kernel_type=liteos_a

# 或
hb build -f powermgr/powermgr_lite --ohos_kernel_type=liteos_m
```

---

### 问题 2: 编译失败 - 静态库问题

**症状** (Mini 系统):
```
error: undefined reference to 'CreateRunningLock'
```

**原因**:
- 静态库链接顺序错误
- 缺少依赖库

**定位路径**:
1. 检查 `frameworks/BUILD.gn` deps 配置
2. 检查应用是否链接 `libpowermgr.a`

**解决方案**:
```bash
# 检查应用构建配置
# 确保链接顺序正确:
# 1. libpowermgr_utils.a
# 2. libpowermgr.a
# 3. libpowermgrservice.a
```

---

### 问题 3: 编译失败 - 屏保功能未编译

**症状** (Small 系统 + `enable_screensaver=true`):
```
error: undefined reference to 'ScreenSaverMgr'
```

**原因**:
- 屏保依赖未配置
- AMS 服务未找到

**定位路径**:
1. 检查 `config.gni`:
   ```bash
   grep enable_screensaver config.gni
   ```
2. 检查屏保 target 是否编译:
   ```bash
   find out -name "*screensaver*" -type f
   ```

**解决方案**:
```bash
# 确保启用屏保
hb build -f powermgr/powermgr_lite --enable-screensaver=true

# 检查 AMS 服务是否可用
hdc shell "sa_list | grep AbilityManagerService"
```

---

## 运行时问题

### 问题 1: 服务未启动

**症状**:
```
Failed to get feature api: powermgr:powermanage
```

**原因**:
- powermgrservice 进程未运行
- 服务注册失败

**定位路径**:
1. 检查服务进程:
   ```bash
   hdc shell ps -A | grep powermgr
   ```
2. 检查服务注册 (hilog):
   ```bash
   hdc shell hilog -x | grep "powermgr"
   ```

**预期日志**:
```
[powermgr] Succeed to init power manager service
```

**解决方案**:
1. 重启设备
2. 检查服务配置:
   ```bash
   hdc shell cat /etc/param/ohos.para.d/
   ```
3. 手动启动服务 (如果支持):
   ```bash
   hdc shell "start_service powermgrservice"
   ```

---

### 问题 2: RunningLock 获取超时

**症状**:
```
AcquireRunningLock() 返回 FALSE
```

**原因**:
- 平台操作超时
- 系统资源耗尽
- 其他进程持有锁

**定位路径**:
1. 检查日志:
   ```bash
   hdc shell hilog -x | grep "running"
   ```
2. 检查内核日志:
   ```bash
   # Small 系统
   hdc shell dmesg | grep -i power

   # Mini 系统
   hdc shell cat /proc/kmsg | grep -i power
   ```
3. 检查锁状态:
   ```bash
   # Small 系统
   hdc shell cat /proc/power/lock_status

   # Mini 系统
   # 需要特定内核接口
   ```

**解决方案**:
1. 重试获取锁:
   ```c
   int retry = 3;
   while (retry-- > 0) {
       if (AcquireRunningLock(lock)) {
           break;
       }
       usleep(100000);  // 等待 100ms
   }
   ```
2. 检查是否有其他锁:
   ```c
   if (IsAnyRunningLockHolding()) {
       printf("Other lock is holding\n");
   }
   ```
3. 增加超时时间 (如平台支持)

---

### 问题 3: 设备无法挂起

**症状**:
```
SuspendDevice() 调用成功,但设备不挂起
```

**原因**:
- 有运行锁被持有
- WakeupHolder 未释放
- 平台挂起功能未工作

**定位路径**:
1. 检查锁状态:
   ```bash
   hdc shell hilog -x | grep "running lock"
   ```
2. 检查阻塞计数器:
   ```bash
   hdc shell hilog -x | grep "suspend block"
   ```
3. 检查内核挂起状态:
   ```bash
   # Small 系统
   hdc shell cat /sys/power/state

   # Mini 系统
   hdc shell cat /sys/power/wakeup_count
   ```

**解决方案**:
1. 释放所有运行锁:
   ```c
   ReleaseRunningLock(lock);
   ```
2. 检查 SuspendController:
   ```bash
   hdc shell hilog -x | grep "suspend controller"
   ```
3. 手动触发挂起 (用于测试):
   ```bash
   # Small 系统
   hdc shell "echo mem > /sys/power/state"

   # Mini 系统
   # 需要特定命令
   ```

---

### 问题 4: 设备无法唤醒

**症状**:
```
WakeupDevice() 调用成功,但设备保持挂起状态
```

**原因**:
- 平台唤醒机制未工作
- 唤醒信号未发送

**定位路径**:
1. 检查唤醒日志:
   ```bash
   hdc shell hilog -x | grep "wakeup"
   ```
2. 检查内核唤醒计数:
   ```bash
   hdc shell cat /sys/power/wakeup_count
   ```
3. 检查唤醒原因:
   ```bash
   hdc shell hilog -x | grep "wakeup reason"
   ```

**解决方案**:
1. 验证唤醒原因是否正确
2. 检查硬件唤醒事件
3. 测试其他唤醒方式:
   ```c
   WakeupDevice(WAKEUP_DEVICE_POWER_BUTTON, "manual wakeup");
   ```

---

## JS API 相关问题

### 问题 1: battery.getStatus 返回 undefined

**症状**:
```javascript
battery.getStatus({
    success: function(data) {
        console.log(data);  // undefined!
    }
});
```

**原因**:
- `GetBatteryStatus()` 实现未正确返回
- 电池模块未初始化

**定位路径**:
1. 检查 JS 模块加载:
   ```bash
   hdc shell hilog -x | grep "battery"
   ```
2. 检查电池服务:
   ```bash
   hdc shell "sa_list | grep battery"
   ```
3. 检查电池接口实现:
   ```bash
   # GetBatteryStatus() 声明在 battery_impl.h
   # 但实现在外部仓库 (powermgr_battery_lite)
   ```

**解决方案**:
1. 确保电池服务运行:
   ```bash
   hdc shell "start_service battery_service"
   ```
2. 检查电池模块初始化:
   ```bash
   hdc shell hilog -x | grep "battery init"
   ```

---

## 安全相关问题

### 问题 1: 权限错误 (预期)

**症状** (如果已实现权限检查):
```
Failed to suspend device: Permission denied
```

**原因**:
- 应用未声明权限
- 权限级别不足

**定位路径**:
1. 检查应用 manifest:
   ```bash
   hdc shell cat /data/app/com.example.app/config.json
   ```
2. 检查权限列表:
   ```bash
   grep "reqPermissions" config.json
   ```

**解决方案**:
1. 在 manifest 中添加权限:
   ```json
   {
     "reqPermissions": [
       "ohos.permission.POWER_MANAGE"
     ]
   }
   ```
2. 重新安装应用
3. 验证权限授予

---

## 调试技巧

### 启用详细日志

```bash
# 启用 powermgr 详细日志
hdc shell "hdc shell param set persist.powermgr.log.level 3"

# 查看日志
hdc shell hilog -x | grep powermgr

# 保存日志到文件
hdc shell hilog -x > powermgr.log
```

### 跟踪 API 调用

```c
// 在应用中添加日志
#include "hilog_wrapper.h"

const RunningLock *lock = CreateRunningLock("test", RUNNINGLOCK_SCREEN, 0);
POWER_HILOGI("Created lock: %p", lock);

BOOL ret = AcquireRunningLock(lock);
POWER_HILOGI("Acquire result: %d", ret);
```

### 使用调试工具

```bash
# 连接设备调试器
hdc file -f local_path remote_path

# 监控日志流
hdc hilog -x --tail

# 检查进程状态
hdc shell "ps -A | grep powermgr"
```

---

## 相关文档

- [04_NAPI_JS_API.md](04_NAPI_JS_API.md#error-handling) - API 错误处理
- [06_GN_Targets.md](06_GN_Targets.md) - 构建配置
- [08_Security_Assessment.md](08_Security_Assessment.md) - 安全风险
- [07_Build_Artifacts.md](07_Build_Artifacts.md#故障排查) - 运行时验证
