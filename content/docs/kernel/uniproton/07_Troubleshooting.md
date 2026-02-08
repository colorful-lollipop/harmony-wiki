# UniProton 故障排查指南

## 概述

本文档收集 UniProton 开发过程中的常见问题及解决方案。

---

## 编译问题

### 问题 1: GN 环境未配置

**错误信息**:
```
'gn' is not recognized as an internal or external command
or
ninja: error: loading 'build.ninja': No such file or directory
```

**解决方案**:

```bash
# 安装 GN
git clone https://gn.googlesource.com/gn
cd gn
python build/gen.py
sudo cp out/gn /usr/local/bin/

# 或使用 prebuilt
wget https://raw.githubusercontent.com/nickel-org/nickel.rs/master/devtools/ninja_1.8.2_ubuntu.zip
```

---

### 问题 2: 工具链路径未设置

**错误信息**:
```
arm-none-eabi-gcc: command not found
```

**解决方案**:

```bash
# 添加到环境变量
export PATH=$PATH:/path/to/gcc-arm-none-eabi-10-2020-q4-major/bin

# 永久生效 (添加到 ~/.bashrc 或 ~/.zshrc)
echo 'export PATH=$PATH:/path/to/gcc-arm-none-eabi-10-2020-q4-major/bin' >> ~/.bashrc
```

---

### 问题 3: config.gni 未生成

**错误信息**:
```
import("$root_out_dir/config.gni"): The file to import was not found.
```

**解决方案**:

```bash
# 1. 确保产品配置文件存在
ls product_path/kernel_configs/*.config

# 2. 手动运行 Kconfig
python3 build/lite/scripts/gen_kconfig.py --product=your_product

# 3. 重新生成
gn gen out/uniproton --args="product_path='...'"
```

---

### 问题 4: 编译选项不支持

**错误信息**:
```
error: unknown option '-fstack-protector-strong'
```

**解决方案**:

检查 GCC 版本，低版本可能不支持某些选项：

```bash
arm-none-eabi-gcc --version
# 要求版本 >= 10-2020-q4-major
```

---

## 运行问题

### 问题 5: 系统无法启动 (HardFault)

**症状**: 程序跑飞，进入 HardFault 中断

**排查步骤**:

1. **检查栈大小**

```c
// 任务栈太小可能导致溢出
TaskParam taskParam = {
    .taskEntry = (TSK_ENTRY_FUNC)userTask,
    .stackAddr = userStack,
    .stackSize = 0x800,  // 栈太小!
    // ...
};
```

**解决方案**: 增加栈大小 (建议 >= 0x1000)

2. **检查堆内存**

```c
// 堆内存不足
#define OS_MEM_HEAP_SIZE 0x10000  // 256KB
```

3. **检查启动代码**

```bash
# 检查链接脚本中的 RAM/Flash 布局
ls out/uniproton/*.map
```

---

### 问题 6: 任务调度异常

**症状**: 任务不执行、优先级反转、死锁

**排查步骤**:

1. **检查任务优先级设置**

```c
// 注意: 优先级 31 保留给 IDLE 任务
if (priority >= OS_TSK_PRIORITY_31) {
    return OS_ERRNO_TSK_PRIORITY_ERROR;
}
```

2. **检查调度锁**

```c
// PRT_TaskLock() 后忘记解锁
PRT_TaskLock();
// ... 临界区代码 ...
PRT_TaskUnlock();  // 忘记调用!
```

3. **检查信号量优先级继承**

```c
// 高优先级任务被低优先级任务阻塞
// 启用优先级继承
SemParam.attr = OS_ATTR_PP_INHERIT;  // Priority Protocol
```

---

### 问题 7: 内存分配失败

**症状**: `PRT_MemAlloc` 返回 NULL

**排查步骤**:

1. **检查堆内存大小**

```c
// 配置中定义堆大小
#define OS_MEM_HEAP_SIZE 0x20000
```

2. **检查内存碎片**

```c
// 频繁分配/释放导致碎片
// 解决方案: 使用固定大小块分配器 (FSC)
PRT_FscMemAlloc(poolId);
PRT_FscMemFree(poolId, pBuf);
```

3. **检查泄漏**

```c
// 确保每次 alloc 都有对应的 free
void *p = PRT_MemAlloc(128);
// ...
PRT_MemFree(p);  // 必须释放
```

---

### 问题 8: 中断处理问题

**症状**: 中断不触发、数据丢失

**排查步骤**:

1. **检查中断使能**

```c
// 中断创建后默认禁用，需要手动使能
PRT_HwiCreate(IRQ_USART1, handler, ...);
PRT_HwiEnable(IRQ_USART1);  // 必须使能
```

2. **检查中断优先级**

```c
// 优先级配置错误
// Cortex-M: 数值越小优先级越高
// 确保与 NVIC 配置一致
```

3. **检查中断处理时间**

```c
// 中断处理过长导致丢失
// 建议: 最小化中断处理，延迟处理放任务中
void FastHwiHandler(void) {
    PRT_SemPost(sem);  // 只发信号
}

void TaskHandler(void) {
    PRT_SemPend(sem, timeout);  // 任务中处理
}
```

---

## 调试方法

### 1. 使用 IDE 调试

#### Keil/μVision

```bash
# 1. 导入工程
# 2. 设置 J-Link/ST-Link
# 3. 下载并调试
```

#### IAR Embedded Workbench

```bash
# 1. 打开工作区
# 2. 配置调试器
# 3. 下载并调试
```

---

### 2. 使用 GDB 调试

```bash
# 1. 启动 GDB Server
JLinkGDBServer -device STM32F407ZG -if SWD -speed 4000

# 2. 连接 GDB
arm-none-eabi-gdb out/uniproton/unstripped/bin/uniproton

(gdb) target remote localhost:2331
(gdb) break main
(gdb) continue
(gdb) backtrace  # 查看调用栈
(gdb) info registers  # 查看寄存器
(gdb) x/32x $sp  # 查看栈内容
```

---

### 3. 使用 ITM/SWO 打印

```c
// 启用 ITM 跟踪
void ITM_Init(void) {
    // 配置 SWO 引脚
}

// 打印日志
printf("Task running: %d\n", taskId);
```

---

### 4. 使用串口调试

```c
// 串口初始化
UartInit(COM1, 115200);

// 打印调试信息
DbgPrint("Hello UniProton!\n");
```

---

### 5. 使用 Hook 调试

```c
// 注册系统 Hook
void MyHook(U32 hookType) {
    if (hookType == OS_HOOK_TYPE_STACK_OVERFLOW) {
        DbgPrint("Stack overflow detected!\n");
    }
}

PRT_HookUserCreate(MyHook);
```

---

## 常见错误码

| 错误码 | 定义位置 | 说明 |
|--------|----------|------|
| `OS_ERRNO_TSK_NO_MEMORY` | `prt_errno.h` | 内存不足 |
| `OS_ERRNO_TSK_PRIORITY_ERROR` | `prt_errno.h` | 优先级无效 |
| `OS_ERRNO_SEM_INVALID_ARGS` | `prt_errno.h` | 参数无效 |
| `OS_ERRNO_QUEUE_INVALID_ARGS` | `prt_errno.h` | 队列参数无效 |
| `OS_ERRNO_EVENT_INVALID` | `prt_errno.h` | 事件无效 |
| `OS_ERRNO_HWI_INVALID_ARGS` | `prt_errno.h` | 中断参数无效 |

**完整错误码列表**: `src/include/uapi/prt_errno.h`

---

## 日志级别

```c
// 编译时启用调试日志
#define OS_DEBUG_ENABLE

// 运行时常量
#define OS_LOG_LEVEL_ERROR   1
#define OS_LOG_LEVEL_WARN    2
#define OS_LOG_LEVEL_INFO    3
#define OS_LOG_LEVEL_DEBUG   4
```

---

## 性能分析

### CPU 占用率

```c
// 获取 CPU 占用率
U32 cpup = PRT_CpupGet(0);  // 核 0 的 CPU 占用率 (百分比)
```

### 栈使用量

```c
// 任务栈使用量分析
// 需要启用栈保护
```

### 中断延迟测量

```c
// 测量中断延迟
U32 start = PRT_TickGet();
U32 end = PRT_TickGet();
U32 latency = end - start;  // Tick 数为单位
```

---

## 相关文档

- [API 参考](./03_API_Reference.md) - 完整 API
- [架构设计](./04_Architecture.md) - 内部机制
- [安全评审](./06_Security_Review.md) - 安全考量
- [官方文档](https://gitee.com/openeuler/UniProton)
