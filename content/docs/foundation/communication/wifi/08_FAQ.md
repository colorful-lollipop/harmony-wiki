# 常见构建/运行/调试问题

**目的**: 提供 WLAN 组件常见问题的解答和定位路径

**适用范围**: 开发者、测试工程师、技术支持人员

**生成时间**: 2026-02-06

---

## 构建相关问题

### Q1: 编译时出现 "undefined reference to xxx"

**症状**:
```
undefined reference to 'OHOS::Wifi::WIFI_DEVICE_ABILITY_ID'
```

**原因**:
- 缺少 `wifi/frameworks/native/interfaces/define.h` 的包含
- 或者头文件路径不正确

**解决方案**:
1. 检查 `.gni` 文件中的 `include_dirs` 配置
2. 确认 `wifi/frameworks/native/interfaces/define.h` 路径正确
3. 重新构建：`./build.sh --clean && ./build.sh`

**证据**: `wifi/frameworks/native/src/wifi_device_impl.cpp:88` 包含 `wifi/frameworks/native/interfaces/define.h`

---

### Q2: NAPI 模块编译失败

**症状**:
```
error: undefined symbol: napi_module_register
```

**原因**:
- N-API 头文件未正确包含
- 或 `NAPI_ENGINE` 未正确设置

**解决方案**:
1. 检查 `wifi/frameworks/js/napi/BUILD.gn` 的 `defines` 配置
2. 确认是否正确引用 Node-API 头文件
3. 查看构建日志获取详细错误信息

**证据**: `wifi/frameworks/js/napi/src/wifi_napi_entry.cpp:476` 使用 `napi_module_register()`

---

### Q3: Feature flag 未生效

**症状**:
```cpp
// 某个功能未按预期工作
#ifdef FEATURE_AP_EXTENSION
    // 这个代码应该被编译
#endif
```

**原因**:
- Feature flag 在 `.gni` 文件中设置，但未传递给 GN
- 或者在不同的 target 中不一致

**解决方案**:
1. 检查 `wifi/wifi.gni` 中的 flag 定义
2. 确认编译命令：`./build.sh --gn-args="wifi_feature_xxx=true"`
3. 检查 `bundle.json` 中的 feature 列表是否包含该 flag

**证据**: `wifi/wifi.gni:20-82` 定义了 40+ 个 feature flags

---

### Q4: 链接时出现 "multiple definition of xxx"

**症状**:
```
multiple definition of 'WifiDeviceProxy'
```

**原因**:
- 同一个头文件被多次包含在不同的 include 目录
- 导致符号重复定义

**解决方案**:
1. 检查头文件是否使用了 `#pragma once` 或 include guard
2. 检查 BUILD.gn 中的 `include_dirs` 顺序
3. 清理构建缓存：`./build.sh --clean`

**证据**: 多数 C++ 头文件使用 include guard（如 `#ifndef WIFI_I_WIFI_DEVICE_H`）

---

## 运行时问题

### Q5: WiFi 无法启动

**症状**:
- `enableWifi()` 返回 false
- `isWifiActive()` 返回 false
- 日志显示 "WiFi not started"

**可能原因**:
1. System Ability 未成功加载
2. HAL 驱动未初始化
3. 配置文件损坏
4. 权限不足

**定位步骤**:
1. 检查 SA 状态：
   ```bash
   hdc shell
   ps -A | grep wifi_manager
   ```
2. 查看 SA 日志：
   ```bash
   hdc shell hilog -T WifiManager
   ```
3. 检查权限：
   ```bash
   hdc shell aa dump -a <bundle_name>
   ```
4. 检查 HAL 服务状态：
   ```bash
   hdc shell service list | grep wifi_hal
   ```

**证据**:
- `wifi_device_mgr_service_impl.cpp:100` - `Publish()` 调用
- `wifi/services/wifi_standard/etc/init/wifi_standard.cfg` - SA 启动配置

---

### Q6: 扫描不到任何网络

**症状**:
- `scan()` 返回 true
- `getScanInfos()` 返回空数组

**可能原因**:
1. WiFi 硬件未启动
2. 扫描权限未授予
3. 无线网络环境干扰
4. 扫描持续时间过短

**定位步骤**:
1. 检查 WiFi 是否启用：
   ```javascript
   if (!await wifi.isWifiActive()) {
       console.error("WiFi is not active");
       return;
   }
   ```
2. 检查权限：
   ```javascript
   // 确保应用有 GET_WIFI_INFO 和 LOCATION 权限
   ```
3. 增加扫描结果延迟：
   ```javascript
   await wifi.scan();
   await new Promise(resolve => setTimeout(resolve, 2000)); // 等待 2 秒
   const results = await wifi.getScanInfos();
   ```

**证据**:
- `wifi_scan_service.cpp:166` - `Scan()` 实现调用 HAL
- `wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_scan_sa/wifi_scan_mgr_service_impl.cpp:153` - 权限检查

---

### Q7: 连接失败，错误码未知

**症状**:
- `connectToDevice()` 返回 false
- 错误码不是标准的枚举值

**可能原因**:
1. 错误码映射不完整
2. HAL 返回了未知的错误值
3. 超时或中断

**定位步骤**:
1. 查看 HAL 日志获取原始错误：
   ```bash
   hdc shell hilog -T WifiHal
   ```
2. 检查 wpa_supplicant 日志：
   ```bash
   hdc shell wpa_cli status
   ```
3. 查看错误码定义：
   - `wifi/interfaces/kits/c/wifi_error_code.h`
   - 对比 HAL 返回值与标准错误码

**证据**:
- `wifi_napi_errcode.cpp` - 定义了标准错误码
- `wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_sta/sta_service.cpp` - 服务实现中调用 HAL 并处理错误

---

### Q8: 热点无法启动

**症状**:
- `enableHotspot()` 返回 false
- 日志显示 "Hotspot start failed"

**可能原因**:
1. AP 实例数量限制（`wifi_feature_with_ap_num`）
2. 系统应用冲突（MDM 限制）
3. 信道冲突
4. 资源不足

**定位步骤**:
1. 检查当前热点配置：
   ```javascript
   const config = await wifi.getHotspotConfig();
   console.log("Current hotspot config:", config);
   ```
2. 检查 AP 实例限制：
   - 查看 `wifi/wifi.gni` 中的 `wifi_feature_with_ap_num` 设置
   - 默认值为 1，某些设备可能限制为 0
3. 检查 MDM 策略：
   ```bash
   hdc shell dump mdm policy
   ```
4. 查看 AP 服务日志：
   ```bash
   hdc shell hilog -T WifiHotspot
   ```

**证据**:
- `wifi_hotspot_mgr_service_impl.cpp:48` - AP SA 实现
- `wifi/wifi.gni:25` - `wifi_feature_with_ap_num = 1`

---

### Q9: 事件回调未触发

**症状**:
- 调用 `on('wifiStateChange', callback)` 后，断开连接时回调未被调用

**可能原因**:
1. 应用进程被杀死
2. 事件订阅失败
3. 回调函数抛出异常
4. 系统资源耗尽

**定位步骤**:
1. 确认事件订阅成功：
   ```javascript
   try {
       await wifi.on('wifiStateChange', (data) => {
           console.log('Event received:', data);
       });
       console.log('Subscription successful');
   } catch (e) {
       console.error('Subscription failed:', e);
   }
   ```
2. 检查应用生命周期：
   - 确保在 `onDestroy()` 中取消订阅
   - `wifi.off('wifiStateChange')`
3. 查看事件分发器日志：
   ```bash
   hdc shell hilog -T WifiEventDispatcher
   ```

**证据**:
- `wifi_napi_event.cpp:474-549` - `On()`/`Off()` 实现事件订阅
- `wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_common/wifi_internal_event_dispatcher.cpp` - 事件分发器

---

## 调试技巧

### 启用详细日志

**方法 1**: 通过 hdc shell 启用详细日志
```bash
# 启用 Wi-Fi 模块详细日志
hdc shell "hilog -r -b WifiManager && hilog -r -b WifiScan && hilog -r -b WifiHotspot"

# 实时查看日志
hdc shell hilog -T WifiManager | grep "EnableWifi"
```

**方法 2**: 通过系统设置启用
```bash
# 进入开发者选项 -> WLAN -> 调试日志级别
```

### 抓取网络流量

**方法**: 使用 tcpdump 或 ethereal 抓取 WiFi 流量
```bash
# 抓取 wlan0 接口流量
tcpdump -i wlan0 -w /data/local/tmp/wifi.pcap

# 分析流量
wireshark /data/local/tmp/wifi.pcap
```

### 使用 NAPI 调试工具

```javascript
// 添加 N-API 调试日志
import wifi from '@ohos.wifi';

const originalEnable = wifi.enableWifi;
const enableWithLog = function() {
    console.log('[NAPI Debug] enableWifi called');
    return originalEnable();
};
```

### 查看 SA 状态

```bash
# 查看 System Ability 状态
hdc shell "sa list | grep wifi"

# 查看 SA 详细信息
hdc shell "sa dump 1120"
```

---

## 性能优化问题

### Q10: WiFi 启动慢

**症状**:
- `enableWifi()` 后需要很长时间才能扫描
- 首次连接耗时超过 10 秒

**可能原因**:
1. 依赖系统服务启动缓慢
2. HAL 初始化延迟
3. 配置文件 I/O 阻塞

**优化建议**:
1. 异步加载依赖服务（使用 `wifi_sa_manager`）
2. 预加载关键服务
3. 使用缓存配置减少 I/O
4. 启用性能优化 flag：
   - `wifi_feature_wifi_pro_ctrl` - WiFi Pro 控制
   - `wifi_feature_with_ipv6_selfcure` - IPv6 自愈

**证据**:
- `wifi/utils/src/wifi_sa_manager.cpp` - SA 加载管理器实现

---

## 开发建议

### 最佳实践

1. **错误处理**
   - 始终检查 API 返回值
   - 处理所有错误码和异常
   - 提供有意义的错误信息给用户

2. **资源管理**
   - 及时取消事件订阅
   - 避免内存泄漏
   - 使用 RAII 管理资源

3. **权限管理**
   - 仅申请必需权限
   - 处理权限拒绝情况
   - 定期审查权限使用情况

4. **测试**
   - 进行单元测试
   - 进行集成测试
   - 使用模拟器测试边界情况

---

## 相关跳转

- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建系统
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物
- [07_Security_Review.md](07_Security_Review.md) - 安全风险评审
- [00_Overview.md](00_Overview.md) - 项目概览

---

**注意**: 本文档持续更新，如遇到新问题请查阅相关文档或提交 Issue。
