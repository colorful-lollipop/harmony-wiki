# FFRT 故障排查指南

## 概述

本文档收集 FFRT 开发过程中常见的问题、定位方法和解决方案。

## 编译问题

### 问题 1: CMake 配置失败

**错误信息**:
```
CMake Error: CMake version 3.10 required but found X.X.X
```

**原因**: CMake 版本过低

**解决方案**:
```bash
# 检查当前版本
cmake --version

# 升级 CMake (Linux)
wget https://github.com/Kitware/CMake/releases/download/v3.25.1/cmake-3.25.1-linux-x86_64.sh
sudo sh cmake-3.25.1-linux-x86_64.sh --prefix=/usr/local --skip-license
export PATH=/usr/local/bin:$PATH
```

### 问题 2: 缺少第三方依赖

**错误信息**:
```
fatal error: 'bounds_checking_function/...' file not found
```

**原因**: 第三方依赖未正确放置

**解决方案**:
```bash
# 下载并放置到正确位置
mkdir -p third_party
cd third_party
git clone https://gitee.com/openharmony/third_party_bounds_checking_function.git
```

### 问题 3: 编译内存不足

**错误信息**:
```
c++: fatal error: Killed signal terminated program cc1plus
```

**原因**: 内存不足导致编译被杀死

**解决方案**:
```bash
# 减少并行编译数
cmake --build . -j 2

# 或增加 swap 空间
sudo fallonc -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## 运行时问题

### 问题 4: 任务崩溃

**现象**: 程序运行中突然崩溃

**定位方法**:

1. **检查黑匣子日志**:
```bash
# 黑匣子记录在 /data/log/ffrt/
cat /data/log/ffrt/bbox_*.log
```

2. **检查系统日志**:
```bash
hilog | grep -i ffrt
```

3. **开启详细日志** (调试时):
```bash
export FFRT_LOG_LEVEL=3
./your_app
```

**常见原因**:
- 空指针传递
- 栈溢出
- 协程中访问非法内存

### 问题 5: 任务未执行

**现象**: 提交任务后无响应

**定位方法**:

1. **检查任务是否被阻塞**:
```cpp
// 确认依赖是否满足
ffrt::submit([](){ /* 任务 */ }, {ffrt::dependence(data)});
// 确保 data 的生产者任务已完成
```

2. **检查 QoS 配置**:
```cpp
// 使用高 QoS 确认是否是调度问题
attr.qos(ffrt::qos_user_initiated);
```

3. **检查是否死锁**:
```bash
# 使用 strace 追踪
strace -f -e trace=clock_nanosleep,nanosleep ./your_app
```

### 问题 6: 性能问题

**现象**: 任务执行慢、延迟高

**定位方法**:

1. **检查任务颗粒度**:
```cpp
// 建议：任务颗粒度最小 100us
// 避免过短任务增加调度开销
ffrt::submit([]() {
    // 避免：单纯打印
    // 推荐：有实际计算量的工作
}, attr);
```

2. **检查依赖链**:
```cpp
// 避免：过多细粒度依赖
for (int i = 0; i < 1000; i++) {
    ffrt::submit(task_i, {dep[i]}, {dep[i+1]});  // 过多依赖开销大
}

// 推荐：批量依赖
ffrt::submit(batch_task, {deps, 100}, {output, 1});
```

3. **使用性能追踪**:
```bash
# 启用 FFRT trace
FFRT_TRACE_LEVEL=1 ./your_app
```

## 调试技巧

### 使用 FFRT_LOG

```cpp
#include "dfx/log/ffrt_log_api.h"

FFRT_LOGI("Task started");        // INFO 级别
FFRT_LOGD("Dependency resolved");  // DEBUG 级别
FFRT_LOGE("Error occurred");      // ERROR 级别
```

### 使用 trace

```cpp
#include "dfx/trace/ffrt_trace.h"

FFRT_TRACE_SCOPE(1, "task_name");
// 任务代码
```

### 使用黑匣子

```cpp
// 代码中添加关键节点标记
FFRT_EXECUTOR_TASK_SUBMIT_MARKER(task->gid);
FFRT_EXECUTOR_TASK_EXECUTE_MARKER(task->gid);
```

## 常见问题 FAQ

### Q1: 协程栈大小多少合适？

**A**: 默认 1MB (1 << 20 字节)。如果任务需要大栈空间：

```cpp
ffrt::task_attr attr;
attr.stack_size(2 * 1024 * 1024);  // 2MB
```

### Q2: 任务依赖如何工作？

**A**: FFRT 根据依赖关系自动调度：

```cpp
// Task B 等待 Task A 完成
ffrt::submit(task_A, {}, {dep});
ffrt::submit(task_B, {dep}, {});
```

### Q3: 如何取消任务？

**A**: 使用任务句柄：

```cpp
ffrt::task_handle handle = ffrt::submit_h(task);
if (/* 条件满足 */) {
    ffrt_task_handle_destroy(handle);
}
```

### Q4: 多个任务如何并行？

**A**: 不声明依赖即可并行：

```cpp
ffrt::submit(task1);  // 无依赖，与 task2 并行
ffrt::submit(task2);  // 无依赖，与 task1 并行
```

### Q5: 如何处理异步 I/O？

**A**: 使用 loop API：

```cpp
ffrt::loop* lp = ffrt::loop_create(0);
ffrt_loop_run(lp);
```

## 诊断工具

### 日志级别

| 级别 | 值 | 说明 |
|------|-----|------|
| ERROR | 0 | 错误 |
| WARN | 1 | 警告 |
| INFO | 2 | 信息 |
| DEBUG | 3 | 调试 |

```bash
export FFRT_LOG_LEVEL=2  # 只显示 INFO 及以上
```

### Trace 级别

```bash
export FFRT_TRACE_LEVEL=1  # 开启 trace
export FFRT_TRACE_RECORD_LEVEL=1  # 记录 trace
```

### perf 分析

```bash
# 使用 perf 采样
perf record -g ./your_app
perf report
```

## 相关文档

- [概览](01_Overview.md) - 项目介绍
- [架构设计](02_Architecture.md) - 内部机制
- [API 参考](03_API_Reference.md) - 接口说明
- [编译构建](04_Build.md) - 构建配置
- [安全风险](05_Security.md) - 安全问题
- [用户指南](docs/README.md) - 官方文档
