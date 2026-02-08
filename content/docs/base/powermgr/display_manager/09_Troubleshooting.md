# 常见问题 - display_manager

> 本文档汇总 display_manager 模块的常见问题及解决方案

---

## 文档目的

本文档提供：
- 常见问题及解决方案
- 调试方法和技巧
- 问题定位路径
- 排查检查清单

## 适用范围

- **适用对象**：应用开发者、系统集成人员、故障排查工程师
- **前置知识**：熟悉 OpenHarmony 系统、基本调试方法

---

## 构建问题

### Q1: 编译失败，提示缺少依赖

**症状**：
```
ERROR: //base/powermgr/display_manager/state_manager/service:displaymgrservice 
  Missing dependency: power_manager:power_permission
```

**原因**：依赖的组件未参与编译

**解决方案**：
1. 检查 `bundle.json` 中的 `deps` 配置
2. 确保依赖的组件在编译目标中：
   ```bash
   # 检查 power_manager 是否参与编译
   hb build -T //base/powermgr/power_manager -v
   ```
3. 如果缺少组件，更新 `productdefine/common/products/{product}.json`

---

### Q2: 链接错误，找不到符号

**症状**：
```
ld.lld: error: undefined symbol: OHOS::DisplayPowerMgr::DisplayPowerMgrClient::GetInstance()
```

**原因**：未链接正确的库或库路径配置错误

**解决方案**：
1. 检查 `BUILD.gn` 中的 `deps` 配置：
   ```gn
   deps = [
     "//base/powermgr/display_manager/state_manager/interfaces/inner_api:displaymgr",
   ]
   ```
2. 确保库文件已生成：
   ```bash
   ls out/{product}/{variant}/system/lib/platformsdk/libdisplaymgr.so
   ```

---

### Q3: IDL 生成代码编译错误

**症状**：
```
error: 'IDisplayPowerMgr' was not declared in this scope
```

**原因**：IDL 未正确生成或包含路径错误

**解决方案**：
1. 清理并重新生成：
   ```bash
   rm -rf out/{product}/{variant}/gen/base/powermgr/display_manager
   hb build -T //base/powermgr/display_manager
   ```
2. 检查 `BUILD.gn` 中的 include_dirs：
   ```gn
   include_dirs = [ "${target_gen_dir}" ]
   ```

---

## 运行时问题

### Q4: 服务无法启动

**症状**：
- 屏幕亮度无法调节
- `dump` 命令无输出
- 日志显示服务未注册

**排查步骤**：

1. **检查 SA 配置**：
   ```bash
   # 查看 SA 配置文件
   cat /system/profile/displaymgr_sa_profile.json
   ```
   应包含：
   ```json
   {
       "name": 3308,
       "libpath": "libdisplaymgrservice.z.so"
   }
   ```

2. **检查服务库**：
   ```bash
   # 确认库文件存在
   ls -la /system/lib/libdisplaymgrservice.z.so
   
   # 检查依赖
   ldd /system/lib/libdisplaymgrservice.z.so
   ```

3. **查看系统日志**：
   ```bash
   # 过滤 display 相关日志
   hilog | grep -i display
   
   # 查看详细错误
   hilog -g | grep DisplayPower
   ```

4. **手动启动测试**（仅调试）：
   ```bash
   # 尝试手动加载
   LD_LIBRARY_PATH=/system/lib:/system/lib/platformsdk \
     ./displaymgrservice_test
   ```

---

### Q5: JS API 调用失败

**症状**：
```javascript
import brightness from '@ohos.display.brightness';
brightness.setValue(128);  // 报错或无效
```

**排查步骤**：

1. **检查权限**：
   ```javascript
   // 确认应用有 system 权限
   import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
   ```

2. **检查 N-API 模块**：
   ```bash
   # 确认模块存在
   ls /system/lib/module/libbrightness.so
   ```

3. **查看 N-API 错误**：
   ```bash
   # 过滤 NAPI 相关日志
   hilog | grep -i "napi\|brightness"
   ```

4. **使用 Inner API 测试**：
   ```cpp
   // 用 Native 代码测试
   #include "display_power_mgr_client.h"
   bool ret = DisplayPowerMgrClient::GetInstance().SetBrightness(128);
   ```

---

### Q6: 亮度设置无效

**症状**：
- 调用 `setValue()` 成功但亮度未变
- 亮度值读取正确但不生效

**可能原因及解决方案**：

1. **自动亮度开启**：
   ```cpp
   // 检查自动亮度状态
   bool autoMode = DisplayPowerMgrClient::GetInstance().IsAutoAdjustBrightness();
   if (autoMode) {
       // 关闭自动亮度或等待传感器更新
       DisplayPowerMgrClient::GetInstance().AutoAdjustBrightness(false);
   }
   ```

2. **亮度被覆盖**：
   ```cpp
   // 检查是否有覆盖亮度
   // 可通过 dump 查看
   dump displaymgr
   ```

3. **硬件问题**：
   ```bash
   # 检查 HAL 层
   cat /sys/class/backlight/panel0-backlight/brightness
   ```

---

### Q7: 自动亮度不工作

**症状**：
- 自动亮度已开启但无变化
- 传感器数据正常但亮度不调整

**排查步骤**：

1. **检查传感器可用性**：
   ```cpp
   bool support = DisplayPowerMgrClient::GetInstance().IsSupportLightSensor();
   ```

2. **检查自动亮度开关**：
   ```cpp
   bool enabled = DisplayPowerMgrClient::GetInstance().IsAutoAdjustBrightness();
   ```

3. **查看光传感器数据**：
   ```bash
   # 查看传感器 HAL 日志
   hilog | grep -i "sensor\|lux"
   ```

4. **检查配置文件**：
   ```bash
   # 确认亮度曲线配置存在
   ls /system/etc/display/
   ```

---

## 调试技巧

### 查看服务状态

```bash
# 方法一：使用 dump
dump displaymgr

# 方法二：使用 hidumper
hidumper -s 3308

# 方法三：直接读取日志
hilog -g | grep -A 20 "DisplayPowerMgrService"
```

**Dump 输出示例**：
```
-------------------------------[ability]-------------------------------

Display Power Manager Service Info:
  Display State: DISPLAY_ON
  Brightness: 128
  Auto Adjust: false
  Override: false
  Boost: false
```

---

### 日志分析

**关键日志标签**：
```bash
# Display 服务日志
hilog | grep "DisplayPowerSvc"

# Brightness 管理日志
hilog | grep "BrightnessManager"

# Screen 控制日志
hilog | grep "ScreenController"

# NAPI 层日志
hilog | grep "COMP_FWK"
```

**日志级别设置**：
```cpp
// 代码中设置调试级别
#define DISPLAY_HILOGD(tag, fmt, ...) HILOG_DEBUG(LOG_CORE, fmt, ##__VA_ARGS__)
```

---

### 使用 GDB 调试

```bash
# 附加到 powermgr 进程
gdb out/{product}/{variant}/system/lib/libdisplaymgrservice.z.so $(pidof powermgr)

# 设置断点
(gdb) b DisplayPowerMgrService::SetBrightnessInner

# 运行
(gdb) continue
```

---

### 跟踪 IPC 调用

```bash
# 使用 binder 跟踪
binder_calls displaymgrservice

# 查看 IPC 统计
cat /sys/kernel/debug/binder/stats
```

---

## 性能优化

### 亮度调节延迟

**问题**：亮度调节有延迟感

**优化建议**：
1. 减少渐变时长：
   ```cpp
   // 直接设置，无渐变
   DisplayPowerMgrClient::GetInstance().SetBrightness(value, 0, false);
   ```

2. 使用连续更新模式：
   ```cpp
   // 拖动亮度条时使用
   DisplayPowerMgrClient::GetInstance().SetBrightness(value, 0, true);
   ```

---

### 内存优化

**检查内存使用**：
```bash
# 查看 powermgr 进程内存
cat /proc/$(pidof powermgr)/status | grep -i "vm\|rss"

# 使用 hisysevent 监控
hisysevent -l -n POWER -o BRIGHTNESS
```

---

## 已知限制

### 功能限制

| 限制 | 说明 | 解决方案 |
|------|------|----------|
| 仅系统应用可用 | 非 system 应用无法调用 | 申请系统应用签名 |
| 单进程限制 | 服务运行在 powermgr 进程 | 跨进程 IPC 调用 |
| 亮度范围 0-255 | 硬件限制 | 使用折扣亮度实现更低 |

### 平台差异

| 平台 | 说明 |
|------|------|
| 标准系统 | 完整功能 |
| 轻量系统 | 部分功能不可用 |
| 无传感器设备 | 自动亮度不可用 |

---

## 联系支持

### 问题反馈

1. **Issue 跟踪**：
   - 仓库：gitee.com/openharmony/powermgr_display_manager
   - 标签：bug、question

2. **邮件列表**：
   - openharmony-dev@openharmony.io

3. **日志收集**：
   ```bash
   # 收集完整日志
   hilog > display_issue_log.txt
   dump displaymgr > display_dump.txt
   ```

---

## 相关链接

- **项目概览**：[00_Overview.md](00_Overview.md)
- **架构文档**：[03_Architecture.md](03_Architecture.md)
- **N-API 接口**：[04_NAPI_Interface.md](04_NAPI_Interface.md)
- **内部 API**：[05_Internal_API.md](05_Internal_API.md)
- **安全分析**：[08_Security_Analysis.md](08_Security_Analysis.md)

---

## 文档更新记录

- **2026-02-07**：初始版本 v1.0，汇总常见问题和解决方案
