# 06_Security_Review - 安全风险评审

## 1. 攻击面分析

### 1.1 外部输入点

| 攻击面 | 输入类型 | 位置 | 风险等级 |
|--------|----------|------|----------|
| **N-API 参数** | JS 字符串、数字、对象 | `napi_hidebug.cpp` | 高 |
| **插件配置** | Protobuf 二进制数据 | `onPluginSessionStart()` | 高 |
| **命令行参数** | hiprofiler_cmd 参数 | `device/cmds/` | 中 |
| **IPC 数据** | Binder 调用数据 | `native_memory_profiler_sa/` | 高 |
| **文件路径** | dump 文件路径 | 多个插件 | 高 |
| **网络数据** | gRPC 传输 | `profiler_service/` | 中 |

### 1.2 插件攻击面详细分析

#### 1.2.1 FTRACE_PLUGIN

**证据**: `device/plugins/ftrace_plugin/src/flow_controller.cpp:875`

| 输入源 | 类型 | 风险 |
|--------|------|------|
| Protobuf Config | `traceConfig.ParseFromArray(configData, size)` | 反序列化攻击 |
| System Parameter | `OHOS::system::GetParameter(TRACE_PROPERTY, "")` | 参数注入 |
| Ftrace FS | `/sys/kernel/tracing` 或 `/sys/kernel/debug/tracing` | 内核数据访问 |
| Kernel Symbols | `/proc/kallsyms` (root only) | 内核地址泄露 |
| Raw Trace Data | `/sys/kernel/debug/tracing/per_cpu/cpuX/trace_pipe_raw` | 内核数据读取 |
| Temp File | `mkstemp("/data/local/tmp/ftrace_rawdata.XXXXXX")` | TOCTOU 风险 |

**敏感操作**:
- `ftrace_data_reader.cpp:36`: `open(realPath, O_CLOEXEC | O_NONBLOCK)` - 路径遍历风险
- `flow_controller.cpp:175`: `fopen(fileName, "wb+")` - 临时文件创建
- `ftrace_fs_ops.cpp:139-146`: kptr_restrict 操作

#### 1.2.2 MEMORY_PLUGIN

**证据**: `device/plugins/memory_plugin/src/memory_data_plugin.cpp:166,1084`

| 输入源 | 类型 | 风险 |
|--------|------|------|
| Protobuf Config | `protoConfig_.ParseFromArray()` | 反序列化攻击 |
| Proc Files | `/proc/meminfo`, `/proc/vmstat` | 系统信息读取 |
| Pid-specific Files | `/proc/{pid}/smaps`, `/proc/{pid}/status` | 进程信息泄露 |
| ZRAM Sysfs | `/sys/block/zram*/mm_stat` | 系统状态读取 |
| HIDumper Command | `/system/bin/hidumper` 执行 | **命令注入风险** |

**敏感操作**:
- `memory_data_plugin.cpp:1084`: `COMMON::CustomPopen(cmdArg, 

### 1.2 信任边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                         信任边界图                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐                                               │
│  │   PC 端/开发者    │  ◄─── 不信任边界                             │
│  └────────┬────────┘                                               │
│           │ gRPC                                                  │
│           ▼                                                        │
│  ┌─────────────────┐                                               │
│  │ Profiler Service │  ◄─── 信任边界起点                            │
│  │    (SA)         │                                               │
│  └────────┬────────┘                                               │
│           │ IPC/Binder                                             │
│           ▼                                                        │
│  ┌─────────────────┐     ┌─────────────────┐                       │
│  │   Plugin Manager │ ──► │ 插件进程        │                       │
│  └────────┬────────┘     └─────────────────┘                       │
│           │                                                          │
│           │ 共享内存                                                 │
│           ▼                                                         │
│  ┌─────────────────┐     ┌─────────────────┐                       │
│  │   目标进程       │ ──► │ Native Hook    │                       │
│  │ (被分析应用)     │     └─────────────────┘                       │
│  └─────────────────┘                                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. 安全机制

### 2.1 访问控制

#### Token 验证

```cpp
// 文件: native_memory_profiler_sa_service.cpp:124-131
uint32_t callingTokenID = IPCSkeleton::GetCallingTokenID();
int res = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
    callingTokenID, "ohos.permission.ENABLE_PROFILER");
if (res != Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
    PROFILER_LOG_ERROR(LOG_CORE, "No profiling permission!");
    return false;
}
```

**所需权限**: `ohos.permission.ENABLE_PROFILER`

#### UID 检查

```cpp
// 文件: file_path_handler.cpp:74-77
int32_t callingUid = OHOS::IPCSkeleton::GetCallingUid();
if (callingUid <= APP_ID_THRESH) {  // APP_ID_THRESH = 20000000
    return false;  // 拒绝沙箱应用
}
```

#### Bundle 验证

```cpp
// 文件: common.cpp:637-679
CHECK_TRUE(!bundleName.empty(), false, "Pid or process name is illegal!");
CHECK_NOTNULL(sam, false, "GetSystemAbilityManager failed!");
CHECK_NOTNULL(remoteObject, false, "Get BundleMgr SA failed!");
```

### 2.2 输入验证

#### 路径验证

```cpp
// 文件: common.cpp:686-698
bool VerifyPath(const std::string& filePath, const std::vector<std::string>& validPaths) {
    for (const std::string& path : validPaths) {
        if (filePath.rfind(path, 0) == 0) {  // 检查前缀
            return true;
        }
    }
    return false;
}
```

#### 路径遍历防护

```cpp
// 文件: common.cpp:700-729
const char* RealPath(std::string &filePath) {
    // 检查路径结尾
    if (filePath.size() > 0 && (filePath.back() == '/' || filePath.back() == '.')) {
        return nullptr;
    }
    
    // 防止 ".." 路径遍历
    for (std::string& pathName: paths) {
        if (pathName == "..") {
            if (validPaths.size() == 0) {
                return nullptr;
            }
            validPaths.pop_back();
        }
    }
}
```

#### PATH_MAX 检查

```cpp
// 文件: memory_data_plugin.cpp:487
CHECK_TRUE((path.length() < PATH_MAX) && 
           (realpath(path.c_str(), realPath) != nullptr), "",
           "Invalid path or path too long");
```

### 2.3 内存安全

#### 空指针检查

```cpp
// 文件: logging.h:192-200
#define CHECK_NOTNULL(ptr, retval, fmt, ...) \
    do { \
        if (ptr == nullptr) { \
            HILOG_BASE_WARN(LOG_CORE, "CHECK_NOTNULL FAILED"); \
            return retval; \
        } \
    } while (0)
```

#### 整数溢出防护

```cpp
// 文件: trace_file_helper.cpp:34
if (size > std::numeric_limits<decltype(header_.data_.length)>::max() 
    - header_.data_.length - sizeof(size)) {
    // 拒绝溢出风险的操作
}
```

#### 安全字符串操作

广泛使用 `memcpy_s`、`snprintf_s`、`strcpy_s` 代替不安全的 C 函数。

### 2.4 IPC 安全

#### 文件描述符传递

```cpp
// 文件: socket_context.cpp:260-295
cmsghdr* cmsg = CMSG_FIRSTHDR(&msg);
cmsg->cmsg_level = SOL_SOCKET;
cmsg->cmsg_type = SCM_RIGHTS;  // 仅允许 SCM_RIGHTS
cmsg->cmsg_len = CMSG_LEN(sizeof(int));

if (memcpy_s(CMSG_DATA(cmsg), sizeof(int), &fd, sizeof(int)) != EOK) {
    PROFILER_LOG_ERROR(LOG_CORE, "memcpy_s error");
}
```

#### 客户端生命周期管理

```cpp
// 文件: native_memory_profiler_sa_service.cpp:79-92
void NativeMemoryProfilerSaService::ClientDisconnectCallback(int socketFd) {
    std::unique_lock<std::mutex> lock(g_pidFdMtx);
    for (auto iter = g_pidFds.begin(); iter != g_pidFds.end(); ++iter) {
        if (iter->second == socketFd) {
            StopHook(static_cast<uint32_t>(iter->first));  // 清理资源
            break;
        }
    }
}
```

---

## 3. 风险清单

### 3.1 高风险项

#### 风险 1: TOCTOU 竞态条件

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-001 |
| **类型** | 竞态条件 |
| **证据** | `process_data_plugin.cpp:159-163`, `memory_data_plugin.cpp:527-595` |
| **触发条件** | 攻击者在 `realpath()` 检查和 `fopen()` 打开之间替换文件 |
| **影响** | 可能读取/写入任意文件 |
| **代码示例** | ```cpp\nif (realpath(fileName, realPath) == nullptr) {\n    return RET_FAIL;\n}\n// ... 竞态窗口 ...\nfp = fopen(realPath, "r");\n``` |

**修复建议**:
1. 使用 `open()` 配合 `O_NOFOLLOW` 标志，原子性打开文件
2. 打开后立即使用 `fstat()` 验证文件类型
3. 考虑使用文件描述符锁

---

#### 风险 2: malloc 后缺少空指针检查

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-002 |
| **类型** | 空指针解引用 |
| **证据** | `socket_context.cpp:191-202` |
| **触发条件** | 内存分配失败时继续使用空指针 |
| **影响** | 程序崩溃 (DoS) |
| **代码示例** | ```cpp\nint8_t* data = reinterpret_cast<int8_t*>(malloc(size));  // 无检查\n// 直接使用 data\n``` |

**修复建议**:
```cpp
int8_t* data = reinterpret_cast<int8_t*>(malloc(size));
if (data == nullptr) {
    PROFILER_LOG_ERROR(LOG_CORE, "malloc failed");
    return false;
}
// ... 安全使用 data ...
free(data);
```

---

#### 风险 3: N-API 路径遍历

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-003 |
| **类型** | 路径遍历 |
| **证据** | `hidebug_util.cpp:125` |
| **触发条件** | 用户提供的文件路径包含 `../` |
| **影响** | 写入任意文件路径 |
| **当前防护** | 仅检查 `"./"` 前缀 |
| **代码示例** | ```cpp\nif (dumpPath.rfind("./", 0) == 0) {\n    return false;\n}\n``` |

**修复建议**:
1. 增强路径验证逻辑
2. 使用 `realpath()` 规范化路径后验证
3. 仅允许特定白名单目录

---

#### 风险 4: 插件命令注入 (高危)

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-004 |
| **类型** | 命令注入 |
| **证据** | `hiperf_plugin/src/hiperf_module.cpp:107`, `memory_plugin/src/memory_data_plugin.cpp:1084` |
| **触发条件** | `config.record_args()` 包含 shell 元字符 |
| **影响** | 以 root 权限执行任意命令 |
| **风险等级** | **高危** |
| **代码示例** | ```cpp\n// hiperf_module.cpp:107\nCOMMON::CustomPopen(cmdArg, "r", pipeFds, childPid);\n\n// memory_data_plugin.cpp:1084\nCOMMON::CustomPopen(cmdArg, "r", pipeFds, childPid);\n``` |

**攻击示例**:
```protobuf
plugin_configs {
  plugin_name: "hiperf-plugin"
  config_data {
    record_args: "-f 1000; cat /etc/passwd > /tmp/leak.txt; echo"
    is_root: true
  }
}
```

**修复建议**:
1. **使用 execve() 替代 system()/popen()**: 传递参数数组而非字符串
2. **严格参数白名单**: 只允许预定义的合法参数
3. **移除 is_root 客户端控制**: 服务端决定是否以 root 执行
4. **输入过滤**: 禁止 `;`, `|`, `&`, `$`, `` ` `` 等 shell 元字符

---

#### 风险 5: Protobuf 反序列化 DoS

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-005 |
| **类型** | 拒绝服务 |
| **证据** | 所有插件的 `ParseFromArray(configData, configSize)` |
| **触发条件** | 超大 protobuf 消息或递归嵌套 |
| **影响** | 内存耗尽、CPU 占满 |
| **代码示例** | ```cpp\n// 所有插件通用模式\nif (!protoConfig_.ParseFromArray(configData, configSize)) {\n    return false;\n}\n``` |

**修复建议**:
1. 添加 `configSize` 上限检查（如 1MB）
2. 使用 protobuf 的 `SetTotalBytesLimit()` 限制
3. 设置解析超时

---

### 3.3 中风险项

#### 风险 6: 日志信息泄露

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-006 |
| **类型** | 信息泄露 |
| **证据** | 多处 `HILOG_BASE_WARN/ERROR` 输出内部路径 |
| **触发条件** | 查看系统日志 |
| **影响** | 泄露内部路径结构、配置信息 |
| **代码示例** | ```cpp\nPROFILER_LOG_ERROR(LOG_CORE, "Open file failed: %{public}s", filePath.c_str());\n``` |

**修复建议**:
1. 日志中移除敏感路径信息
2. 使用 `%{private}s` 格式化
3. 建立日志脱敏机制

---

#### 风险 7: GWP-ASan 配置参数验证不足

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-007 |
| **类型** | 资源耗尽 |
| **证据** | `napi_hidebug.cpp:954-976` |
| **触发条件** | 恶意应用启用过多 GWP-ASan 采样 |
| **影响** | 内存消耗过高 |
| **当前限制** | `OVER_ENABLE_LIMIT` 检查 |

**修复建议**:
1. 限制单个应用可启用的 GWP-ASan 总数
2. 添加系统级资源配额

---

#### 风险 8: 插件路径遍历

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-008 |
| **类型** | 路径遍历 |
| **证据** | `memory_plugin/src/memory_data_plugin.cpp:586`, `network_plugin/src/network_plugin.cpp:110` |
| **触发条件** | PID 或文件名包含 `../` |
| **影响** | 读取/写入任意文件 |
| **代码示例** | ```cpp\n// memory_plugin.cpp:586\nsnprintf_s(fileName, sizeof(fileName), "%s/%d/%s", testpath_, pid, pFileName);\n``` |

**修复建议**:
1. 验证 PID 为纯数字
2. 验证文件名不包含路径分隔符
3. 使用 `realpath()` 规范化后检查前缀

---

#### 风险 9: Native Hook Socket 安全

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-009 |
| **类型** | IPC 劫持 |
| **证据** | `native_hook/src/hook_socket_client.cpp:94` |
| **触发条件** | Unix Socket 路径被劫持 |
| **影响** | 恶意进程注入 Hook 配置 |
| **代码示例** | ```cpp\nConnect(DEFAULT_UNIX_SOCKET_HOOK_FULL_PATH);\n``` |

**修复建议**:
1. 使用抽象命名空间或随机化 socket 名称
2. 验证连接对等身份
3. 使用权限位限制访问

---

### 3.3 低风险项

#### 风险 10: 插件符号导出过多

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-010 |
| **类型** | 符号泄露 |
| **证据** | `libnative_hook.map` 控制导出 |
| **影响** | 攻击者可能调用内部函数 |
| **当前防护** | 版本脚本限制符号导出 |

**建议**: 定期审查导出符号列表

---

#### 风险 11: 整数溢出

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-011 |
| **类型** | 整数溢出 |
| **证据** | `memory_data_plugin.cpp:461,769,816-854`, `network_plugin.cpp:273,308,379` |
| **触发条件** | `strtol`/`strtoul`/`strtoull` 转换超大数值 |
| **影响** | 缓冲区计算错误、资源分配异常 |

**修复建议**:
1. 添加范围检查
2. 使用 `std::stoll` 替代并捕获异常
3. 验证转换后的值在合理范围内

---

## 4. 安全建议

### 4.1 高优先级 (P0)

| 建议 | 优先级 | 复杂度 | 预估影响 |
|------|--------|--------|----------|
| **修复插件命令注入漏洞** | **P0** | **中** | **消除高危代码执行风险** |
| 修复 TOCTOU 竞态条件 | P0 | 中 | 消除任意文件读写风险 |
| 添加 malloc 空指针检查 | P0 | 低 | 防止 DoS |
| 增强路径遍历防护 | P0 | 低 | 消除路径注入 |
| 添加 Protobuf 大小限制 | P0 | 低 | 防止反序列化 DoS |

### 4.2 中优先级

| 建议 | 优先级 | 复杂度 | 预估影响 |
|------|--------|--------|----------|
| 日志脱敏 | P1 | 低 | 减少信息泄露 |
| 资源配额限制 | P1 | 中 | 防止资源耗尽 |
| 插件沙箱化 | P1 | 高 | 隔离插件崩溃 |

### 4.3 低优先级

| 建议 | 优先级 | 复杂度 | 预估影响 |
|------|--------|--------|----------|
| 安全编码规范培训 | P2 | 中 | 长期改进 |
| 定期安全审计 | P2 | 低 | 持续合规 |
| 模糊测试覆盖 | P2 | 中 | 发现潜在漏洞 |

---

## 5. 测试建议

### 5.1 模糊测试

当前已有 Fuzz 测试框架:

```bash
# 运行 profiler service fuzz tests
./profilerservice_fuzzer

# 运行 native daemon fuzz tests  
./nativedaemon_fuzzer
```

**建议扩展**:
- N-API 参数模糊测试
- 插件配置 protobuf 模糊测试
- IPC 数据模糊测试

### 5.2 安全测试用例

| 测试场景 | 输入 | 预期行为 |
|----------|------|----------|
| 路径遍历 | `"../../../etc/passwd"` | 拒绝 |
| 溢出路径 | 长度 > PATH_MAX | 拒绝 |
| 非法权限调用 | 无 token 调用 SA | 拒绝 |
| 竞态窗口 | TOCTOU 利用尝试 | 防护生效 |

---

## 6. 相关跳转

| 主题 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |
| 故障排查 | [07_Troubleshooting.md](./07_Troubleshooting.md) |

---

## 附录 A: 安全相关源码索引

### A.1 访问控制与权限检查

| 主题 | 文件路径 | 行号 |
|------|----------|------|
| Token 验证 | `native_memory_profiler_sa_service.cpp` | 124-131 |
| UID 检查 | `file_path_handler.cpp` | 74-77 |
| Bundle 验证 | `common.cpp` | 637-679 |
| Beta 版本检查 | `hidebug_util.cpp` | 205-208 |
| 开发者模式检查 | `hidebug_util.cpp` | 216-219 |

### A.2 输入验证

| 主题 | 文件路径 | 行号 |
|------|----------|------|
| 路径验证 | `common.cpp` | 686-729 |
| 路径合法性 | `hidebug_util.cpp` | 123-126 |
| N-API 字符串验证 | `napi_util.cpp` | 110-128 |
| 资源限制范围 | `napi_hidebug.cpp` | 672-696 |

### A.3 内存安全

| 主题 | 文件路径 | 行号 |
|------|----------|------|
| malloc 检查缺失 | `socket_context.cpp` | 191-202 |
| 整数溢出防护 | `trace_file_helper.cpp` | 34 |
| 空指针检查宏 | `logging.h` | 192-200 |

### A.4 IPC 安全

| 主题 | 文件路径 | 行号 |
|------|----------|------|
| IPC FD 传递 | `socket_context.cpp` | 260-295 |
| Socket 路径 | `socket_context.h` | 26-44 |
| Unix Socket 客户端 | `hook_socket_client.cpp` | 94 |

### A.5 插件安全（新增）

| 主题 | 文件路径 | 行号 |
|------|----------|------|
| **命令注入 - HiPerf** | `hiperf_plugin/src/hiperf_module.cpp` | 107 |
| **命令注入 - Memory** | `memory_plugin/src/memory_data_plugin.cpp` | 1084 |
| Protobuf 解析 - Ftrace | `ftrace_plugin/src/flow_controller.cpp` | 875 |
| Protobuf 解析 - Memory | `memory_plugin/src/memory_data_plugin.cpp` | 166 |
| PID 路径构造 | `memory_plugin/src/memory_data_plugin.cpp` | 586 |
| kptr_restrict 操作 | `ftrace_plugin/src/ftrace_fs_ops.cpp` | 139-146 |
| 网络统计路径 | `network_plugin/src/network_plugin.cpp` | 110-112 |
| Native Hook Socket | `native_hook/src/hook_socket_client.cpp` | 94 |
| 插件加载 | `api/src/plugin_module.cpp` | - |
| 整数溢出 - Memory | `memory_plugin/src/memory_data_plugin.cpp` | 461, 769, 816-854 |
| 整数溢出 - Network | `network_plugin/src/network_plugin.cpp` | 273, 308, 379 |

### A.6 错误码定义

| 主题 | 文件路径 |
|------|----------|
| N-API 错误码 | `hidebug/interfaces/js/kits/napi/util/error_code.h` |
| ANI 错误码 | `hidebug/interfaces/ets/ani/hidebug/include/error_code.h` |

---

*最后更新: 2026-02-06*
