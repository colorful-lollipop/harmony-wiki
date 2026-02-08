# 常见问题 (FAQ)

> **目的**: 收集和解答 powermgr_cangjie_wrapper 的常见问题
> **适用范围**: 问题排查、开发调试、学习参考
> **最后更新**: 2025-02-06

---

## 构建相关

### Q1: 编译时提示找不到 `battery_manager:cj_battery_info_ffi`

**症状**:
```
ERROR at //base/powermgr/powermgr_cangjie_wrapper/ohos/battery_info/BUILD.gn:35:12: Unable to resolve "battery_manager:cj_battery_info_ffi"
```

**原因**: 未正确配置 `battery_manager` 组件的路径或未编译该组件

**解决方法**:
1. 检查 `battery_manager` 组件是否在代码库中
2. 先编译 `battery_manager` 组件：
   ```bash
   ./build.sh --product-name <product> --build-target battery_manager
   ```
3. 确认 `battery_manager` 的 `BUILD.gn` 中有 `cj_battery_info_ffi` target

**证据**: `ohos/battery_info/BUILD.gn:35`

---

### Q2: Windows/macOS 编译总是返回默认值

**症状**: API 调用返回 0 或 `Unknown` 状态

**原因**: Windows/macOS 使用 Mock 实现，返回固定的默认值

**说明**: 这是正常行为，Mock 实现用于开发环境调试

**证据**: `ohos/battery_info/BUILD.gn:21-28`, `mock/ohos.battery_info.cj`

**解决方法**:
- 在真实 OpenHarmony 设备上测试
- 或使用模拟器（如果支持）

---

### Q3: 编译产物名称和路径在哪里？

**症状**: 编译成功但找不到 `.so` 文件

**原因**: 编译输出路径取决于产品配置和构建类型

**预期路径**:
```
out/<product>/<variant>/libs/libohos.battery_info.so
```

**解决方法**:
1. 查找编译输出：
   ```bash
   find out -name "libohos.battery_info.so"
   ```
2. 检查构建日志中的输出路径
3. 确认产品配置正确

**TODO(需确认)**: 完整的输出路径结构需通过构建输出验证

---

## 运行时相关

### Q4: 应用启动时提示找不到 `ohos.battery_info` 包

**症状**:
```
Error: Package ohos.battery_info not found
```

**原因**: 未正确导入或 SDK 未正确配置

**解决方法**:
1. 确认导入语句正确：
   ```cangjie
   import ohos.battery_info
   ```
2. 检查项目依赖配置（如 `cjpm.toml`）
3. 确认 `libohos.battery_info.so` 已安装在设备上：
   ```bash
   ls -l /system/lib64/libohos.battery_info.so
   ```

**证据**: `ohos/battery_info/battery_info.cj:18`

---

### Q5: 调用 API 时抛出 `BusinessException(401, "Parameter error.")`

**症状**:
```
BusinessException: code=401, message="Parameter error."
```

**原因**: C 层返回了无效的枚举值

**证据**: `ohos/battery_info/battery_info.cj:225, 282, 359, 446`

**可能原因**:
1. `battery_manager` 服务异常
2. 设备硬件故障
3. C 层实现 bug

**解决方法**:
1. 检查设备硬件是否正常
2. 查看系统日志（hilog）获取详细错误信息
3. 重启设备或系统服务

**示例**:
```cangjie
try {
    let status = BatteryInfo.chargingStatus
} catch (e: BusinessException) {
    println("错误: ${e.code} - ${e.message}")
    // 处理错误
}
```

---

### Q6: `BatteryInfo.technology` 返回空字符串

**症状**:
```cangjie
let tech = BatteryInfo.technology  // 返回 ""
```

**原因**:
1. Mock 环境（Windows/macOS）
2. 设备不支持电池技术信息
3. C 层返回空字符串

**证据**: `mock/ohos.battery_info.cj:99`

**解决方法**:
- 在真实 OpenHarmony 设备上测试
- 检查设备是否支持该信息

---

## API 使用相关

### Q7: 如何获取当前充电状态？

**代码示例**:
```cangjie
import ohos.battery_info

let status = BatteryInfo.chargingStatus
match (status) {
    case BatteryChargeState.Enabled => println("正在充电")
    case BatteryChargeState.Full => println("已充满")
    case BatteryChargeState.Disabled => println("未充电")
    case BatteryChargeState.UnknownChargeState => println("状态未知")
}
```

**证据**: `ohos/battery_info/battery_info.cj:55-60`

---

### Q8: 如何获取电池温度？单位是什么？

**代码示例**:
```cangjie
let temp = BatteryInfo.batteryTemperature
println("电池温度: ${temp / 10.0}℃")
```

**说明**: 温度单位是 0.1℃，需要除以 10 得到摄氏度

**证据**: `ohos/battery_info/battery_info.cj:122-130`

---

### Q9: 是否支持异步调用或事件监听？

**答案**: **不支持**

**说明**:
- 当前 API 都是同步的静态属性
- 无异步/Promise 机制
- 无事件回调或监听器

**替代方案**:
- 使用定时器轮询（注意不要过于频繁）

**证据**: 所有 API 都是静态属性 getter，无异步机制

---

### Q10: 如何判断电池电量是否低？

**代码示例**:
```cangjie
let level = BatteryInfo.batteryCapacityLevel
match (level) {
    case BatteryCapacityLevel.LevelCritical |
         BatteryCapacityLevel.LevelShutdown => println("电量严重不足")
    case BatteryCapacityLevel.LevelWarning => println("电量警告")
    case _ => println("电量正常")
}
```

**证据**: `ohos/battery_info/battery_info.cj:152-157`

---

## 权限相关

### Q11: 是否需要特殊权限才能使用电池信息 API？

**答案**: **API 层无权限检查**

**说明**:
- 当前 API 层没有显式权限检查代码
- 任何应用都可以调用电池信息 API

**TODO(需确认)**: 系统层面是否有权限控制

**证据**: `ohos/battery_info/battery_info.cj` - 无权限检查代码

**建议**:
- 查阅 OpenHarmony 官方文档确认是否需要权限声明

---

## 性能相关

### Q12: 高频查询电池信息会有性能问题吗？

**说明**:
- API 是同步阻塞调用，每次调用都需要访问底层服务
- 频繁查询可能影响应用性能

**建议**:
- 避免过于频繁的轮询（如每秒多次）
- 使用合理的查询间隔（如每分钟一次）
- 考虑使用系统提供的电量变化事件（如果未来支持）

**证据**: 所有 API 都是同步调用

---

## 错误调试

### Q13: 如何调试 FFI 绑定问题？

**步骤**:
1. 检查 C 层库是否正确安装：
   ```bash
   ls -l /system/lib64/libcj_battery_info_ffi.so
   ```

2. 检查库的符号：
   ```bash
   nm -D /system/lib64/libcj_battery_info_ffi.so | grep FfiBatteryInfo
   ```

3. 查看系统日志：
   ```bash
   hilog | grep -i battery
   ```

4. 在应用中添加异常处理：
   ```cangjie
   try {
       let soc = BatteryInfo.batterySoc
   } catch (e: Exception) {
       println("FFI 错误: ${e}")
   }
   ```

---

### Q14: Mock 环境下如何测试？

**说明**: Mock 环境返回固定值，可用于测试 API 调用流程

**Mock 返回值**:
- `batterySoc`: 0
- `chargingStatus`: `UnknownChargeState`
- `healthStatus`: `UnknownHealthState`
- `pluggedType`: `UnknownType`
- `voltage`: 0
- `nowCurrent`: 0
- `technology`: `""` (空字符串)
- `batteryTemperature`: 0
- `isBatteryPresent`: true
- `batteryCapacityLevel`: `LevelFull`

**证据**: `mock/ohos.battery_info.cj:30-135`

---

## 限制和已知问题

### Q15: 不支持哪些功能？

**不支持的功能** (对比 ArkTS 版本):
- ❌ 重启服务（系统重启和关机）
- ❌ 系统电源管理服务
- ❌ 显示相关功耗调整
- ❌ 节能模式
- ❌ 电池状态监听和上报
- ❌ 温控管理
- ❌ 功耗统计
- ❌ 轻量级设备支持
- ❌ 异步 API

**证据**: `README.md:47-56`

---

## 参考资源

### 官方文档
- [OpenHarmony 电源管理子系统](https://docs.openharmony.cn/)
- [Cangjie 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/)
- [Battery Information Development Guide](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/basic-services/cj-battery-info-development-guide.md)

### 相关仓库
- [powermgr_power_manager](https://gitcode.com/openharmony/powermgr_power_manager)
- [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [对外 API](03_Public_API.md) - API 详细说明
- [安全风险评审](07_Security_Review.md) - 了解安全注意事项
- [编译产物](06_Build_Artifacts.md) - 构建产物和安装路径
