# 安全风险评审 - 08_Security

## 概述

本文档基于代码证据，对 LiteOS-M 内核进行安全风险评审。

## 评审范围

| 目录/文件 | 状态 | 说明 |
|----------|------|------|
| `kernel/` | 已扫描 | 核心内核代码 |
| `components/` | 已扫描 | 可选组件 |
| `kal/` | 已扫描 | API 抽象层 |
| `utils/` | 已扫描 | 公共工具 |
| `testsuites/` | **忽略** | 测试代码不作为安全证据 |

## 攻击面分析

### 1. 动态加载 (dynlink)

**组件**: `components/dynlink/los_dynlink.c`

**风险等级**: 高

**描述**: 动态加载模块运行时加载共享库。

**README 原文**:
> "As for dynamic loading module, the shared library to be loaded needs signature verification or source restriction to ensure security." (README.md:73)

**可利用点**:
- 路径遍历加载恶意 `.so` 文件
- 未校验的符号解析

**代码证据**: `components/dynlink/los_dynlink.c` (26KB 实现)

**修复建议**:
- 实现签名校验 (RSA/ECDSA)
- 限制加载路径白名单
- 校验 ELF 文件头完整性

### 2. 系统调用接口

**组件**: `kal/libc/syscall/` (syscall.c)

**风险等级**: 中

**描述**: 系统调用入口可能存在参数校验不足。

**代码证据**:
- `kal/libc/syscall/syscall.c`
- `kal/libc/syscall/syscall.h`

**可利用点**:
- 空指针解引用 (null dereference)
- 整数溢出
- 缓冲区溢出

**修复建议**:
- 所有参数添加 NULL 检查
- 边界校验
- 使用 `__attribute__((nonnull))`

### 3. 消息队列 (los_queue)

**组件**: `kernel/src/los_queue.c`

**风险等级**: 低

**描述**: 多任务间消息传递。

**代码证据**: `kernel/src/los_queue.c:1` (27KB)

**可利用点**:
- 队列满时的拒绝服务
- 消息长度溢出

**修复建议**:
- 添加消息最大长度限制
- 超时机制防阻塞

### 4. 内存分配

**组件**: `kernel/src/mm/`

**风险等级**: 中

**描述**: 动态内存分配可能存在 double-free 或 use-after-free。

**代码证据**:
- `kernel/include/los_memory.h`
- `kernel/src/mm/`

**可利用点**:
- 双重释放 (double-free)
- 释放后使用 (use-after-free)
- 堆溢出

**修复建议**:
- 启用 `LOSCFG_KERNEL_MEM_SAFE_CHECK`
- 使用内存保护机制
- 添加调试模式检测

### 5. Shell 命令注入

**组件**: `components/shell/`

**风险等级**: 低

**描述**: Shell 命令解析可能受注入攻击。

**代码证据**: `components/shell/`

**可利用点**:
- 命令字符串拼接
- 特殊字符未转义

**修复建议**:
- 输入过滤
- 命令白名单

### 6. 栈溢出

**组件**: `kernel/src/los_task.c`

**风险等级**: 高

**描述**: 任务栈溢出检测不足。

**代码证据**: `kernel/src/los_task.c` (50KB)

**可利用点**:
- 深度递归导致栈溢出
- 大局部变量

**修复建议**:
- 启用栈保护 (`-fstack-protector`)
- 添加栈溢出检测
- 栈 Canary 机制

### 7. 权限缺失

**组件**: 整体

**风险等级**: 低

**描述**: 本内核为单特权模式，无用户/内核态分离。

**说明**:
- LiteOS-M 面向 MCU，无 MMU
- 所有代码运行在特权模式
- 依赖芯片级内存保护 (MPU)

## 信任边界

```
+-------------------+
|  外部输入 (串口/网)|
+-------------------+
         ↓
+-------------------+
|  Shell 解析层     | ← 信任边界1
+-------------------+
         ↓
+-------------------+
|  Syscall 接口     | ← 信任边界2
+-------------------+
         ↓
+-------------------+
|  LOS_* 内核 API   | ← 核心代码
+-------------------+
         ↓
+-------------------+
|  硬件抽象层 (HAL) |
+-------------------+
```

## 安全配置建议

| 配置项 | 建议值 | 说明 |
|--------|--------|------|
| `LOSCFG_DEBUG_VERSION` | `n` | 生产版本关闭调试 |
| `LOSCFG_COMPONENTS_DYNLINK` | 关闭或受限 | 动态加载高风险 |
| `LOSCFG_KERNEL_MEM_SAFE_CHECK` | `y` | 内存安全检查 |
| `LOSCFG_SECURITY_CAPABILITY` | `y` | 启用安全能力 |

## 已识别问题汇总

| # | 风险点 | 等级 | 证据 | 修复建议 |
|---|--------|------|------|----------|
| 1 | 动态加载无签名验证 | 高 | README.md:73 | 实现签名校验 |
| 2 | 系统调用参数校验不足 | 中 | syscall.c | 添加边界检查 |
| 3 | 内存双重释放 | 中 | mm/ | 启用安全检查 |
| 4 | 栈溢出检测不足 | 高 | los_task.c | 启用栈保护 |
| 5 | Shell 注入 | 低 | shell/ | 输入过滤 |

## 安全最佳实践

1. **禁用危险组件**: 不使用 dynlink 时关闭
2. **最小权限**: 任务使用最小必要栈空间
3. **输入验证**: 所有外部输入必须校验
4. **安全编译**: 启用编译器安全选项
5. **运行时检测**: 启用内存 sanitizer (lms)

## 相关文档

- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [构建系统](07_Build.md)
