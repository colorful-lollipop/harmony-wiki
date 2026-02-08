# FaultLoggerd 安全风险评审

## 目的

本文档分析 faultloggerd 组件的安全模型、攻击面、信任边界及可被利用点。

## 适用范围

- 目标读者：安全审计员、系统开发者
- 分析范围：生产代码（排除测试目录）
- 分析方法：代码审计、威胁建模

## 安全模型

### 信任边界

```
┌─────────────────────────────────────────────────────────┐
│              不可信区域（应用/工具）            │
└─────────────────────┬──────────────────────────────────┘
                      │
         ┌───────────┴───────────┐
         │                       │
    [Socket 通信边界]          [权限检查层]
         │                       │
         └───────────┬─────────────┘
                      │
            ┌───────────┴───────────┐
            │                       │
       [故障日志文件]          [进程内存访问]
            │                       │
            └───────────┬─────────────┘
                      │
              ┌───────────┴───────────┐
              │                       │
        [临时目录]              [用户进程内存]
              │                       │
              └───────────┬──────────────┘
                          │
                   ┌─────────────┴───────────┐
                   │                       │
              [系统服务]              [内核能力]
                   │                       │
              └──────────────────────────────┘
```

### 安全原则

1. **最小权限原则**：仅授予必要的权限
2. **防御深度**：多层权限验证
3. **输入验证**：严格验证所有外部输入
4. **隔离原则**：不同权限级别的服务隔离

## 攻击面分析

### 1. Socket 通信攻击面

#### 攻击点：未授权连接

**位置**：`services/fault_logger_server.cpp:88-160`

**代码证据**：
```cpp
bool CheckRequestCredential(int32_t connectionFd, int32_t requestPid)
{
    struct ucred creds{};
    if (!FaultCommonUtil::GetUcredByPeerCred(creds, connectionFd)) {
        return false;
    }
    if (CheckCallerUID(creds.uid)) {
        return true;
    }
    if (creds.pid != requestPid) {
        DFXLOGW("Failed to check request credential request:%{public}d: cred:%{public}d fd:%{public}d",
                requestPid, creds.pid, connectionFd);
        return false;
    }
    return true;
}
```

**威胁场景**：
1. 恶意应用通过 Socket 请求 dump 不属于自己的进程
2. 利用 UID 白名单绕过检查（通过伪造 UID）
3. 拒绝服务攻击（发送大量请求耗尽资源）

**可利用性**：⚠️ **中风险**

**影响**：未授权的进程信息泄露、恶意进程崩溃

**修复建议**：
1. ✅ 已实现：UID 白名单检查（`services/fault_logger_service.cpp:72-78`）
2. ✅ 已实现：PID 匹配验证（`services/fault_logger_service.cpp:97-102`）
3. 💡 建议：添加连接频率限制
4. 💡 建议：实现连接黑名单机制

---

#### 攻击点：Socket 劫持

**位置**：`interfaces/innerkits/faultloggerd_client/faultloggerd_client.cpp:70-120`

**代码证据**：
```cpp
int32_t SendRequestToServer(const char* socketName, const SocketRequestData& socketRequestData, 
    int32_t timeout, SocketFdData* socketFdData = nullptr, bool signalSafely = false)
{
    FaultLoggerdSocket faultLoggerdSocket(signalSafely);
    int32_t retCode{ResponseCode::DEFAULT_ERROR_CODE};
    if (!faultLoggerdSocket.InitSocket(socketName, timeout)) {
        return ResponseCode::CONNECT_FAILED;
    }
    retCode = socketFdData ? faultLoggerdSocket.RequestFdsFromServer(socketRequestData, *socketFdData) :
            faultLoggerdSocket.RequestServer(socketRequestData);
    return retCode;
}
```

**威胁场景**：
1. 恶意应用创建伪造的 Socket 服务
2. 在 `/dev/unix/socket/` 下创建同名 socket
3. 拦截或篡改崩溃日志请求

**可利用性**：⚠️ **中风险**

**影响**：崩溃日志劫持、错误诊断信息误导

**修复建议**：
1. ✅ 已实现：Socket 路径在 `/dev/unix/socket/` 下，需要相应权限
2. ✅ 已实现：SELinux 策略限制（`services/config/faultloggerd.cfg:22-75`）
3. 💡 建议：添加 Socket 所有者验证（通过 SELinux context）

---

### 2. 进程内存访问攻击面

#### 攻击点：ptrace 权限滥用

**位置**：`interfaces/innerkits/dump_catcher/dfx_dump_catcher.cpp`

**代码证据**：通过 PTRACE_ATTACH 访问目标进程内存

**威胁场景**：
1. 非授权进程调用 DumpCatcher 抓取系统进程
2. 利用 UID 白名单绕过检查（如果是白名单 UID）
3. 读取敏感进程的内存信息

**可利用性**：⚠️ **中风险**

**影响**：敏感信息泄露（密钥、密码、令牌）

**修复建议**：
1. ✅ 已实现：UID 白名单限制（`services/fault_logger_service.cpp:72-78`）
2. 💡 建议：添加 ptrace 权限检查日志
3. 💡 建议：对白名单 UID 实施额外审查

---

#### 攻击点：/proc 文件读取权限

**位置**：`interfaces/innerkits/procinfo/procinfo.cpp`

**代码证据**：读取 `/proc/[pid]/` 目录下的进程信息

**威胁场景**：
1. 读取环境变量（`/proc/[pid]/environ`）
2. 读取命令行参数（`/proc/[pid]/cmdline`）
3. 读取进程的文件描述符（`/proc/[pid]/fd`）

**可利用性**：⚠️ **中风险**

**影响**：进程机密泄露、运行环境泄露

**修复建议**：
1. ✅ 已实现：只读取必要的文件（maps、status、task）
2. ✅ 已实现：UID 隔离（不同 UID 无法互相读取）
3. 💡 建议：实现更精细的 SELinux 策略

---

### 3. 文件系统攻击面

#### 攻击点：崩溃日志文件注入

**位置**：`services/temp_file_manager.cpp`

**代码证据**：写入 `/data/log/faultlog/temp/` 目录

**威胁场景**：
1. 恶意应用伪造崩溃日志
2. 利用路径遍历覆盖系统文件
3. 竞态条件导致日志清理失败

**可利用性**：⚠️ **低风险**

**影响**：误导诊断、掩盖真实崩溃

**修复建议**：
1. ✅ 已实现：文件命名包含 PID 和时间戳（唯一性）
2. ✅ 已实现：固定目录 `/data/log/faultlog/temp/`
3. 💡 建议：添加日志文件完整性校验（哈希验证）

---

#### 攻击点：SELinux 规则绕过

**位置**：`services/config/faultloggerd.cfg:22-75`

**代码证据**：
```json
{
    "secon": "u:r:faultloggerd:s0"
}
```

**威胁场景**：
1. 通过符号链接创建恶意目录
2. 滥用文件描述符提升权限
3. 利用配置文件漏洞

**可利用性**：⚠️ **低风险**

**影响**：服务劫持、权限提升

**修复建议**：
1. ✅ 已实现：严格的 SELinux 上下文
2. 💡 建议：定期审计 SELinux 策略
3. 💡 建议：使用最小化文件描述符

---

### 4. 资源耗尽攻击面

#### 攻击点：Pipe 资源耗尽

**位置**：`services/fault_logger_pipe.cpp:461-520`

**代码证据**：
```cpp
int LitePerfPipeService::CheckPerfLimit(int32_t uid, bool checkLimit)
{
    auto& perfCount = perfCountsMap_[uid];
    constexpr int32_t uidPerfLimit = 20;
    if (perfCount >= uidPerfLimit) {
        DFXLOGW("%{public}s :: perf resource is limited for uid %{public}d.",
                  FAULTLOGGERD_SERVICE_TAG, uid);
        return false;
    }
    return deviceAndUidPerfLimit.CheckPerfLimit(uid);
}
```

**威胁场景**：
1. 恶意应用创建大量 pipe 连接耗尽资源
2. 并发发送大量 dump 请求
3. 长时间占用 pipe 不释放

**可利用性**：⚠️ **中风险**

**影响**：服务拒绝攻击、资源耗尽

**修复建议**：
1. ✅ 已实现：UID 级别的资源限制（`services/fault_logger_pipe.cpp:467-471`）
2. ✅ 已实现：pipe 超时自动清理（`services/fault_logger_pipe.cpp:167-188`）
3. 💡 建议：添加全局连接数限制（当前 30，`services/main.cpp:18`）

---

#### 攻击点：并发 Dump 限制绕过

**位置**：`services/fault_logger_service.cpp:51-62`

**代码证据**：
```cpp
constexpr int LITE_DUMP_LIMIT_ONE_DAY = 60;
constexpr int ONE_DAY_SEC = 24 * 60 * 60;
```

**威胁场景**：
1. 恶意应用在短时间内发起大量 dump 请求
2. 利用时间窗口绕过限制（如跨天）
3. 并发请求占用所有配额

**可利用性**：⚠️ **中风险**

**影响**：资源耗尽、服务拒绝

**修复建议**：
1. ✅ 已实现：每日限制（60 次/天）
2. 💡 建议：添加滑动窗口限制（如每小时限制）
3. 💡 建议：添加令牌桶算法

---

### 5. 信号处理攻击面

#### 攻击点：信号处理器竞态条件

**位置**：`interfaces/innerkits/signal_handler/dfx_signal_handler.c`

**代码证据**：信号处理器中的 async-safety 问题

**威胁场景**：
1. 在信号处理器中调用非 async-safe 函数（如 malloc、printf）
2. 死锁导致子进程永久阻塞
3. 信号处理器递归触发

**可利用性**：⚠️ **中风险**

**影响**：堆损坏、死锁、二次崩溃

**修复建议**：
1. 💡 建议：在信号处理器中仅执行 async-safe 函数
2. 💡 建议：使用 sig_atomic_t 确保线程安全
3. 💡 建议：避免在信号处理器中使用互斥锁

---

## 可被利用点总结

| # | 攻击面 | 位置 | 风险等级 | 证据 | 修复状态 |
|---|----------|------|--------|------|----------|
| 1 | 未授权 Socket 连接 | `services/fault_logger_server.cpp:88-160` | ⚠️ 中 | ✅ 已实现 UID 检查 |
| 2 | Socket 劫持 | `interfaces/innerkits/faultloggerd_client/faultloggerd_client.cpp:70-120` | ⚠️ 中 | ✅ SELinux 保护 |
| 3 | ptrace 权限滥用 | `interfaces/innerkits/dump_catcher/dfx_dump_catcher.cpp` | ⚠️ 中 | ✅ UID 白名单 |
| 4 | /proc 文件读取 | `interfaces/innerkits/procinfo/procinfo.cpp` | ⚠️ 中 | ✅ UID 隔离 |
| 5 | 日志文件注入 | `services/temp_file_manager.cpp` | ⚠️ 低 | ✅ 文件命名唯一 |
| 6 | Pipe 资源耗尽 | `services/fault_logger_pipe.cpp:461-520` | ⚠️ 中 | ✅ UID 级别限制 |
| 7 | 并发 Dump 绕过 | `services/fault_logger_service.cpp:51-62` | ⚠️ 中 | ✅ 每日限制 |
| 8 | 信号处理器竞态 | `interfaces/innerkits/signal_handler/dfx_signal_handler.c` | ⚠️ 中 | 💡 建议 async-safe |

## 权限控制机制

### UID 白名单（`services/fault_logger_service.cpp:72-78`）

```cpp
const uint32_t whitelist[] = {
    0,      // rootUid
    1000,    // bmsUid
    1201,    // hiviewUid
    1212,    // hidumperServiceUid
    5523,    // foundationUid
    7400,    // dev_assistant
};
```

**安全原则**：
1. 仅白名单 UID 可以请求 dump 任意进程
2. 非白名单 UID 只能 dump 自己的进程
3. PID 匹配验证（请求 PID 必须等于连接 PID）

### Capability 管理

**服务能力**（`services/config/faultloggerd.cfg:22-75`）：
- `CAP_DAC_READ_SEARCH`：读取目录权限
- `CAP_KILL`：发送信号权限

**风险缓解**：
1. ✅ 最小化 capability 集合
2. ✅ 仅在需要时请求

### SELinux 策略

**上下文**：`u:r:faultloggerd:s0`

**保护措施**：
1. ✅ 严格的文件访问控制
2. ✅ Unix Domain Socket 权限（`0666`）
3. ✅ Capability 限制

## 安全最佳实践

### 1. 输入验证

| 输入类型 | 验证方式 | 风险 |
|----------|----------|------|
| PID | 范围检查、存在性检查 | 注入 |
| TID | 范围检查、线程存在性检查 | 注入 |
| UID | 白名单检查、范围检查 | 权限提升 |
| Socket 路径 | 白名单、权限检查 | 劫持 |
| 文件路径 | 规范化、遍历检查 | 路径遍历 |

### 2. 错误处理

| 错误类型 | 安全风险 | 推荐做法 |
|----------|----------|----------|
| 网络错误 | 不暴露内部状态 | 返回通用错误码 |
| 权限错误 | 不泄露内部信息 | 记录日志但返回通用错误 |
| 内存分配失败 | 不崩溃服务 | 优雅降级 |

### 3. 日志安全

| 信息类型 | 敏感度 | 处理建议 |
|----------|----------|----------|
| PID | 低 | 可记录 |
| 线程栈 | 中 | 仅记录帧地址 |
| 寄存器值 | 低 | 可记录（崩溃现场需要） |
| 内存映射 | 低 | 已脱敏 |
| 环境变量 | 高 | 不记录 |

### 4. 并发控制

| 资源类型 | 当前限制 | 建议改进 |
|----------|----------|----------|
| Socket 连接 | 30 | 动态调整、优先级队列 |
| Pipe 资源 | 20/UID | 全局限制、时间窗口 |
| Dump 请求 | 60/天 | 令牌桶算法 |

## 未实现的安全特性

### 检查范围

- ✅ 已检查：进程内存访问权限
- ✅ 已检查：Socket 通信权限
- ✅ 已检查：文件系统访问权限
- ⚠️ 部分：信号处理器 async-safety（需要改进）

### 检查方法

1. **代码审计**：手动审查所有权限检查点
2. **模糊测试**：使用 fuzz 测试工具
3. **静态分析**：使用静态分析工具

## 相关跳转

- [项目定位与核心能力](00_Overview.md) - 项目概览
- [架构设计](02_Architecture.md) - 组件交互和通信机制
- [对外 API](03_External_API.md) - API 权限要求

## 参考资料

- OpenHarmony 安全指南：https://gitee.com/openharmony/docs/blob/master/zh-cn/security/
- SELinux 策略参考：`services/config/faultloggerd.cfg`
