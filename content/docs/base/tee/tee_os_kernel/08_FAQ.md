# 常见问题

> **文档版本**: 1.0
> **生成时间**: 2026-02-06
> **适用范围**: OpenHarmony TEE OS Kernel 开发与调试

---

## 文档目的

本文档提供 TEE OS Kernel 开发和调试过程中常见问题的解决方案。

---

## 目录

- [构建问题](#构建问题)
- [运行时问题](#运行时问题)
- [调试问题](#调试问题)
- [性能问题](#性能问题)
- [安全相关问题](#安全相关问题)

---

## 构建问题

### 1.1 编译错误

#### Q1: undefined reference to `CHCORE_OH_TEE`

**现象**：
```
kernel/object/memory.c:30: error: undefined reference to 'CHCORE_OH_TEE'
```

**原因**：
- 未在 `config.mk` 中定义 `CHCORE_OH_TEE=ON`
- 某些 TEE 特有代码缺少 `#ifdef CHCORE_OH_TEE`

**解决方案**：
```makefile
# 在 config.mk 中确保定义
CHCORE_OH_TEE = ON
```

---

### 1.2 链接错误：undefined reference to `CHCORE_ARCH_AARCH64`

**现象**：
```
error: undefined reference to 'CHCORE_ARCH_AARCH64'
```

**原因**：
- 未正确定架构变量

**解决方案**：
检查 `config.mk` 中的 `CHCORE_ARCH` 定义：
```makefile
CHCORE_ARCH = aarch64
```

---

### 1.3 GN 构建失败

#### Q2: `oh_build_tee.sh: command not found`

**现象**：
```
error: oh_build_tee.sh: command not found
```

**原因**：
- GN 构建系统是 stub，实际构建使用 `build/build_tee.sh`

**解决方案**：
直接使用完整构建命令：
```bash
cd /path/to/openharmony
./build.sh --product-name rk3568 --build-target tee --ccache
```

---

## 运行时问题

### 2.1 启动失败

#### Q3: TEE 内核未启动

**现象**：
- 系统进入 normal world 但 TEE 未响应
- `bl32.bin` 加载失败

**排查步骤**：

**1. 检查 bl32.bin 是否正确生成**
```bash
ls -la kernel/bl32.bin
file kernel/bl32.bin
```

**2. 检查 ATF (ARM Trusted Firmware）日志**
```
在 ATF 日志中查找 TEE OS 加载信息
```

**3. 验证 TEXT_OFFSET**
```bash
# 检查 config.mk 中的 TEXT_OFFSET
# RK3568: TEXT_OFFSET = 0x8400000
# RK3399: TEXT_OFFSET = 0x8408000
```

---

### 2.2 服务未启动

#### Q4: chanmgr/fsm/tmpfs 未响应

**现象**：
- IPC 调用超时
- 文件操作失败
- 进程无法创建

**排查步骤**：

**1. 检查服务是否运行**
```bash
# 通过系统调用检查进程状态
# 或使用调试工具查看进程列表
```

**2. 检查 Badge 是否正确**
```c
// 确保服务进程使用正确的 Badge
// ROOT_CAP_GROUP_BADGE = 1 (procmgr)
// FSM_BADGE = 2
// TMPFS_BADGE = 4
```

**3. 检查 IPC Channel 是否正确注册**
```c
// 确认服务通过 chanmgr 正确注册了 Channel
```

---

## 调试问题

### 3.1 内核调试

#### Q5: 如何启用内核调试输出

**方法 1：修改 config.mk**
```makefile
# 在 config.mk 中修改
CHCORE_KERNEL_DEBUG = ON
```

**方法 2：使用系统调用输出**
```c
// 在用户态程序中使用
sys_debug_log("debug message", len);
```

**方法 3：查看进程列表**
```c
// 使用系统调用
sys_top();
```

---

### 3.2 用户态调试

#### Q6: 如何调试用户态程序

**方法 1：使用 printf**
```c
#include <stdio.h>
printf("debug info: %d\n", value);
```

**方法 2：查看内存使用**
```c
// 使用系统调用
sys_get_mem_usage_msg(&info, 0);
```

---

### 3.3 GDB 调试

#### Q7: 如何使用 GDB 调试 TEE

**方法 1：在 GDB 中加载符号**
```gdb
# 设置架构
set architecture aarch64

# 加载符号
symbol-file kernel/kernel.img

# 设置断点
break main
```

**方法 2：查看变量**
```gdb
(gdb) print current_thread
(gdb) info locals
```

---

## 性能问题

### 4.1 内存泄漏

#### Q8: 如何检测内存泄漏

**方法 1：查看内存统计**
```bash
# 使用系统调用
sys_get_free_mem_size();  // 获取空闲内存
sys_get_mem_usage_msg(&info, 1);  // 获取使用详情
```

**方法 2：启用 tmpfs 内存追踪**
```makefile
# 在 user/system-services/system-servers/tmpfs/defs.h 中修改
#define DEBUG_MEM_USAGE 1

# 然后重新编译
make user
```

---

### 4.2 性能分析

#### Q9: 如何分析性能瓶颈

**方法 1：使用性能计数器**
```c
// 系统提供性能测试接口
sys_perf_start();  // 开始计数
// ... 执行测试 ...
sys_perf_end();    // 结束计数
```

**方法 2：分析调度延迟**
- 检查线程就绪队列长度
- 分析上下文切换频率
- 查看优先级分布

---

## 安全相关问题

### 5.1 权限被拒

#### Q10: 系统调用返回 -EPERM

**现象**：
```c
int ret = sys_some_operation();
if (ret < 0) {
    printf("Error: %d (EPERM)\n", ret);
}
```

**常见原因**：
- Badge 不在允许范围
- Capability 不存在或无权限
- 操作需要特定 Badge（如 ROOT 或 FSM）

**排查步骤**：
1. 检查调用者的 Badge
2. 确认是否有 Capability
3. 查看系统调用钩子中的权限检查

---

### 5.2 IPC 通信失败

#### Q11: IPC 调用超时

**现象**：
- `sys_ipc_call()` 或 `sys_tee_msg_call()` 长时间阻塞
- 没有返回或返回超时错误

**原因**：
- 服务端未处理请求
- 服务线程阻塞
- 共享内存访问冲突

**排查步骤**：
1. 检查服务进程状态（`sys_top()`）
2. 增加 IPC 超时时间
3. 检查共享内存配置

---

## 开发建议

### 6.1 代码规范

- **遵循 ChCore 微内核模式**
- **使用 Capability 进行资源访问**
- **避免在内核态做耗时操作**
- **所有用户空间数据必须经过验证**
- **错误检查必须完整**

### 6.2 调试技巧

- **使用 `sys_debug_log()` 输出关键信息**
- **使用 `sys_top()` 查看进程状态**
- **使用 `sys_get_mem_usage_msg()` 监控内存**
- **保存崩溃现场日志**

### 6.3 性能优化

- **减少上下文切换次数**
- **使用共享内存减少数据复制**
- **合理设置线程优先级**
- **避免频繁的 IPC 调用**

---

## 相关跳转

- [架构设计](02_Architecture.md) - 了解系统架构有助于调试
- [系统调用接口](03_Syscall_Interfaces.md) - 查阅系统调用文档
- [安全评审](07_Security_Review.md) - 了解安全机制

---

## 工具参考

### 7.1 ELF 查看工具

**位置**：`tools/read_procmgr_elf_tool/`

**用途**：
- 查看 procmgr 二进制结构
- 分析进程布局
- 调试进程加载问题

```bash
cd tools/read_procmgr_elf_tool
make
./read_procmgr_elf_tool /path/to/procmgr
```

---

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2026-02-06
