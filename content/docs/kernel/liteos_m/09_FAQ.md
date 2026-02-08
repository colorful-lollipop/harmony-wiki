# 常见问题 - 09_FAQ

## 构建问题

### Q1: 如何设置目标芯片架构？

**答**: 在 `hb set` 时选择开发板，或手动配置 `LOSCFG_ARCH_*`。

```bash
# menuconfig 配置
make menuconfig

# 路径
Arch --->
    [*] Enable ARM AARCH32
    (cortex-m4) ARM CPU type
```

### Q2: 构建产物在哪里？

**答**: 构建产物位于 out 目录：

```
out/<product>/libs/libkernel.a      # 静态库
out/<product>/unstripped/bin/liteos # 可执行镜像
```

### Q3: 如何启用/禁用组件？

**答**: 通过 menuconfig 或直接修改 `.config`：

```bash
make menuconfig

# 启用 dynlink
Kernel --->
    Enable Components --->
        [*] Enable dynlink
```

### Q4: GN 构建失败？

**答**: 检查以下项：

1. Python3 是否在 PATH
2. GN 工具是否安装
3. config.gni 是否正确生成

```bash
# 检查 GN
gn --version

# 检查配置
cat out/xxx/config.gni
```

## 运行时问题

### Q5: 任务不运行？

**答**: 常见原因：

1. **优先级问题**: 任务优先级是否过低
2. **栈溢出**: 栈空间是否不足
3. **无限阻塞**: 任务是否在循环中阻塞

```c
// 检查任务是否创建成功
UINT32 ret = LOS_TaskCreate(NULL, &taskParam);
if (ret != LOS_OK) {
    // 错误处理
}
```

### Q6: 内存分配失败？

**答**:

1. 检查堆大小配置 (`LOSCFG_SYS_HEAP_SIZE`)
2. 检查是否有内存泄漏
3. 启用内存调试 (`LOSCFG_DEBUG`)

### Q7: 消息队列满？

**答**:

1. 增大队列深度
2. 添加超时机制
3. 检查消费者是否正常运行

## 调试问题

### Q8: 如何查看内核日志？

**答**:

1. 串口输出 (默认)
2. Shell 命令 `log`
3. ITM/SWO 调试 (Cortex-M)

### Q9: 如何调试任务切换？

**答**:

1. 启用调度追踪 (`LOSCFG_SCHED_DEBUG`)
2. 使用 Shell `task` 命令
3. ITM/ETM 硬件追踪

### Q10: 栈溢出如何定位？

**答**:

1. 启用栈保护
2. 任务创建时指定足够栈空间
3. 使用 `LOS_InspectStack` 检查

## 移植问题

### Q11: 如何移植到新芯片？

**答**:

1. 实现 HAL 层 (硬件抽象层)
2. 配置时钟源
3. 配置串口/UART
4. 实现系统节拍

### Q12: 支持哪些开发板？

**答**: 社区移植项目：

| 芯片 | 链接 |
|------|------|
| STM32F103 | https://gitee.com/rtos_lover/stm32f103_simulator_keil |
| STM32F429 | https://gitee.com/harylee/stm32f429ig_firechallenger |
| Qemu | https://gitee.com/openharmony/device_qemu |

> 参考: README.md:97-107

## 组件问题

### Q13: Shell 不工作？

**答**:

1. 启用 Shell 组件 (`LOSCFG_COMPONENTS_SHELL`)
2. 配置串口
3. 检查波特率设置

### Q14: 动态加载失败？

**答**:

1. 检查签名 (如启用签名验证)
2. 确认路径正确
3. 检查 ELF 格式兼容性

## 工具问题

### Q15: hb 命令找不到？

**答**:

```bash
# 安装 HarmonyOS 开发工具
pip3 install ohos-build
```

### Q16: 如何查看内存使用？

**答**:

```bash
# Shell 命令
Shell> mem

# API 调用
VOID PrintMemUsage(VOID) {
    // 调用 LOS_MemInfoGet
}
```

## 定位路径

| 问题类型 | 定位文件 |
|----------|----------|
| 构建失败 | `BUILD.gn`, `liteos.gni` |
| 任务问题 | `kernel/src/los_task.c` |
| 内存问题 | `kernel/src/mm/`, `utils/los_error.c` |
| 调度问题 | `kernel/src/los_sched.c` |
| IPC 问题 | `kernel/src/los_queue.c`, `los_mux.c`, `los_sem.c` |

## 相关文档

- [构建系统](07_Build.md)
- [内核 API](04_Kernel_API.md)
- [架构说明](03_Architecture.md)
