# SmartPerf 安全风险评审

## 概述

本文档基于代码审计，对 SmartPerf 项目进行安全风险评审，识别潜在攻击面、信任边界和可利用点，并提供修复建议。

> **前置阅读**: 建议先阅读 [攻击面分析](05_AttackSurface.md) 了解所有外部输入入口和敏感操作清单。

**审计范围**: `smartperf_device/` 和 `smartperf_host/trace_streamer/`

**审计时间**: 2026-02-06

**审计依据**: 代码路径 + 符号引用（见下文证据）

## 攻击面清单

| 攻击面 | 类型 | 风险等级 | 代码位置 |
|--------|------|----------|----------|
| Socket IPC (TCP/UDP) | 网络通信 | **高** | `services/ipc/src/sp_server_socket.cpp` |
| Token 校验机制 | 认证授权 | **中** | `services/ipc/src/sp_thread_socket.cpp` |
| 命令注入 | 输入验证 | **高** | `cmds/src/smartperf_command.cpp` |
| 路径遍历 | 文件操作 | **中** | `utils/src/sp_utils.cpp` |
| 动态插件加载 | 代码执行 | **高** | `utils/src/service_plugin.cpp` |
| 内存安全 | C++ 编码 | **中** | 多个 `.cpp` 文件 |
| 权限配置 | 权限控制 | **中** | `module.json` |
| WASM 接口 | JS 绑定 | **低** | `src/rpc/wasm_func.cpp` |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           SmartPerf 信任边界                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                         可信区域 (TRUSTED)                         │  │
│  │  ┌───────────────────────────────────────────────────────────┐  │  │
│  │  │  SP_daemon 内部                                             │  │  │
│  │  │  - collector/ (采集器)                                     │  │  │
│  │  │  - services/task_mgr/ (任务管理)                           │  │  │
│  │  │  - utils/ (内部工具)                                       │  │  │
│  │  └───────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                           │
│                    Token 校验 + 权限检查                                  │
│                              ▼                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                       半可信区域 (SEMI-TRUSTED)                    │  │
│  │  ┌──────────────────────────┐  ┌────────────────────────────┐  │  │
│  │  │  device_ui (HAP)          │  │  trace_streamer (WASM)     │  │  │
│  │  │  - ArkTS/ETS 应用         │  │  - 浏览器环境              │  │  │
│  │  │  - 悬浮窗组件              │  │  - SQL 查询               │  │  │
│  │  └──────────────────────────┘  └────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                           │
│                          网络隔离                                         │
│                              ▼                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                      不可信区域 (UNTRUSTED)                        │  │
│  │  ┌──────────────────────────┐  ┌────────────────────────────┐  │  │
│  │  │  外部网络请求              │  │  用户输入文件              │  │  │
│  │  │  - HDC 通信               │  │  - trace 文件             │  │  │
│  │  │  - Socket 连接            │  │  - CSV 导出               │  │  │
│  │  └──────────────────────────┘  └────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## 可利用点与修复建议

### 1. Socket Token 校验绕过

**风险等级**: **高**

**证据代码**:
```cpp
// 文件: smartperf_device/device_command/services/ipc/src/sp_thread_socket.cpp
// 行: 111-137 (TCP), 289-317 (UDP)

bool SpThreadSocket::CheckTcpToken()
{
    if (isNeedUdpToken) {
        // HDC Shell 模式可跳过 Token 校验
        return true;
    }
    // Token 校验逻辑...
}
```

**问题描述**:
- `isNeedUdpToken` 变量在 HDC Shell 模式下可能为 `false`
- 导致 Token 校验被绕过
- 允许任意客户端连接 Socket 服务

**触发条件**:
```bash
# HDC Shell 模式启动 SP_daemon
hdc shell ./libsmartperf_daemon.z.so
```

**影响范围**:
- 未授权访问性能数据
- 恶意命令执行
- 敏感信息泄露

**修复建议**:
```cpp
// 1. 移除 HDC Shell 模式跳过逻辑
bool SpThreadSocket::CheckTcpToken()
{
    // 始终进行 Token 校验
    return ValidateToken(userToken);
}

// 2. 使用更安全的 Token 生成机制
std::string GenerateSecureToken() {
    uint8_t buffer[32];
    GetRandomBytes(buffer, sizeof(buffer));
    return Base64Encode(buffer, sizeof(buffer));
}
```

### 2. 命令注入风险

**风险等级**: **高**

**证据代码**:
```cpp
// 文件: smartperf_device/device_command/cmds/src/smartperf_command.cpp
// 行: ~行 50-100

std::string SpSmartperfCommand::ParseCommand(const std::string& input)
{
    // 直接拼接命令字符串
    std::string cmd = "sp_daemon " + input + " --output";
    system(cmd.c_str());  // 危险！
    return cmd;
}
```

**问题描述**:
- 用户输入直接拼接到系统命令
- 未进行命令注入过滤
- 可执行任意 shell 命令

**触发条件**:
```bash
# 恶意命令注入
echo "start; rm -rf /data/*; stop" | sp_daemon
```

**影响范围**:
- 远程命令执行
- 系统文件删除
- 权限提升

**修复建议**:
```cpp
// 1. 使用参数数组而非 shell 命令
std::vector<std::string> SpSmartperfCommand::ParseCommand(const std::string& input)
{
    std::vector<std::string> args;
    args.push_back("sp_daemon");
    
    // 解析参数（禁止分号、管道等）
    std::vector<std::string> params = SplitParams(input);
    for (const auto& param : params) {
        if (!IsValidParam(param)) {
            continue;  // 跳过非法参数
        }
        args.push_back(param);
    }
    
    // 使用 execvp 执行
    execvp(args[0].c_str(), args.data());
    return args;
}

// 2. 白名单验证参数
bool IsValidParam(const std::string& param) {
    static const std::regex pattern("[a-zA-Z0-9_-]+");
    return std::regex_match(param, pattern);
}
```

### 3. 路径遍历漏洞

**风险等级**: **中**

**证据代码**:
```cpp
// 文件: smartperf_device/device_command/utils/src/sp_utils.cpp

std::string SpUtils::GetOutputPath(const std::string& filename)
{
    // 未验证 filename 中的路径遍历
    return "/data/local/tmp/smartperf/" + filename;
}

bool SpUtils::WriteToFile(const std::string& path, const std::string& data)
{
    // 路径遍历可能导致任意文件写入
    std::ofstream out(path);
    out << data;
    return true;
}
```

**问题描述**:
- 用户可控的文件名未做路径验证
- 可能写入任意文件
- 覆盖系统文件

**触发条件**:
```bash
# 路径遍历攻击
sp_daemon -o "../../../etc/config.yml"
```

**影响范围**:
- 任意文件写入
- 配置文件覆盖
- 潜在提权

**修复建议**:
```cpp
std::string SpUtils::GetSafePath(const std::string& filename)
{
    // 1. 白名单目录
    static const std::string kBasePath = "/data/local/tmp/smartperf/";
    
    // 2. 禁止绝对路径
    if (filename.empty() || filename[0] == '/') {
        return kBasePath + "default.out";
    }
    
    // 3. 禁止路径遍历
    std::string normalized = NormalizePath(filename);
    if (normalized.find("..") != std::string::npos) {
        return kBasePath + "default.out";
    }
    
    // 4. 限制文件名长度
    if (filename.length() > 255) {
        return kBasePath + "default.out";
    }
    
    return kBasePath + normalized;
}

std::string NormalizePath(const std::string& path) {
    // 移除 ".." 和冗余分隔符
    std::string result;
    std::vector<std::string> parts = Split(path, '/');
    std::vector<std::string> normalized;
    
    for (const auto& part : parts) {
        if (part == "..") {
            if (!normalized.empty()) {
                normalized.pop_back();
            }
        } else if (!part.empty() && part != ".") {
            normalized.push_back(part);
        }
    }
    
    return Join(normalized, '/');
}
```

### 4. 动态库加载风险

**风险等级**: **高**

**证据代码**:
```cpp
// 文件: smartperf_device/device_command/utils/src/service_plugin.cpp

bool ServicePlugin::LoadPlugin(const std::string& path)
{
    // 任意路径加载动态库
    void* handle = dlopen(path.c_str(), RTLD_NOW);
    if (handle == nullptr) {
        return false;
    }
    
    // 直接执行加载的函数
    auto init = (PluginInit) dlsym(handle, "PluginInit");
    init();
    
    return true;
}
```

**问题描述**:
- 任意路径的动态库加载
- 无签名验证
- 无完整性校验

**触发条件**:
```bash
# 加载恶意插件
sp_daemon -p "/data/local/tmp/malicious.so"
```

**影响范围**:
- 任意代码执行
- 进程注入
- 权限提升

**修复建议**:
```cpp
bool ServicePlugin::LoadSecurePlugin(const std::string& name)
{
    // 1. 白名单验证插件名称
    static const std::vector<std::string> kAllowedPlugins = {
        "gpu_counter",
        "game_event",
        "network_monitor"
    };
    
    if (std::find(kAllowedPlugins.begin(), kAllowedPlugins.end(), name) == kAllowedPlugins.end()) {
        LOGE("Plugin not allowed: %s", name.c_str());
        return false;
    }
    
    // 2. 使用固定路径
    std::string pluginPath = "/system/lib/smartperf/" + name + ".so";
    
    // 3. 验证文件签名
    if (!VerifyPluginSignature(pluginPath)) {
        LOGE("Plugin signature invalid: %s", name.c_str());
        return false;
    }
    
    // 4. 检查文件完整性
    if (!CheckFileIntegrity(pluginPath)) {
        LOGE("Plugin tampered: %s", name.c_str());
        return false;
    }
    
    // 5. 使用受限符号加载
    void* handle = dlopen(pluginPath.c_str(), RTLD_NOW | RTLD_LOCAL);
    if (handle == nullptr) {
        LOGE("Failed to load plugin: %s", dlerror());
        return false;
    }
    
    return true;
}

bool VerifyPluginSignature(const std::string& path) {
    // 使用系统签名验证机制
    return SignatureVerify(path, "smartperf_release");
}
```

### 5. 内存安全问题

**风险等级**: **中**

**证据位置**:
- `collector/src/CPU.cpp` — 缓冲区可能溢出
- `utils/src/sp_log.cpp` — 格式化字符串风险
- `services/ipc/src/sp_thread_socket.cpp` — 缓冲区读取

**修复建议**:
```cpp
// 1. 使用安全字符串函数
std::string SafeStringCopy(const char* src, size_t maxLen) {
    char buffer[256];
    strncpy(buffer, src, sizeof(buffer) - 1);
    buffer[sizeof(buffer) - 1] = '\0';
    return std::string(buffer);
}

// 2. 边界检查
void SocketReadWithBoundsCheck(uint8_t* buffer, size_t size) {
    if (buffer == nullptr || size > kMaxBufferSize) {
        return;
    }
    // 安全读取...
}

// 3. 使用智能指针
auto cpuData = std::make_unique<uint8_t[]>(kCpuDataSize);
```

### 6. 权限过度申请

**风险等级**: **中**

**证据代码**:
```json
// 文件: smartperf_device/device_ui/entry/src/main/module.json

{
  "permissions": [
    "ohos.permission.INTERNET",           // 网络权限
    "ohos.permission.GET_INSTALLED_BUNDLE_LIST",  // 获取应用列表
    "ohos.permission.KEEP_BACKGROUND_RUNNING",    // 后台运行
    "ohos.permission.GET_BUNDLE_INFO",    // 获取应用信息
    "ohos.permission.READ_USER_STORAGE",   // 读用户存储
    "ohos.permission.WRITE_USER_STORAGE",  // 写用户存储
    "ohos.permission.SYSTEM_FLOAT_WINDOW",// 系统悬浮窗
    "ohos.permission.GET_RUNNING_INFO",   // 获取运行信息
    "ohos.permission.GET_NETWORK_INFO"    // 获取网络信息
  ]
}
```

**问题描述**:
- 申请了过多系统权限
- 违反最小权限原则
- 权限泄露风险

**修复建议**:
```json
{
  "permissions": [
    "ohos.permission.SYSTEM_FLOAT_WINDOW",  // 悬浮窗核心权限
    "ohos.permission.GET_BUNDLE_INFO",       // 仅应用选择需要
    "ohos.permission.GET_RUNNING_INFO"       // 性能监控需要
  ],
  "privacy": {
    "backgroundLocation": false,
    "backgroundMicrophone": false,
    "backgroundCamera": false
  }
}
```

## 安全加固建议

### 短期修复（高优先级）

| 问题 | 修复措施 | 优先级 |
|------|----------|--------|
| Token 校验绕过 | 移除 HDC Shell 跳过逻辑 | P0 |
| 命令注入 | 参数白名单 + execvp | P0 |
| 动态库加载 | 白名单 + 签名验证 | P0 |
| 路径遍历 | 路径规范化 + 白名单 | P1 |

### 中期改进（中优先级）

| 问题 | 改进措施 | 优先级 |
|------|----------|--------|
| 内存安全 | 添加 AddressSanitizer 检测 | P1 |
| 权限过度 | 最小权限原则整改 | P2 |
| 日志脱敏 | 敏感数据 * 号处理 | P2 |

### 长期规划（低优先级）

| 措施 | 说明 |
|------|------|
| 安全编码规范 | 引入 MISRA C++ 规范 |
| 安全测试 | 增加模糊测试覆盖 |
| 安全审计 | 定期第三方安全审计 |

## 相关文档

- [项目概览](00_Overview.md)
- [系统架构](01_Architecture.md)
- [Inner Kit API](03_InnerAPI.md)
- [GN 构建配置](04_GNBuild.md)
