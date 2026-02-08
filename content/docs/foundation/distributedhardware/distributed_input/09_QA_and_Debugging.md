# 常见问题与调试指南

**文档版本**: v1.0  
**更新日期**: 2025-02-07  
**模块版本**: 3.2

---

## 目录

- [1. 构建问题](#1-构建问题)
- [2. 运行问题](#2-运行问题)
- [3. 调试方法](#3-调试方法)
- [4. 错误码参考](#4-错误码参考)
- [5. 日志分析](#5-日志分析)
- [6. 性能优化](#6-性能优化)

---

## 1. 构建问题

### 1.1 GN 构建失败

**问题现象**: 执行 `hb build` 时构建失败

**常见原因**:
1. 依赖组件未配置
2. 头文件路径缺失
3. GN 语法错误

**排查步骤**:

```bash
# 1. 检查依赖组件
cat bundle.json | grep -A 20 "deps"

# 2. 检查 GN 路径配置
cat distributedinput.gni

# 3. 执行完整构建
hb build -f
```

**解决方案**:

| 问题 | 解决方案 |
|------|---------|
| 依赖缺失 | 确保 `bundle.json` 中包含所有必需组件 |
| 头文件找不到 | 检查 `include` 路径是否正确配置 |
| GN 变量未定义 | 确认 `distributedinput.gni` 包含正确路径 |

### 1.2 编译产物未生成

**问题现象**: 构建成功但未生成 `.so` 文件

**排查命令**:

```bash
# 1. 检查构建产物目录
ls -la out/standard_arm64/\
    foundation/distributedhardware/distributed_input/

# 2. 检查 GN target
cat interfaces/inner_kits/BUILD.gn

# 3. 检查链接配置
nm -D out/standard_arm64/\
    foundation/distributedhardware/distributed_input/\
    services/source/sourcemanager/libdinput_source.z.so | grep "StartRemoteInput"
```

---

## 2. 运行问题

### 2.1 SA 启动失败

**问题现象**: dinput 进程未启动或启动后崩溃

**排查步骤**:

```bash
# 1. 检查 SA 注册状态
hidumper -sa

# 2. 查看系统日志
hilog | grep -E "dinput|DISTINPUT"

# 3. 检查进程状态
ps -ef | grep dinput
```

**常见错误码**:

| 错误码 | 含义 | 排查方向 |
|--------|------|---------|
| `-67000` | IPC 描述符无效 | SA 初始化异常 |
| `-67061` | 权限校验失败 | 权限配置问题 |
| `-67045` | 令牌校验失败 | IPC 通信异常 |

### 2.2 跨设备连接失败

**问题现象**: Source 设备无法连接到 Sink 设备

**排查步骤**:

```bash
# 1. 检查设备发现
hiChainInnerKit -l

# 2. 检查网络连通性
ping <sink_ip>

# 3. 检查 SoftBus 状态
hilog | grep -i softbus
```

**可能原因**:

| 原因 | 解决方案 |
|------|---------|
| 设备不在同一局域网 | 确保设备在同一网络 |
| 同账号校验失败 | 检查账户绑定状态 |
| ACL 配置问题 | 检查设备管理器 ACL 配置 |

---

## 3. 调试方法

### 3.1 日志调试

**启用详细日志**:

```cpp
// 在代码中添加
#include "dinput_log.h"

// 设置日志级别
DHLOGD("Debug message");
DHLOGI("Info message");
DHLOGW("Warning message");
DHLOGE("Error message");
```

**日志过滤**:

```bash
# 过滤分布式输入日志
hilog | grep -i "dinput|DISTINPUT"

# 过滤特定标签
hilog | grep -E "Source|Sink|Transport"
```

### 3.2 IPC 调试

**查看 IPC 调用**:

```bash
# 检查 SA 状态
hidumper -sa | grep -A 5 dinput

# 查看 IPC 通信
hilog | grep -i "IPCSkeleton|OnRemoteRequest"
```

### 3.3 事件流调试

**追踪输入事件**:

```bash
# 查看 RawEvent 流转
hilog | grep -i "RawEvent|CollectEvents|InjectEvent"

# 检查虚拟设备
ls -la /dev/uinput
cat /sys/class/input/*/name
```

### 3.4 GDB 调试

**附加到进程**:

```bash
# 找到进程 PID
ps -ef | grep dinput

# 附加 GDB
gdb -p <pid>

# 或启动调试版本
gdb --args ./out/.../dinput
```

**常用 GDB 命令**:

```gdb
# 设置断点
break distributed_input_source_manager.cpp:123

# 查看变量
print devId
print inputTypes

# 单步执行
next
step

# 查看调用栈
backtrace

# 继续执行
continue
```

---

## 4. 错误码参考

### 4.1 IPC 错误码

| 错误码 | 定义 | 含义 |
|--------|------|------|
| `-67000` | `ERR_DH_INPUT_IPC_INVALID_DESCRIPTOR` | IPC 描述符无效 |
| `-67001` | `ERR_DH_INPUT_IPC_WRITE_FAIL` | IPC 写入失败 |
| `-67045` | `ERR_DH_INPUT_IPC_WRITE_TOKEN_VALID_FAIL` | 写入令牌校验失败 |
| `-67046` | `ERR_DH_INPUT_IPC_READ_TOKEN_VALID_FAIL` | 读取令牌校验失败 |
| `-67047` | `ERR_DH_INPUT_IPC_READ_VALID_FAIL` | 读取数据校验失败 |

### 4.2 权限错误码

| 错误码 | 定义 | 含义 |
|--------|------|------|
| `-67061` | `ERR_DH_INPUT_SRC_ENABLE_PERMISSION_CHECK_FAIL` | Source ENABLE 权限失败 |
| `-67062` | `ERR_DH_INPUT_SRC_ACCESS_PERMISSION_CHECK_FAIL` | Source ACCESS 权限失败 |
| `-67063` | `ERR_DH_INPUT_SINK_ENABLE_PERMISSION_CHECK_FAIL` | Sink ENABLE 权限失败 |

### 4.3 传输错误码

| 错误码 | 定义 | 含义 |
|--------|------|------|
| `-65040` | `ERR_DH_INPUT_SERVER_SOURCE_TRANSPORT_SESSION_NOT_READY` | 会话未就绪 |
| `-65041` | `ERR_DH_INPUT_SERVER_SOURCE_TRANSPORT_SEND_FAIL` | 发送失败 |
| `-65042` | `ERR_DH_INPUT_SERVER_SOURCE_TRANSPORT_PERMISSION_DENIED` | 权限被拒 |

### 4.4 设备错误码

| 错误码 | 定义 | 含义 |
|--------|------|------|
| `-64001` | `ERR_DH_INPUT_SINK_SERVER_OPEN_DEVICE_FAIL` | 打开设备失败 |
| `-64002` | `ERR_DH_INPUT_SINK_SERVER_GET_DEVICE_INFO_FAIL` | 获取设备信息失败 |

---

## 5. 日志分析

### 5.1 日志标签

| 标签 | 来源 | 说明 |
|------|------|------|
| `DINPUT` | 主日志标签 | 通用日志 |
| `DistributedInput` | 类名 | 按组件分类 |
| `SoftBus` | 传输层 | SoftBus 相关 |

### 5.2 日志格式

```
# 标准格式
[时间戳][日志级别][标签] 消息内容

# 示例
02-07 10:30:15.123  1234  5678 E/DINPUT: HandleStartRemoteInput permission check fail
```

### 5.3 关键日志点

#### Source 启动流程

```
[Init] Source SA 开始初始化
[Init] 注册 DistributedInputSourceManager
[Init] 启动传输层
[Init] Source SA 初始化完成
```

#### Sink 启动流程

```
[Init] Sink SA 开始初始化
[Init] 启动事件采集
[Init] Sink SA 初始化完成
```

#### 跨设备连接流程

```
[Transport] OpenInputSoftbusSession 开始
[Transport] CheckSrcPermission 校验
[Transport] Socket 创建成功
[Transport] 会话建立成功
```

---

## 6. 性能优化

### 6.1 事件传输延迟

**监控指标**:
- 端到端延迟
- 事件丢失率
- 带宽占用

**优化建议**:

| 优化项 | 配置方法 |
|--------|---------|
| QOS 带宽 | 配置 `QOS_TYPE_MIN_BW` |
| 延迟控制 | 配置 `QOS_TYPE_MAX_LATENCY` |
| 批量发送 | 调整事件缓冲大小 |

### 6.2 内存优化

**关注指标**:
- 事件缓冲区大小
- 设备描述符缓存
- 连接状态内存

**配置参数** (见 `constants_dinput.h`):

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `INPUT_EVENT_BUFFER_SIZE` | 256 | 事件缓冲区大小 |
| `MSG_MAX_SIZE` | 32KB | 消息最大长度 |

### 6.3 CPU 优化

**关注场景**:
- 事件采集循环
- 序列化/反序列化
- 白名单匹配

**优化建议**:
- 使用哈希表加速白名单匹配
- 批量处理事件减少系统调用
- 使用零拷贝技术减少内存复制

---

## 附录

### A. 常用调试命令

| 命令 | 用途 |
|------|------|
| `hilog | grep dinput` | 查看 dinput 日志 |
| `hidumper -sa | grep dinput` | 查看 SA 状态 |
| `ps -ef | grep dinput` | 查看进程 |
| `ls -la /dev/input/` | 查看输入设备 |
| `cat /sys/class/input/*/name` | 查看设备名称 |

### B. 配置文件路径

| 文件 | 路径 | 说明 |
|------|------|------|
| 白名单 | `/etc/minidp_dinput_whitelist.cfg` | 组合键过滤 |
| SA 配置 | `/system/profile/dinput.json` | SA 参数 |
| 日志配置 | - | 通过 hilog 系统 |

### C. 相关工具

| 工具 | 用途 |
|------|------|
| `hidumper` | 系统信息转储 |
| `hilog` | 日志查看 |
| `hiChainInnerKit` | 设备认证 |
| `bm` | bundle 管理 |

---

**文档版本**: v1.0  
**维护者**: OpenHarmony Wiki Generator  
**下次更新**: 建议每季度更新一次
