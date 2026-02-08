# 常见问题

## 目的

本文档汇总 `telephony_core_service` 开发、构建和调试过程中的常见问题及解决方法。

---

## 构建问题

### Q1: 编译时提示缺少 `drivers_interface_ril` 依赖

**现象**:
```
ERROR: //base/telephony/core_service:tel_core_service
  //drivers/interface/ril:libril_proxy_1.5 not found
```

**原因**: 缺少 RIL 驱动接口依赖

**解决**:
1. 确保已同步 `drivers_interface` 仓库
2. 在 `bundle.json` 中确认 `drivers_interface_ril` 在 deps 列表中

**证据** (`bundle.json:51`):
```json
"components": [
    "drivers_interface_ril",
    // ...
]
```

---

### Q2: eSIM 功能编译失败

**现象**:
```
ERROR: esim_manager.cpp:10:10: fatal error: 'asn1_decoder.h' file not found
```

**原因**: 未启用 `core_service_support_esim` 但代码尝试编译 eSIM 相关文件

**解决**:
1. 在构建命令中添加参数:
   ```bash
   gn gen --args='core_service_support_esim=true'
   ```
2. 或在 `products` 配置中设置:
   ```gn
   core_service_support_esim = true
   ```

**证据** (`BUILD.gn:131-137`):
```gn
if (core_service_support_esim) {
  sources += [
    "$TELEPHONY_SIM_ROOT/src/esim_controller.cpp",
    "$TELEPHONY_SIM_ROOT/src/esim_file.cpp",
  ]
}
```

---

### Q3: 链接时 CFI 错误

**现象**:
```
error: undefined symbol: __cfi_check
```

**原因**: Sanitizer CFI 配置与某些库不兼容

**解决**:
1. 检查所有依赖库是否都启用了 CFI
2. 临时禁用 CFI 测试:
   ```gn
   sanitize = {
     cfi = false
   }
   ```

**证据** (`BUILD.gn:30-35`):
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

---

## 运行时问题

### Q4: CoreService 启动失败

**现象**: 日志显示 `CoreService::Init Publish failed!`

**原因**: SA Manager 未就绪或权限问题

**排查步骤**:
1. 检查 `sa_profile/4010.json` 是否正确部署
2. 检查日志:
   ```bash
   hilog | grep -i "coreservice"
   ```
3. 确认 `libtel_core_service.z.so` 存在于 `/system/lib64/`

**证据** (`services/core/src/core_service.cpp:64-70`):
```cpp
if (!registerToService_) {
    bool ret = Publish(DelayedSingleton<CoreService>::GetInstance().get());
    if (!ret) {
        TELEPHONY_LOGE("CoreService::Init Publish failed!");
        return;
    }
}
```

---

### Q5: RIL 连接失败

**现象**: 日志显示 `TelRilManager init is failed!`

**原因**: RIL Adapter 服务未启动或 HDI 接口不匹配

**排查步骤**:
1. 检查 RIL 服务状态:
   ```bash
   service_control status ril_adapter
   ```
2. 检查 HDI 版本兼容性 (`sa_profile/4010.json:10`):
   ```json
   "min_hdi_proxy_version": ["libril_proxy_1.1.z.so"]
   ```
3. 检查日志中的详细错误信息

**证据** (`services/core/src/core_service.cpp:97-101`):
```cpp
telRilManager_ = std::make_shared<TelRilManager>();
if (!telRilManager_->OnInit()) {
    TELEPHONY_LOGE("TelRilManager init is failed!");
    return false;
}
```

---

### Q6: JS API 调用返回权限错误

**现象**: `getSimIccId()` 返回 `401 (Permission denied)`

**原因**: 应用未申请或未获得相应权限

**解决**:
1. 在 `module.json5` 中声明权限:
   ```json
   "requestPermissions": [
     {
       "name": "ohos.permission.GET_TELEPHONY_STATE"
     }
   ]
   ```
2. 运行时申请权限 (如果是 user_grant 权限)

**证据** (`utils/common/src/telephony_permission.cpp:70-101`):
```cpp
bool TelephonyPermission::CheckPermission(const std::string &permissionName)
{
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int result = AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    if (result != PermissionState::PERMISSION_GRANTED) {
        return false;
    }
    return true;
}
```

---

### Q7: 双卡设备 SlotId 无效错误

**现象**: `slotId: 1` 返回 `ERROR_SLOT_ID_INVALID`

**原因**: `SIM_SLOT_COUNT` 配置与实际硬件不匹配

**排查步骤**:
1. 检查 `telephony_config.h` 中的 `SIM_SLOT_COUNT` 定义
2. 确认设备是否支持双卡
3. 使用 `getMaxSimCount()` 获取支持的最大卡槽数

**证据** (`frameworks/js/sim/src/napi_sim.cpp:52-55`):
```cpp
static inline bool IsValidSlotId(int32_t slotId)
{
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT));
}
```

---

## 调试方法

### 查看 CoreService 日志

```bash
# 过滤 CoreService 日志
hilog | grep -E "CoreService|CoreServiceApi|CoreServiceSimJsApi|CoreServiceRadioJsApi"

# 设置日志级别为 DEBUG
param set telephony.log.level 3
```

### Dump 服务状态

```bash
# 使用 hidumper 查看服务状态
hidumper -s 4010 -a "-h"

# 查看帮助
hidumper -s 4010
```

**证据** (`services/core/src/core_service_dump_helper.cpp`):
```cpp
int32_t CoreServiceDumpHelper::Dump(int32_t fd, const std::vector<std::u16string> &args) {
    // 输出 SIM 状态、网络状态等
}
```

### 使用 GDB 调试

```bash
# 附加到 telephony 进程
gdb-pid $(pidof telephony)

# 设置断点
(gdb) b CoreService::OnStart
(gdb) b SimManager::OnInit

# 继续执行
(gdb) c
```

### 分析崩溃日志

```bash
# 查看崩溃堆栈
hilog | grep -A 20 "Crash"

# 使用 addr2line 解析地址
addr2line -e libtel_core_service.z.so -Cfa <address>
```

---

## 性能优化

### FFRT 线程调优

**代码证据** (`services/core/src/core_service.cpp:46,72`):
```cpp
const int32_t MAX_FFRT_THREAD_NUM = 32;
ffrt_set_cpu_worker_max_num(ffrt::qos_default, MAX_FFRT_THREAD_NUM);
```

**建议**: 根据设备核心数调整 `MAX_FFRT_THREAD_NUM`

### IPC 超时处理

**现象**: 某些 API 调用超时

**解决**:
1. 检查 RIL 响应时间
2. 调整 IPC 超时配置
3. 检查是否有阻塞操作

---

## 相关链接

- [架构设计](./02_Architecture.md)
- [安全风险](./06_Security.md)
- [GN 构建](./05_GN_Build.md)
