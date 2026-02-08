# 常见构建/运行/调试问题

## 目的

提供 hdc 项目的常见问题排查指南、定位路径和解决方案。

## 适用范围

本文档适用于：
- 构建失败排查
- 运行时错误定位
- 连接问题排查
- 性能问题诊断

## 相关跳转

- [项目概览](./00_Overview.md) - 项目定位和核心能力
- [目录结构](./02_Directory_Structure.md) - 模块职责
- [架构说明](./03_Architecture.md) - Client-Server-Daemon 通信
- [构建产物](./07_Build_Artifacts.md) - 安装路径和依赖

---

## 构建问题

### 问题 1：编译失败 - 找不到头文件

**症状**：
```
error: 'session.h' file not found
```

**可能原因**：
1. 编译命令未正确指定包含路径
2. `hdc.gni` 配置错误
3. 依赖的 headers 缺失

**定位路径**：
1. 检查 `BUILD.gn` 的 `include_dirs` 配置
2. 检查头文件是否存在：`src/common/*.h`
3. 查看编译输出中的错误路径

**证据**：`BUILD.gn:66-70` - `config("hdc_config")` 配置

```gn
config("hdc_config") {
  include_dirs = [
    "src/common",
    "${py_out_dir}",
  ]
}
```

**解决方案**：
1. 确保完整构建：`./build.sh --product-name <name>`
2. 检查 `hdc.gni` 配置是否正确
3. 清理构建产物：`rm -rf out/`

---

### 问题 2：链接错误 - undefined reference

**症状**：
```
undefined reference to 'HdcSessionBase::MallocSession'
```

**可能原因**：
1. 编译选项不正确（缺少特定宏定义）
2. 源文件版本不匹配
3. 链接顺序问题

**定位路径**：
1. 查看编译输出中的符号定义
2. 检查 `src/common/session.h:96` - `MallocSession()` 是否声明
3. 检查相关宏是否定义：`HDC_HOST`, `HDC_DAEMON`

**证据**：`BUILD.gn:385-393` - Defines 配置

```gn
defines = [
  "HDC_HOST",  # Host 端
  "HARMONY_PROJECT",
  # ...
]
```

**解决方案**：
1. 确认使用正确的 GN 命令
2. 检查条件编译分支是否匹配当前平台
3. 查看头文件中的条件编译保护：`#ifdef HDC_HOST`

---

### 问题 3：Rust 编译失败

**症状**：
```
error[E0432]: failed to resolve: could not find `serialize_structs` in `hdc`
```

**可能原因**：
1. `product_name == "ohos-sdk"`（SDK 构建）
2. Rust 模块未正确配置

**定位路径**：
1. 检查 `hdc_rust/BUILD.gn:316-327`
2. 查看条件编译：`product_name != "ohos-sdk"`

**证据**：`BUILD.gn:195-261` - Rust target 条件

```gn
if (product_name != "ohos-sdk") {
  ohos_static_library("serialize_structs") { ... }
  ohos_rust_static_library("lib") { ... }
}
```

**解决方案**：
1. 如果需要 SDK 构建，使用正确的 `product_name`
2. 如果不需要 Rust 模块，可以禁用（但 HDC 需要 C++ 版本）

---

## 运行时问题

### 问题 4：无法找到设备

**症状**：
```
$ hdc list targets
[Empty]
```

**可能原因**：
1. 设备未连接（USB/网络）
2. hdcd 未运行
3. hdc server 未运行
4. USB 驱动问题

**定位路径**：
1. 检查设备连接：`hdc list targets [-v]`
2. 检查 server 状态：`hdc checkserver`
3. 查看 hdc 日志（如果启用）：`hdc -l5 start`

**证据**：`README_zh.md:137-141` - `hdc list targets` 命令说明

**排查步骤**：
1. **PC 端检查**：
   - `hdc checkserver` - 确认 server 版本
   - `hdc kill -r` - 重启 server
   - 检查 USB 设备：`lsusb` (Linux), 设备管理器 (Windows/macOS)

2. **设备端检查**：
   - 登录设备 shell：通过串口或网络
   - 检查 hdcd 进程：`ps -ef | grep hdcd`
   - 查看日志：`hilog -x hdc | grep error`

3. **USB 连接问题**（Linux）：
   - 检查权限：`ls -l /dev/bus/usb/`
   - 如果非 root，设置权限：`sudo chmod 666 /dev/bus/usb/*/*`
   - 或创建 udev 规则（见 README_zh.md:72-90）

---

### 问题 5：版本不匹配

**症状**：
```
[Fail]Error: Version mismatch, host version 3.1.0, daemon version 3.0.0
```

**可能原因**：
1. PC 端和设备端 hdc 版本不一致
2. 使用了旧版本的工具

**定位路径**：
1. 检查版本：`hdc -v` (PC 端), `hdcd -v` (设备端）
2. 查看版本不匹配错误日志

**证据**：
- 版本检查宏：`BUILD.gn:459-460` - `hdc_version_check` defines
- 版本检查实现：`src/daemon/daemon.cpp` - 应该有版本比较逻辑

**解决方案**：
1. 更新到相同版本：
   - PC 端：下载最新的 SDK
   - 设备端：刷入包含新版本 hdcd 的系统镜像
2. 如果必须使用不同版本：
   - 禁用版本检查（如果开发环境允许）
   - 注意：版本不匹配可能导致兼容性问题

---

### 问题 6：权限被拒绝

**症状**：
```
[Fail]Error: Permission denied, operation requires root privileges
```

**可能原因**：
1. 非开发者模式
2. 缺少 sudo 权限
3. SELinux 策略阻止

**定位路径**：
1. 检查开发者模式：`getprop const.debuggable`
2. 检查 root 模式：`getprop persist.hdc.root`
3. 检查 SELinux：`dmesg | grep avc | grep hdc`

**证据**：`src/daemon/main.cpp:268-293` - `NeedDropRootPrivileges()` 函数

```cpp
bool NeedDropRootPrivileges()
{
    string rootMode;
    string debugMode;
    SystemDepend::GetDevItem("const.debuggable", debugMode);
    SystemDepend::GetDevItem("persist.hdc.root", rootMode);
    if (debugMode == "1") {
        if (rootMode == "1") {
            int rc = setuid(0);  // 保持 root
        } else if (rootMode == "0") {
            if (getuid() == 0) {
                return DropRootPrivileges();  // 降权到 shell
            }
        }
    }
}
```

**解决方案**：
1. **方法 1 - 使用 sudo**：
   ```bash
   sudo hdc shell
   ```

2. **方法 2 - 启用 root 模式**：
   ```bash
   # 在设备 shell
   setprop persist.hdc.root 1
   hdcd &
   ```

3. **方法 3 - 启用开发者模式**：
   ```bash
   # 在设备 shell
   setprop const.debuggable 1
   ```

4. **方法 4 - 检查 SELinux**：
   ```bash
   # 查看 AVC 拒绝日志
   dmesg | grep avc | grep hdc
   # 如果有拒绝，需要调整 SELinux 策略
   ```

---

### 问题 7：连接超时

**症状**：
```
[Fail]Error: Connection timeout
```

**可能原因**：
1. 网络延迟高
2. 防火墙阻止
3. 设备繁忙

**定位路径**：
1. 检查网络连接：`ping <device_ip>`
2. 检查防火墙规则
3. 查看 hdc 日志：`hdc -l5 start`

**证据**：`src/common/session.h` - 心跳机制（`HeartbeatMsg` 结构）

**解决方案**：
1. **网络问题**：
   - 使用 USB 而非 TCP（如果可能）
   - 使用有线连接
   - 检查网络设备和路由

2. **防火墙问题**：
   - Windows：检查防火墙规则，允许 TCP 端口
   - Linux：检查 `iptables` 或 `ufw` 规则

3. **增加超时**：
   - 在启动 hdc 时增加超时参数（如果支持）
   - TODO(需确认)：是否支持超时配置

---

### 问题 8：文件传输失败

**症状**：
```
[Fail]Error: Failed to send file, permission denied
```

**可能原因**：
1. 设备端存储空间不足
2. 权限不足
3. 文件路径不存在

**定位路径**：
1. 检查设备存储：`hdc shell df -h`
2. 检查文件权限：`hdc shell ls -l /path`
3. 检查 bundle 路径：验证 debug bundle 路径有效

**证据**：`src/daemon/daemon_unity.cpp:77-94` - `CheckbundlePath()` 函数

**解决方案**：
1. **检查存储空间**：
   ```bash
   hdc shell df -h /data
   ```

2. **正确使用目标路径**：
   ```bash
   # 确保目标路径在可写目录
   hdc file send local.txt /data/local/tmp/target.txt
   ```

3. **使用 debug bundle 路径**：
   ```bash
   # 将文件发送到 debug bundle 目录
   hdc file send local.txt /data/hdc/hdc_debug/ <bundle_name>/
   ```

---

## 性能问题

### 问题 9：文件传输慢

**可能原因**：
1. 未启用压缩
2. USB 缓冲区大小不合适
3. 网络带宽限制

**定位路径**：
1. 检查传输参数：`hdc file send --help`
2. 查看压缩使用情况
3. 监控传输速度

**证据**：`src/common/define_plus.h:367-436` - `FeatureFlagsUnion` 结构包含 `hugeBuf` 和 `compressLz4` 位

**解决方案**：
1. 启用 LZ4 压缩（默认应启用）
2. 使用大缓冲区（如果网络带宽足够）
3. 使用 USB 3.0 高速传输（如果设备支持）

### 问题 10：Shell 响应慢

**可能原因**：
1. 设备负载高
2. Shell 参数解析慢
3. 数据传输量大

**定位路径**：
1. 检查设备负载：`hdc shell top`
2. 查看 hdc 日志：`hilog | grep shell`

**解决方案**：
1. 使用非交互模式（减少终端输出）
2. 减少日志级别：`hdc -l0`
3. 优化命令（减少不必要的操作）

---

## 调试技巧

### 启用调试日志

**PC 端**：
```bash
# 启动 hdc server 并设置日志级别为 5（最详细）
hdc -l5 start
```

**设备端**：
```bash
# 在设备 shell 设置参数
setprop hdc.log.level 5
# 重启 hdcd
killall hdcd
hdcd &
```

**证据**：`BUILD.gn:106` - `HDC_HILOG` define

### 查看 HDC 日志

**设备端**：
```bash
# 使用 hilog 查看 hdc 日志
hilog -x hdc | grep -E "error|fail|warn"
```

### 网络抓包

**PC 端**（TCP 连接）：
```bash
# 使用 tcpdump 抓包
sudo tcpdump -i any port <hdc_port> -w capture.pcap
```

---

## 已知限制

1. **多设备并发**：
   - USB 模式下通常不支持多设备并发
   - 建议：一次只连接一个设备

2. **系统分区访问**：
   - `/system` 分区默认为只读
   - 需要先挂载为读写：`hdc target mount`

3. **非开发者模式限制**：
   - 某些功能在非开发者模式下被禁用
   - 需要启用开发者模式或使用 root 模式

4. **版本要求**：
   - Client 和 Daemon 版本必须一致
   - 版本不匹配时拒绝连接

---

## 关键结论

1. **常见问题分类**：构建问题、运行时问题、性能问题
2. **核心排查工具**：`list targets`, `checkserver`, 日志查看
3. **权限问题最常见**：需要 root 或开发者模式，注意 `persist.hdc.root` 参数
4. **版本检查严格**：版本不匹配会拒绝连接，需保持版本一致
5. **调试支持完善**：支持多级日志、hilog 集成、网络抓包

---

## 待确认事项

**TODO(需确认)**：
1. 超时参数的具体配置方法
2. 网络断线重连机制的细节
3. 各平台特定的已知问题和解决方案
