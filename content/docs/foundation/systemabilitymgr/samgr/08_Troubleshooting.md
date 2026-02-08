# 常见问题与排查

## 启动问题

### 问题 1: samgr 启动失败

**现象**:
```
init: Service "samgr" failed to start
```

**排查步骤**:

1. **检查配置文件**:
   ```bash
   cat /system/etc/init/samgr_standard.cfg
   # 确认 path 指向正确的可执行文件
   ```

2. **检查可执行文件存在**:
   ```bash
   ls -la /system/bin/samgr
   file /system/bin/samgr
   ```

3. **检查依赖库**:
   ```bash
   ldd /system/bin/samgr
   # 确认所有依赖库存在
   ```

4. **查看日志**:
   ```bash
   hilog | grep -i samgr
   # 关注 E (Error) 级别日志
   ```

**常见原因**:
- 配置文件 JSON 格式错误
- 依赖库缺失 (libipc_core.so, libffrt.so 等)
- SELinux 拒绝

---

### 问题 2: SA 注册失败

**现象**:
```cpp
int32_t ret = samgr->AddSystemAbility(saId, ability, extraProp);
// ret != ERR_OK
```

**排查步骤**:

1. **检查错误码**:
   ```cpp
   #include "samgr_err_code.h"
   // 对照 SamgrErrCode 枚举
   ```

2. **检查 SA ID 有效性**:
   ```cpp
   // SA ID 范围: 0x00000001 - 0x00ffffff
   if (saId < 0 || saId > 0x00ffffff) {
       // 无效 SA ID
   }
   ```

3. **检查权限**:
   ```bash
   # 查看 SELinux 拒绝日志
   dmesg | grep -i avc | grep samgr
   ```

4. **检查是否达到上限**:
   ```cpp
   // abilityMap_.size() >= MAX_SERVICES
   // MAX_SERVICES 通常为 1000
   ```

**代码定位**:
```cpp
// services/samgr/native/source/system_ability_manager.cpp:1042
int32_t SystemAbilityManager::AddSystemAbility(...) {
    if (!CheckInputSysAbilityId(systemAbilityId) || ability == nullptr) {
        HILOGE("AddSystemAbility input params is invalid.");
        return ERR_INVALID_VALUE;  // 错误码 -1
    }
    // ...
    if (saSize >= MAX_SERVICES) {
        HILOGE("map size error");
        return ERR_INVALID_VALUE;
    }
}
```

---

## 服务发现问题

### 问题 3: GetSystemAbility 返回 nullptr

**现象**:
```cpp
sptr<IRemoteObject> obj = samgr->GetSystemAbility(saId);
// obj == nullptr
```

**排查步骤**:

1. **确认 SA 已注册**:
   ```cpp
   // 使用 CheckSystemAbility 非阻塞检查
   sptr<IRemoteObject> obj = samgr->CheckSystemAbility(saId);
   ```

2. **确认 SA 进程已启动**:
   ```bash
   ps -A | grep {process_name}
   # 从 sa_profiles 中获取 process 字段
   ```

3. **检查是否为 On-Demand SA**:
   ```cpp
   // 如果是 On-Demand，需要先调用 LoadSystemAbility
   samgr->LoadSystemAbility(saId, callback);
   ```

4. **查看 Samgr 日志**:
   ```bash
   hilog | grep "NOT found service"
   # 确认日志中出现目标 SA ID
   ```

**代码定位**:
```cpp
// services/samgr/native/source/system_ability_manager.cpp:496
sptr<IRemoteObject> SystemAbilityManager::GetSystemAbility(int32_t saId) {
    // 重试 7 次，每次间隔 200ms
    for (int i = 0; i < MAX_RETRY; i++) {
        auto ability = CheckSystemAbility(saId);
        if (ability != nullptr) {
            return ability;
        }
        usleep(RETRY_INTERVAL);
    }
    return nullptr;  // 最终返回空
}
```

---

### 问题 4: 分布式服务无法获取

**现象**:
```cpp
sptr<IRemoteObject> obj = samgr->GetSystemAbility(saId, deviceId);
// obj == nullptr，但本地服务正常
```

**排查步骤**:

1. **检查网络连接**:
   ```bash
   # 确认 SoftBus 连接正常
   softbus_tool
   ```

2. **检查 DBinder 服务**:
   ```bash
   # 确认 DBinder 已启动
   ps -A | grep dbinder
   ```

3. **检查分布式权限**:
   ```cpp
   // 调用者需要是 root (0) 或 system (1000)
   // CheckDistributedPermission() 会检查 UID
   ```

4. **查看 DBinder 日志**:
   ```bash
   hilog | grep -i dbinder
   ```

**代码定位**:
```cpp
// services/samgr/native/source/system_ability_manager.cpp:600
sptr<IRemoteObject> SystemAbilityManager::GetSystemAbility(
    int32_t saId, const std::string& deviceId) {
    if (!CheckDistributedPermission()) {
        return nullptr;  // 权限检查失败
    }
    // ... 分布式获取逻辑
}
```

---

## 动态加载问题

### 问题 5: LoadSystemAbility 超时

**现象**:
```cpp
sptr<IRemoteObject> obj = samgr->LoadSystemAbility(saId, 4);  // 4秒超时
// obj == nullptr，或回调收到 OnLoadSystemAbilityFail
```

**排查步骤**:

1. **检查 SA 配置文件**:
   ```bash
   cat /system/profile/{sa_name}.json
   # 确认 ondemand 配置正确
   ```

2. **检查进程启动配置**:
   ```bash
   cat /system/etc/init/{process_name}.cfg
   # 确认 ondemand: true
   ```

3. **查看进程启动日志**:
   ```bash
   hilog | grep {process_name}
   # 查看进程是否正常启动
   ```

4. **检查内存限制**:
   ```bash
   # 确认系统内存充足
   cat /proc/meminfo
   ```

**常见原因**:
- 配置文件格式错误
- 进程启动依赖缺失
- 系统内存不足
- 启动超时（默认 4s，手表 12s）

---

### 问题 6: OnLoadSystemAbilitySuccess 未被调用

**现象**:
```cpp
// 回调未被触发，或触发了 OnLoadSystemAbilityFail
```

**排查步骤**:

1. **检查回调对象生命周期**:
   ```cpp
   // 确保 callback 在加载完成前不被销毁
   // 使用 sptr 保持引用
   ```

2. **检查 SA 注册**:
   ```bash
   hilog | grep "AddSystemAbility"
   # 确认目标 SA 已注册
   ```

3. **检查回调死亡通知**:
   ```bash
   hilog | grep "callback died"
   # 确认回调对象未被意外销毁
   ```

**代码定位**:
```cpp
// services/samgr/native/source/system_ability_manager.cpp:1614
int32_t SystemAbilityManager::LoadSystemAbility(
    int32_t saId, const sptr<ISystemAbilityLoadCallback>& callback) {
    // 将 callback 加入等待队列
    // 当 SA 注册时，调用 NotifySystemAbilityLoaded
}
```

---

## 订阅问题

### 问题 7: 订阅状态变化不生效

**现象**:
```cpp
samgr->SubscribeSystemAbility(saId, listener);
// SA 状态变化时，OnAddSystemAbility 未被调用
```

**排查步骤**:

1. **检查监听器生命周期**:
   ```cpp
   // 确保 listener 在订阅期间有效
   // listener 被销毁会自动取消订阅
   ```

2. **检查监听器实现**:
   ```cpp
   class MyListener : public SystemAbilityStatusChangeStub {
   public:
       void OnAddSystemAbility(int32_t saId, const std::string& deviceId) override {
           // 确认正确实现
       }
   };
   ```

3. **查看订阅日志**:
   ```bash
   hilog | grep "SubscribeSystemAbility"
   # 确认订阅成功
   ```

**代码定位**:
```cpp
// services/samgr/native/source/system_ability_manager.cpp:907
int32_t SystemAbilityManager::SubscribeSystemAbility(
    int32_t saId, const sptr<ISystemAbilityStatusChange>& listener) {
    // 将 listener 加入 listenerMap_
    // 当 SA 添加时，调用 OnAddSystemAbility
}
```

---

## 性能问题

### 问题 8: GetSystemAbility 耗时过长

**现象**:
```cpp
// 调用 GetSystemAbility 需要数百毫秒
```

**排查步骤**:

1. **检查是否为 On-Demand SA**:
   ```cpp
   // On-Demand SA 需要等待进程启动
   // 首次获取会有延迟
   ```

2. **检查系统负载**:
   ```bash
   top
   # 确认 CPU/内存使用正常
   ```

3. **查看重试日志**:
   ```bash
   hilog | grep "Retry get SA"
   # 确认是否在重试
   ```

**优化建议**:
- 使用 `CheckSystemAbility` 非阻塞查询
- 预先调用 `LoadSystemAbility` 启动 On-Demand SA
- 使用 `SubscribeSystemAbility` 监听状态，避免轮询

---

### 问题 9: 内存占用过高

**现象**:
```bash
ps -A | grep samgr
# RSS 内存占用超过 10MB
```

**排查步骤**:

1. **检查 abilityMap_ 大小**:
   ```cpp
   // 确认注册的 SA 数量
   // 每个 SA 持有 IRemoteObject 引用
   ```

2. **检查 listenerMap_ 大小**:
   ```cpp
   // 确认订阅者数量
   // 泄露的订阅可能导致内存增长
   ```

3. **查看内存 dump**:
   ```bash
   # 使用 hisysevent 查看内存统计
   hisysevent -r -n SAMGR_MEMORY
   ```

**内存优化**:
```cpp
// 及时取消订阅
samgr->UnSubscribeSystemAbility(saId, listener);

// 及时释放回调
// callback 在加载完成后自动释放
```

---

## 调试工具

### 1. 使用 Samgr Dumper

```bash
# 查看 Samgr 状态
hidumper -s 0

# 输出示例:
# System Ability Manager Status:
#   Total registered SA: 50
#   Total processes: 10
#   On-demand SA: 20
```

### 2. 查看系统日志

```bash
# 实时查看 Samgr 日志
hilog | grep samgr

# 查看错误日志
hilog -E | grep samgr

# 查看特定 SA 日志
hilog | grep "SA:401"
```

### 3. 使用 hisysevent

```bash
# 查看 Samgr 事件
hisysevent -r -n SAMGR_BEHAVIOR

# 查看 SA 加载事件
hisysevent -r -n SAMGR_LOAD
```

### 4. 代码调试

```cpp
// 开启详细日志
#define LOG_DEBUG 1

// 在关键位置添加日志
HILOGD("Debug: saId=%{public}d, state=%{public}d", saId, state);
```

---

## 错误码速查

| 错误码 | 值 | 含义 | 常见原因 |
|--------|-----|------|----------|
| SAMGR_OK | 0 | 成功 | - |
| INVALID_SYSTEM_ABILITY_ID | 1000 | 无效的 SA ID | SA ID 超出范围 |
| INVALID_INPUT_PARA | 1001 | 输入参数无效 | nullptr 或空字符串 |
| PROFILE_NOT_EXIST | 1002 | 配置文件不存在 | JSON 配置缺失 |
| SA_NOT_EXIST | 1007 | SA 不存在 | 未注册或未启动 |
| ONDEMAND_SIZE_LIMIT | 1009 | 按需加载数量超限 | 超过 MAX_SERVICES |
| SUBSCRIBE_SIZE_LIMIT | 1010 | 订阅数量超限 | 未实现限制 |
| ABILITY_MAP_SIZE_LIMIT | 1011 | SA 映射表已满 | MAX_SERVICES 限制 |
| PROC_NOT_EXIST | 1006 | 进程不存在 | 进程名错误 |

---

## 日志关键字

| 关键字 | 说明 | 日志级别 |
|--------|------|----------|
| "AddSystemAbility" | SA 注册 | INFO |
| "NOT found service" | SA 未找到 | WARNING |
| "insert SA" | SA 插入成功 | INFO |
| "rm SA" | SA 移除 | INFO |
| "LoadSystemAbility" | 加载请求 | INFO |
| "Load SA success" | 加载成功 | INFO |
| "Load SA failed" | 加载失败 | ERROR |
| "SubscribeSystemAbility" | 订阅 | INFO |
| "input params is invalid" | 参数无效 | ERROR |
| "Permission denied" | 权限拒绝 | ERROR |
