# 常见构建/运行/调试问题与定位路径

## 目的

本文档提供 thermal_manager 的常见问题、调试方法和定位路径，帮助开发者快速解决问题。

## 适用范围

- OpenHarmony thermal_manager 模块
- 构建问题
- 运行时问题
- 调试技巧

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [06_GN_Targets.md](06_GN_Targets.md) - GN Targets 详解

---

## 构建问题

### 1. 编译错误：找不到 NAPI 头文件

**错误信息**:
```
fatal error: napi/native_api.h: No such file or directory
```

**原因**: N-API 头文件路径配置不正确

**解决方法**:
1. 检查 `bundle.json` 中是否包含 `napi` 依赖
2. 确认 N-API SDK 正确安装
3. 检查 `external_deps` 配置：
   ```python
   external_deps = [
       "napi:ace_napi",
   ]
   ```

**证据**: `frameworks/napi/BUILD.gn:48`

---

### 2. 链接错误：未定义的符号

**错误信息**:
```
undefined reference to `IThermalSrv::GetThermalLevel'
```

**原因**: Inner API 库链接问题

**解决方法**:
1. 确认 `libthermalsrv_client.so` 已正确生成
2. 检查 `deps` 配置：
   ```python
   deps = [
       "${thermal_inner_api}:thermalsrv_client",
   ]
   ```
3. 清理编译缓存：`gn clean`

---

### 3. 条件编译错误

**错误信息**:
```
error: 'has_thermal_airplane_manager_part' was not declared
```

**原因**: Feature flags 未定义

**解决方法**:
1. 检查 `thermalmgr.gni` 中的条件编译配置
2. 确保相关 subsystem 已加入编译：
   ```gni
   if (!defined(global_parts_info) ||
       defined(global_parts_info.communication_netmanager_base)) {
       has_thermal_airplane_manager_part = true
   }
   ```
3. 检查 `bundle.json` 中的 `parts` 依赖

**证据**: `thermalmgr.gni:28-33`

---

### 4. CFI 链接错误

**错误信息**:
```
error: cannot compile this file without -fcf-protection=full
```

**原因**: CFI 配置不兼容

**解决方法**:
1. 确认编译器支持 CFI
2. 检查 `sanitize` 配置：
   ```python
   sanitize = {
       cfi = true
       cfi_cross_dso = true
   }
   ```

**证据**: `services/BUILD.gn:31-34`

---

## 运行时问题

### 1. 服务未启动

**症状**:
```
GetThermalLevel() 失败，返回 -1
Dump 无响应
```

**定位步骤**:
1. 检查 SA 是否注册：
   ```bash
   hdc shell hidumper -l ThermalService
   ```
2. 检查进程是否存在：
   ```bash
   hdc shell ps -A | grep powermgr
   ```
3. 查看系统日志：
   ```bash
   hdc shell hilog -T ThermalSrv
   ```

**可能原因**:
- 配置文件解析失败
- HDI 服务未就绪
- 权限问题

**证据**: `services/native/src/thermal_service.cpp:192-204`

---

### 2. 回调未触发

**症状**:
```
注册了 ThermalLevelCallback，但热级别变化时未收到回调
```

**定位步骤**:
1. 检查回调是否正确注册：
   ```bash
   hdc shell hidumper -s ThermalService -a SubscribeThermalLevelCallback
   ```
2. 检查回调引用是否有效：
   - N-API 层的 callback 是否被正确保存
3. 查看服务日志：
   ```bash
   hdc shell hilog -T ThermalSrv | grep callback
   ```

**可能原因**:
- 回调引用被过早释放
- IPC 连接断开
- 回调异常

**证据**: `frameworks/napi/thermal_manager_napi.cpp:51-59`

---

### 3. 动作未执行

**症状**:
```
温度超过阈值，但未触发相应动作（如 CPU 降频）
```

**定位步骤**:
1. Dump 动作状态：
   ```bash
   hdc shell hidumper -s ThermalService -a -a action
   ```
2. 查看策略配置：
   ```bash
   hdc shell hidumper -s ThermalService -a -a policy
   ```
3. 检查配置文件路径：
   ```bash
   hdc shell cat /system/etc/thermal_config/thermal_service_config.xml
   ```

**可能原因**:
- 策略配置不正确
- 动作未启用
- 设备状态不满足条件

**证据**: `services/native/include/thermal_policy/thermal_policy.h:40-82`

---

### 4. HDI 通信失败

**症状**:
```
Thermal HDI 服务未连接
温度传感器数据未上报
```

**定位步骤**:
1. 检查 HDI 服务状态：
   ```bash
   hdc shell hidumper -l
   | grep thermal_interface_service
   ```
2. 检查 thermal 进程日志：
   ```bash
   hdc shell hilog -T ThermalSrv | grep HDI
   ```
3. 查看设备树：
   ```bash
   hdc shell cat /sys/class/thermal/thermal_zone0/
   ```

**可能原因**:
- HDI 服务未启动
- 驱动未加载
- 版本不匹配

**证据**: `services/native/src/thermal_service.cpp:540-550`

---

## 调试技巧

### 1. 启用详细日志

**方法**: 通过 hidumper 启用调试

```bash
# 启用 Thermal Service 调试
hdc shell param set persist.thermal.debug 1
# 重启服务
hdc shell param set persist.thermal.dumplevel 1
```

**相关代码**: `services/native/src/thermal_service.cpp:735-749` (Dump 方法)

---

### 2. 温度仿真模式

**用途**: 模拟温度变化，用于测试

**配置方法**:
1. 修改配置文件添加 `sim_tz`：
   ```xml
   <base>
       <item tag="sim_tz" value="1"/>
   </base>
   ```
2. 或通过 shell 命令：
   ```bash
   hdc shell param set persist.thermal.simulation 1
   ```

**证据**: `services/native/src/thermal_srv_config_parser.cpp:159-176`

---

### 3. 使用 hidumper 查看 SA 状态

**常用命令**:

```bash
# 查看 Thermal Service 所有信息
hdc shell hidumper -s ThermalService -a

# 查看订阅者列表
hdc shell hidumper -s ThermalService -a SubscribeThermalLevelCallback
hdc shell hidumper -s ThermalService -a SubscribeThermalTempCallback

# 查看动作状态
hdc shell hidumper -s ThermalService -a -a action

# 查看策略配置
hdc shell hidumper -s ThermalService -a -a policy

# 查看当前温度和级别
hdc shell hidumper -s ThermalService -a -a -a level
```

**SA 接口**: `services/native/include/thermal_service.h:731-749`

---

### 4. 监控日志

**日志组件**:
- `COMP_FWK` - N-API Framework 日志
- `COMP_SVC` - Thermal Service 日志

**日志标签**:
- `ThermalSrv`
- `ThermalMgrNapi`

**常用命令**:

```bash
# 实时查看日志
hdc shell hilog -T ThermalSrv -v

# 按级别过滤
hdc shell hilog -T ThermalSrv -v | grep ERROR
hdc shell hilog -T ThermalSrv -v | grep WARN

# 保存日志到文件
hdc shell hilog -T ThermalSrv -v -f thermal.log
```

**证据**: `utils/native/src/thermal_log.cpp` (推测日志模块)

---

### 5. 性能分析

**工具**: HiDumper + HiLog 时间戳

**分析方法**:
1. 查看策略执行频率
2. 分析动作执行时间
3. 检查是否有频繁的回调触发

**关键指标**:
- 策略决策延迟
- 动作执行延迟
- 温度上报频率

---

## 定位路径

### 问题定位流程图

```
问题发现
    ↓
是否是构建问题？
    ├─ 是 → 检查 GN 配置和依赖
    └─ 否 → 继续
            ↓
是否是启动问题？
    ├─ 是 → 检查 SA 注册、进程状态
    └─ 否 → 继续
            ↓
是否是功能问题？
    ├─ 是 → Dump 配置、日志、HDI 状态
    └─ 否 → 继续
            ↓
是否是性能问题？
    ├─ 是 → 性能分析、优化策略
    └─ 否 → 继续分析
```

---

## 常见配置问题

### 配置文件未生效

**症状**: 修改配置后无效果

**解决方法**:
1. 确认配置文件路径：
   - 系统配置: `/system/etc/thermal_config/thermal_service_config.xml`
   - 厂商配置: `/vendor/etc/thermal_config/thermal_service_config.xml`
2. 检查文件格式是否正确（XML 语法）
3. 重启服务或设备：
   ```bash
   hdc shell "sa_restart 3303"
   # 或
   hdc shell reboot
   ```
4. 检查配置解析日志：
   ```bash
   hdc shell hilog -T ThermalSrv | grep "ParseXmlFile"
   ```

**证据**: `services/native/src/thermal_srv_config_parser.cpp:40-65`

---

### 特性开关未生效

**症状**: 编译时指定了 feature flag，但功能未启用

**解决方法**:
1. 确认子系统已加入编译：
   - 检查 `bundle.json` 中的 `parts` 列表
2. 检查 GN 参数：
   ```bash
   gn args --thermal_manager_audio_framework_enable=true
   ```
3. 确认宏定义：
   ```cpp
   #ifdef HAS_THERMAL_AUDIO_FRAMEWORK_PART
   ```
4. 清理并重新编译

**证据**: `services/BUILD.gn:36-40`

---

## 故障排除清单

### 构建检查清单

- [ ] 确认 GN 版本正确
- [ ] 确认所有依赖子系统已加入编译
- [ ] 检查 feature flags 正确
- [ ] 清理编译缓存：`gn clean`
- [ ] 确认外部依赖正确配置

### 运行时检查清单

- [ ] SA 3303 是否已注册
- [ ] thermal 服务进程是否运行
- [ ] HDI 服务是否可用
- [ ] 配置文件是否正确加载
- [ ] 日志级别是否足够
- [ ] 权限配置是否正确

---

## 相关资源

### 官方文档

- [OpenHarmony 电源管理子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/power-management.md)
- [热管理文档](https://gitee.com/openharmony/powermgr_thermal_manager)

### 工具

- `hdc shell` - OpenHarmony 命令行工具
- `hilog` - 日志查看工具
- `hidumper` - System Ability 调试工具

---

## 总结

Thermal Manager 的调试可以通过以下方式进行：

1. **使用 hidumper** - 查看 SA 状态、配置、订阅者
2. **使用 hilog** - 查看详细日志，过滤级别
3. **Dump 功能** - 导出服务内部状态
4. **温度仿真** - 模拟温度变化进行测试
5. **参数控制** - 通过系统参数启用调试模式

**关键定位点**:
- 配置文件解析
- HDI 服务状态
- IPC 连接状态
- 策略执行流程
- 回调通知机制

**建议**: 遇到问题时，先收集日志和状态信息，再根据问题类型定位具体模块。
