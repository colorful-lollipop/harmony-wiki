# 安全风险评估

> 目的：详细分析 HiDumper 的安全风险，提供代码级证据和修复建议
> 
> **受众**: 安全研究员、安全审计人员、开发工程师

---

## 风险评估总览

| 风险等级 | 数量 | 风险项 |
|---------|------|--------|
| 🔴 **高危** | 1 | 命令注入风险 (CmdDumper) |
| 🟠 **中危** | 3 | 路径遍历、整数溢出、权限绕过 |
| 🟡 **低危** | 3 | 信息泄露、DoS、配置注入 |

---

## R1: 命令注入风险 (高危)

### 风险描述

`CmdDumper` 使用 `popen()` 执行系统命令，若命令参数来源不可控，可能导致命令注入攻击。

### 代码证据

**位置**: `frameworks/native/src/executor/cmd_dumper.cpp:58`

```cpp
const std::string CMD_PREFIX = "/system/bin/";

// 第58行
if ((fp_ = popen((CMD_PREFIX + cmd_).c_str(), "r")) == nullptr) {
    DUMPER_HILOGE(MODULE_COMMON, "popen %{public}s failed.", cmd_.c_str());
    return DumpStatus::DUMP_FAIL;
}
```

**位置**: `frameworks/native/src/executor/cmd_dumper.cpp:110`

```cpp
// 第110行
FILE* fp = popen((CMD_PREFIX + cmd).c_str(), "r");
```

**命令来源**: `frameworks/native/src/executor/cmd_dumper.cpp:165`

```cpp
// PID 替换逻辑
std::string CMDDumper::SetCmdParameter(const std::shared_ptr<DumpCfg> &ptrDumpCfg)
{
    std::string cmd = ptrDumpCfg->target_;  // 来自配置!
    if (ptrDumpCfg->args_ != nullptr && ptrDumpCfg->args_->GetPid() != DEFAULT_PID) {
        std::string pid = std::to_string(ptrDumpCfg->args_->GetPid());
        cmd.replace(cmd.find(PLACEHOLDER_PID), PLACEHOLDER_PID.size(), pid);
    }
    return cmd;
}
```

### 触发路径

```
用户输入 --(间接)--▶ 配置文件/配置数据 --▶ ptrDumpCfg->target_ --▶ popen()
```

### 当前缓解措施

| 措施 | 有效性 | 说明 |
|------|--------|------|
| 命令前缀固定 | ⚠️ 部分 | `CMD_PREFIX = "/system/bin/"` 限制命令搜索路径 |
| PID 类型转换 | ✅ 有效 | `std::to_string()` 确保 PID 为纯数字 |
| 配置文件控制 | ⚠️ 部分 | 命令来自配置，配置通常受保护 |

### 影响评估

| 属性 | 评估 |
|------|------|
| **可利用性** | 中 - 需控制配置或发现注入点 |
| **影响范围** | 高 - 可执行任意系统命令 |
| **权限提升** | 是 - 以 SA 权限执行 |

### 修复建议

1. **使用白名单**: 仅允许预定义的命令
   ```cpp
   static const std::set<std::string> ALLOWED_CMDS = {
       "ls", "cat", "ps", "top", ...
   };
   if (ALLOWED_CMDS.find(cmd) == ALLOWED_CMDS.end()) {
       return DumpStatus::DUMP_FAIL;
   }
   ```

2. **使用 execve() 替代 popen()**:
   ```cpp
   // 避免 shell 解释器，直接执行程序
   execve("/system/bin/cmd", args, envp);
   ```

3. **参数校验**: 对命令字符串进行严格字符过滤
   ```cpp
   // 禁止特殊字符: ; | & $ ` \n < >
   if (cmd.find_first_of(";|&$`\n<>") != std::string::npos) {
       return DumpStatus::DUMP_FAIL;
   }
   ```

---

## R2: 路径遍历风险 (中危)

### 风险描述

文件操作使用用户提供的或间接来源的路径，可能导致路径遍历，读取敏感文件。

### 代码证据

**位置**: `frameworks/native/src/executor/fd_output.cpp:48`

```cpp
// 用户路径直接用于 open()
fd_ = open(path_.c_str(), O_WRONLY | O_CREAT | O_APPEND, OPEN_ARGV);
```

**位置**: `frameworks/native/src/executor/file_stream_dumper.cpp:196`

```cpp
// 目录遍历
DIR* dir = opendir(target.c_str());
```

**位置**: `services/native/src/dump_manager_service.cpp:254-258`

```cpp
// /proc 路径拼接
std::string fdPath = "/proc/" + std::to_string(pid) + "/fd/";
// ...
std::string linkPath = fdPath + node;
std::string linkDest;
char buf[PATH_MAX] = {0};
ssize_t len = readlink(linkPath.c_str(), buf, sizeof(buf) - 1);
```

### 触发路径

```
用户输入 --▶ DumperOptions --▶ path_ --▶ open() / fopen()
```

### 当前缓解措施

| 措施 | 位置 | 有效性 |
|------|------|--------|
| `realpath()` 规范化 | `file_utils.cpp:60` | ✅ 有效 |
| `O_NOFOLLOW` 标志 | `dump_utils.cpp:330` | ✅ 有效 |

```cpp
// 良好实践示例
char canonicalPath[PATH_MAX];
if (realpath(path.c_str(), canonicalPath) == nullptr) {
    return false;
}
// 验证前缀
if (strncmp(canonicalPath, "/allowed/path", strlen("/allowed/path")) != 0) {
    return false;
}
```

### 影响评估

| 属性 | 评估 |
|------|------|
| **可利用性** | 中 - 需要绕过路径检查 |
| **影响范围** | 中 - 可能读取任意文件 |
| **权限提升** | 否 - 受限于文件系统权限 |

### 修复建议

1. **统一路径验证**: 所有文件操作前使用 `realpath()` + 前缀检查
2. **chroot 沙箱**: 在受限目录下执行文件操作
3. **文件描述符传递**: 由可信组件打开文件后传递 FD

---

## R3: 整数溢出风险 (中危)

### 风险描述

PID 和内存大小等数值参数可能存在整数溢出，导致越界访问或逻辑错误。

### 代码证据

**位置**: `frameworks/native/src/executor/memory/parse/parse_smaps_info.cpp`

```cpp
// 解析 smaps 文件中的数值
// 缺乏溢出检查
size_t size = std::stoll(sizeStr);  // 可能溢出
```

**位置**: `frameworks/native/src/manager/dump_implement.cpp`

```cpp
// PID 转 int
int pid = std::stoi(pidStr);  // 无范围检查
```

### 触发条件

```bash
# 超大 PID
hidumper --mem-smaps 99999999

# 异常 smaps 文件
# 修改 /proc/<pid>/smaps 中的数值 (需 root)
```

### 当前缓解措施

| 措施 | 有效性 |
|------|--------|
| PID 有效性检查 | ⚠️ 部分 |
| 范围检查 | ❌ 缺失 |

### 修复建议

1. **范围检查**:
   ```cpp
   constexpr int MAX_PID = 32768;  // 典型 Linux PID 上限
   if (pid <= 0 || pid > MAX_PID) {
       return DumpStatus::DUMP_INVALID_PID;
   }
   ```

2. **使用安全转换**:
   ```cpp
   try {
       size_t size = std::stoull(sizeStr);
       if (size > MAX_MEMORY_SIZE) {
           throw std::out_of_range("Memory size too large");
       }
   } catch (const std::exception& e) {
       return DumpStatus::DUMP_FAIL;
   }
   ```

---

## R4: IPC 拒绝服务 (中危)

### 风险描述

缺乏 IPC 请求速率限制，恶意应用可通过高频调用导致服务不可用。

### 代码证据

**位置**: `services/zidl/src/dump_broker_stub.cpp:32`

```cpp
int DumpBrokerStub::OnRemoteRequest(uint32_t code, MessageParcel &data, 
                                     MessageParcel &reply, MessageOption &option)
{
    // 无速率限制逻辑
    switch (code) {
        case static_cast<uint32_t>(HidumperServiceInterfaceCode::DUMP_REQUEST_FILEFD):
            return RequestFileFdStub(data, reply);
        // ...
    }
}
```

**位置**: `services/native/src/dump_manager_service.cpp:165`

```cpp
int32_t DumpManagerService::Request(std::vector<std::u16string> &args, int outfd)
{
    // 直接处理，无队列限制
    auto rawParam = AddRequestRawParam(args, outfd);
    return StartRequest(rawParam);
}
```

### 触发条件

```cpp
// 恶意代码
while (true) {
    proxy->Request(fd, args);  // 无限循环调用
}
```

### 当前缓解措施

| 措施 | 有效性 |
|------|--------|
| 请求队列 | ⚠️ 存在但未验证容量 |
| 超时机制 | ❌ 缺失 |
| 频率限制 | ❌ 缺失 |

### 修复建议

1. **令牌桶限流**:
   ```cpp
   class RateLimiter {
       bool AllowRequest();
       // 实现令牌桶算法
   };
   ```

2. **请求队列上限**:
   ```cpp
   if (requestRawParamMap_.size() > MAX_PENDING_REQUESTS) {
       return ERR_TOO_MANY_REQUESTS;
   }
   ```

3. **执行超时**:
   ```cpp
   // 使用 ffrt 或类似机制设置超时
   auto future = ffrt::submit([&]() { ExecuteDump(); });
   if (future.wait_for(TIMEOUT) == std::future_status::timeout) {
       CancelRequest();
   }
   ```

---

## R5: 信息泄露 (低危)

### 风险描述

应用可能获取其他进程的敏感信息（内存布局、线程状态等）。

### 代码证据

**位置**: `frameworks/native/src/executor/memory/get_process_info.cpp`

```cpp
// 可获取任意进程内存信息
int GetProcessInfo(pid_t pid, ProcessInfo& info) {
    // 读取 /proc/<pid>/maps, smaps 等
}
```

**位置**: `frameworks/native/src/executor/cpu_dumper.cpp`

```cpp
// 可获取任意进程 CPU 使用率
GetCpuUsageByPid(pid, cpuUsage);
```

### 当前缓解措施

| 措施 | 有效性 |
|------|--------|
| `ohos.permission.DUMP` | ✅ 有效 |
| UID 检查 | ✅ 有效 |

### 修复建议

当前缓解措施已足够，建议：
1. 定期审计哪些应用拥有 `ohos.permission.DUMP` 权限
2. 考虑添加应用签名白名单

---

## R6: 权限绕过风险 (中危)

### 风险描述

权限检查可能存在绕过漏洞，如 UID 伪造、Token 重放等。

### 代码证据

**位置**: `services/native/src/dump_manager_service.cpp:195-204`

```cpp
bool DumpManagerService::HasDumpPermission() const
{
    uint32_t callingTokenID = IPCSkeleton::GetCallingTokenID();
    int res = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callingTokenID, "ohos.permission.DUMP");
    return res == PermissionState::PERMISSION_GRANTED;
}
```

**位置**: `services/native/src/dump_manager_service.cpp:172`

```cpp
// HIPROFILER_UID 白名单
if (!HasDumpPermission() && uid != HIPROFILER_UID) {
    return ERR_NO_PERMISSION;
}
```

### 当前缓解措施

| 措施 | 有效性 |
|------|--------|
| AccessTokenKit | ✅ 标准权限框架 |
| UID 验证 | ✅ 基础检查 |
| Token 验证 | ✅ Binder 内置 |

### 修复建议

当前机制已较为完善，建议：
1. 记录所有权限检查失败日志
2. 监控异常访问模式

---

## 安全检查点汇总

### 已实施的安全控制

| 控制点 | 位置 | 状态 |
|--------|------|------|
| CLI 参数长度限制 | `dump_client_main.cpp:58` | ✅ 256 字节 |
| CLI 参数数量限制 | `dump_client_main.cpp:40` | ✅ 64 个 |
| IPC 权限检查 | `dump_manager_service.cpp:195` | ✅ AccessToken |
| UID 白名单 | `dump_manager_service.cpp:172` | ✅ HIPROFILER_UID |
| 路径规范化 | `file_utils.cpp:60` | ✅ realpath |
| 命令前缀限制 | `cmd_dumper.cpp:22` | ⚠️ 部分 |

### 建议添加的安全控制

| 控制点 | 优先级 | 建议实现位置 |
|--------|--------|-------------|
| 命令白名单 | **P0** | `cmd_dumper.cpp` |
| IPC 速率限制 | P1 | `dump_manager_service.cpp` |
| PID 范围检查 | P1 | `dump_implement.cpp` |
| 路径白名单 | P2 | `fd_output.cpp` |
| 请求超时 | P2 | `dump_manager_service.cpp` |

---

## 安全测试建议

### 静态分析

```bash
# CodeCheck 扫描
codecheck -p . --enable-security

# 关注规则:
# - R001: 命令注入
# - R002: 路径遍历
# - R003: 整数溢出
```

### 动态测试

| 测试类型 | 工具 | 目标 |
|---------|------|------|
| Fuzzing | libFuzzer | CLI 参数解析 |
| Fuzzing | AFL | IPC 接口 |
| 渗透测试 | 自定义脚本 | 命令注入 |

### 已有测试覆盖

**Fuzz 测试位置**: `test/fuzztest/`

| Fuzzer | 目标 |
|--------|------|
| `client_fuzzer` | CLI 参数 |
| `processdump_fuzzer` | 进程 Dump |
| `memdump_fuzzer` | 内存 Dump |

---

## 修复优先级

| 优先级 | 风险 | 修复建议 | 预计工作量 |
|--------|------|---------|-----------|
| **P0** | R1 命令注入 | 命令白名单 | 1 天 |
| **P0** | R3 整数溢出 | PID 范围检查 | 0.5 天 |
| P1 | R2 路径遍历 | 统一路径验证 | 2 天 |
| P1 | R4 DoS | IPC 限流 | 3 天 |
| P2 | R5 信息泄露 | 权限审计 | 1 天 |
| P2 | R6 权限绕过 | 日志监控 | 0.5 天 |

---

## 相关文档

- [攻击面分析](./05_AttackSurface.md) - 完整攻击面识别
- [代码地图](./03_CodeMap.md) - 安全敏感代码位置
- [系统架构](./01_Architecture.md) - 信任边界分析
- [API 参考](./02_API_Reference.md) - IPC 接口定义
