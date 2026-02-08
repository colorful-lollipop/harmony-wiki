# 常见问题与故障排查

## 概述

本文档整理 ArkCompiler Runtime Core 开发和使用过程中的常见问题及排查方法。

## 构建问题

### 1. GN 构建失败

**症状**: `gn gen out` 或 `ninja` 命令失败

**排查步骤**:

```bash
# 1. 检查 GN 配置
gn args out --list

# 2. 清理并重新生成
rm -rf out
gn gen out

# 3. 详细错误信息
ninja -C out -v 2>&1 | tee build.log
```

**常见原因**:
- `ark_root` 路径配置错误
- 依赖组件未找到（如 `hilog`, `zlib`）
- 工具链配置错误

**代码位置**: `BUILD.gn`, `ark_config.gni`

### 2. 链接错误

**症状**: `undefined reference` 或 `multiple definition`

**排查方法**:

```bash
# 检查符号定义
nm -C libarkbase.so | grep SymbolName

# 检查依赖关系
ninja -C out -t deps target_name
```

**常见原因**:
- 循环依赖
- 静态库/共享库混用
- 符号未导出

**修复建议**:
- 检查 `BUILD.gn` 中的 `deps` 和 `public_deps`
- 确保头文件中的函数有实现
- 使用 `extern "C"` 处理 C 接口

### 3. 头文件找不到

**症状**: `'header.h' file not found`

**排查方法**:

```bash
# 检查 include_dirs 配置
gn desc out //target_name include_dirs

# 查找头文件位置
find . -name "header.h" -not -path "./out/*"
```

**代码位置**: 各 `BUILD.gn` 中的 `include_dirs`

## 运行时问题

### 1. VM 启动失败

**症状**: `arkts_bin` 无法启动，报错或崩溃

**排查步骤**:

```bash
# 1. 检查日志
arkts_bin --log-level=debug app.abc Entry 2>&1 | tee vm.log

# 2. 检查依赖库
ldd /system/bin/ark

# 3. 验证 ABC 文件
ark_verifier app.abc
```

**常见原因**:
- ABC 文件损坏或格式不兼容
- 缺少启动类（boot-panda-files 配置错误）
- 内存不足

**代码位置**:
- `static_core/panda/panda.cpp` - 入口点
- `static_core/runtime/include/runtime.h` - 运行时初始化

### 2. 类找不到

**症状**: `ClassNotFoundException` 或类似错误

**排查方法**:

```bash
# 1. 检查 ABC 文件内容
arkts_disasm app.abc | grep "class"

# 2. 确认类路径
# 类名格式: "Lpackage/ClassName;"
```

**代码位置**:
- `static_core/runtime/include/class_linker.h` - 类加载
- `static_core/libarkfile/` - 文件解析

**修复建议**:
- 检查类名拼写和格式
- 确认 ABC 文件已正确加载
- 检查 ClassLinkerContext 配置

### 3. 内存不足 / OOM

**症状**: `OutOfMemoryError` 或进程被杀死

**排查方法**:

```bash
# 1. 查看内存使用
cat /proc/$(pidof ark)/status | grep VmRSS

# 2. 启用内存日志
arkts_bin --log-gc --gc-heap-size=512m app.abc Entry
```

**代码位置**:
- `static_core/runtime/mem/heap_manager.h` - 堆管理
- `static_core/runtime/mem/gc/gc.h` - GC

**修复建议**:
- 增加堆大小：`--gc-heap-size=1024m`
- 检查内存泄漏
- 优化对象创建

### 4. GC 暂停过长

**症状**: 应用卡顿，GC 日志显示长暂停

**排查方法**:

```bash
# 启用 GC 日志
arkts_bin --log-gc --gc-verbose app.abc Entry

# 分析 GC 统计
# 查看 pause time 和 throughput
```

**代码位置**:
- `static_core/runtime/mem/gc/g1/g1-gc.h` - G1 GC
- `static_core/runtime/mem/gc/gc_stats.h` - GC 统计

**修复建议**:
- 使用 G1 GC（默认）
- 调整 GC 参数：`--gc-target-pause-time=10`
- 减少大对象分配

## ANI/N-API 问题

### 1. ANI 调用失败

**症状**: ANI API 返回错误码

**排查方法**:

```cpp
// 检查错误码
ani_status status = ani_find_class(env, name, &cls);
if (status != ANI_OK) {
    // 打印错误码
    LOG_ERROR("ANI error: %d", status);
    
    // 检查异常
    ani_boolean has_exception;
    ani_exception_check(env, &has_exception);
}
```

**代码位置**:
- `static_core/plugins/ets/runtime/ani/ani_interaction_api.cpp`

**常见错误码**:
- `ANI_INVALID_ARGS` - 参数错误
- `ANI_NOT_FOUND` - 类/方法未找到
- `ANI_OUT_OF_MEMORY` - 内存不足

### 2. N-API 模块加载失败

**症状**: JS 无法调用 ETS 函数

**排查方法**:

```javascript
// 检查模块是否加载
try {
    const etsvm = require('etsvm');
    console.log('ETS VM loaded:', etsvm.Version());
} catch (e) {
    console.error('Failed to load:', e);
}
```

**代码位置**:
- `static_core/plugins/ets/runtime/interop_js/ets_vm_plugin.cpp`

**修复建议**:
- 确认 `ETS_INTEROP_JS_NAPI` 模块已注册
- 检查 N-API 版本兼容性
- 查看系统日志

### 3. 类型转换错误

**症状**: 数据类型不匹配，调用失败

**排查方法**:

```cpp
// 检查参数类型
ani_type type;
ani_get_object_type(env, obj, &type);
// 与期望类型比较
```

**代码位置**:
- `static_core/plugins/ets/runtime/ani/ani_converters.h`

## 调试技巧

### 1. 启用调试日志

```bash
# 设置日志级别
export ARK_LOG_LEVEL=DEBUG

# 运行应用
arkts_bin app.abc Entry
```

**代码位置**:
- `libpandabase/utils/logger.h`
- `static_core/libarkbase/utils/logger.h`

### 2. 使用 GDB 调试

```bash
# 启动 GDB
gdb --args arkts_bin app.abc Entry

# 常用命令
(gdb) break Runtime::Create
(gdb) run
(gdb) bt          # 查看调用栈
(gdb) info locals # 查看局部变量
(gdb) continue
```

### 3. 堆栈分析

```bash
# 崩溃时获取堆栈
cat /data/crash/app_crash.log

# 使用 addr2line
addr2line -e libarkruntime.so -C -f <address>
```

**代码位置**:
- `static_core/runtime/tooling/backtrace/`
- `libpandabase/os/native_stack.cpp`

### 4. 内存调试

```bash
# 使用 Address Sanitizer
./build.sh --asan

# 检查内存泄漏
export ARK_TRACK_ALLOCATIONS=1
arkts_bin app.abc Entry
```

**代码位置**:
- `static_core/runtime/mem/alloc_tracker.h`

## 性能问题

### 1. 启动慢

**排查方法**:

```bash
# 测量启动时间
time arkts_bin app.abc Entry

# 启用性能分析
arkts_bin --enable-sampling app.abc Entry
```

**优化建议**:
- 使用 AOT 编译：`ark_aot` 生成 .an 文件
- 减少启动类加载
- 延迟初始化

### 2. 执行慢

**排查方法**:

```bash
# 启用 JIT
arkts_bin --compiler-enable-jit app.abc Entry

# 查看热点方法
arkts_bin --compiler-dump-stats app.abc Entry
```

**代码位置**:
- `static_core/compiler/` - JIT 编译器
- `static_core/runtime/interpreter/` - 解释器

**优化建议**:
- 启用 JIT 编译
- 优化热点代码
- 调整编译阈值：`--compiler-hotness-threshold=1000`

## 平台相关问题

### 1. OpenHarmony 特定问题

**症状**: 在 OpenHarmony 上运行异常

**排查方法**:

```bash
# 检查系统日志
hilog | grep ark

# 检查权限
ls -la /system/lib64/libark*
```

**代码位置**:
- `platforms/ohos/` - OpenHarmony 适配
- `arkplatform/` - 平台层

### 2. 跨平台兼容性

**症状**: 代码在 Linux 正常，在 OpenHarmony 异常

**排查方法**:

```cpp
// 检查平台宏
#ifdef PANDA_TARGET_OHOS
    // OpenHarmony 特定代码
#elif PANDA_TARGET_LINUX
    // Linux 特定代码
#endif
```

**代码位置**:
- `platforms/common/` - 通用平台代码
- `platforms/unix/` - Unix 实现
- `platforms/ohos/` - OpenHarmony 实现

## 工具使用

### 1. 反汇编器

```bash
# 反汇编 ABC 文件
ark_disasm app.abc app.pa

# 查看特定方法
ark_disasm --method="Main.main" app.abc
```

### 2. 验证器

```bash
# 验证 ABC 文件
ark_verifier --load-runtimes=ets app.abc

# 详细验证
ark_verifier --verbose app.abc
```

### 3. AOT 编译器

```bash
# 编译为 AOT
ark_aot --boot-panda-files=etsstdlib.abc \
        --load-runtimes=ets \
        --paoc-panda-files=app.abc \
        --paoc-output=app.an

# 运行 AOT
arkts_bin --enable-an app.abc Entry
```

## 获取帮助

### 日志收集模板

```bash
#!/bin/bash
# collect_debug_info.sh

echo "=== System Info ==="
uname -a
cat /etc/os-release

echo "=== Ark Version ==="
arkts_bin --version

echo "=== Libraries ==="
ls -la /system/lib64/libark*

echo "=== Running Process ==="
ps | grep ark

echo "=== Logs ==="
hilog | grep -i "ark\|error\|crash" | tail -100

echo "=== Memory ==="
cat /proc/meminfo

echo "=== ABC File ==="
file app.abc
ls -la app.abc
```

### 提交 Issue 模板

```
问题描述:
[清晰描述问题]

复现步骤:
1. 
2. 
3. 

期望结果:
[期望发生什么]

实际结果:
[实际发生什么]

环境信息:
- OS: [e.g. OpenHarmony 4.0]
- Arch: [e.g. ARM64]
- Version: [commit hash]

日志:
[粘贴相关日志]
```

## 参考资源

- [Runtime Core README](../README.md)
- [静态核心开发指南](../static_core/README.md)
- [设计文档](../docs/)
- [OpenHarmony 文档](https://gitee.com/openharmony/docs)

## 下一步

- 查看 [对外 API](04_Public_API.md) 了解 API 使用
- 参考 [安全风险](08_Security.md) 了解安全问题
- 阅读源码 `docs/` 目录获取更多设计信息
