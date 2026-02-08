# 常见问题与故障排查

## 构建问题

### Q1: 编译报错 "bgtaskmgr_innerkits not found"

**现象**:
```
ninja: error: 'out/.../libbgtaskmgr_innerkits.so', needed by '...', missing and no known rule to make it
```

**原因**: 依赖顺序错误或innerkits未先编译

**解决**:
```bash
# 1. 先编译innerkits
ninja -C out background_task_mgr/interfaces/innerkits:bgtaskmgr_innerkits

# 2. 再编译完整模块
ninja -C out background_task_mgr:service_group_background_task_mgr_all
```

**证据**: `interfaces/BUILD.gn:17-22`
```gn
group("bgtaskmgr_interfaces") {
  deps = [
    "innerkits:bgtaskmgr_innerkits",
    # ...
  ]
}
```

---

### Q2: IDL生成代码编译失败

**现象**:
```
error: 'BackgroundTaskMgrStub' was not declared in this scope
```

**原因**: IDL文件修改后未重新生成代码

**解决**:
```bash
# 清理IDL生成代码
rm -rf out/.../gen/background_task_mgr/interfaces/innerkits/

# 重新生成并编译
ninja -C out background_task_mgr/interfaces/innerkits:background_task_mgr_interface
```

**证据**: `interfaces/innerkits/BUILD.gn:25-35`
```gn
idl_gen_interface("background_task_mgr_interface") {
  src_idl = rebase_path("IBackgroundTaskMgr.idl")
  # 自动生成BackgroundTaskMgrStub/Proxy
}
```

---

### Q3: N-API模块加载失败

**现象**:
```javascript
Error: module 'backgroundTaskManager' not found
```

**原因**: N-API库未正确安装到/system/lib/module/

**排查步骤**:
```bash
# 1. 检查库是否存在
ls /system/lib/module/resourceschedule/libbackgroundtaskmanager_napi.z.so

# 2. 检查文件权限
ls -lZ /system/lib/module/resourceschedule/libbackgroundtaskmanager_napi.z.so

# 3. 检查SELinux标签
getenforce
# 应为Permissive或正确配置策略
```

**证据**: `interfaces/kits/BUILD.gn:85-98`
```gn
ohos_shared_library("backgroundtaskmanager_napi") {
  relative_install_dir = "module/resourceschedule"
  # ...
}
```

---

## 运行时问题

### Q4: SA 1903 启动失败

**现象**: 后台任务服务无法启动，日志显示:
```
BackgroundTaskMgrService: OnStart failed, dependency not ready
```

**原因**: 依赖的System Ability未就绪

**依赖检查**:
```cpp
// services/core/src/background_task_mgr_service.cpp:180-220
void BackgroundTaskMgrService::OnAddSystemAbility(
    int32_t systemAbilityId, const std::string& deviceId) {
    switch (systemAbilityId) {
        case APP_MGR_SERVICE_ID:  // 501
            SetReady(TRANSIENT_SERVICE_READY);
            break;
        case BUNDLE_MGR_SERVICE_SYS_ABILITY_ID:  // 401
            SetReady(CONTINUOUS_SERVICE_READY);
            break;
        // ...
    }
}
```

**排查步骤**:
```bash
# 检查依赖SA状态
hdc shell "sactl -l | grep -E '501|401'"

# 查看详细日志
hdc shell "hilog | grep -i background"
```

**解决**: 确保依赖SA正常启动

---

### Q5: 短时任务申请失败 (ERR_BGTASK_EXCEEDS_THRESHOLD)

**现象**:
```javascript
Error: ERR_BGTASK_EXCEEDS_THRESHOLD
```

**原因**: 应用当日配额已耗尽

**配额计算**:
```cpp
// services/transient_task/src/decision_maker.cpp:77-89
bool DecisionMaker::IsExceedMaxBgTaskDurationPerDay(int32_t uid) {
    int32_t maxDurationPerDay = GetQuota();  // 默认10分钟
    int32_t duration = GetTodayDuration(uid);
    return duration >= maxDurationPerDay;
}
```

**排查步骤**:
```bash
# 查看应用配额使用情况
hdc shell "hidumper -s 1903 -a '-h'"
# 查找Transient Task部分
```

**解决**:
1. 等待次日配额重置
2. 检查是否有未取消的任务占用配额
3. 联系系统管理员增加配额（需配置）

---

### Q6: 长时任务通知不显示

**现象**: 调用startBackgroundRunning成功，但通知栏没有提示

**排查步骤**:
```bash
# 1. 检查通知权限
hdc shell "notification -l"

# 2. 查看服务日志
hdc shell "hilog | grep -i 'notification\|continuous'"

# 3. 检查通知服务状态
hdc shell "sactl -l | grep notification"
```

**常见原因**:
1. `distributed_notification_service` SA未启动
2. WantAgent配置错误
3. 通知权限被拒绝

**代码检查点**: `services/continuous_task/src/bg_continuous_task_mgr.cpp:687-730`
```cpp
ErrCode BgContinuousTaskMgr::SendContinuousTaskNotification(...) {
    // 检查通知服务可用性
    if (!distributed_notification_enable) {
        return ERR_BGTASK_NOTIFICATION_ERR;
    }
    // ...
}
```

---

### Q7: IPC调用超时 (ERR_BGTASK_TRANSACT_FAILED)

**现象**:
```cpp
ErrCode: 980000301  // ERR_BGTASK_TRANSACT_FAILED
```

**原因**: 服务端处理超时或死锁

**排查步骤**:
```bash
# 1. 检查服务进程状态
hdc shell "ps -ef | grep resource_schedule"

# 2. 查看线程状态
hdc shell "cat /proc/$(pidof resource_schedule_service)/stack"

# 3. 检查服务是否ANR
hdc shell "ls -l /data/anr/"
```

**常见原因**:
1. 服务死锁（EventHandler单线程阻塞）
2. 依赖服务响应慢
3. 系统负载过高

**解决**: 重启服务
```bash
hdc shell "sactl -r 1903"  # 重启BackgroundTaskMgrService
```

---

## 调试方法

### Dump调试信息

**命令**:
```bash
# 查看所有后台任务信息
hdc shell "hidumper -s 1903"

# 查看帮助
hdc shell "hidumper -s 1903 -a '-h'"

# 清理任务
hdc shell "hidumper -s 1903 -a '-c'"
```

**输出示例**:
```
-------------------------------[usage]-------------------------------

-h: help
-t: dump all transient task
-c: dump all continuous task
-a: dump all info
-e: dump efficiency resources
-r: dump reset efficiency resources, {app all} {proc all} or {app bundleName} {proc bundleName}
-s: dump stop continuous task {bundleName} {abilityName}
```

**实现代码**: `services/core/src/background_task_mgr_service.cpp:118-178`

---

### 日志级别设置

**动态调整日志级别**:
```bash
# 设置DEBUG级别
hdc shell "hilog -b D -T background_task_mgr"

# 恢复INFO级别
hdc shell "hilog -b I -T background_task_mgr"
```

**日志标签**:
- `background_task_mgr` - 主服务日志
- `BgTransientTaskMgr` - 短时任务管理器
- `BgContinuousTaskMgr` - 长时任务管理器
- `BgEfficiencyResourcesMgr` - 能效资源管理器

---

### GDB调试

**附加到服务进程**:
```bash
# 1. 获取进程PID
hdc shell "pidof resource_schedule_service"

# 2. 进入容器调试
hdc shell "gdbserver :1234 --attach <PID>"

# 3. 本地连接
gdb out/soong/.../libbgtaskmgr_service.so
(gdb) target remote <device_ip>:1234
```

**关键断点**:
```gdb
# 短时任务入口
break BgTransientTaskMgr::RequestSuspendDelay

# 长时任务入口
break BgContinuousTaskMgr::StartBackgroundRunning

# 权限检查
break BackgroundTaskMgrService::CheckHapCalling
```

---

## 配置问题

### Q8: 如何修改短时任务配额

**配置位置**: `services/common/src/bgtask_config.cpp`

**方法1: 代码修改**:
```cpp
// bgtask_config.cpp
int32_t BgtaskConfig::GetMaxBgTaskDurationPerDay() {
    return 10 * 60 * 1000;  // 默认10分钟，修改为需要值
}
```

**方法2: 运行时配置** (如支持):
```cpp
// 通过Dump接口设置
hdc shell "hidumper -s 1903 -a 'config -t 20'"
```

**证据**: `services/transient_task/include/key_info.h`
```cpp
struct KeyInfo {
    int32_t delayTime_ {DEFAULT_DELAY_TIME};  // 默认3分钟
};
```

---

### Q9: 如何添加CPU级别白名单

**配置位置**: `services/common/src/bg_task_config_file_info.cpp`

**格式**:
```json
{
    "allowApplyCpuBundleInfoList": [
        {
            "bundleName": "com.example.app",
            "appId": "com.example.app_BQ1bKlbLSQGvfN7...",
            "maxCpuLevel": 3
        }
    ]
}
```

**加载代码**: `services/common/src/bgtask_config.cpp:290-314`
```cpp
bool BgtaskConfig::ParseBundleSignature(jsonObj) {
    // 解析签名和CPU级别配置
    for (const auto& item : jsonObj.items()) {
        std::string bundleName = item.key();
        int32_t maxCpuLevel = item.value()["maxCpuLevel"];
        allowApplyCpuBundleInfoMap_[bundleName] = {signature, maxCpuLevel};
    }
}
```

---

## 性能问题

### Q10: 服务CPU占用高

**排查**:
```bash
# 1. 查看CPU占用
hdc shell "top -p $(pidof resource_schedule_service)"

# 2. 查看线程占用
hdc shell "ps -T -p $(pidof resource_schedule_service)"
```

**常见原因**:
1. 大量短时任务频繁申请/取消
2. 通知更新过于频繁
3. 数据库操作阻塞

**优化建议**:
```cpp
// 1. 增加批处理逻辑
// services/transient_task/src/bg_transient_task_mgr.cpp
void BgTransientTaskMgr::BatchProcessRequests() {
    // 批量处理请求，减少EventHandler投递次数
}

// 2. 增加通知更新频率限制
// services/continuous_task/src/bg_continuous_task_mgr.cpp
if (now - lastUpdateTime < MIN_UPDATE_INTERVAL) {
    return;  // 跳过过于频繁的更新
}
```

---

### Q11: 内存泄漏

**排查工具**:
```bash
# 1. 查看内存占用
hdc shell "dumpsys meminfo resource_schedule_service"

# 2. 使用Valgrind（如支持）
hdc shell "valgrind --leak-check=full /system/bin/resource_schedule_service"
```

**常见泄漏点**:
1. `expiredCallbackMap_` 中死亡回调未清理
2. `continuousTaskInfosMap_` 中任务记录残留
3. `subscriberList_` 中订阅者未移除

**检查代码**:
```cpp
// bg_transient_task_mgr.cpp:75
void BgTransientTaskMgr::HandleExpiredCallbackDeath(
    const wptr<IRemoteObject>& remote) {
    // 确保死亡回调被正确移除
    std::lock_guard<std::mutex> lock(expiredCallbackLock_);
    // 清理逻辑...
}
```

---

## 相关文档

- [架构说明](02_Architecture.md) - 组件关系
- [内部API](04_Inner_API.md) - 接口说明
- [安全风险](06_Security.md) - 安全注意事项
