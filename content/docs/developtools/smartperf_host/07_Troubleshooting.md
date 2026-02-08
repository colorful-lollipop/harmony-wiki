# SmartPerf 问题排查指南

## 概述

本文档汇总 SmartPerf 常见构建、运行和调试问题，提供排查路径和解决方案。

**代码位置**: `smartperf_device/` 和 `smartperf_host/`

## 构建问题

### 1. GN 构建失败

**错误现象**:
```
gn gen out/[config] failed
```

**排查步骤**:

```bash
# 1. 检查 GN 工具链
which gn
gn --version

# 2. 检查 Ninja
which ninja
ninja --version

# 3. 检查 OHOS SDK
echo $OHOS_SDK
ls $OHOS_SDK/build-tools/

# 4. 重新生成构建配置
rm -rf out/[config]
gn gen out/[config] --check
```

**常见原因**:

| 原因 | 解决方案 |
|------|----------|
| GN 版本不兼容 | 使用 OHOS SDK 内置 GN (`$OHOS_SDK/gn/gn`) |
| Ninja 未安装 | `brew install ninja` 或 `apt install ninja-build` |
| 路径配置错误 | 检查 `OHOS_SDK` 环境变量 |
| 依赖缺失 | `hb set` 配置依赖 |

### 2. 编译错误：头文件找不到

**错误现象**:
```
fatal error: 'xxx.h' file not found
```

**排查步骤**:

```bash
# 1. 检查 include_dirs 配置
cat smartperf_device/device_command/BUILD.gn | grep include_dirs

# 2. 检查依赖路径
cat smartperf_device/device_command/BUILD.gn | grep external_deps

# 3. 验证头文件存在
find $OHOS_SDK -name "xxx.h"
```

**解决方案**:

```gn
# 修改 BUILD.gn 添加路径
include_dirs = [
  ".",
  "include",
  "$OHOS_SDK/path/to/headers",
]
```

### 3. 链接错误：符号未定义

**错误现象**:
```
undefined reference to `xxx'
```

**排查步骤**:

```bash
# 1. 检查依赖的库
cat smartperf_device/device_command/BUILD.gn | grep external_deps

# 2. 查找符号定义
nm $OHOS_SDK/libs/*.a | grep xxx

# 3. 检查链接顺序
# GN deps 中顺序可能导致链接问题
```

**解决方案**:

```gn
deps = [
  ":smartperf_daemon",
  # 确保正确顺序
]
```

### 4. HAP 构建失败

**错误现象**:
```
error: failed to compile ets module
```

**排查步骤**:

```bash
# 1. 检查 ArkTS SDK
ls $OHOS_SDK/ets-sdk/

# 2. 检查 Node.js 版本
node --version  # 需要 Node.js 18+

# 3. 清理并重新构建
rm -rf node_modules
npm install
```

**解决方案**:

```bash
# 使用正确的 Node.js 版本
nvm use 18
npm install
```

## 运行问题

### 1. SP_daemon 启动失败

**错误现象**:
```
sp_daemon: cannot execute
```

**排查步骤**:

```bash
# 1. 检查文件权限
ls -la /system/bin/sp_daemon

# 2. 检查依赖库
ldd /system/bin/sp_daemon

# 3. 查看系统日志
hilog | grep -i smartperf

# 4. 检查错误码
hilog | grep -i error
```

**常见原因与解决方案**:

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| Permission denied | 权限不足 | `chmod +x sp_daemon` |
| Library not found | 依赖库缺失 | 检查 `ldd` 输出，补齐依赖 |
| Segmentation fault | 内存错误 | 使用 `gdb` 调试 |
| Socket bind failed | 端口占用 | `lsof -i :8283` |

### 2. Socket 连接失败

**错误现象**:
```
connect to localhost:8283 failed: Connection refused
```

**排查步骤**:

```bash
# 1. 检查 SP_daemon 是否运行
ps aux | grep sp_daemon

# 2. 检查端口监听
netstat -tlnp | grep 828

# 3. 检查 Token 配置
cat /etc/smartperf/token.conf

# 4. 测试 UDP 连接
nc -u localhost 8283
```

**解决方案**:

```bash
# 启动 SP_daemon
./libsmartperf_daemon.z.so

# 或指定端口
./libsmartperf_daemon.z.so -p 8284
```

### 3. device_ui 悬浮窗不显示

**错误现象**:
- 悬浮窗无法启动
- 无数据显示

**排查步骤**:

```bash
# 1. 检查悬浮窗权限
hdc shell dumpsys permission | grep SYSTEM_FLOAT_WINDOW

# 2. 检查 HAP 安装
hdc shell bm get --quick-mode com.ohos.gameperceptio

# 3. 检查 SP_daemon 连接
hdc shell ./libsmartperf_daemon.z.so -c "status"

# 4. 查看应用日志
hdc shell hilog | grep -i "SmartPerf\|gameperceptio"
```

**解决方案**:

```bash
# 授予悬浮窗权限
hdc shell shell pm grant com.ohos.gameperceptio ohos.permission.SYSTEM_FLOAT_WINDOW

# 重启应用
hdc shell aa force-stop com.ohos.gameperceptio
hdc shell aa start -b com.ohos.gameperceptio
```

### 4. trace_streamer 解析失败

**错误现象**:
```
Parse failed: invalid trace format
```

**排查步骤**:

```bash
# 1. 检查输入文件格式
file trace_data.dat
hexdump -C trace_data.dat | head

# 2. 检查文件完整性
md5sum trace_data.dat

# 3. 调试解析过程
./trace_streamer -d trace_data.dat

# 4. 查看错误日志
./trace_streamer --log-level debug trace_data.dat
```

**支持的输入格式**:

| 格式 | 文件扩展名 | 说明 |
|------|------------|------|
| ftrace | `.txt`, `.dat` | 文本格式 |
| Hiperf | `.perf`, `.pb` | Protobuf 格式 |
| HiSysEvent | `.he` | 事件格式 |
| XPower | `.xp` | 功耗数据 |

### 5. WASM 模块加载失败

**错误现象**:
```
WebAssembly: invalid magic number
```

**排查步骤**:

```javascript
// 1. 检查 WASM 文件
const buffer = await fetch('trace_streamer_builtin.wasm').then(r => r.arrayBuffer());
console.log(new Uint8Array(buffer.slice(0, 8)));

// 2. 检查 MIME 类型
// 应为: application/wasm

// 3. 检查跨域配置
// 服务器需配置 CORS 头
```

**解决方案**:

```javascript
// 正确加载 WASM
async function loadWasmModule(url) {
    const response = await fetch(url);
    const bytes = await response.arrayBuffer();
    const module = await WebAssembly.compile(bytes);
    const instance = await WebAssembly.instantiate(module, imports);
    return instance.exports;
}
```

## 调试方法

### 1. 日志调试

**启用详细日志**:

```cpp
// C++ 代码
#define HI_LOG_ENABLE
#define LOG_DOMAIN 0xD004100

#include "log.h"

void DebugFunction() {
    HI_LOG_DEBUG("Debug message: %{public}s", data.c_str());
    HI_LOG_INFO("Info message");
    HI_LOG_ERROR("Error message: %{public}d", errorCode);
}
```

**运行时日志级别**:

```bash
# 设置日志级别
./sp_daemon --log-level debug
./trace_streamer --log-level verbose
```

### 2. GDB 调试

```bash
# 启动 GDB
gdb ./libsmartperf_daemon.z.so

# 设置断点
(gdb) break SpThreadSocket::CheckTcpToken

# 运行程序
(gdb) run

# 查看变量
(gdb) print token
(gdb) print isNeedUdpToken

# 单步执行
(gdb) next
(gdb) step

# 堆栈跟踪
(gdb) bt
```

### 3. 系统调用跟踪

```bash
# 使用 strace 跟踪系统调用
strace -f -o sp_daemon.log ./libsmartperf_daemon.z.so

# 使用 ltrace 跟踪库调用
ltrace -f -o sp_daemon_lib.log ./libsmartperf_daemon.z.so

# 使用 perf 跟踪性能
perf record -g ./libsmartperf_daemon.z.so
perf report
```

### 4. 网络调试

```bash
# 使用 tcpdump 抓包
tcpdump -i any -w socket_trace.pcap port 8283 or port 8284

# 使用 Wireshark 分析
wireshark socket_trace.pcap

# 使用 nc 测试 Socket
nc -u localhost 8283
```

### 5. HDC 调试

```bash
# 设备信息
hdc list targets
hdc shell device-info

# 文件传输
hdc file send local_file /data/local/tmp/
hdc file recv /data/local/tmp/remote_file ./

# 日志获取
hdc hilog > device.log

# 进程监控
hdc shell top -b -n 1 | grep smartperf
```

## 性能问题

### 1. SP_daemon 内存占用过高

**诊断步骤**:

```bash
# 1. 查看内存使用
hdc shell cat /proc/$(pgrep sp_daemon)/status | grep VmRSS

# 2. 跟踪内存分配
hdc shell ./libsmartperf_daemon.z.so --memory-profile

# 3. 使用 heapprofd
hdc shell heapprofd -p $(pgrep sp_daemon)
```

**优化建议**:
- 降低采集频率
- 减少采集指标数量
- 启用采样模式

### 2. trace_streamer 解析慢

**诊断步骤**:

```bash
# 1. 性能分析
time ./trace_streamer large_trace.dat

# 2. 使用 perf
perf record -g ./trace_streamer large_trace.dat
perf report

# 3. 检查 I/O 瓶颈
iotop -b -n 3
```

**优化建议**:
- 使用 SSD 存储
- 增加内存缓存
- 启用并行解析

## 回退策略

### 1. 回退到上一版本

```bash
# 回退 GN 配置
git checkout HEAD~1 -- bundle.json BUILD.gn

# 回退源代码
git checkout HEAD~1 -- smartperf_device/
```

### 2. 最小化测试

```bash
# 仅编译 SP_daemon
ninja -C out/[config] //developtools/smartperf_host/smartperf_device/device_command:SP_daemon

# 跳过 HAP 构建
gn gen out/[config] --args="smartperf_host_device=true support_jsapi=false"
```

### 3. 降级依赖

```bash
# 查看依赖版本
cat bundle.json | grep external_deps

# 使用固定版本
gn gen out/[config] --args="hilog_version='1.0.0'"
```

## 相关文档

- [项目概览](00_Overview.md)
- [系统架构](01_Architecture.md)
- [WASM 接口](02_WASM_API.md)
- [GN 构建配置](04_GNBuild.md)
- [编译产物](05_BuildArtifacts.md)
- [安全风险评审](06_Security.md)
