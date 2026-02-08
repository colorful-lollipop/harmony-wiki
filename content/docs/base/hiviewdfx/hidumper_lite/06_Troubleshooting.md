# 问题排查指南

## 概述

本文档收集了 `hidumper_lite` 项目在构建、运行过程中常见的问题及其解决方案。问题按类别组织，并提供详细的排查步骤和代码证据。

---

## 构建问题

### 问题一：编译报错找不到头文件

**错误信息**：

```
fatal error: xxx.h: No such file or directory
```

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| 依赖未编译 | 相关子系统未先编译 | 先编译依赖子系统 |
| 头文件路径错误 | include_dirs 配置不正确 | 检查 BUILD.gn 配置 |
| 工具链未配置 | 交叉编译器未安装 | 配置 ARM 交叉编译工具链 |

**排查步骤**：

```bash
# 1. 检查依赖是否已编译
hb build -f

# 2. 检查头文件是否存在
ls -la {依赖路径}/include/

# 3. 检查 BUILD.gn 配置
cat mini/BUILD.gn | grep include_dirs
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `mini/BUILD.gn:19-25` | include_dirs 配置 |
| `bundle.json:25-29` | deps 依赖配置 |

---

### 问题二：链接失败

**错误信息**：

```
undefined reference to `xxx'
```

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| 依赖库未链接 | deps 配置缺失 | 添加缺失的 deps |
| 库顺序错误 | 链接顺序导致符号未解析 | 调整 deps 顺序 |
| 库版本不匹配 | 依赖库版本不一致 | 确保版本一致 |

**排查步骤**：

```bash
# 1. 查看链接命令
cat out/{product}/build.ninja | grep "ld "

# 2. 检查依赖库是否存在
ls out/{product}/libs/

# 3. 检查符号是否在库中
nm out/{product}/libs/libxxx.a | grep xxx
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `lite/BUILD.gn:18` | deps 配置 |
| `mini/BUILD.gn:28` | deps 配置 |

---

### 问题三：条件编译代码未生效

**错误信息**：调试功能打印 "Unsupported!"

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| OHOS_DEBUG 未定义 | 调试宏未定义 | 在产品配置中添加 |
| 产品类型错误 | 选择了非调试版本 | 选择正确的产品 |

**排查步骤**：

```bash
# 1. 检查编译标志
arm-linux-gnueabi-gcc -dM -E - < /dev/null | grep OHOS_DEBUG

# 2. 检查产品配置
cat {product}/config.json | grep debug
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `lite/hidumper.c:120` | OHOS_DEBUG 条件编译 |
| `lite/hidumper.c:134` | OHOS_DEBUG 条件编译 |

---

## 运行问题

### 问题一：执行时提示 "Failed to open [/dev/hidumper]"

**错误信息**：

```
Failed to open [/dev/hidumper], error [No such file or directory]
```

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| 设备节点未创建 | 内核驱动未加载 | 检查内核配置 |
| 设备节点路径错误 | 路径配置不正确 | 检查设备路径 |
| 权限不足 | 无访问权限 | 添加访问权限 |

**排查步骤**：

```bash
# 1. 检查设备节点是否存在
ls -la /dev/hidumper

# 2. 检查内核日志
dmesg | grep hidumper

# 3. 检查驱动是否加载
lsmod | grep hidumper
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `lite/hidumper.c:28` | 设备路径定义 |
| `lite/hidumper.c:208-211` | 设备打开失败处理 |

---

### 问题二：执行时提示 "No adapter has been registered!"

**错误信息**：

```
No adapter has been registered!
```

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| 初始化未执行 | CORE_INIT_PRI 未触发 | 检查初始化优先级 |
| 适配器注册失败 | HiDumperRegisterAdapter 返回错误 | 检查平台实现 |
| 平台代码缺失 | 平台未实现适配器 | 添加平台实现 |

**排查步骤**：

```bash
# 1. 检查适配器注册标志
# 在代码中添加调试日志
if (g_isAdapterRegistered == 0) {
    printf("Debug: Adapter not registered\n");
}

# 2. 检查初始化函数是否被调用
# 在 HiDumperAdapterInit 中添加日志
printf("Debug: HiDumperAdapterInit called\n");
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `mini/hidumper_core.c:35` | g_isAdapterRegistered 标志 |
| `mini/hidumper_core.c:157-160` | 未注册检测 |
| `mini/hidumper_adapter.c:84-101` | 初始化函数 |

---

### 问题三：AT 命令无响应

**现象**：发送 AT+HIDUMPER 命令无响应

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| AT 框架未初始化 | AT 任务未启动 | 检查 AT 框架配置 |
| at_hidumper 未注册 | AT 命令回调未注册 | 检查 AT 命令注册 |
| 参数解析错误 | 参数格式不正确 | 检查参数格式 |

**排查步骤**：

```bash
# 1. 检查 AT 任务状态
# 在目标设备上查看任务列表

# 2. 发送基本 AT 命令测试
AT

# 3. 检查 at_hidumper 返回值
# 在代码中添加返回值日志
unsigned int ret = at_hidumper(argc, argv);
printf("Debug: at_hidumper returned %u\n", ret);
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `mini/hidumper_core.c:155-165` | at_hidumper 函数 |
| `mini/hidumper_core.c:105-153` | ParameterMatching 函数 |

---

### 问题四：内存转储输出为空

**现象**：执行内存转储命令后无输出

**常见原因**：

| 原因 | 描述 | 解决方案 |
|------|------|----------|
| 调试功能未启用 | OHOS_DEBUG 未定义 | 启用调试版本 |
| 内存地址无效 | 地址超出有效范围 | 检查地址范围 |
| 内核不支持 | 内核驱动未实现 | 检查内核驱动 |

**排查步骤**：

```bash
# 1. 确认调试版本已编译
strings hidumper | grep OHOS_DEBUG

# 2. 尝试有效内存地址
hidumper -m 0x20000000 0x100

# 3. 检查内核支持
cat /proc/devices | grep hidumper
```

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `lite/hidumper.c:120-130` | DumpMemData 函数 |
| `mini/hidumper_core.c:126-130` | DumpAllMem 条件编译 |

---

## 功能问题

### 问题一：转储信息不完整

**现象**：部分转储功能无输出

**排查步骤**：

1. 检查平台适配器是否完整实现所有函数
2. 检查函数指针是否为空
3. 检查平台实现是否有内部错误

**代码证据**：

| 检查项 | 证据位置 |
|--------|----------|
| 适配器函数指针校验 | `mini/hidumper_core.c:86-95` |
| 弱函数默认实现 | `mini/hidumper_adapter.c:33-82` |

---

### 问题二：命令参数解析错误

**现象**：参数格式正确但解析失败

**排查步骤**：

1. 检查参数分隔符（空格 vs 逗号）
2. 检查十六进制格式（前缀 0x）
3. 检查参数个数

**AT 命令格式对比**：

| 版本 | 格式示例 |
|------|----------|
| LiteOS_A | `hidumper -m 0x20000000 0x100` |
| LiteOS_M | `AT+HIDUMPER=-m,memstart,memsize` |

**代码证据**：

| 文件 | 检查项 |
|------|--------|
| `lite/hidumper.c:156-203` | 命令行参数解析 |
| `mini/hidumper_core.c:105-153` | AT 参数解析 |

---

## 调试技巧

### 技巧一：启用详细日志

在代码中添加调试日志：

```c
// 在 ParameterMatching 函数入口添加
printf("Debug: argc=%u, argv[0]=%s\n", argc, argv ? argv[0] : "NULL");

// 在各功能函数入口添加
printf("Debug: DumpCpuUsage called\n");
```

### 技巧二：检查适配器注册状态

```c
// 在 at_hidumper 入口添加
printf("Debug: g_isAdapterRegistered=%d\n", g_isAdapterRegistered);
```

### 技巧三：验证参数解析

```c
// 检查参数值
for (unsigned int i = 0; i < argc; i++) {
    printf("Debug: argv[%u]=%s\n", i, argv[i]);
}
```

---

## 日志位置

| 日志类型 | 位置 | 说明 |
|----------|------|------|
| 构建日志 | `out/{product}/build.log` | 编译过程日志 |
| 运行日志 | 串口输出 | 命令执行结果 |
| 内核日志 | `dmesg` | 内核驱动日志 |

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [02_Architecture.md](02_Architecture.md) | 架构说明 |
| [03_API.md](03_API.md) | 接口文档 |
| [04_Build.md](04_Build.md) | 构建配置 |
| [05_Security.md](05_Security.md) | 安全评审 |
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链 |

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0.0 | 2026-02-06 | 初始版本 |
