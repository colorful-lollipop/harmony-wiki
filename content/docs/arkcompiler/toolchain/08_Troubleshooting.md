# 故障排查指南

本文档汇总方舟工具链在构建、运行和调试过程中常见的问题及其解决方案。

## 构建问题

### 问题 1：编译找不到头文件

**错误信息**:
```
fatal error: 'ecmascript/napi/include/jsnapi.h' file not found
```

**原因**: 未正确设置 arkcompiler/runtime_core 或 ets_runtime 的 include 路径。

**解决方案**:

1. 确认构建系统正确配置了 ark_root 和 js_root：
```bash
# 在 toolchain_config.gni 中确认
ark_root = "//arkcompiler/runtime_core"
js_root = "//arkcompiler/ets_runtime"
```

2. 检查 GN 依赖配置：
```gn
# 在对应的 BUILD.gn 中确认依赖
external_deps = [
    "runtime_core:libarkruntime",
    "ets_runtime:libark_jsruntime",
]
```

### 问题 2：链接错误 - 找不到库

**错误信息**:
```
ld: library not found for -lwebsocket_server
```

**原因**: websocket 模块未正确编译或依赖配置错误。

**解决方案**:

1. 确认编译了正确的 target：
```bash
./build.sh --product-name <product> --build-target ark_toolchain_packages
```

2. 检查 inspector 对 websocket 的依赖：
```gn
# inspector/BUILD.gn
deps = [ "../websocket:libwebsocket_server" ]
```

### 问题 3：平台相关编译错误

**错误信息**:
```
error: use of undeclared identifier 'shared_mutex'
```

**原因**: macOS 平台的 C++ 标准库版本不支持 `std::shared_mutex`。

**解决方案**:

检查 C++ 标准库配置（`BUILD.gn`）：
```gn
if (is_mingw || is_mac) {
    cflags = [ "-std=c++17" ]  # 确保使用 C++17
}
```

## 运行问题

### 问题 1：WebSocket 连接失败

**错误信息**:
```
WebSocket connection failed: Connection refused
```

**可能原因**:
1. 调试服务器未启动
2. 端口被占用
3. 防火墙阻止连接

**排查步骤**:

1. 检查服务器是否运行：
```bash
# 查看进程
ps aux | grep ark_inspector
```

2. 检查端口监听：
```bash
# Linux
netstat -tlnp | grep <port>

# OHOS
hdc shell netstat -tlnp
```

3. 检查日志输出（启用 hilog）：
```bash
# OHOS 设备
hdc shell hilog -x | grep -i inspector
```

### 问题 2：断点无法命中

**可能原因**:
1. 断点设置在未执行的代码路径
2. 源文件与字节码不同步
3. 调试会话未正确初始化

**排查步骤**:

1. 确认断点位置在可执行代码中：
```bash
# 在 DevEco Studio 中检查断点图标状态
# 灰色断点：未绑定到任何代码
# 红色断点：已绑定，等待命中
```

2. 验证源文件与 ABC 文件同步：
```bash
# 重新编译应用确保源文件同步
hdc shell rm -rf /data/cache/*
```

3. 检查调试会话日志：
```bash
# 查看调试初始化日志
hdc shell hilog -x | grep -i debugger
```

### 问题 3：CPU Profiler 数据异常

**现象**: 采样数据显示空白或异常数据

**可能原因**:
1. Profiler 功能未启用（平台限制）
2. 采样配置错误
3. 数据传输中断

**排查步骤**:

1. 确认平台支持：
```gn
# BUILD.gn 中确认宏定义
if (!is_mingw && !is_mac && target_os != "ios") {
    defines += [
        "ECMASCRIPT_SUPPORT_CPUPROFILER",
        "ECMASCRIPT_SUPPORT_HEAPPROFILER",
    ]
}
```

2. 检查采样配置：
```json
{
    "method": "Profiler.start",
    "params": {
        "samplingInterval": 1000  // 采样间隔（微秒）
    }
}
```

### 问题 4：内存泄漏检测误报

**现象**: HeapProfiler 报告不存在的内存泄漏

**可能原因**:
1. 缓存对象未被 GC 回收
2. 调试会话期间的对象保留
3. 静态分析工具的误报

**排查建议**:
1. 在稳定运行状态下进行多次堆快照对比
2. 区分"增长"和"泄漏"（增长不一定是泄漏）
3. 考虑调试工具本身的内存开销

## 调试技巧

### 启用详细日志

```bash
# OHOS 设备启用 hilog
hdc shell hilog -x &

# 设置日志级别
hdc shell param set debuggable.config true
```

### 本地调试（Host 模式）

```bash
# 构建 host 版本
./build.sh --product-name <host_product> --build-target ark_toolchain_host_unittest

# 运行测试
./out/<product>/tests/<test_executable>
```

### 使用 arkdb CLI 工具

```bash
# 启动命令行调试器
./out/<product>/ark_db/arkdb <panda_file>.abc

# 常用命令
(lldb) breakpoint set --name main
(lldb) run
(lldb) bt  # backtrace
```

## 常见错误码

| 错误码 | 说明 | 排查方向 |
|--------|------|----------|
| -32600 | 无效请求 | JSON 格式、协议版本 |
| -32601 | 方法未找到 | 方法名拼写、域支持 |
| -32602 | 参数无效 | 参数类型、范围 |
| -32603 | 内部错误 | 服务器日志 |
| 1000 | 正常关闭 | 正常退出 |
| 1001 | 端点离开 | 连接断开 |
| 1009 | 消息过大 | 消息大小限制 |

## 性能问题

### 调试器性能优化

1. **减少断点数量**: 过多的断点影响性能
2. **禁用不必要的域**: 仅启用需要的调试域
3. **调整采样间隔**: Profiler 采样间隔不宜过短

### 内存优化

1. **及时关闭调试会话**: 长时间运行的调试会话会占用内存
2. **限制快照数量**: 避免同时生成多个堆快照
3. **清理临时文件**: 定期清理调试数据缓存

---

*相关文档：[05_GN_Build.md](./05_GN_Build.md) | [06_Build_Artifacts.md](./06_Build_Artifacts.md) | [07_Security_Review.md](./07_Security_Review.md)*
