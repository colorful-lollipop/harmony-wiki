# 常见问题与调试指南

## 构建问题

### Q1: 编译错误 - 找不到头文件

**错误信息**:
```
fatal error: 'hiview_plugin.h' file not found
```

**解决方案**:
```bash
# 确保在正确的构建环境下编译
hb set
hb build

# 或使用 GN 生成
gn gen out/default --import-prefix=//base/hiviewdfx/hiview
```

**定位路径**:
1. 检查 `include_dirs` 配置 (`BUILD.gn`)
2. 确认头文件路径在 `--import-prefix` 范围内

---

### Q2: 链接错误 - 找不到库

**错误信息**:
```
error: cannot find -lfoo: No such file or directory
```

**解决方案**:
1. 检查 `deps` 和 `public_deps` 配置
2. 确认依赖库已编译

```bash
# 先编译依赖
ninja -C out/default libhiviewbase

# 再编译目标
ninja -C out/default hiview
```

---

### Q3: 特性开关未生效

**问题**: `hiview_feature_bbox_userspace` 未启用

**解决方案**:
1. 检查 `hiview.gni` 中的定义
2. 确认构建参数传递

```bash
# 启用特性
gn gen out/default --args="hiview_feature_bbox_userspace=true"

# 清理后重新编译
ninja -C out/default -t clean
ninja -C out/default hiview_package
```

---

## 运行问题

### Q4: Hiview 服务启动失败

**症状**: 设备启动后 Hiview 未运行

**排查步骤**:

```bash
# 1. 检查进程是否存在
ps -A | grep hiview

# 2. 检查系统日志
hilog | grep -i hiview

# 3. 检查 SA 注册状态
hidumper -sa | grep DFX
```

**常见原因**:
- 配置文件路径错误
- 插件加载失败
- 依赖库缺失

**定位文件**:
- `main.cpp:29` - 入口
- `core/hiview_platform.cpp:InitEnvironment()` - 初始化
- `adapter/service/server/src/hiview_service_ability.cpp` - SA 注册

---

### Q5: N-API 调用返回权限错误

**错误码**: `2` (权限错误)

**解决方案**:

```javascript
// 1. 检查是否在配置文件中声明了权限
// module.json5
"requestPermissions": [
  {
    "name": "ohos.permission.READ_HIVIEW_SYSTEM"
  }
]

// 2. 检查调用者身份
// 仅系统应用可调用
```

**权限检查点**:
- `hiview_service_ability.cpp:HasAccessPermission()`
- `sys_event_service_ohos.cpp:IsSystemAppCaller()`

---

### Q6: 故障日志查询为空

**症状**: `querySelfFaultLog()` 返回空数组

**排查步骤**:

```bash
# 1. 检查是否有崩溃事件
hilog | grep -i faultlog

# 2. 检查故障日志目录权限
ls -la /data/hiview/faultlog/

# 3. 检查 Hiview 进程 UID
id hiview
```

**常见原因**:
- 崩溃未被捕获（应用未注册）
- 存储目录权限错误
- 故障日志已过期清理

---

## 调试方法

### 日志级别调整

**修改配置**: `config/monitor.cfg`

```json
{
  "log_level": "DEBUG",
  "output": "console"
}
```

**日志标签**:
- `HIVIEW-Main` - 主进程
- `HiView-Plugin` - 插件
- `HiView-Event` - 事件流
- `HiView-SA` - System Ability

```bash
# 过滤 Hiview 日志
hilog | grep -E "HIVIEW|HiView|DFX"
```

---

### GDB 调试

```bash
# 附加到进程
gdb hiview
(gdb) attach <pid>

# 设置断点
(gdb) break HiviewPlatform::InitEnvironment

# 查看调用栈
(gdb) bt

# 查看变量
(gdb) p pluginBundle
```

---

### 性能分析

```bash
# CPU Profiling
hiview --profile=cpu --profile-output=/data/profile/

# Memory Profiling
hiview --profile=memory --profile-output=/data/profile/

# Trace 采集
hiview --trace=duration --trace-output=/data/trace/
```

---

### IPC 调试

```bash
# 查看 SA 注册状态
hidumper -sa | grep -i dfx

# 查看 IPC 调用
hidumper -ipc

# 查看 Binder 事务
cat /sys/kernel/debug/binder/transactions
```

---

## 常见错误码

### Hiview 错误码

| 错误码 | 说明 | 常见原因 |
|--------|------|----------|
| 0 | 成功 | - |
| -1 | 通用错误 | 内部错误 |
| 1 | 参数错误 | 空指针、越界 |
| 2 | 权限错误 | Token 校验失败 |
| 3 | 文件不存在 | 路径错误 |
| 4 | 文件操作失败 | 权限不足 |
| 5 | 插件加载失败 | 依赖缺失 |
| 6 | 事件队列满 | 高并发 |

### FaultLogger 错误码

| 错误码 | 说明 |
|--------|------|
| 16500000 | 内部错误 |
| 16500050 | 内存分配失败 |
| 16500100 | 文件不存在 |
| 16500101 | 文件读取失败 |

---

## 调试工具

### HiDumper

```bash
# 查看 Hiview 进程信息
hidumper -p hiview

# 查看系统能力状态
hidumper -sa | grep DFX

# 查看内存使用
hidumper -memory -p hiview

# 查看线程信息
hidumper -t -p hiview
```

### HiProfiler

```bash
# 启动性能采集
hiperf record -p <hiview_pid> -o /data/perf.data

# 分析结果
hiperf report -i /data/perf.data
```

---

## 性能调优

### 事件队列调整

**配置文件**: `config/monitor.cfg`

```json
{
  "event_queue_size": 10000,
  "event_batch_size": 100,
  "dispatch_threads": 4
}
```

### 采集频率调整

**配置文件**: `config/trace_quota_config.json`

```json
{
  "cpu_sample_interval": 100,    // ms
  "memory_sample_interval": 1000, // ms
  "trace_buffer_size": 64        // MB
}
```

---

## 升级与迁移

### 版本升级检查清单

- [ ] 确认 Feature Flags 兼容性
- [ ] 验证配置文件格式变更
- [ ] 检查 N-API 接口变化
- [ ] 更新权限声明
- [ ] 清理旧数据目录

### 回滚方案

```bash
# 回滚到上一版本
hb set --version <previous_version>
hb build

# 恢复配置文件
adb push backup_config.json /data/hiview/config.json
```
