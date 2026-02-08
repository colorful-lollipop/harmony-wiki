# HiCollie 常见问题与定位路径

> 构建、运行、调试问题与定位路径

---

## 目的与适用范围

### 文档目的
本文档提供 HiCollie 的常见问题、诊断方法和解决方案，帮助开发者快速定位和解决问题。

### 适用场景
- 🔧 **问题定位** - 遇到问题时快速诊断
- 🐛 **调试指南** - 理解调试方法和工具
- 📋 **问题报告** - 有效的 Bug 报告方式

---

## 构建问题

### 问题 1: 构建失败 - 找不到头文件

**现象**:
```
error: xcollie.h: No such file or directory
```

**原因**: 头文件路径配置错误

**解决**:
```bash
# 检查 BUILD.gn 中的 include_dirs
cat frameworks/native/BUILD.gn | grep include_dirs

# 确认头文件存在
ls interfaces/native/innerkits/include/xcollie/

# 重新构建
hb clean && hb build
```

**证据**: `frameworks/native/BUILD.gn:17-20` - include_dirs 配置

---

### 问题 2: 链接错误 - 未定义符号

**现象**:
```
undefined reference to `Watchdog::GetInstance()'
```

**原因**: 源文件未包含正确的头文件或库未链接

**解决**:
```bash
# 检查源文件包含
grep "#include.*watchdog" frameworks/native/*.cpp

# 检查 BUILD.gn 依赖
cat frameworks/native/BUILD.gn | grep deps

# 使用 gn 查看依赖图
gn desc out/ //base/hiviewdfx/hicollie/frameworks/native
```

**证据**: `watchdog.h` - Watchdog 类定义

---

### 问题 3: 编译警告 - 隐式声明

**现象**:
```
warning: implicit declaration of function 'memset_s'
```

**原因**: 头文件顺序或缺失

**解决**:
```cpp
// 确保包含必要的头文件
#include <string.h>
#include <securec.h>
```

**证据**: `frameworks/native/BUILD.gn:17` - include_dirs 配置

---

## 运行时问题

### 问题 1: 超时检测未触发

**现象**: 定时器超时但未触发回调或日志

**诊断步骤**:
```bash
# 1. 检查 HiLog 日志
hilog -T HiCollie

# 2. 检查定时器是否正确注册
# 查看 /data/log/faultlog/faultloggerd/

# 3. 检查时间计算
# 在代码中添加调试日志
XCOLLIE_LOGI("Timer registered: name=%s, timeout=%u", name, timeout);
```

**可能原因**:
| 原因 | 检查方法 | 解决方案 |
|-----|---------|---------|
| 进程类型错误 | 检查 UID | 使用正确的进程类型 |
| 定时器未启动 | 检查看门狗线程 | 确认 `InitFfrtWatchdog()` 被调用 |
| 超时时间过长 | 检查 timeout 值 | 使用合理的超时值 |

**证据**: `interfaces/ndk/hicollie.cpp:234-266` - SetTimer 实现

---

### 问题 2: 主线程检测失败

**现象**: `HICOLLIE_WRONG_THREAD_CONTEXT` 错误

**诊断步骤**:
```bash
# 1. 检查当前线程
ps -T -p <pid>

# 2. 检查 UID
id -u <pid>

# 3. 查看代码中的检查逻辑
# interfaces/ndk/hicollie.cpp:50-56
bool IsAppMainThread() {
    static uint64_t uid = getuid();
    if (g_pid == gettid() && uid >= MIN_APP_UID) {
        return true;
    }
    return false;
}
```

**解决方案**:
```c
// 不在主线程调用时，使用单独的检测线程
pthread_t checkThread;
pthread_create(&checkThread, NULL, CheckFunction, NULL);
```

**证据**: `interfaces/ndk/hicollie.cpp:50-56` - IsAppMainThread 实现

---

### 问题 3: 动态库加载失败

**现象**: `dlopen failed: libthread_sampler.z.so`

**诊断步骤**:
```bash
# 1. 检查库是否存在
find /system/lib64 -name "libthread_sampler*"

# 2. 检查依赖
ldd /system/lib64/libthread_sampler.z.so

# 3. 查看 HiLog 错误日志
hilog -T HiCollie | grep dlopen
```

**可能原因**:
| 原因 | 检查方法 | 解决方案 |
|-----|---------|---------|
| 库未安装 | `ls -l` | 确认库已正确安装 |
| 依赖缺失 | `ldd` | 检查所有依赖库 |
| 权限不足 | `ls -l` | 检查文件权限 |

**证据**: `watchdog_inner.cpp:366-396` - dlopen 调用

---

## 性能问题

### 问题 1: 高 CPU 占用

**现象**: HiCollie 导致 CPU 占用过高

**诊断步骤**:
```bash
# 1. 查看 CPU 使用情况
top -p <pid>

# 2. 统计看门狗线程调用
# 添加调试日志
XCOLLIE_LOGI("FetchNextTask called");
```

**优化建议**:
| 优化项 | 配置 | 说明 |
|---------|------|------|
| 增加检测间隔 | 调整 interval 参数 | 减少检测频率 |
| 禁用不用的功能 | 修改 `hicollie.gni` | 关闭不需要的功能 |
| 限制任务数量 | 使用 `CancelTimer()` 及时取消 | 减少队列压力 |

**证据**: `watchdog_inner.h:164` - MAX_WATCH_NUM 定义

---

### 问题 2: 内存泄漏

**现象**: 进程内存持续增长

**诊断步骤**:
```bash
# 1. 使用 Valgrind 检测
valgrind --leak-check=full <application>

# 2. 查看 VmRSS
cat /proc/<pid>/status | grep VmRSS

# 3. 检查智能指针使用
# 确保使用 std::shared_ptr 而非裸指针
```

**常见泄漏点**:
| 位置 | 检查项 | 解决方案 |
|-----|---------|---------|
| 回调函数 | 确保回调清理资源 | 使用 RAII 包装 |
| 任务队列 | 及时移除完成的任务 | `RemoveXCollieTask()` |
| 动态库加载 | 及时释放 dlopen 返回的句柄 | `dlclose()` |

**证据**: `watchdog_inner.h:28-35` - 智能指针使用

---

## 调试方法

### 启用详细日志

```cpp
// 在代码中添加详细日志
XCOLLIE_LOGI("WatchdogInner::AddThread: name=%s, interval=%llu", name.c_str(), interval);
```

### 使用 gdb 调试

```bash
# 附加到运行中的进程
gdb -p <pid>

# 查看调用栈
(gdb) bt

# 设置断点
(gdb) break WatchdogInner::AddThread

# 运行
(gdb) continue
```

### 使用 hilog 查看日志

```bash
# 实时查看 HiCollie 日志
hilog -T HiCollie

# 保存到文件
hilog -T HiCollie > hicollie.log

# 过滤特定级别
hilog -T HiCollie -L D
```

### 系统事件监控

```bash
# 查看 HiSysEvent 上报的事件
hisysevent -c HiCollie

# 监控实时事件
hisysmonitor
```

---

## 故障日志分析

### 故障日志位置

| 日志类型 | 路径 | 说明 |
|---------|------|------|
| **冻结日志** | `/data/log/faultlog/faultloggerd/` | SERVICE_BLOCK, SERVICE_TIMEOUT |
| **IPC 日志** | `/data/log/faultlog/faultloggerd/` | IPC_FULL |
| **Jank 日志** | `/data/log/faultlog/faultloggerd/` | MAIN_THREAD_JANK |

**证据**: `hisysevent.yaml` - 事件定义

### 日志解读

#### SERVICE_BLOCK 事件

```
事件类型: SERVICE_BLOCK
可能原因:
1. Watchdog 检测到超时
2. 主线程阻塞
3. 任务未及时响应

关键字段:
- MSG: 超时描述
- STACK: 堆栈信息
- PID/TGID/UID: 进程信息
```

#### MAIN_THREAD_JANK 事件

```
事件类型: MAIN_THREAD_JANK
可能原因:
1. 事件处理时间过长
2. 主线程被阻塞
3. 系统资源竞争

关键字段:
- JANK_LEVEL: 卡顿级别
- HEAVIEST_STACK: 最重堆栈
- EXTERNAL_LOG: 外部日志
```

**证据**: `hisysevent.yaml:16-77` - 事件字段定义

---

## 问题报告

### Bug 报告信息

报告 Bug 时应包含以下信息：

| 信息项 | 内容 | 示例 |
|---------|------|------|
| **版本** | OpenHarmony 版本 | API 12 |
| **组件版本** | HiCollie 版本 | 3.1 |
| **重现步骤** | 详细步骤 | 1. 调用 OH_HiCollie_SetTimer... |
| **预期行为** | 预期结果 | 定时器超时时应触发回调 |
| **实际行为** | 实际结果 | 定时器超时未触发 |
| **日志** | HiLog 输出 | XCOLLIE: ... |
| **堆栈** | 崩溃堆栈 | gdb bt 输出 |

### 日志收集

```bash
# 收集 HiCollie 日志
hilog -T HiCollie > hicollie_log.txt

# 收集故障日志
tar -czf faultlog.tar.gz /data/log/faultlog/faultloggerd/

# 收集系统信息
version > system_info.txt
```

---

## 常见配置错误

### 配置错误 1: 超时值超出范围

**错误码**: `HICOLLIE_INVALID_TIMEOUT_VALUE`

**说明**: 超时值必须在 [3, 15] 秒范围内（API 12）

**解决**:
```c
// 使用有效的超时值
OH_HiCollie_Init_StuckDetectionWithTimeout(task, 5); // 有效
OH_HiCollie_Init_StuckDetectionWithTimeout(task, 20); // 无效
```

**证据**: `interfaces/ndk/hicollie.cpp:178-188` - 超时值检查

---

### 配置错误 2: 线程上下文错误

**错误码**: `HICOLLIE_WRONG_THREAD_CONTEXT`

**说明**: 不能在主线程调用初始化函数

**解决**:
```c
// 在业务线程而非主线程调用
pthread_create(&thread, NULL, InitHiCollie, NULL);
```

**证据**: `interfaces/ndk/hicollie.cpp:59-66` - 线程上下文检查

---

## 相关资源

### 文档资源

- [项目概览](00_Overview.md) - 了解组件功能
- [NDK C API](03_NDK_API.md) - API 参考手册
- [架构说明](02_Architecture.md) - 理解设计

### 外部资源

- [OpenHarmony 文档](https://gitee.com/openharmony/docs)
- [HiLog 工具指南](https://gitee.com/openharmony/hiviewdfx_hilog)
- [HiSysEvent 文档](https://gitee.com/openharmony/hiviewdfx_hisysevent)

---

## 关键结论

### 问题分类
1. **构建问题** - 配置错误、依赖缺失
2. **运行时问题** - API 使用错误、初始化失败
3. **性能问题** - 配置不当、资源泄漏
4. **调试方法** - 日志、gdb、性能工具

### 诊断建议
1. **查看日志** - 首先检查 HiLog 和故障日志
2. **检查配置** - 确认参数在有效范围
3. **验证环境** - 确认运行时环境和依赖
4. **使用工具** - gdb、valgrind、top 等工具辅助诊断

### 预防措施
1. **正确初始化** - 确保按正确顺序和参数初始化
2. **及时清理** - 取消不再需要的定时器
3. **监控资源** - 定期检查内存和 CPU 使用
4. **添加日志** - 关键操作添加详细日志

---

## 相关跳转

- [项目概览](00_Overview.md) - 了解运行环境
- [安全评审](07_Security_Review.md) - 了解安全机制
- [编译产物](06_Build_Artifacts.md) - 了解产物依赖
