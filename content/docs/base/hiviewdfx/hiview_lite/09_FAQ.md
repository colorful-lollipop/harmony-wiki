# 常见问题

## 目的

本文档描述 HiView Lite 的常见构建/运行/调试问题和定位路径。

## 适用范围

本文档适用于：
- 遇到构建问题的工程师
- 遇到运行时错误的开发者
- 需要调试的维护者

## 相关跳转

- [GN Targets](06_GN_Targets.md) - 了解构建配置
- [编译产物](07_Build_Artifacts.md) - 了解编译输出
- [架构说明](03_Architecture.md) - 了解组件关系

---

## 构建问题

### Q1: 编译时提示 "undefined reference to 'xxx'"

**问题描述**：编译时提示未定义的引用错误。

**可能原因**：
1. 依赖模块未正确链接
2. include 路径配置错误
3. 函数声明缺失

**定位路径**：
1. 检查 `BUILD.gn` 中的 `deps` 和 `public_deps`
2. 检查 `hiview_lite_config` 的 `include_dirs`
3. 确认函数在对应头文件中声明

**证据来源**：
- BUILD.gn:31-37 - include_dirs 定义
- BUILD.gn:78 - public_deps 定义

**解决方案**：
```gn
# 确保依赖正确
public_deps = [
  ":hiview_lite_static",
]
```

---

### Q2: 编译时提示 "error: macro redefined"

**问题描述**：宏重定义错误。

**可能原因**：
1. 宏在多个文件中定义
2. 宏冲突

**定位路径**：
1. 查找宏定义位置：`grep -r "MACRO_NAME ."`
2. 检查宏定义是否有保护（`#ifndef` / `#define` / `#endif`）

**证据来源**：
- hiview_config.h:30-32 - `#ifndef HIVIEW_FILE_DIR` / `#define` / `#endif`

**解决方案**：
```c
// ✅ 好的做法：使用宏保护
#ifndef HIVIEW_FILE_DIR
#define HIVIEW_FILE_DIR ""
#endif
```

---

### Q3: 静态库大小超过预期

**问题描述**：编译生成的 `libhiview_lite.a` 大小超出预期。

**可能原因**：
1. 启用了不必要的功能（LOG/DUMP/EVENT）
2. 调试符号未剥离
3. 编译优化级别不对

**定位路径**：
1. 检查 `BUILD.gn` 中的 feature flags
2. 检查 `ohos_build_type`（debug/release）
3. 使用 `size` 工具分析静态库

**证据来源**：
- BUILD.gn:15-20 - feature flags
- BUILD.gn:57-61 - 根据 ohos_build_type 设置不同宏
- bundle.json:33-34 - ROM/RAM 占用

**解决方案**：
```bash
# 构建时指定 release 模式
hb build -f --ccache --product-name <product> --build-type release

# 或在 GN args 中关闭不需要的功能
hiview_lite_hilog_lite_log_switch = 0
hiview_lite_dump_lite_dump_switch = 0
```

---

## 运行时问题

### Q4: HiView 服务未启动

**问题描述**：系统启动后，HiView 服务未运行。

**可能原因**：
1. CORE_INIT 或 SYS_SERVICE_INIT 未触发
2. 内存不足
3. 栈溢出

**定位路径**：
1. 查看启动日志，搜索 "hiview init success"
2. 检查 SAMGR Lite 日志，确认服务注册
3. 使用调试器单步跟踪 `Init` 函数

**证据来源**：
- hiview_config.c:37 - `CORE_INIT_PRI(HiviewConfigInit, 0)`
- hiview_service.c:51 - `SYS_SERVICE_INIT(Init)`
- hiview_service.c:70 - `HIVIEW_UartPrint("hiview init success.");`

**解决方案**：
```c
// 检查初始化宏是否正确
CORE_INIT_PRI(HiviewConfigInit, 0);
SYS_SERVICE_INIT(Init);

// 检查栈大小是否足够
TaskConfig config = {
    LEVEL_LOW,
    HIVIEW_STACK_PRIO,    // 默认 24
    HIVIEW_STACK_SIZE,     // 默认 4096
    10,
    SINGLE_TASK
};
```

---

### Q5: 日志文件未生成

**问题描述**：运行后日志文件不存在。

**可能原因**：
1. `hiview_lite_dir` 配置错误
2. 文件系统未初始化
3. 权限不足

**定位路径**：
1. 检查 `hiview_lite_dir` 编译参数
2. 检查文件系统是否挂载
3. 查看文件操作错误日志

**证据来源**：
- BUILD.gn:22 - `hiview_lite_dir = ""`
- hiview_config.h:34-44 - 文件路径定义
- hiview_file.c:46-55 - 文件打开错误处理

**解决方案**：
```bash
# 在 GN args 中配置文件目录
hiview_lite_dir = "/data/log/"

# 确保目录存在并有写权限
mkdir -p /data/log/
chmod 777 /data/log/
```

---

### Q6: 文件满后未重命名

**问题描述**：日志文件达到最大大小后，未自动重命名。

**可能原因**：
1. 文件监视器未注册
2. `outPath` 配置错误
3. 互斥锁超时

**定位路径**：
1. 检查是否调用了 `RegisterFileWatcher`
2. 检查 `outPath` 配置
3. 查看文件操作错误日志

**证据来源**：
- hiview_file.h:186 - `RegisterFileWatcher` 函数
- hiview_file.c:291-311 - `RegisterFileWatcher` 实现
- hiview_file.c:233 - `HIVIEW_MutexLockOrWait(fp->mutex, OUT_PATH_WAIT_TIMEOUT)`

**解决方案**：
```c
// 确保注册文件监视器
RegisterFileWatcher(&logFile, MyFileWatcher, "/data/log/output.log");

// 检查互斥锁超时配置
#define OUT_PATH_WAIT_TIMEOUT (5 * 1000)  // 5 秒
```

---

## 调试技巧

### 查看日志输出

HiView Lite 支持 UART 输出，可通过 UART 查看日志：

```c
// 在代码中添加调试信息
HIVIEW_UartPrint("Debug message: xxx");
```

**证据来源**：
- hiview_util.c:133-136 - `HIVIEW_UartPrintDef` 实现
- hiview_util.c:138-141 - `HIVIEW_UartPrint` 实现

---

### 使用 Hook 机制

可以通过 Hook 机制覆盖底层函数，便于调试：

```c
// 自定义文件操作函数
int my_open(const char *path, int flags, ...) {
    printf("Opening file: %s\n", path);
    return open(path, flags, ...);
}

// 初始化 Hook
HIVIEW_Hooks hooks = {
    .open_fn = my_open,
    // ... 其他函数
};
HIVIEW_InitHook(&hooks);
```

**证据来源**：
- hiview_util.h:52-74 - `HIVIEW_Hooks` 结构体
- hiview_util.c:148-174 - `HIVIEW_InitHook` 实现

---

### 单步跟踪初始化

使用调试器单步跟踪初始化流程：

1. 在 `HiviewConfigInit` 设置断点
2. 在 `Init` 设置断点
3. 在 `InitHiviewComponent` 设置断点
4. 单步执行，观察变量值

**证据来源**：
- hiview_config.c:29-34 - `HiviewConfigInit` 实现
- hiview_service.c:45-51 - `Init` 实现
- hiview_service.c:107-115 - `InitHiviewComponent` 实现

---

### 检查内存使用

使用工具检查内存泄漏和栈使用：

```bash
# 使用 valgrind 检查内存泄漏
valgrind --leak-check=full --show-leak-kinds=all ./your_app

# 检查栈使用
arm-none-eabi-objdump -d hiview_service.o | grep -A 20 "InitHiviewComponent"
```

**证据来源**：
- hiview_util.c:55-59 - `HIVIEW_MemAlloc` 实现
- BUILD.gn:23 - `hiview_lite_stack_size = 4096`

---

## 配置问题

### Q7: 如何禁用某个功能（LOG/DUMP/EVENT）？

**问题描述**：需要禁用某个 DFX 功能以节省资源。

**解决方案**：
```bash
# 在构建时设置对应的 switch 为 0
hiview_lite_hilog_lite_log_switch = 0
hiview_lite_dump_lite_dump_switch = 0
hiview_lite_hievent_lite_event_switch = 0
```

**证据来源**：
- BUILD.gn:18-20 - feature switches
- BUILD.gn:50-53 - defines 生成

---

### Q8: 如何修改日志级别？

**问题描述**：需要调整日志输出级别（DEBUG/INFO/WARN/ERROR）。

**解决方案**：
```bash
# debug 模式
hiview_lite_hilog_lite_level = 1  # HILOG_LV_DEBUG

# release 模式
hiview_lite_hilog_lite_level_release = 3  # HILOG_LV_ERROR
```

**证据来源**：
- BUILD.gn:16-17 - 日志级别参数
- BUILD.gn:57-61 - 根据 ohos_build_type 选择级别
- hiview_config.h:79 - `uint8 level : 3;`

---

### Q9: 如何修改日志输出模式？

**问题描述**：需要调整日志输出模式（UART/文件）。

**解决方案**：
```bash
# 输出到 UART（不推荐用于商业版本）
hiview_lite_output_option = 0

# 输出到文本文件
hiview_lite_output_option = 2

# 输出到二进制文件
hiview_lite_output_option = 3
```

**证据来源**：
- BUILD.gn:15 - `hiview_lite_output_option = 1`
- hiview_config.h:88-95 - `HiviewOutputOption` 枚举

---

## 平台适配问题

### Q10: 如何适配新的平台？

**问题描述**：需要将 HiView Lite 移植到新平台。

**解决方案**：
1. 修改文件系统接口（通过 `HIVIEW_InitHook`）
2. 调整栈大小和优先级（通过 GN args）
3. 调整文件目录路径（通过 `hiview_lite_dir`）

**证据来源**：
- hiview_util.c:148-174 - `HIVIEW_InitHook` 实现
- BUILD.gn:23-24 - 栈配置
- BUILD.gn:22 - 目录配置

---

## 关键结论

1. **构建问题** - 大多数构建问题与依赖和配置有关。
2. **运行时问题** - 大多数运行时问题与初始化和文件系统有关。
3. **调试技巧** - Hook 机制和日志输出是有效的调试手段。
4. **配置灵活** - 通过 GN args 可以灵活配置各种功能。

---

*最后更新：2026-02-06*
