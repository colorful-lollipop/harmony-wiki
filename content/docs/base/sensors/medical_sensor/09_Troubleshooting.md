# 常见问题与排查

## 目的

本文档汇总 Medical_Sensor 的常见构建、运行和调试问题，提供定位路径和解决方法。

---

## 适用范围

本文档适用于需要：
- 解决编译错误
- 定位运行时问题
- 进行系统调试

---

## 构建问题

### 1. N-API 模块编译失败

**现象**：
```
error: 'medical' was not declared in this scope
```

**原因**：N-API 模块未正确注册或命名冲突。

**定位**：
- 检查 `medical_js.cpp:292-295` 的模块注册
- 确认 `nm_modname` 为 `"medical"`

**解决方法**：
```cpp
// 确保模块名正确
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = NULL,
    .nm_register_func = Init,
    .nm_modname = "medical",  // ⚠️ 必须与 JS 导入名一致
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

---

### 2. HDI 连接失败

**现象**：
```
ERROR: Failed to connect HDI service, ret: -1
```

**原因**：HDI 服务未启动或接口版本不匹配。

**定位**：
- HDI 连接：`services/medical_sensor/hdi_connection/adapter/src/hdi_connection.cpp:48-70`
- 检查日志 `HdiConnection::ConnectHdi()`

**解决方法**：
1. 确认传感器驱动已正确安装并启动
2. 检查 `drivers_interface_sensor` 版本是否匹配
3. 验证 `HdiServiceImpl` 是否正确注册到 HDI 框架

---

### 3. System Ability 发布失败

**现象**：
```
ERROR: publish MedicalSensorService error
```

**原因**：SA 已存在或资源初始化失败。

**定位**：
- SA 发布：`services/medical_sensor/src/medical_service.cpp:86-91`
- 检查 `OnStart()` 的错误日志

**解决方法**：
1. 检查 `sa_profile/3605.xml` 配置是否正确
2. 确认 `REGISTER_SYSTEM_ABILITY_BY_ID` 的 SA ID 为 3605
3. 验证所有依赖（HDI、权限工具等）初始化成功

---

### 4. 权限检查失败

**现象**：
```
ERROR: sensorId:129 permission failed, result:-1
```

**原因**：应用未申请 `ohos.permission.READ_HEALTH_DATA` 权限。

**定位**：
- 权限检查：`utils/src/permission_util.cpp:40-49`
- 检查应用的 `module.json` 或 `app.json` 权限配置

**解决方法**：
1. 在应用 `module.json` 中添加权限声明：
```json
"module": {
  "reqPermissions": [
    {
      "name": "ohos.permission.READ_HEALTH_DATA",
      "reason": "$string:medical_sensor_permission_reason",
      "usedScene": {
        "abilities": [
          "FormAbility"
        ],
        "when": "inuse"
      }
    }
  ]
}
```

2. 在应用代码中动态请求权限：
```javascript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
let result = await abilityAccessCtrl.requestPermissionsFromUser(
    this.context, ["ohos.permission.READ_HEALTH_DATA"]
);
```

---

## 运行时问题

### 1. 传感器数据未上报

**现象**：
```javascript
medical.on(medical.MedicalSensorType.TYPE_ID_PHOTOPLETHYSMOGRAPH, (data) => {
    console.info("PPG data: " + data.dataArray);
});
// 回调从未被调用
```

**原因**：
- 传感器硬件未启用
- 权限被拒绝
- 订阅参数错误
- 驱动层未正常工作

**定位**：
1. 检查权限日志：`utils/src/permission_util.cpp:55-60`
2. 检查服务端日志：`services/medical_sensor/src/medical_service.cpp`
3. 检查 HDI 连接日志：`services/medical_sensor/hdi_connection/adapter/src/hdi_connection.cpp`

**解决方法**：
1. 确认应用已获得 `READ_HEALTH_DATA` 权限
2. 检查传感器是否支持（调用 `GetSensorList()` 查询）
3. 验证 `interval` 参数是否合理（建议 1μs ~ 10s）
4. 使用 `hdc shell hidumper -s 3605` 查看 SA 状态

---

### 2. 应用崩溃

**现象**：
```
Fatal error: Signal 11 (SIGSEGV)
```

**原因**：
- 内存越界访问
- Use-After-Free
- 竞态条件
- 空指针解引用

**定位**：
1. 获取崩溃堆栈
2. 检查全局变量访问（如 `g_onCallbackInfos`）
3. 检查内存分配（如 `new AsyncCallbackInfo`）

**解决方法**：
1. 添加全局变量互斥锁保护（参考安全评审第 4 点）
2. 使用智能指针（`std::shared_ptr`, `std::unique_ptr`）
3. 添加空指针检查

**证据**：
- 竞态风险：`interfaces/plugin/src/medical_js.cpp:40`

---

### 3. IPC 调用超时

**现象**：
```
ERROR: EnableSensor timeout
```

**原因**：
- 服务端进程卡死
- Binder 线程阻塞
- 死锁

**定位**：
1. 检查服务端日志：`services/medical_sensor/src/medical_service.cpp`
2. 检查客户端代理状态：`frameworks/native/medical_sensor/src/medical_service_client.cpp`
3. 使用 `hdc shell ps -A | grep sensors` 查看进程状态

**解决方法**：
1. 重启 MedicalSensorService：
```bash
hdc shell aa force-stop com.example.sensor
hdc shell aa start -D 3605
```

2. 使用 `hidumper` 查看 SA 状态：
```bash
hdc shell hidumper -s 3605
```

3. 检查是否存在死锁：
   - 检查所有互斥锁的加锁/解锁顺序
   - 确保不会出现循环等待

---

### 4. 数据通道断开

**现象**：
```
ERROR: TransferDataChannel failed, ret: -1
```

**原因**：
- 服务端拒绝通道传输
- 客户端进程异常终止
- 共享内存创建失败

**定位**：
1. 检查数据通道创建：`interfaces/native/src/medical_native_impl.cpp:62-92`
2. 检查服务端通道接收：`services/medical_sensor/src/medical_service.cpp`
3. 检查客户端死亡通知：`services/medical_sensor/src/medical_service.cpp:452-481`

**解决方法**：
1. 检查应用进程是否存活
2. 确认 SA 是否正常运行（`hidumper -s 3605`）
3. 重新建立数据通道（取消订阅后重新订阅）

---

## 调试技巧

### 1. 启用详细日志

**位置**：`interfaces/plugin/BUILD.gn:24-27`

```gn
defines = [
    "APP_LOG_TAG = \"medicalJs\"",
    "LOG_DOMAIN = 0xD002701"
]
```

**方法**：
1. 修改 `LOG_DOMAIN` 为调试值（如 0xD002799）
2. 重新编译 N-API 模块
3. 使用 `hdc shell hilog -x` 过滤日志：
```bash
hdc shell hilog -x | grep medical
```

---

### 2. 使用 Dump 功能

**SA Dump**：
```bash
hdc shell hidumper -s 3605
```

**Dump 输出示例**：
```
----------------------------------------------------------
AbilityManagerService dump begin:
----------------------------------------------------------
Dump all ability info:
  SystemAbility:
    MedicalSensorService:
      state: RUNNING
      sensor count: 1
      client count: 1
----------------------------------------------------------
AbilityManagerService dump end
----------------------------------------------------------
```

**证据**：
- Dump 接口：`services/medical_sensor/include/medical_sensor_service.h:46-51`

### 3. GDB 调试

**附加到进程**：
```bash
# 找到传感器服务进程 ID
pid=$(hdc shell ps -A | grep sensors | awk '{print $2}')

# 附加 GDB
hdc file recv /tmp/gdbserver < /tmp/gdbserver
hdc shell gdbserver :5039 &
hdc shell gdbserver :5039 --attach $pid
```

**断点设置**：
```bash
(gdb) break PermissionUtil::CheckSensorPermission
(gdb) break MedicalSensorService::EnableSensor
(gdb) continue
```

### 4. 性能分析

**使用 HiSysEvent**：
```bash
# 查看系统事件
hdc shell hisysevent -l | grep medical
```

**使用 Perf 工具**：
```bash
# CPU 性能分析
hdc shell perf top -p $(pidof sensors)
```

---

## 常见错误码

| 错误码 | 说明 | 常见原因 |
|--------|------|----------|
| `SUCCESS` | 操作成功 | - |
| `ERROR` | 通用错误 | 参数错误、资源不足 |
| `ERR_PERMISSION_DENIED` | 权限拒绝 | 未申请权限 |
| `ERR_NO_INIT` | 未初始化 | HDI 未连接 |
| `INVALID_POINTER` | 空指针 | 资源未分配 |
| `PERMISSION_GRANTED` | 权限通过 | - |

**证据**：
- 错误定义：`utils/include/medical_errors.h`

---

## 代码证据索引

| 问题类别 | 关键文件路径 | 行号 |
|---------|-------------|------|
| N-API 注册 | `interfaces/plugin/src/medical_js.cpp` | 292-295 |
| 权限检查 | `utils/src/permission_util.cpp` | 40-49 |
| HDI 连接 | `services/medical_sensor/hdi_connection/adapter/src/hdi_connection.cpp` | 48-70 |
| SA 启动 | `services/medical_sensor/src/medical_service.cpp` | 62-95 |
| 数据通道 | `interfaces/native/src/medical_native_impl.cpp` | 62-92 |
| Dump 接口 | `services/medical_sensor/include/medical_sensor_service.h` | 46-51 |

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [N-API 参考](03_N-API_Reference.md) - JS API 文档
- [安全评审](07_Security_Review.md) - 安全风险和修复建议
