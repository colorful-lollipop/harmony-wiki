# 安全风险评审

## 目的

本文档基于代码证据分析 hiperf 的安全风险，包括攻击面、信任边界和可被利用点。

## 适用范围

- 安全审计人员
- 系统安全架构师
- 代码审查人员

## 检查范围

- **代码范围**: `src/`, `include/`, `interfaces/` 目录下的生产代码
- **排除范围**: `test/` 目录下的测试代码
- **分析深度**: 静态代码分析，重点关注输入验证、权限控制和数据处理

## 权限模型

### 1. 入口权限检查

**代码位置**: `src/main.cpp:64-67`

```cpp
if (!GetDeveloperMode() && !IsAllowProfilingUid()) {
    printf("error: not in developermode, exit.\n");
    return -1;
}
```

**允许的 UID** (`include/utilities.h:104`):
```cpp
inline const std::set<int> ALLOW_UIDS = {1201};
```

### 2. 应用调试检查

**代码位置**: `src/ipc_utilities.cpp:68-99`

```cpp
bool IsDebugableApp(const std::string& bundleName) {
    // 通过 SAMGR 获取 BundleMgr 服务
    sptr<ISystemAbilityManager> sam = 
        SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    // 检查应用是否标记为 debuggable
    proxy->IsDebuggableApplication(bundleName, isDebugApp);
}
```

### 3. 加密应用检查

**代码位置**: `src/ipc_utilities.cpp:101-130`

```cpp
bool IsApplicationEncryped(const int pid) {
    // 获取应用信息
    proxy->GetApplicationInfo(bundleName, ...);
    // 检查 ENCRYPTED_APPLICATION 标志
    bool isEncrypted = (appInfo.applicationReservedFlag &
        static_cast<uint32_t>(AppExecFwk::ApplicationReservedFlag::ENCRYPTED_APPLICATION)) != 0;
}
```

## 攻击面清单

### 1. 命令行接口

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| 参数解析 | 中 | 大量命令行参数需要验证 |
| 文件路径 | 高 | 输出文件路径可控 |
| 进程/线程 ID | 中 | PID/TID 参数需要验证 |

### 2. 文件系统访问

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| /proc 访问 | 高 | 读取进程信息 |
| ELF 解析 | 高 | 解析外部 ELF 文件 |
| HAP 解析 | 高 | 解析应用包 |
| 数据文件 | 中 | 读写 perf.data |

### 3. 系统调用

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| perf_event_open | 高 | 内核接口 |
| mmap | 中 | 内存映射 |
| fork/exec | 高 | 创建子进程 |

### 4. IPC 通信

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| SAMGR | 中 | 系统服务通信 |
| 匿名管道 | 中 | Client API 通信 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                     不可信区域                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   用户输入    │  │   外部文件    │  │   网络数据    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ 输入验证
┌─────────────────────────────────────────────────────────────┐
│                     半可信区域                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   命令参数    │  │   /proc 数据  │  │   IPC 响应    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ 权限检查
┌─────────────────────────────────────────────────────────────┐
│                      可信区域                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   内核接口    │  │   系统服务    │  │   内部数据    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## 可被利用点

### 1. UID 硬编码绕过

**风险等级**: 高

**证据**:
```cpp
// include/utilities.h:104
inline const std::set<int> ALLOW_UIDS = {1201};

// src/main.cpp:64-67
if (!GetDeveloperMode() && !IsAllowProfilingUid()) {
    printf("error: not in developermode, exit.\n");
    return -1;
}
```

**触发路径**:
1. 获取 UID 1201 的进程权限
2. 直接执行 hiperf 命令

**影响**:
- 绕过开发者模式限制
- 可采集任意进程性能数据

**修复建议**:
- 使用能力（Capability）机制替代 UID 白名单
- 添加额外的身份验证

### 2. 路径遍历风险

**风险等级**: 高

**证据**:
```cpp
// src/subcommand_record.cpp:508
printf("Invalid output file path, permission denied\n");

// src/utilities.cpp:1020-1037
bool NeedAdaptSandboxPath(char *filename, const int pid, u16 &headerSize) {
    std::string newFilename = "/proc/" + std::to_string(pid) + "/root" + oldFilename;
    // ...
    strncpy_s(filename, KILO, newFilename.c_str(), newFilename.size());
}
```

**触发路径**:
1. 使用 `--app` 参数指定应用
2. 构造特殊的路径名
3. 触发沙箱路径适配逻辑

**影响**:
- 可能访问受限目录
- 符号链接攻击

**修复建议**:
- 严格验证路径格式
- 使用 `O_NOFOLLOW` 打开文件
- 限制路径前缀白名单

### 3. 命令注入风险

**风险等级**: 高

**证据**:
```cpp
// interfaces/innerkits/native/hiperf_client/src/hiperf_client.cpp:527-547
void Client::ChildRunExecv(std::vector<std::string> &cmd) {
    char *argv[cmd.size() + SIZE_ARGV_TAIL];
    for (i = 0; i < cmd.size(); ++i) {
        argv[i] = cmd[i].data();
    }
    argv[i] = nullptr;
    execv(argv[0], argv);
}
```

**触发路径**:
1. 应用调用 `hiperf_client` API
2. 构造包含特殊字符的参数
3. 通过管道传递给 hiperf 进程

**影响**:
- 可能执行任意命令
- 权限提升

**修复建议**:
- 严格验证所有参数
- 使用参数白名单
- 转义特殊字符

### 4. 整数溢出风险

**风险等级**: 中

**证据**:
```cpp
// src/subcommand_record.cpp:336
uint64_t dataSizeLimit_ = 0;

// src/perf_file_writer.cpp:130
CHECK_TRUE(record.GetSize() <= RECORD_SIZE_LIMIT_SPE, false, 1, ...);
```

**触发路径**:
1. 设置极大的 `--data-limit` 参数
2. 导致整数溢出

**影响**:
- 内存分配异常
- 程序崩溃

**修复建议**:
- 添加范围检查
- 使用饱和算术

### 5. 竞态条件风险

**风险等级**: 中

**证据**:
```cpp
// src/subcommand_record.cpp:749-775
const auto endTime = startTime + std::chrono::seconds(CHECK_TIMEOUT);
while (std::chrono::steady_clock::now() < endTime) {
    // 检查应用状态
    std::this_thread::sleep_for(milliseconds(CHECK_FREQUENCY));
}
```

**触发路径**:
1. 使用 `--app` 参数采样应用
2. 应用在检查间隙快速重启
3. 导致 PID 混淆

**影响**:
- 采样错误进程
- 数据污染

**修复建议**:
- 使用原子操作
- 添加额外的验证步骤

### 6. 缓冲区溢出风险

**风险等级**: 中

**证据**:
```cpp
// src/subcommand_record.cpp:333
uint32_t callStackDwarfSize_ = MAX_SAMPLE_STACK_SIZE;  // 65528

// include/subcommand_record.h:333
bool isCallStackDwarf_ = false;
bool isCallStackFp_ = false;
```

**触发路径**:
1. 使用 `-s dwarf,65528` 参数
2. 分配大缓冲区
3. 可能导致内存压力

**影响**:
- 内存耗尽
- 系统不稳定

**修复建议**:
- 限制缓冲区大小
- 添加内存使用监控

### 7. 信息泄露风险

**风险等级**: 低

**证据**:
```cpp
// src/utilities.cpp:1010-1018
std::string GetProcessName(const int pid) {
    std::string filePath = "/proc/" + std::to_string(pid) + "/cmdline";
    std::string bundleName = ReadFileToString(filePath);
    return bundleName.substr(0, strlen(bundleName.c_str()));
}
```

**触发路径**:
1. 采样系统进程
2. 获取进程命令行信息

**影响**:
- 泄露敏感进程信息

**修复建议**:
- 限制可采样进程范围
- 敏感信息脱敏

### 8. 资源耗尽风险

**风险等级**: 中

**证据**:
```cpp
// include/subcommand_record.h:50
static constexpr int MAX_SAMPLE_FREQUENCY = 100000;

// src/subcommand_record.cpp:1154
CHECK_TRUE(SetPerfMaxSampleRate(), false, 1, ...);
```

**触发路径**:
1. 设置极高的采样频率
2. 消耗大量 CPU/内存资源

**影响**:
- 系统性能下降
- 服务不可用

**修复建议**:
- 实施资源配额
- 动态调整采样率

## 修复建议汇总

### 高优先级

1. **移除 UID 硬编码**: 使用 Capability 机制
2. **路径验证**: 实现严格的路径白名单
3. **命令参数验证**: 过滤/转义特殊字符

### 中优先级

4. **整数溢出防护**: 添加范围检查
5. **竞态条件修复**: 使用原子操作
6. **资源限制**: 实施配额机制

### 低优先级

7. **信息泄露防护**: 敏感信息脱敏
8. **日志安全**: 避免敏感信息泄露

## 检查局限性

1. **静态分析限制**: 未进行动态 fuzz 测试
2. **依赖分析**: 未深入分析第三方库安全
3. **内核接口**: 依赖内核 perf_event 安全实现
4. **系统服务**: 依赖 SAMGR 和 BundleMgr 的安全实现

## 相关跳转

- [项目定位](01_Overview.md) - 了解权限模型
- [架构说明](03_Architecture.md) - 了解数据流
- [GN Targets](06_GN_Targets.md) - 了解编译选项
