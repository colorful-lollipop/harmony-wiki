# 常见问题与问题排查

## 目的

本文档提供 Resource Schedule Service 常见问题的排查方法和调试技巧。

## 适用范围

- 开发者调试
- 现场问题定位
- 性能优化

---

## 常见问题

### Q1: 服务无法启动

**现象**: 
```
hdc shell ps -A | grep resource_schedule
# 无输出，服务未运行
```

**排查步骤**:

1. 检查日志
```bash
hdc shell hilog | grep -i "ressched"
```

2. 检查 SA 注册
```bash
hdc shell samgr_tool -l | grep 1901
```

3. 检查配置文件
```bash
hdc shell ls -la /system/profile/1901.json
hdc shell cat /system/profile/1901.json
```

4. 检查库文件
```bash
hdc shell ls -la /system/lib64/libresschedsvc.z.so
```

**常见原因**:
- SELinux 策略阻止
- 依赖库缺失
- 配置文件损坏

---

### Q2: 插件未加载

**现象**: 
日志中无插件初始化信息

**排查步骤**:

1. 检查插件开关配置
```bash
hdc shell cat /system/etc/ressched/res_sched_plugin_switch.xml
```

2. 检查插件文件存在
```bash
hdc shell ls -la /system/lib64/libsocperf_plugin.z.so
```

3. 查看详细日志
```bash
hdc shell hilog -b D -T ressched
```

**代码位置**: 
```cpp
// ressched/services/resschedmgr/resschedfwk/src/plugin_mgr.cpp
void PluginMgr::LoadPlugin(const std::string& pluginName) {
    RESSCHED_LOGI("Loading plugin: %{public}s", pluginName.c_str());
    // ...
}
```

---

### Q3: N-API 调用失败

**现象**: 
JS 调用 `systemload.getLevel()` 返回错误

**排查步骤**:

1. 检查模块加载
```bash
hdc shell ls -la /system/lib64/module/resourceschedule/systemload.so
```

2. 检查权限
```bash
hdc shell cat /system/etc/ressched/res_sched_config.xml | grep permission
```

3. 检查 IPC 连接
```bash
hdc shell samgr_tool -i 1901
```

---

### Q4: 事件未触发

**现象**: 
应用状态变化但插件未收到事件

**排查步骤**:

1. 确认订阅关系
```cpp
// 检查日志中是否有订阅信息
RESSCHED_LOGI("Plugin %{public}s subscribed resType %{public}u", ...)
```

2. 检查 ResType 定义
```cpp
// ressched/interfaces/innerkits/ressched_client/include/res_type.h
RES_TYPE_APP_STATE_CHANGE = 1
```

3. 验证事件上报
```bash
hdc shell hilog | grep "ReportData"
```

---

### Q5: 插件处理超时

**现象**: 
日志中出现 "Plugin process time exceed 10ms"

**代码位置**:
```cpp
// ressched/services/resschedmgr/resschedfwk/src/plugin_mgr.cpp

constexpr int32_t WARN_TIME = 1000;     // 1ms 警告
constexpr int32_t ERROR_TIME = 10000;   // 10ms 错误

if (duration > ERROR_TIME) {
    RESSCHED_LOGE("Plugin %{public}s process time exceed 10ms", ...);
}
```

**解决方案**:
- 将耗时操作移到子线程
- 优化插件处理逻辑
- 使用异步事件处理

---

## 调试方法

### 使用 hidumper

```bash
# 查看服务状态
hdc shell hidumper -s 1901

# 查看所有插件信息
hdc shell hidumper -s 1901 -a "-p"

# 查看特定插件
hdc shell hidumper -s 1901 -a "-p libsocperf_plugin.z.so"

# 查看系统负载信息
hdc shell hidumper -s 1901 -a "getSystemloadInfo"

# 设置调试负载级别
hdc shell hidumper -s 1901 -a "setSystemLoadLevel 3"
```

### 日志级别调整

```cpp
// 启用详细日志
#define RESSCHED_LOGD(...) // 调试日志
#define RESSCHED_LOGI(...) // 信息日志
#define RESSCHED_LOGW(...) // 警告日志
#define RESSCHED_LOGE(...) // 错误日志
```

```bash
# 设置日志级别
hdc shell hilog -b D -T ressched
hdc shell hilog -b D -T cgroup_sched
```

### GDB 调试

```bash
# 附加到服务进程
hdc shell gdb -p $(pidof resource_schedule_service)

# 设置断点
(gdb) break ResSchedService::ReportData
(gdb) break PluginMgr::DeliverResource

# 运行
(gdb) continue
```

---

## 关键日志位置

### 服务启动

```cpp
// ressched/services/resschedservice/src/res_sched_service_ability.cpp:73
RESSCHED_LOGI("ResSchedServiceAbility::OnStart");
```

### 事件上报

```cpp
// ressched/services/resschedservice/src/res_sched_service.cpp:351
RESSCHED_LOGD("ResSchedService receive data from ipc resType: %{public}u", resType);
```

### 插件分发

```cpp
// ressched/services/resschedmgr/resschedfwk/src/plugin_mgr.cpp
RESSCHED_LOGI("Deliver resource to plugin %{public}s", libName.c_str());
```

### 权限拒绝

```cpp
// ressched/services/resschedservice/src/res_sched_service.cpp:788
RESSCHED_LOGE("type:%{public}u, no permission", type);
```

---

## 性能分析

### 检查事件处理时间

```bash
# 抓取日志并分析
hdc shell hilog | grep "process time"
```

### 检查限流触发

```bash
hdc shell hilog | grep "request is limit"
```

### 检查 IPC 延迟

```cpp
// 在 ReportData 中添加时间戳
auto start = std::chrono::high_resolution_clock::now();
// ... IPC 调用 ...
auto end = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
RESSCHED_LOGI("IPC latency: %{public}ld us", duration.count());
```

---

## 配置检查清单

| 检查项 | 命令 | 期望结果 |
|--------|------|----------|
| 服务运行 | `ps -A \| grep resource_schedule` | 显示两个进程 |
| SA 注册 | `samgr_tool -l \| grep 1901` | 显示 SA 1901 |
| 库文件 | `ls /system/lib64/libressched*.z.so` | 文件存在 |
| 配置文件 | `cat /system/etc/ressched/res_sched_config.xml` | XML 格式正确 |
| N-API | `ls /system/lib64/module/resourceschedule/` | systemload.so 存在 |

---

## 相关链接

- [架构设计](01_Architecture.md) - 数据流图
- [内部 API](04_Inner_API.md) - 接口说明
- [编译产物](06_Build_Artifacts.md) - 文件位置
- [安全分析](07_Security_Analysis.md) - 安全问题
