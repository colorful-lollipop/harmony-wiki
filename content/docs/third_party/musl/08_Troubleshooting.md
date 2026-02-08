# 常见问题排查

> OpenHarmony musl 常见问题与定位方法

---

## 目的与适用范围

**目的**: 提供 musl 构建、运行和调试过程中的常见问题定位方法。

**适用范围**: 开发者、测试人员、维护工程师。

---

## 构建问题

### 问题 1: 编译失败 - 头文件找不到

**现象**:
```
error: 'elf.h' file not found
```

**原因**:
- 生成的头文件未生成
- include 路径配置错误

**定位方法**:
```bash
# 检查生成的头文件
ls -la out/{target}/obj/third_party/musl/usr/include/

# 检查依赖关系
ninja -C out -t deps third_party/musl:musl_headers
```

**解决方案**:
1. 确保 `create_alltypes_h` 等 target 已执行
2. 检查 `musl_inc_out_dir` 配置

---

### 问题 2: 链接错误 - 符号未定义

**现象**:
```
undefined reference to `__libc_malloc_impl'
```

**原因**:
- 源文件未正确编译
- 符号被错误过滤

**定位方法**:
```bash
# 检查符号是否在库中
nm out/{target}/obj/third_party/musl/libc.a | grep malloc

# 检查符号导出
grep malloc third_party/musl/libc.map.txt
```

**解决方案**:
1. 检查 `musl_src_file` 列表
2. 检查 `libc.map.txt` 导出配置

---

### 问题 3: 架构不匹配

**现象**:
```
error: incompatible architecture
```

**原因**:
- 编译目标架构与实际不符
- 交叉编译配置错误

**定位方法**:
```bash
# 检查当前架构
echo $musl_arch

# 检查编译器配置
echo $current_cpu
```

**解决方案**:
1. 确认 `musl_arch` 与 `current_cpu` 匹配
2. 检查工具链配置

---

## 运行时问题

### 问题 4: 程序启动失败 - 动态链接器错误

**现象**:
```
/system/bin/ld-musl-aarch64.so.1: not found
```

**原因**:
- 动态链接器未正确安装
- 路径配置错误

**定位方法**:
```bash
# 检查动态链接器存在
ls -la /system/bin/ld-musl-*

# 检查程序 INTERP 段
readelf -l /path/to/program | grep INTERP
```

**解决方案**:
1. 确保 `soft_create_linker` 执行成功
2. 检查 `install_images` 配置

---

### 问题 5: 库加载失败 - Namespace 错误

**现象**:
```
cannot open shared object file: No such file or directory
```

**原因**:
- 库不在 Namespace 搜索路径中
- Namespace 配置错误

**定位方法**:
```bash
# 检查 Namespace 配置
cat /etc/ld-musl-namespace-aarch64.ini

# 检查库路径
ls -la /system/lib64/libxxx.so
```

**解决方案**:
1. 检查 `namespace.default.lib.paths` 配置
2. 确认库文件存在且权限正确

---

### 问题 6: 内存分配失败

**现象**:
- malloc 返回 NULL
- 程序异常退出

**原因**:
- 内存不足
- 内存碎片
- 安全机制触发

**定位方法**:
```bash
# 检查内存状态
cat /proc/meminfo

# 启用 musl 日志
setprop libc.hook_mode debug
```

**解决方案**:
1. 检查 `musl_secure_level` 设置
2. 调整安全级别或内存限制

---

## 调试方法

### 启用日志

```c
// 在代码中添加日志
#include "musl_log.h"
MUSL_LOG(MUSL_LOG_INFO, "message");
```

### 使用 GDB

```bash
# 调试动态链接器
gdb /system/bin/ld-musl-aarch64.so.1

# 设置断点
b dlopen
b malloc

# 运行程序
run /path/to/program
```

### 检查符号

```bash
# 查看动态符号
readelf -s libc.so | grep GLOBAL

# 查看静态符号
nm libc.a | grep T
```

---

## 性能问题

### 问题 7: 内存分配性能下降

**原因**:
- 高安全级别启用过多检查
- GWP-ASan 采样率过高

**定位方法**:
```bash
# 检查当前配置
grep musl_secure_level out/args.gn

# 性能分析
perf record -g ./program
perf report
```

**解决方案**:
1. 生产环境降低 `musl_secure_level`
2. 调整 GWP-ASan 采样率

---

## Hook 相关问题

### 问题 8: Hook 不生效

**现象**:
- Hook 函数未被调用
- 内存追踪无输出

**原因**:
- Hook 未启用
- Hook 库加载失败

**定位方法**:
```bash
# 检查 Hook 标志
grep HOOK_ENABLE out/args.gn

# 检查 Hook 库
ls -la /system/lib64/libmemleak*.so
```

**解决方案**:
1. 确认 `HOOK_ENABLE` 宏已定义
2. 检查 Hook 库路径和权限

---

## 相关跳转

- [GN 构建目标](05_GN_Targets.md) - 构建配置
- [编译产物](06_Build_Artifacts.md) - 输出文件
- [安全分析](07_Security_Analysis.md) - 安全相关问题

