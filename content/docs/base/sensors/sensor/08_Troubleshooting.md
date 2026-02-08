# 常见构建/运行/调试问题与定位路径

> **目的**: 记录 Sensor 子系统的常见构建、运行和调试问题，以及相应的定位路径
> **适用范围**: /base/sensors/sensor（排除 test/ 目录）
> **关键结论**: Sensor 子系统日志系统完善，通过 HiLog 可以定位大多数问题，SA 管理器提供 dump 命令
> **相关跳转**: [项目概览](00_Overview.md) | [目录结构](01_Directory_Structure.md)

---

## 日志系统

### HiLog 使用

**日志域**: 0xD002700

**日志标签**:
- `sensorJs` - JS N-API 层
- `ISensorServiceIdl` - IDL 接口层
- `SensorService` - 服务层（默认）
- `SensorManager` - 传感器管理器
- `SensorDataManager` - 数据管理器
- `SensorAgent` - 传感器代理

**日志命令** (TODO: 补完整命令):

| 命令 | 说明 | 使用方法 |
|------|------|----------|
| `hilog -T sensorJs` | 查看 JS N-API 日志 |
| `hilog -T SensorService` | 查看服务日志 |
| `hilog -x sensor` | 查看所有传感器日志 |

> **证据**: `frameworks/js/napi/BUILD.gn:25-26`, `services/src/sensor_service.cpp`

---

## 构建问题

### 问题 1: SA 注册失败

**症状**:
- 传感器服务无法启动
- 应用调用传感器 API 时报错 "SENSOR_NATIVE_GET_SERVICE_ERR"
- SA 列表中没有 Sensor Service (3601)

**可能原因**:
- `libsensor_service.z.so` 未正确安装
- `3601.json` 配置错误
- HDF 驱动未加载

**定位路径**:
1. 检查 SA 列表:
   ```bash
   hdc shell sm dump
   ```
   或
   ```bash
   hdc shell hidumper -l
   ```
2. 检查 Sensor Service 状态:
   ```bash
   hdc shell samgr list -p sensors
   ```
3. 查看 Sensor Service 日志:
   ```bash
   hdc shell hilog -T SensorService
   ```
4. 检查 SA 配置文件:
   ```bash
   hdc shell cat /system/profile/3601.json
   ```

**解决方案**:
1. 确认 `libsensor_service.z.so` 是否正确构建和安装
2. 检查 `services/BUILD.gn` 中的 `shlib_type = "sa"` 配置
3. 重新构建 sensor 服务:
   ```bash
   ./build.sh --build-target sensor_service_target
   ```

> **证据**: `sa_profile/3601.json`, `services/BUILD.gn:17,137`

---

### 问题 2: N-API 模块加载失败

**症状**:
- JS 应用调用 `import sensor from '@ohos.sensor'` 报错
- 应用启动时无法加载传感器模块
- 错误信息包含 "Module not found" 或类似

**可能原因**:
- `libsensor.z.so` 未正确安装
- N-API 模块注册失败
- 依赖库缺失

**定位路径**:
1. 检查模块文件是否存在:
   ```bash
   hdc shell ls -l /system/lib/module/libsensor.z.so
   ```
2. 查看 JS 应用日志:
   ```bash
   hdc shell hilog -T sensorJs
   ```
3. 检查依赖库:
   ```bash
   hdc shell ldd /system/lib/module/libsensor.z.so
   ```

**解决方案**:
1. 确认 `frameworks/js/napi/BUILD.gn` 中的安装路径配置
2. 检查 `relative_install_dir = "module"` 是否正确
3. 重新构建 N-API 模块:
   ```bash
   ./build.sh --build-target sensor_js_target
   ```

> **证据**: `frameworks/js/napi/BUILD.gn:49`

---

### 问题 3: HDF 驱动连接失败

**症状**:
- 传感器服务启动但无法获取传感器列表
- 日志中显示 HDI 连接失败
- 传感器数据无法上报

**可能原因**:
- HDF 传感器驱动未加载
- `libsensor_proxy_3.0.z.so` 未找到
- HDF 服务未启动

**定位路径**:
1. 检查 HDF 服务状态:
   ```bash
   hdc shell hidumper -l
   ```
2. 查看 Sensor Service 日志中的 HDI 错误:
   ```bash
   hdc shell hilog -T SensorService | grep -i "hdi"
   ```
3. 检查 HDF 驱动配置:
   ```bash
   hdc shell cat /vendor/etc/hdf_config/
   ```

**解决方案**:
1. 确保 HDF 传感器驱动已编译
2. 检查 `sensor.gni` 中的 `hdf_drivers_interface_sensor` 配置
3. 重新构建并刷入传感器驱动

> **TODO**: 确认 HDF 驱动的具体编译命令

---

## 运行问题

### 问题 1: 权限拒绝

**症状**:
- 应用调用传感器 API 时报错 "PERMISSION_DENIED"
- 日志显示权限检查失败
- 即使声明了权限仍然失败

**可能原因**:
- bundle.json 中权限声明错误
- 权限未在系统设置中授予
- Access Token 验证失败
- 应用签名不匹配

**定位路径**:
1. 查看应用日志:
   ```bash
   hdc shell hilog -T sensorJs | grep -i "permission"
   ```
2. 查看 Sensor Service 权限日志:
   ```bash
   hdc shell hilog -T SensorService | grep -i "permission"
   ```
3. 检查应用权限配置:
   ```bash
   hdc shell aa dump -b <bundle-name>
   ```

**解决方案**:
1. 检查 `app.json` 中的权限声明
2. 确认权限名称正确（如 `ohos.permission.ACCELEROMETER`）
3. 在系统设置中授予用户权限（user_grant 类型）
4. 检查应用签名是否正确

> **证据**: `utils/common/src/permission_util.cpp:41-54`

---

### 问题 2: 传感器数据未上报

**症状**:
- 应用成功订阅传感器但未收到数据回调
- 日志显示订阅成功但无数据
- 其他应用可以正常接收数据

**可能原因**:
- 传感器硬件未启用
- HDF 驱动未正确初始化
- 权限问题（某些系统 API）
- 采样率设置过低

**定位路径**:
1. 检查传感器状态:
   ```bash
   hdc shell cat /sys/class/sensors/
   ```
2. 查看 Sensor Service 日志:
   ```bash
   hdc shell hilog -T SensorService | grep -i "enable\|disable"
   ```
3. 检查订阅信息:
   ```bash
   hdc shell hilog -T sensorJs | grep -i "subscribe"
   ```

**解决方案**:
1. 确认应用使用的传感器类型正确
2. 检查传感器硬件是否已启用
3. 尝试使用不同的采样间隔
4. 检查权限是否足够

> **证据**: `services/src/sensor_service.cpp:596-622`

---

### 问题 3: 传感器数据延迟过高

**症状**:
- 传感器数据上报延迟过大
- 数据时间戳与当前时间相差很大
- 应用响应慢

**可能原因**:
- 采样间隔设置过大
- 系统负载过高
- HDF 驱动性能问题
- FIFO 缓存溢出

**定位路径**:
1. 检查采样间隔设置:
   ```bash
   hdc shell hilog -T sensorJs | grep -i "interval"
   ```
2. 查看服务端处理延迟:
   ```bash
   hdc shell hilog -T SensorDataManager
   ```
3. 检查 FIFO 状态:
   ```bash
   hdc shell hilog -T SensorService | grep -i "fifo\|cache"
   ```

**解决方案**:
1. 调整采样间隔参数（使用预设值：normal/ui/game）
2. 检查系统负载和后台进程
3. 重启传感器服务（如适用）

> **证据**: `frameworks/js/napi/src/sensor_js.cpp:55-59`

---

### 问题 4: 内存泄漏

**症状**:
- 长时间运行后内存占用持续增长
- 应用频繁崩溃
- 系统变慢

**可能原因**:
- 订阅后未正确取消订阅
- 回调引用未释放
- 数据通道未关闭
- AsyncCallbackInfo 对象泄漏

**定位路径**:
1. 查看内存统计:
   ```bash
   hdc shell ps -A | grep sensor
   ```
2. 使用 HiCollie 查看堆栈:
   ```bash
   hdc shell xcollie -s sensors
   ```
3. 查看泄漏检测日志:
   ```bash
   hdc shell hilog -T SensorXcollie
   ```

**解决方案**:
1. 确保调用 `sensor.off()` 取消订阅
2. 检查代码中的资源释放逻辑
3. 使用内存分析工具（ASan、Valgrind）重新编译

> **证据**: `frameworks/js/napi/src/sensor_js.cpp:68-69`

---

### 问题 5: 服务崩溃

**症状**:
- Sensor Service 进程频繁重启
- 所有应用都无法使用传感器
- 日志显示崩溃或 FATAL 错误

**可能原因**:
- 空指针解引用
- 数组越界访问
- HDF 回调异常
- 内存耗尽

**定位路径**:
1. 查看崩溃堆栈:
   ```bash
   hdc shell hilog -b
   ```
2. 查看崩溃日志:
   ```bash
   hdc shell hilog -T SensorService | grep -i "fatal\|crash"
   ```
3. 使用 Debug 版本重现问题

**解决方案**:
1. 分析崩溃堆栈定位问题代码位置
2. 检查相关代码的空指针检查
3. 添加边界检查和错误处理
4. 使用 Address Sanitizer 等工具重新测试

> **TODO**: 补充常见崩溃模式

---

## Dump 命令

### Sensor Service Dump

**命令**:
```bash
hdc shell hidumper -s 3601
```

**输出内容** (待补充):
- 当前订阅列表
- 传感器信息
- 客户端信息
- 数据通道状态

> **TODO**: 补充 dump 输出格式

---

## 调试工具

### HiLog Viewer

**工具**:
- `hilog` 命令行工具
- DevEco Studio 日志查看器

### System Ability Manager

**命令**:
```bash
hdc shell samgr list
hdc shell samgr dump -i 3601
```

### IPC 调试

**工具**:
- `binder_driver` - Binder 驱动调试工具
- `hidumper` - HDF 服务调试工具

### 内存分析

**工具**:
- `AddressSanitizer` - 地址检查
- `LeakSanitizer` - 内存泄漏检测
- Valgrind - 内存分析工具

---

## 性能调优

### 采样间隔选择

| 预设 | 间隔 (纳秒) | 说明 | 适用场景 |
|------|----------|------|---------|
| `game` | 20000000 (20ms) | 游戏场景，需要高频率 |
| `ui` | 60000000 (60ms) | UI 场景，平衡性能和功耗 |
| `normal` | 200000000 (200ms) | 普通场景，低功耗优先 |

> **证据**: `frameworks/js/napi/src/sensor_js.cpp:55-59`

### 电源管理

**策略**:
- 自动暂停无活动的传感器
- 根据客户端数量动态调整
- 降低后台传感器功耗

> **证据**: `services/src/sensor_power_policy.cpp`

---

## 相关跳转

- [项目概览](00_Overview.md) - 查看日志系统和运行环境
- [对外 N-API 参考](03_NAPI_Reference.md) - 查看 API 使用指南
- [安全风险评审](07_Security_Audit.md) - 查看安全问题分析
