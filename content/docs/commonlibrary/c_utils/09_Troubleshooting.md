# 问题排查

## 目的

本文档汇总 c_utils 常见构建、运行、调试问题及定位方法。

## 适用范围

- 开发者日常问题排查
- 集成问题定位
- 调试技巧参考

---

## 构建问题

### 1. 编译失败：找不到头文件

**症状**:
```
fatal error: 'refbase.h' file not found
#include "refbase.h"
         ^~~~~~~~~~~
```

**原因**:
- 未正确添加依赖
- include 路径未配置

**解决**:
```gn
# 在 BUILD.gn 中添加依赖
ohos_shared_library("my_module") {
  external_deps = [
    "c_utils:utils",  # 或 "c_utils:utilsbase"
  ]
}
```

**验证**:
```bash
# 检查 include 路径是否正确传递
gn desc out/rk3568 //my/module:my_module include_dirs
```

---

### 2. 链接失败：符号未定义

**症状**:
```
undefined reference to `OHOS::RefBase::RefBase()'
undefined reference to `OHOS::Parcel::Parcel()'
```

**原因**:
- 只添加了头文件路径，未链接库
- 链接顺序问题

**解决**:
```gn
ohos_shared_library("my_module") {
  sources = [ "my_source.cpp" ]
  
  # 必须添加 external_deps
  external_deps = [
    "c_utils:utils",
  ]
}
```

---

### 3. 静态库与动态库混用冲突

**症状**:
```
multiple definition of `OHOS::RefBase::RefBase()'
```

**原因**:
- 同时链接了 `libutils.so` 和 `libutilsbase.a`
- 不同模块分别使用静态和动态库

**解决**:
统一使用同一种链接方式：
```gn
# 方案1: 全部使用动态库
external_deps = [ "c_utils:utils" ]

# 方案2: 全部使用静态库（不推荐，除非特殊需求）
external_deps = [ "c_utils:utilsbase" ]
```

---

### 4. Rust 编译失败

**症状**:
```
error: could not find `rust_cxx` in dependencies
```

**原因**:
- 未启用 rust_cxx 部件
- 平台不支持（macOS 不支持 Rust）

**解决**:
```bash
# 确保 rust_cxx 在构建目标中
./build.sh --product-name rk3568 --build-target thirdparty_rust_cxx

# 或在配置中启用
# 检查 bundle.json 中 deps 包含 rust_cxx
```

---

## 运行问题

### 1. 找不到 libutils.so

**症状**:
```
CANNOT LINK EXECUTABLE "my_service": library "libutils.so" not found
```

**排查步骤**:

```bash
# 1. 检查文件是否存在
ls -la /system/lib/libutils.so
ls -la /system/lib64/libutils.so

# 2. 检查文件权限
ls -laZ /system/lib/libutils.so  # SELinux 上下文

# 3. 检查 linker 配置
cat /system/etc/ld.config.txt | grep search

# 4. 使用 ldd 检查依赖
ldd /system/bin/my_service

# 5. 检查系统属性
getprop | grep linker
```

**解决**:
```bash
# 如果是开发版本，手动推送
adb push out/rk3568/commonlibrary/c_utils/base/libutils.so /system/lib/
adb shell chmod 644 /system/lib/libutils.so
adb shell sync
```

---

### 2. 崩溃：段错误（Segmentation Fault）

**症状**:
```
Signal 11 (SIGSEGV), code 1 (SEGV_MAPERR)
```

**常见原因**:

| 场景 | 原因 | 定位方法 |
|------|------|----------|
| RefBase | 对象已销毁仍使用 | 启用 DEBUG_REFBASE |
| Parcel | 读写越界 | 检查容量和游标 |
| unique_fd | 使用已关闭的fd | 检查生命周期 |
| SafeMap | 迭代时修改 | 检查并发访问 |

**调试步骤**:

```bash
# 1. 获取 tombstone
cat /data/tombstones/tombstone_00

# 2. 使用 addr2line 定位
aarch64-linux-gnu-addr2line -e out/rk3568/commonlibrary/c_utils/base/libutils.so \
    -f -C <pc_address>

# 3. 启用调试日志
setprop persist.debug.c_utils 1
```

---

### 3. 内存泄漏

**症状**:
- 进程内存持续增长
- 最终 OOM 被杀死

**排查**:

```cpp
// 启用 RefBase 跟踪
#define DEBUG_REFBASE 1
#define TRACK_ALL 1
```

```bash
# 使用 valgrind（模拟器）
valgrind --leak-check=full --show-leak-kinds=all ./my_app

# 使用 AddressSanitizer
# 在 BUILD.gn 中添加
sanitize = {
  address = true
}
```

**常见泄漏点**:

| 模块 | 泄漏场景 | 检查点 |
|------|----------|--------|
| RefBase | 循环引用 | 检查 sptr/wptr 使用 |
| Parcel | 未释放 | 检查 Parcelable 对象 |
| ThreadPool | 任务未执行 | 检查 Stop() 调用 |
| Timer | 未 Shutdown | 检查 Shutdown() 调用 |

---

### 4. 死锁

**症状**:
- 进程卡死无响应
- 看门狗超时重启

**排查**:

```bash
# 1. 获取线程状态
cat /proc/<pid>/task/*/stack

# 2. 使用 debuggerd
debuggerd -b <pid>

# 3. 检查锁持有情况
cat /proc/lock_stat
```

**常见死锁场景**:

```cpp
// 场景1: 嵌套锁
SafeMap<int, Data> map1;
SafeMap<int, Data> map2;

// 线程1
map1.Insert(1, data);
map2.Insert(2, data);  // 可能死锁

// 线程2
map2.Insert(2, data);
map1.Insert(1, data);  // 可能死锁
```

**解决**: 统一锁获取顺序

---

## 调试技巧

### 1. 启用 RefBase 调试

```cpp
// 在代码中启用
#define DEBUG_REFBASE 1
#define PRINT_TRACK_AT_ONCE 1
```

或在 BUILD.gn 中:
```gn
declare_args() {
  c_utils_debug_refbase = true
  c_utils_print_track_at_once = true
}
```

**输出示例**:
```
RefBase: curCount: 2, operation: incStrong, countType: strong
RefBase: curCount: 1, operation: decStrong, countType: strong
```

---

### 2. 日志级别控制

```cpp
// 启用 DEBUG_UTILS 日志
#define DEBUG_UTILS 1
```

日志宏定义 (`base/src/utils_log.h`):
```cpp
#ifdef CONFIG_HILOG
#include "hilog/log.h"
#define UTILS_LOGD(...) HILOG_DEBUG(LOG_CORE, __VA_ARGS__)
#define UTILS_LOGI(...) HILOG_INFO(LOG_CORE, __VA_ARGS__)
#define UTILS_LOGE(...) HILOG_ERROR(LOG_CORE, __VA_ARGS__)
#else
#define UTILS_LOGD(...)
#define UTILS_LOGI(...)
#define UTILS_LOGE(...)
#endif
```

---

### 3. 使用 GDB 调试

```bash
# 1. 启动 gdbserver
adb shell gdbserver :5039 /system/bin/my_service

# 2. 本地连接
aarch64-linux-gnu-gdb out/rk3568/my/module/my_service
(gdb) target remote device-ip:5039

# 3. 设置断点
(gdb) b OHOS::RefBase::IncStrongRef
(gdb) b OHOS::Parcel::WriteInt32

# 4. 继续执行
(gdb) continue
```

---

### 4. 性能分析

```bash
# 使用 simpleperf
adb shell simpleperf record -p <pid> -g --duration 10
adb pull /data/local/tmp/perf.data
simpleperf report -g --dsos libutils.so

# 使用 Tracy（如已集成）
```

---

## 常见问题速查

### Q: sptr 和 wptr 如何选择？

**A**:
- 正常持有对象 → `sptr`（强引用）
- 避免循环引用 → `wptr`（弱引用）
- 缓存场景 → `wptr` + `promote()`

### Q: Parcel 最大容量是多少？

**A**: 默认 200KB，可通过 `SetMaxCapacity()` 调整，但不建议超过 1MB

### Q: ThreadPool 默认线程数？

**A**: 无默认值，必须调用 `Start(numThreads)` 指定

### Q: SafeMap 是否线程安全？

**A**: 是，使用 `std::mutex` 保护所有操作。但迭代时持有锁，不适合长时间操作。

### Q: 如何检查文件是否存在？

**A**:
```cpp
#include "file_ex.h"
if (OHOS::FileExists("/path/to/file")) {
    // 存在
}
```

### Q: unique_fd 和 int 如何转换？

**A**:
```cpp
OHOS::unique_fd fd(open("/path", O_RDONLY));
int raw_fd = fd.get();  // 获取原始fd（不转移所有权）
int released = fd.release();  // 释放所有权，返回原始fd
```

---

## 相关跳转

- [对外 API](04_Public_API.md) - API 使用指南
- [内部 API](05_Inner_API.md) - 实现细节
- [安全风险](08_Security_Review.md) - 安全问题
