# 问题排查

> 常见问题、日志分析、调试命令

---

## 常见问题

### Q1: 设置时间失败，返回权限错误

**现象**: `setTime` 返回 `E_TIME_NO_PERMISSION`

**原因**: 
1. 应用未声明 `ohos.permission.SET_TIME` 权限
2. 应用不是系统应用（API9+ 需要 `CheckSystemUidCallingPermission`）

**解决**:
```json
// module.json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.SET_TIME"
      }
    ]
  }
}
```

**验证**:
```cpp
// 服务端日志
TIME_HILOGE(TIME_MODULE_SERVICE, "permission check setTime failed");
```

---

### Q2: 定时器不触发

**检查清单**:

1. **是否正确启动？**
   ```javascript
   // 必须先 create，再 start
   const timerId = await systemTimer.createTimer(options);
   await systemTimer.startTimer(timerId, triggerTime);
   ```

2. **triggerTime 是否已过？**
   - `triggerTime` 必须是未来的时间戳
   - 使用 `systemTime.getCurrentTime()` 获取当前时间

3. **应用是否被冻结？**
   - 检查 `TimerProxy` 日志
   - 后台应用定时器可能被延迟

4. **系统是否休眠？**
   - 非 `TIMER_TYPE_WAKEUP` 定时器休眠不触发

---

### Q3: 时区设置不生效

**验证步骤**:

```cpp
// 1. 检查设置是否成功
int32_t result = TimeSystemAbility::SetTimeZone(timeZoneId, apiVersion);

// 2. 验证时区文件是否存在
std::string tzFile = "/data/misc/zoneinfo/" + timeZoneId;
// 检查文件是否存在

// 3. 检查持久化
// 时区信息存储在 /data/service/el1/public/time/time_zone_config.json
```

---

## 日志分析

### 日志标签

| 模块 | 标签 | 说明 |
|------|------|------|
| N-API | `TIME_MODULE_JS_NAPI` | JS 接口层 |
| Client | `TIME_MODULE_CLIENT` | 客户端 |
| Service | `TIME_MODULE_SERVICE` | 服务端 |
| Common | `TIME_MODULE_COMMON` | 通用工具 |
| Timer | `TIME_MODULE_TIMER` | 定时器核心 |

### 关键日志

#### 权限检查失败
```
TIME_MODULE_SERVICE: permission check setTime failed
TIME_MODULE_SERVICE: not system applications
```

#### 定时器操作
```
TIME_MODULE_SERVICE: CreateTimer uid:xxx, pid:xxx
TIME_MODULE_TIMER: StartTimer timerId:xxx, triggerTime:xxx
TIME_MODULE_TIMER: Timer triggered id:xxx
```

#### IPC 连接
```
TIME_MODULE_CLIENT: GetProxy success
TIME_MODULE_CLIENT: OnRemoteDied, recipient death
```

### 日志级别

```cpp
TIME_HILOGD(TAG, "debug message");   // Debug
TIME_HILOGI(TAG, "info message");    // Info
TIME_HILOGW(TAG, "warning message"); // Warning
TIME_HILOGE(TAG, "error message");   // Error
```

---

## 调试命令

### HIDumper 命令

**条件**: 编译时启用 `time_service_hidumper_able=true`

```bash
# 查看时间信息
hidumper -s 3702 -a "-time"

# 查看所有定时器
hidumper -s 3702 -a "-timer -a"

# 查看特定定时器
hidumper -s 3702 -a "-timer -i 12345"

# 查看定时器触发统计
hidumper -s 3702 -a "-timer -s 12345"

# 查看空闲定时器
hidumper -s 3702 -a "-idle -a"

# 查看代理定时器
hidumper -s 3702 -a "-ProxyTimer -l"

# 查看 UID 定时器映射
hidumper -s 3702 -a "-UidTimer -l"
```

### 系统服务状态

```bash
# 查看 TimeService 是否运行
ps -A | grep timeservice

# 查看 SA 注册状态
ls /system/profile/ | grep 3702

# 查看配置文件
cat /system/etc/init/timeservice.cfg
```

### 数据库检查

```bash
# 查看定时器数据库（RDB 启用时）
ls /data/service/el1/public/database/time/

# 查看时区配置
cat /data/service/el1/public/time/time_zone_config.json
```

---

## 调试技巧

### 1. 开启 Debug 日志

修改 `time.gni`:
```gn
time_service_debug_able = true
```

### 2. 使用 HiCollie 检测卡死

TimeService 已集成 HiCollie，当接口响应超过阈值时会打印告警：
```
TimeXCollie: TimeService::SetTime timeout
```

### 3. 手动触发 NTP 同步

```cpp
// 调用 SetAutoTime(true) 触发 NTP 同步
TimeServiceClient::GetInstance()->SetAutoTime(true);
```

### 4. 检查 TimerManager 状态

```cpp
// 在 HIDumper 命令中添加自定义检查
TimerManager::GetInstance()->ShowTimerEntryMap(fd);
```

---

## 错误码参考

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `E_TIME_OK` | 0 | 成功 |
| `E_TIME_DEAL_FAILED` | 1 | 处理失败 |
| `E_TIME_NULLPTR` | 2 | 空指针 |
| `E_TIME_NO_PERMISSION` | 3 | 无权限 |
| `E_TIME_NOT_SYSTEM_APP` | 4 | 非系统应用 |
| `E_TIME_PARAMETERS_INVALID` | 5 | 参数无效 |
| `E_TIME_SET_RTC_FAILED` | 6 | 设置 RTC 失败 |
| `E_TIME_PUBLISH_FAIL` | 7 | 发布 SA 失败 |
| `E_TIME_AUTO_RESTORE_ERROR` | 8 | 自动恢复错误 |
| `E_TIME_READ_PARCEL_ERROR` | 100 | 读取 Parcel 错误 |
| `E_TIME_NTP_UPDATE_FAILED` | 200 | NTP 更新失败 |
| `E_TIME_NTP_NOT_UPDATE` | 201 | NTP 未更新 |

---

## 相关链接

- [架构说明](./02_Architecture.md) - 组件关系
- [安全分析](./06_Security_Analysis.md) - 权限说明
- [N-API 参考](./03_NAPI_Reference.md) - 接口详情
