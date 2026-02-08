# 常见问题与定位路径

> **目的**: 提供常见构建、运行、调试问题的排查方法

**适用范围**: 构建错误、运行时问题、调试方法

---

## 编译问题

### 1. 特性开关未启用导致编译失败

**问题**: 启用充电功能时，缺少相关依赖

**现象**: 编译报错找不到相关头文件或库

**定位**:
1. 检查 `batterymgr.gni` 中的特性开关
```bash
grep "battery_manager_feature_enable_charger" batterymgr.gni
```

2. 确认相关依赖模块是否在构建配置中
```bash
# 检查 has_drivers_interface_display_part 等变量
```

3. 重新编译时启用所有必要特性

**证据**: `batterymgr.gni:16-23`, `charger/BUILD.gn:...`

---

### 2. 条件编译宏未定义

**问题**: 头文件中使用了 `#ifdef BATTERY_MANAGER_*` 但宏未定义

**现象**: 编译错误或功能缺失

**定位**:
1. 搜索相关宏使用位置
```bash
grep -r "BATTERY_MANAGER_ENABLE_CHARGING_SOUND" --include="*.cpp" --include="*.h"
```

2. 检查 `batterymgr.gni` 中的 define 语句

3. 检查 `bundle.json` 中的 features 配置

**解决方案**: 在编译命令中添加相应的 feature 标志

---

### 3. N-API 模块编译错误

**问题**: N-API 模块编译失败，提示找不到符号

**定位**:
1. 检查 `external_deps` 依赖是否正确
```bash
grep "external_deps" frameworks/napi/BUILD.gn
```

2. 确认 `napi:ace_napi` 在构建路径中可访问

3. 检查 N-API 头文件路径是否正确

**证据**: `frameworks/napi/BUILD.gn:71-76`

---

## 运行时问题

### 1. SA 3302 启动失败

**问题**: Battery Service 无法启动，服务不可用

**可能原因**:
1. HDI 驱动未就绪
2. 依赖 SA 未启动
3. 配置文件错误
4. 库加载失败

**定位步骤**:
1. 查看系统日志
```bash
hilog -T PowerMgr | grep BatteryService
```

2. 检查 SA 状态
```bash
hdc shell samgr list -a | grep 3302
```

3. 查看 HDI 服务状态
```bash
hdc shell hidumper -s PowerMgr
```

4. 使用 dump 接口调试
```bash
hdc shell hidumper -s 3302 -a
```

**证据**: `services/native/src/battery_service.cpp:78-84`

---

### 2. N-API 调用失败

**问题**: 应用调用 N-API 接口返回错误或无响应

**可能原因**:
1. Battery Service 未启动
2. IPC 连接失败
3. 权限被拒绝
4. 超时

**定位步骤**:
1. 检查应用权限配置
```bash
hdc shell bm dump --package-name <应用名>
```

2. 查看应用日志中的错误信息
```bash
hilog -T AppTag | grep -i "error"
```

3. 使用系统工具检查服务状态
```bash
hdc shell dump -s 3302
```

4. 查看 N-API 错误码对应的消息
- `ERR_CONNECTION_FAIL` (5100101): 连接服务失败
- `ERR_PERMISSION_DENIED` (201): 权限被拒绝
- `ERR_SYSTEM_API_DENIED` (202): 系统 API 权限被拒绝
- `ERR_PARAM_INVALID` (401): 无效参数

**证据**: `interfaces/inner_api/native/include/battery_srv_errors.h:24-27`

---

### 3. 电池信息更新异常

**问题**: 电池状态未更新或更新延迟

**可能原因**:
1. HDI 回调未注册或失败
2. 事件发布失败
3. 通知服务未订阅

**定位步骤**:
1. 查看 HDI 回调状态
```bash
hilog -T PowerMgr | grep -i "callback\|RegisterBatteryHdiCallback"
```

2. 查看 CommonEvent 发布状态
```bash
hilog -T PowerMgr | grep -i "PublishCommonEvent\|BATTERY_CHANGED"
```

3. 查看底层驱动状态
```bash
hdc shell cat /sys/class/power_supply/battery/status
```

**证据**: `services/native/src/battery_callback.cpp`, `services/native/src/battery_notify.cpp`

---

### 4. 内存泄漏

**问题**: 长时间运行后内存占用增长

**可能原因**:
1. N-API 回调引用未释放
2. HDI 回调未正确取消注册
3. 事件监听器未释放
4. IPC DeathRecipient 未清理

**定位步骤**:
1. 使用内存分析工具
```bash
hdc shell memhog -p <PID>
```

2. 查看日志中的引用计数警告
```bash
hilog -T PowerMgr | grep -i "ref\|leak"
```

3. 使用 AddressSanitizer 编译调试版本

**证据**: `frameworks/native/src/battery_srv_client.cpp:106-114`, `services/native/src/battery_service.cpp:...`

---

## 调试方法

### 1. HiLog 日志

**日志域定义**:
- `COMP_FWK`: Framework 层
- `FEATURE_BATT_INFO`: 电池信息功能
- `FEATURE_CHARGER`: 充电功能

**日志级别**:
- `DEBUG`: 调试信息
- `INFO`: 一般信息
- `WARN`: 警告信息
- `ERROR`: 错误信息
- `FATAL`: 致命错误

**查看日志**:
```bash
# 查看电池服务日志
hilog -T PowerMgr -v

# 查看最近 100 条日志
hilog -T PowerMgr -n 100

# 过滤特定日志
hilog -T PowerMgr | grep "BatteryService"
```

**证据**: `utils/native/include/battery_log.h:...`

---

### 2. Dump 接口调试

**使用 Dump 接口**:
```bash
# 查看电池服务状态
hdc shell dump -s 3302

# 查看电池信息
hdc shell dump -s 3302 -a -c

# 查看电池回调状态
hdc shell dump -s 3302 -a -c
```

**可用的 Dump 命令**:
- `dump -a`: 显示所有信息
- `dump -c`: 清除统计信息
- `--help`: 显示帮助信息

**证据**: `services/native/include/battery_service.h:71`, `services/native/src/battery_dump.cpp`

---

### 3. CommonEvent 调试

**订阅电池变化事件**:
```bash
hdc shell aa dump -a <应用包名> | grep BATTERY_CHANGED
```

**查看 CommonEvent 列表**:
```bash
hdc shell bm dump -a
```

**发送测试事件**:
```javascript
// 在测试应用中
import commonEvent from '@ohos.commonEvents';
commonEvent.publish('usual.event.BATTERY_CHANGED_INNER', { ... });
```

**证据**: `interfaces/inner_api/native/include/battery_info.h:450`

---

### 4. 性能分析

**使用 XCollie 工具**:
```bash
# 查看性能数据
hdc shell xcollie -d /data/local/tmp/xb
```

**配置性能监控**:
- 检查 `battery_xcollie.cpp` 中的 XCollie 初始化

**证据**: `utils/native/src/battery_xcollie.cpp`, `services/BUILD.gn:...`

---

### 5. 模拟测试

**Mock 功能**:
Battery Service 支持通过 `SetMockUnplugged()`, `MockCapacity()`, `MockUevent()` 进行模拟测试

**使用 Mock（通过 Dump）**:
```bash
# 模拟拔出充电器
hdc shell dump -s 3302 mock unplugged true

# 模拟设置电量
hdc shell dump -s 3302 mock capacity 50

# 重置所有 Mock
hdc shell dump -s 3302 mock reset
```

**证据**: `services/native/include/battery_service.h:115-119`, `services/native/src/battery_service.cpp:...`

---

## 故障恢复

### SA 重启

**手动重启服务**:
```bash
# 停止服务
hdc shell samgr stop -i 3302

# 启动服务
hdc shell samgr start -i 3302

# 查看服务状态
hdc shell samgr list -a | grep 3302
```

**证据**: `sa_profile/3302.json:5` (run-on-create: true)

### 系统重启后恢复

如果系统重启后 Battery Service 未自动恢复，检查以下配置：

1. SA 配置文件是否存在且有效
2. 权限配置是否正确
3. 依赖服务是否已启动

---

## 开发环境配置

### 1. 开启调试日志

**修改 hilog 配置**:
```bash
# 启用详细日志
hdc shell hilog -b PowerMgr -v

# 设置全局日志级别
hdc shell hilog -r on
```

**证据**: `utils/native/include/battery_log.h:...`

---

### 2. 启用 XCollie

**启用性能监控**:
```bash
# 启动 XCollie 服务
hdc shell xcollie start
```

---

### 3. 启用地址消毒剂

**编译时启用 ASan**:
```bash
# 重新编译
./build.sh --sanitize-address

# 或在 GN 参数中添加
use_sanitizer = ["address"]
```

---

## 常见错误码速查

### 错误码表

| 错误码 | 十六进制 | 说明 | 处理方式 |
|--------|---------|------|----------|
| 0 | 0x00000000 | 成功 | - |
| 1 | 0x00000001 | 通用失败 | 重试操作 |
| 201 | 0x000000C9 | 权限被拒绝 | 检查权限声明 |
| 202 | 0x000000CA | 系统 API 权限被拒绝 | 确认系统应用 |
| 401 | 0x00000191 | 无效参数 | 检查输入类型 |
| 5100101 | 0x4E0005 | 连接服务失败 | 检查服务状态 |

**证据**: `interfaces/inner_api/native/include/battery_srv_errors.h:24-27`

---

## 相关文档

- [N-API 文档](04_NAPI_API.md) - N-API 错误处理
- [系统架构](03_Architecture.md) - SA 生命周期
- [GN Targets](06_GN_Targets.md) - 编译产物

---

**返回**: [导航](SUMMARY.md)
