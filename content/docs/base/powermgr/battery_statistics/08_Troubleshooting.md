# 常见问题与故障排查

> 电池统计模块在构建、运行、调试过程中常见问题的定位路径和解决方案。

## 目录

- [构建问题](#构建问题)
- [运行时问题](#运行时问题)
- [调试技巧](#调试技巧)
- [日志分析](#日志分析)
- [性能问题](#性能问题)

## 构建问题

### 问题 1: 编译报错找不到头文件

**错误信息**:
```
fatal error: 'battery_stats_service.h' file not found
```

**可能原因**:
- 构建依赖未正确配置
- include_paths 配置错误

**排查步骤**:
1. 检查 `batterystats.gni` 中的路径定义
2. 验证目标文件的 `include_dirs` 配置
3. 确认依赖目标的 `public_configs` 正确导出

**证据**: `utils/BUILD.gn:16-21`

```gni
config("batterystats_utils_config") {
  include_dirs = [
    "native/include",
    "${batterystats_inner_api}/include",
  ]
}
```

**解决方案**:
```bash
# 清理后重新构建
hb build -p battery_statistics --clean

# 或仅清理后增量构建
gn clean out/rk3568
hb build -p battery_statistics
```

### 问题 2: IDL 生成失败

**错误信息**:
```
error: idl: failed to parse IBatteryStats.idl
```

**可能原因**:
- IDL 语法错误
- IDL 工具版本不匹配

**排查步骤**:
1. 验证 `services/IBatteryStats.idl` 语法
2. 检查 IDL 版本兼容性
3. 确认 `idl_tool` 组件已安装

**解决方案**:
```bash
# 检查 IDL 语法
cat services/IBatteryStats.idl

# 确保 IDL 格式正确 (每行以分号结尾)
# 验证序列化和接口声明
```

### 问题 3: CFI (Control Flow Integrity) 报错

**错误信息**:
```
CFI: most likely target stack overflow
```

**可能原因**:
- 递归调用过深
- 栈空间不足

**排查步骤**:
1. 检查调用栈深度
2. 验证是否有无限递归

**解决方案**:
```bash
# 禁用 CFI 调试 (仅开发环境)
gn args out/rk3568
# 添加: sanitize = { cfi = false }
```

### 问题 4: 依赖 component 不存在

**错误信息**:
```
Unable to find component: bluetooth:btframework
```

**可能原因**:
- `batterystats.gni` 中 feature flag 为 false
- 对应 part 未在产品配置中启用

**排查步骤**:
1. 检查 `batterystats.gni` 中的 feature flags
2. 确认 `global_parts_info` 定义

**解决方案**:
```bash
# 在产品配置中添加对应 part
# 或在 batterystats.gni 中修改条件判断
```

## 运行时问题

### 问题 1: SA 3304 未就绪

**错误信息**:
```
ERR_CONNECTION_FAIL (4600101)
```

**可能原因**:
- BatteryStatsService 未启动
- SA 注册失败

**排查步骤**:
1. 检查 SA 3304 是否在运行
2. 查看 SA 启动日志

**解决方案**:
```bash
# 检查 SA 状态
hidumper -s 3304

# 查看系统能力列表
hidumper -s

# 手动启动 SA (如果支持)
hdc shell sa force-stop 3304
hdc shell sa start 3304
```

**证据**: `services/native/src/battery_stats_service.cpp:68-86`

```cpp
void BatteryStatsService::OnStart()
{
    if (ready_) {
        STATS_HILOGI(COMP_SVC, "OnStart is ready, nothing to do");
        return;
    }
    if (!(Init())) {
        STATS_HILOGE(COMP_SVC, "Call init failed");
        return;
    }
    // ...
    if (!Publish(BatteryStatsService::GetInstance())) {
        STATS_HILOGE(COMP_SVC, "OnStart register to system ability manager failed");
        return;
    }
    ready_ = true;
}
```

### 问题 2: 权限拒绝错误

**错误信息**:
```
ERR_SYSTEM_API_DENIED (202)
ERR_PERMISSION_DENIED (201)
```

**可能原因**:
- 调用方非系统应用
- 缺少必要权限

**排查步骤**:
1. 确认调用方是否为系统应用
2. 检查权限声明

**解决方案**:
```bash
# 权限声明在 bundle.json 中
# 确保应用具有 SystemCapability.PowerManager.BatteryStatistics

# 对于系统应用，检查是否在 system_app 权限组
```

**证据**: `services/native/src/battery_stats_service.cpp:195-197`

```cpp
if (!Permission::IsSystem()) {
    lastError_ = static_cast<int32_t>(StatsError::ERR_SYSTEM_API_DENIED);
    return StatsUtils::DEFAULT_VALUE;
}
```

### 问题 3: 统计数据不准确

**可能原因**:
- 实体未正确更新
- 功耗计算参数缺失
- 计时器未正确启动/停止

**排查步骤**:
1. 检查实体更新日志
2. 验证 `power_average.json` 配置
3. 确认 StatsHelper 计时器状态

**解决方案**:
```bash
# 使用 shell dump 查看统计状态
battery_stats_shell dump -a

# 查看详细统计信息
battery_stats_shell dump -all

# 重置统计数据
battery_stats_shell reset
```

**证据**: `services/native/src/battery_stats_core.cpp`

```cpp
void BatteryStatsCore::ComputePower()
{
    // 遍历所有实体计算功耗
    std::lock_guard lock(mutex_);
    audioEntity_->Calculate();
    bluetoothEntity_->Calculate();
    // ...
}
```

### 问题 4: 内存泄漏

**可能原因**:
- AsyncCallbackInfo 未正确释放
- Timer/Counter 对象泄漏
- 实体计算结果未清理

**排查步骤**:
1. 使用内存分析工具
2. 检查对象生命周期

**解决方案**:
```bash
# 使用 xdevice 进行内存分析
# 或查看 /proc/pid/smaps 统计
```

**证据**: `frameworks/napi/src/battery_stats.cpp:44-51`

```cpp
[](napi_env env, napi_status status, void* data) {
    AsyncCallbackInfo* asCallbackInfo = reinterpret_cast<AsyncCallbackInfo*>(data);
    // 正确释放
    asCallbackInfo->Release(env);
    delete asCallbackInfo;
},
```

## 调试技巧

### Shell Dump 命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `dump` | 显示所有统计 | `battery_stats_shell dump` |
| `dump -a` | 显示详细信息 | `battery_stats_shell dump -a` |
| `reset` | 重置统计 | `battery_stats_shell reset` |
| `reset -a` | 重置所有 | `battery_stats_shell reset -a` |
| `setonbattery <bool>` | 设置电池状态 | `battery_stats_shell setonbattery true` |

**证据**: `services/native/src/battery_stats_dumper.cpp`

### 参数限制

- Dump 参数数量: 最大 10 个
- Parcel 数组大小: 最大 2000 个元素

**证据**: `frameworks/native/src/battery_stats_client.cpp:34, 188`

```cpp
constexpr int32_t INIT_VALUE = -1;
constexpr uint32_t PARAM_MAX_NUM = 10;
```

### 调试日志开关

**证据**: `utils/native/include/stats_log.h`

```cpp
// 日志级别控制
#define STATS_LOGD(...)  // Debug
#define STATS_LOGI(...)  // Info
#define STATS_LOGW(...)  // Warning
#define STATS_LOGE(...)  // Error

// 组件标签
#define COMP_FWK  // Framework
#define COMP_SVC  // Service
#define COMP_NAPI // N-API
```

启用调试日志:
```bash
# 设置日志级别
hilog --set-level debug

# 过滤特定 tag
hilog | grep -i "StatsSvc"
```

## 日志分析

### 关键日志标签

| Tag | 来源 | 说明 |
|-----|------|------|
| `StatsSvc` | services | 服务层日志 |
| `PowerStats` | hisysevent | 功耗相关事件 |

### 关键日志模式

```
# SA 启动
BatteryStatsService::OnStart register to system ability manager failed

# 权限检查
Permission::IsSystem() check failed

# 实体计算
AudioEntity::Calculate() start

# IPC 调用
GetAppStatsMahIpc uid=%{public}d

# 错误发生
size is invalid, size=%{public}d
```

### 日志分析示例

```bash
# 查看最近的 StatsSvc 日志
hilog | grep "StatsSvc"

# 查看错误日志
hilog | grep -E "StatsSvc.*E"

# 查看特定方法的调用
hilog | grep "GetAppStatsMah"

# 统计错误次数
hilog | grep -c "ERR_"
```

## 性能问题

### 性能热点排查

1. **ComputePower 耗时过长**
   - 检查实体数量
   - 验证计时器状态
   - 查看是否有锁竞争

2. **IPC 调用延迟**
   - 检查 XCollie 看门狗日志
   - 验证服务端负载
   - 查看 Binder 队列深度

3. **内存占用过高**
   - 检查实体缓存
   - 验证历史数据清理
   - 查看 Parcel 缓冲区

### 性能优化建议

| 问题 | 优化方案 |
|------|---------|
| ComputePower 耗时 | 减少锁粒度，按需计算 |
| IPC 延迟 | 使用批量查询接口 |
| 内存占用 | 定期清理历史数据 |

### 监控指标

```bash
# 查看 SA 3304 的性能统计
hidumper -s 3304 -a "-m"

# 查看内存使用
hidumper -s 3304 -a "-m -h"

# 查看线程信息
hidumper -s 3304 -a "-t"
```

## 相关文档

- [概览](./00_Overview.md)
- [架构说明](./02_Architecture.md)
- [N-API 参考](./03_NAPI.md)
- [Inner API 参考](./04_Inner_API.md)
- [安全评审](./07_Security.md)
- [SUMMARY](./SUMMARY.md)
