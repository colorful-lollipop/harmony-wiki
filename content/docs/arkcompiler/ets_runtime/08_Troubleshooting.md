# 常见问题与定位路径

本文档汇总 ArkCompiler ETS Runtime 的常见构建、运行和调试问题，提供问题定位路径和解决方案。

## 构建问题

### 1. GN 构建失败

**问题描述**：`gn gen` 或 `ninja` 构建失败

**定位路径**：

```
1. 检查错误输出
   └── 查看终端错误信息

2. 检查依赖配置
   └── js_runtime_config.gni
   └── BUILD.gn

3. 检查环境变量
   └── PYTHONPATH
   └── clang/llvm 路径
```

**常见原因**：

| 原因 | 解决方案 |
|------|----------|
| Python 版本不兼容 | 使用 Python 3.8+ |
| 依赖库缺失 | 安装 libuv、zlib 等依赖 |
| Ninja 版本过低 | 升级 Ninja 到 1.10+ |
| 磁盘空间不足 | 清理构建缓存 |

**调试命令**：

```bash
# 查看详细错误
ninja -C out/<target> -v

# 清理后重新构建
rm -rf out/<target>
gn gen out/<target>
ninja -C out/<target>
```

### 2. 编译选项不生效

**问题描述**：修改了编译选项但未生效

**定位路径**：

```
1. 检查 gn args 配置
   └── out/<target>/args.gn

2. 检查配置继承链
   └── js_runtime_config.gni
   └── BUILD.gn 中的 configs
```

**解决方案**：

```bash
# 查看当前配置
gn args out/<target> --list

# 修改配置
gn args out/<target>
# 添加: enable_ark_intl=true
```

## 运行问题

### 1. 运行时库加载失败

**问题描述**：`./ark_js_vm` 运行时报 "cannot open shared object file"

**定位路径**：

```
1. 检查 LD_LIBRARY_PATH
   └── echo $LD_LIBRARY_PATH

2. 检查库文件是否存在
   └── out/<target>/arkcompiler/ets_runtime/libark_jsruntime.so

3. 检查依赖库
   └── ldd out/<target>/arkcompiler/ets_runtime/libark_jsruntime.so
```

**解决方案**：

```bash
# 设置库路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:\
  out/<target>/arkcompiler/ets_runtime:\
  out/<target>/thirdparty/icu:\
  prebuilts/clang/ohos/linux-x86_64/llvm/lib

# 运行
./ark_js_vm helloworld.abc
```

### 2. ABC 文件加载失败

**问题描述**：加载 ABC 文件时崩溃或报错

**定位路径**：

```
1. 检查文件路径
   └── abc 文件是否存在

2. 检查文件格式
   └── file <abc_file>

3. 检查字节码版本
   └── 使用 hexdump 查看文件头
```

**调试方法**：

```bash
# 检查文件
file helloworld.abc

# 查看文件头
hexdump -C helloworld.abc | head -n 5
```

### 3. 内存不足错误

**问题描述**：运行时报 "Out of memory"

**定位路径**：

```
1. 检查堆内存配置
   └── Runtime 配置选项

2. 检查系统内存
   └── free -h

3. 检查进程限制
   └── ulimit -a
```

**解决方案**：

```bash
# 增加进程内存限制
ulimit -v unlimited

# 调整 GC 参数
# 在代码中设置 Runtime 选项
```

## 调试问题

### 1. 调试器连接失败

**问题描述**：无法连接到调试器

**定位路径**：

```
1. 检查调试端口
   └── 默认端口 5001

2. 检查防火墙
   └── iptables -L

3. 检查调试配置
   └── debug 配置选项
```

**调试命令**：

```bash
# 启动调试模式
./ark_js_vm --debugger --port=5001 helloworld.abc

# 使用 Chrome DevTools
# 打开 chrome://inspect
```

### 2. 性能分析数据收集

**问题描述**：无法收集性能分析数据

**定位路径**：

```
1. 检查 profiler 配置
   └── ECMA 性能分析器配置

2. 检查输出路径
   └── profiler 输出目录

3. 检查权限
   └── 写入权限
```

**调试命令**：

```bash
# 收集 CPU Profiler 数据
./ark_js_vm --cpu-profiler --profiler-output=profile.json helloworld.abc

# 收集 Heap Profiler 数据
./ark_js_vm --heap-profiler --profiler-output=heap.json helloworld.abc
```

### 3. 崩溃问题定位

**问题描述**：运行时崩溃

**定位路径**：

```
1. 检查崩溃日志
   └── dmesg | tail
   └── /var/log/syslog

2. 使用 ASAN 构建
   └── 构建时启用 ASAN

3. 检查调用栈
   └── addr2line -e <binary> <address>
```

**调试方法**：

```bash
# 使用 ASAN 构建
gn args out/<target>
# 添加: is_asan=true

# 运行并捕获崩溃
./ark_js_vm helloworld.abc

# 分析 core 文件
gdb ./ark_js_vm core
```

## 日志与诊断

### 启用详细日志

```bash
# 设置日志级别
export PANDA_LOG_LEVEL=DEBUG

# 运行
./ark_js_vm helloworld.abc
```

### 常用日志标签

| 标签 | 说明 |
|------|------|
| `GC` | 垃圾回收日志 |
| `Interpreter` | 解释器日志 |
| `Compiler` | 编译器日志 |
| `Module` | 模块加载日志 |
| `NAPI` | N-API 调用日志 |

## 相关文档

- [架构说明](../03_Architecture.md)
- [N-API 参考](../04_NAPI_Reference.md)
- [构建系统](../06_Build_System.md)
- [配置开关](../appendix/Config_Flags.md)
