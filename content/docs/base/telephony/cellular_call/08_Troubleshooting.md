# 08 - 常见问题与调试指南

## 目的

本文档收集 Cellular Call 模块的**常见构建/运行问题与调试方法**，帮助开发者快速定位和解决问题。

## 适用范围

- **读者对象**：开发者、测试工程师
- **使用场景**：
  - 构建失败排查
  - 运行问题定位
  - 性能问题分析
  - Crash 分析

---

## 8.1 构建问题

### 8.1.1 常见构建错误

#### 问题 1: 依赖找不到

**错误信息**:
```
error: dependency 'xxx' not found
```

**原因**: 外部依赖未正确配置

**解决方案**:
```bash
# 1. 检查 bundle.json 中的依赖配置
# 2. 同步依赖
hb deps sync

# 3. 检查依赖组件是否已构建
hb build --parts
```

**证据位置**: `bundle.json:36-63`

---

#### 问题 2: 头文件找不到

**错误信息**:
```
fatal error: 'xxx.h' file not found
```

**原因**: include_dirs 配置缺失

**解决方案**:
```bash
# 1. 检查 BUILD.gn 中的 include_dirs
# 2. 确认路径正确性
# 3. 检查头文件是否存在
find . -name "xxx.h"
```

**证据位置**: `BUILD.gn:68-78`

---

#### 问题 3: 条件编译问题

**错误信息**:
```
error: 'CELLULAR_CALL_SATELLITE' file not found
```

**原因**: 条件编译开关未开启

**解决方案**:
```bash
# 1. 在产品配置中启用对应 feature
# 2. 修改 cellularcall.gni
cellular_call_satellite = true

# 3. 重新配置并构建
hb set
hb build
```

**证据位置**: `cellularcall.gni:18`

---

### 8.1.2 构建性能优化

| 优化项 | 方法 |
|--------|------|
| **增量构建** | `hb build --patch` |
| **并行构建** | 使用 `hb build -j <N>` |
| **缓存利用** | 保留 out 目录 |
| **分布式构建** | 使用 buildfs |

---

## 8.2 运行问题

### 8.2.1 SA 启动失败

#### 问题: CellularCall Service 无法启动

**错误信息**:
```
[ERROR] Service 4006 start failed
```

**排查步骤**:

```bash
# 1. 检查 SA 配置
cat sa_profile/4006.json

# 2. 检查依赖 SA 是否启动
hdc shell sa -l

# 3. 查看系统日志
hdc shell hidumper -s 4006

# 4. 检查权限
hdc shell permission check
```

**常见原因**:

| 原因 | 解决方案 |
|------|----------|
| Core Service (4010) 未启动 | 等待或启动 4010 |
| 权限不足 | 检查 CONNECT_CELLULAR_CALL_SERVICE |
| 库文件缺失 | 检查 libtel_cellular_call.z.so 是否存在 |

---

#### 问题: SA 依赖超时

**错误信息**:
```
[ERROR] SA 4006 depend 4010 timeout
```

**解决方案**:
```bash
# 1. 检查依赖 SA 状态
hdc shell sa -l | grep 4010

# 2. 手动启动依赖
hdc shell start 4010

# 3. 配置动态启动（如果网络不稳定）
cellular_call_dynamic_start = true
```

**证据位置**: `sa_profile/4006.json:8-10`

---

### 8.2.2 通话功能异常

#### 问题: 拨号无响应

**排查步骤**:

```bash
# 1. 检查通话状态
hdc shell telephony call status

# 2. 检查 CellularCall 服务状态
hdc shell telephony call sa status

# 3. 查看日志
hdc shell logcat | grep CellularCall

# 4. 检查 RIL 连接
hdc shell telephony ril status
```

**常见原因**:

| 原因 | 检查方法 |
|------|----------|
| Modem 异常 | `hdc shell ril at` |
| RIL 适配器问题 | `hdc shell logcat | grep RIL` |
| Control 未初始化 | Dump 服务状态 |
| 权限问题 | 检查 CONNECT_CELLULAR_CALL_SERVICE |

---

#### 问题: 来电无响铃

**排查步骤**:

```bash
# 1. 检查事件流
hdc shell logcat | grep -E "RING|INCOMING"

# 2. 检查 Call Manager 回调注册
#    证据位置: services/manager/src/cellular_call_register.cpp

# 3. 检查 Handler 事件处理
#    证据位置: services/manager/src/cellular_call_handler.cpp
```

---

### 8.2.3 IMS 相关问题

#### 问题: VoLTE 不可用

**排查步骤**:

```bash
# 1. 检查 IMS 开关状态
hdc shell telephony ims status

# 2. 检查 IMS 配置
hdc shell telephony ims config list

# 3. 检查 Core Service IMS 状态
hdc shell sa -l | grep 4010
```

**常见原因**:

| 原因 | 解决方案 |
|------|----------|
| IMS 功能未开启 | `SetImsSwitchStatus(true)` |
| IMS 配置错误 | 检查 `SetImsConfig()` |
| Core Service 问题 | 检查 SA 4010 日志 |

---

## 8.3 调试方法

### 8.3.1 日志调试

#### 日志标签

```cpp
// 证据位置: BUILD.gn:80-84
TELEPHONY_LOG_TAG = "CellularCall"
LOG_DOMAIN = 0xD001F11
```

#### 日志查看

```bash
# 查看所有 CellularCall 日志
hdc shell logcat | grep CellularCall

# 按日志级别过滤
hdc shell logcat | grep -E "ERROR|WARN"

# 保存日志到文件
hdc shell logcat > cellular_call.log
```

#### 增加调试日志

```cpp
// 在关键路径添加
TELEPHONY_LOGD("Entering Dial(), slotId=%{public}d", slotId);
TELEPHONY_LOGI("Dial success, callId=%{public}d", callId);
TELEPHONY_LOGE("Dial failed, error=%{public}d", error);
```

---

### 8.32 服务 Dump

#### 服务状态 Dump

```bash
# Dump CellularCall 服务状态
hdc shell hidumper -s 4006

# Dump 所有 telephony 服务
hdc shell hidumper -s telephony
```

#### Dump 内容解读

| Dump 项 | 含义 |
|---------|------|
| ServiceRunningState | 服务运行状态 |
| bindTime | 绑定时间 |
| endTime | 结束时间 |
| spendTime | 运行时长 |
| handlerMap | 处理器映射 |
| csControlMap | CS Control 映射 |
| imsControlMap | IMS Control 映射 |

---

### 8.3.3 IPC 调试

#### 权限检查调试

**证据位置**: `services/manager/src/cellular_call_stub.cpp:45-50`

```bash
# 检查调用方 UID
# 代码中自动获取: IPCSkeleton::GetCallingUid()

# 调试时添加日志
auto callingUid = IPCSkeleton::GetCallingUid();
TELEPHONY_LOGD("Calling UID: %{public}d", callingUid);
```

#### IPC 消息追踪

```bash
# 启用 IPC 追踪
hdc shell ipcperf on

# 查看 IPC 调用
hdc shell ipcperf dump
```

---

### 8.3.4 GDB/LLDB 调试

#### 附加进程

```bash
# 找到 SA 进程
hdc shell pidof cellular_call_service

# 附加调试器
lldb-server p --server --attach=<pid>
```

#### 常用断点

| 断点 | 用途 |
|------|------|
| `CellularCallStub::OnRemoteRequest` | 入口调试 |
| `CellularCallService::Dial` | 拨号调试 |
| `CSControl::Dial` | CS 拨号调试 |
| `IMSControl::Dial` | IMS 拨号调试 |
| `CellularCallHandler::ProcessEvent` | 事件处理调试 |

---

## 8.4 性能问题

### 8.4.1 性能指标

| 指标 | 目标值 | 说明 |
|------|--------|------|
| **拨号时延** | < 500ms | 从拨号到听到回铃音 |
| **接通时延** | < 2s | 从响铃到通话建立 |
| **切换时延** | < 300ms | CS↔IMS 切换 |

### 8.4.2 性能分析

#### Trace 追踪

```bash
# 启用 Trace
hdc shell systrace --list

# 开始追踪
hdc shell systrace -o trace.html telephony

# 结束追踪并查看
```

#### FFRT 任务分析

```bash
# 查看 FFRT 任务
hdc shell ffrt top

# 任务耗时分析
hdc shell ffrt trace
```

---

## 8.5 Crash 分析

### 8.5.1 Crash 现场收集

```bash
# 查看 Tombstone
hdc shell ls /data/tombstones/
hdc shell cat /data/tombstones/tombstone_xx

# 查看日志
hdc shell logcat | grep -E "FATAL|signal"

# 查看内存
hdc shell dumpsys meminfo <pid>
```

### 8.5.2 常见 Crash 原因

| 原因 | 日志特征 | 解决方案 |
|------|----------|----------|
| **空指针** | `NullPointerException` | 检查参数有效性 |
| **内存越界** | `SIGSEGV` | 检查数组/指针访问 |
| **锁顺序** | `DeadLock` | 规范锁使用 |
| **RIL 超时** | `IPC timeout` | 检查 RIL 响应 |

### 8.5.3 调试技巧

#### 1. 使用 Address Sanitizer

```gn
# BUILD.gn 中启用
sanitize = {
  address = true
}
```

#### 2. 使用 Thread Sanitizer

```gn
sanitize = {
  thread = true
}
```

#### 3. 内存调试

```bash
# 启用 Valgrind
valgrind --tool=memcheck ./cellular_call
```

---

## 8.6 定位路径速查

| 问题 | 排查入口 | 关键文件 |
|------|----------|----------|
| **SA 启动失败** | SA 日志 | `cellular_call_service.cpp` |
| **IPC 权限问题** | Stub 日志 | `cellular_call_stub.cpp:45-50` |
| **拨号失败** | Control 日志 | `cs_control.cpp` / `ims_control.cpp` |
| **RIL 无响应** | Handler 日志 | `cellular_call_handler.cpp` |
| **状态异常** | 状态机日志 | `cellular_call_handler.cpp` |
| **补充业务失败** | Supplement 日志 | `cellular_call_supplement.cpp` |
| **视频通话问题** | Video Control 日志 | `ims_video_call_control.cpp` |
| **紧急呼叫异常** | Emergency Utils 日志 | `emergency_utils.cpp` |

---

## 8.7 常用调试命令汇总

### 8.7.1 服务管理

```bash
# 启动服务
hdc shell start 4006

# 停止服务
hdc shell stop 4006

# 查看服务状态
hdc shell sa -l | grep 4006

# Dump 服务信息
hdc shell hidumper -s 4006
```

### 8.7.2 日志管理

```bash
# 实时日志
hdc shell logcat | grep CellularCall

# 保存日志
hdc shell logcat -f /data/log/cellular_call.log &

# 清除日志
hdc shell logcat -c
```

### 8.7.3 通话调试

```bash
# 查看通话状态
hdc shell telephony call status

# IMS 状态
hdc shell telephony ims status

# RIL 状态
hdc shell telephony ril status
```

---

## 相关跳转

| 目标 | 链接 |
|------|------|
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| 接口规范 | [04_Interfaces.md](./04_Interfaces.md) |
| 安全评审 | [07_Security_Review.md](./07_Security_Review.md) |

---

*最后更新：2026-02-06*
