# 安全风险评审

## 概述

本章节对 appspawn 组件进行安全风险评审，涵盖攻击面分析、信任边界、潜在安全风险及修复建议。

## 攻击面清单

| 攻击面 | 类型 | 描述 | 风险等级 |
|--------|------|------|---------|
| Unix Domain Socket | IPC | 客户端连接入口 | 高 |
| 消息解析 (TLV) | 数据解析 | 解析客户端发送的二进制消息 | 高 |
| JSON配置解析 | 配置文件 | 解析沙箱配置JSON | 中 |
| fork/execve | 系统调用 | 创建子进程 | 高 |
| setuid/setgid | 权限设置 | 设置进程UID/GID | 高 |
| mount namespace | 隔离 | 创建沙箱挂载 | 中 |
| SELinux | MAC | 安全上下文设置 | 中 |
| 文件路径操作 | 文件系统 | 读写配置文件 | 中 |

## 信任边界

```
┌─────────────────────────────────────────────────────────┐
│                      信任边界                            │
│  ┌─────────────────────────────────────────────────┐   │
│  │              appspawn 服务进程                    │   │
│  │  (UID=root, SELinux=u:r:appspawn:s0)           │   │
│  │                                                 │   │
│  │  信任:                                          │   │
│  │  - 配置文件 (/system/etc/appdata-sandbox*.json) │   │
│  │  - 系统库 (/system/lib/*.so)                    │   │
│  │  - 来自 clients (AMS等) 的 socket 消息          │   │
│  │                                                 │   │
│  │  不信任:                                        │   │
│  │  - 客户端输入的所有数据                          │   │
│  │  - 用户空间文件路径                              │   │
│  │  - 外部网络输入                                  │   │
│  └─────────────────────────────────────────────────┘   │
│                        ↓                                │
│              fork/execve 创建子进程                      │
│                        ↓                                │
│  ┌─────────────────────────────────────────────────┐   │
│  │              应用子进程                          │   │
│  │  (UID/GID已设置, 沙箱已配置)                     │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## 数据流安全分析

```
客户端(AMS)
    ↓ (Unix Socket)
┌──────────────────────────────────────────────────────┐
│ appspawn 服务进程                                      │
│  1. Socket接收 (access control via UID check)        │
│  2. TLV消息解析 (长度校验, 类型校验)                   │
│  3. JSON配置解析 (路径验证)                           │
│  4. 权限设置 (setuid/setgid/setgroups)               │
│  5. 沙箱配置 (mount namespace)                        │
│  6. fork + execve                                   │
└──────────────────────────────────────────────────────┘
    ↓
子进程 (低权限)
```

## 安全机制

### 1. Socket 连接认证

**代码位置**: `standard/appspawn_service.c:348-366`

```c
static bool OnConnectionUserCheck(uid_t uid)
{
    // 只允许特定UID连接
    const uid_t uids[APPSPAWN_MSG_USER_CHECK_COUNT] = {
        #include "app_spawn_msg_user_check.inc"
    };
    // ... 检查逻辑
}
```

**评估**: ✅ 已实现UID白名单检查

### 2. DAC 权限设置

**代码位置**: `modules/common/appspawn_common.c:296-317`

```c
// 验证UID/GID范围
APPSPAWN_CHECK(dacInfo->uid >= MIN_VALID_APP_UID && 
               dacInfo->gid >= MIN_VALID_APP_GID, 
               return APPSPAWN_MSG_INVALID, "uid invalid");

// 设置groups
int ret = setgroups(dacInfo->gidCount, (const gid_t *)(&dacInfo->gidTable[0]));

// 设置GID
ret = setresgid(dacInfo->gid, dacInfo->gid, dacInfo->gid);

// 设置UID
ret = setresuid(dacInfo->uid, dacInfo->uid, dacInfo->uid);
```

**评估**: ✅ 已实现完整的DAC权限设置流程

### 3. SELinux 安全上下文

**代码位置**: `modules/common/appspawn_common.c:597-606`

```c
// 加载SELinux配置
APPSPAWN_CHECK_LOGV(LoadSeLinuxConfig() == 0, 
                    return -1, "Failed to load selinux config");

// 设置子进程安全上下文
const char *selinuxLabel = GetAppSandboxSelinuxLabel(context);
// ...
setcon(selinuxLabel);
```

**评估**: ✅ 已实现SELinux标签设置（可选）

### 4. Sandbox 隔离

**代码位置**: `modules/sandbox/modern/appspawn_sandbox.c`

```c
// 检查挂载点路径
ret = access(buffer, F_OK);
// ...

// 构建沙箱根路径
len = sprintf_s(buffer, bufferLen, "%s/%d", 
                sandbox->rootPath, uid);

// Mount namespace
unshare(CLONE_NEWNS);
```

**评估**: ✅ 已实现完整的沙箱隔离机制

### 5. 消息完整性校验

**代码位置**: `modules/module_engine/include/appspawn_msg.h`

```c
#define APPSPAWN_MSG_MAGIC 0xEF201234
#pragma pack(4)
typedef struct {
    uint32_t magic;      // 魔数校验
    uint32_t msgType;    // 消息类型
    uint32_t msgLen;     // 消息长度
    // ...
} AppSpawnMsg;
```

**评估**: ✅ 已实现消息魔数校验

## 识别的安全风险

### 高风险

#### 1. Socket 路径竞争条件 (TOCTOU)

**证据**: `appspawn_service.c:1185-1200`

```c
int ret = snprintf_s(path, sizeof(path), sizeof(path) - 1, 
                     "%s%s", APPSPAPAWN_SOCKET_DIR, socketName);
// ...
int socketId = GetControlSocket(socketName);
```

**问题**: 在创建socket前后存在时间窗口，可能被恶意利用

**影响**: 可能创建恶意socket或劫持连接

**修复建议**: 
- 使用原子操作创建socket
- 创建后立即设置权限
- 使用seccomp限制操作

#### 2. 消息长度溢出风险

**证据**: `modules/module_engine/include/appspawn_msg.h`

```c
#define MAX_MSG_TOTAL_LENGTH (64 * 1024)
#define MAX_TLV_COUNT 128
```

**问题**: 虽有限制，但TLV解析循环未完全防止整数溢出

**影响**: 可能导致缓冲区溢出

**修复建议**:
```c
// 添加长度总和检查
if (msgHeader.msgLen > MAX_MSG_TOTAL_LENGTH) {
    return APPSPAWN_MSG_INVALID;
}
```

#### 3. JSON 解析器安全

**证据**: 使用 cJSON 库解析沙箱配置

**问题**: cJSON 不提供内置的JSON Schema验证

**影响**: 恶意JSON可能导致解析错误或拒绝服务

**修复建议**:
- 实现JSON Schema验证
- 限制JSON解析深度
- 添加超时机制

### 中风险

#### 4. 挂载路径验证不完整

**证据**: `modules/sandbox/modern/appspawn_sandbox.c`

```c
// 仅检查路径是否存在
ret = access(buffer, F_OK);
```

**问题**: 未验证路径是否在允许的目录范围内

**影响**: 可能挂载敏感系统目录

**修复建议**:
```c
// 添加路径前缀检查
if (strncmp(buffer, SANDBOX_ROOT_PATH, 
            strlen(SANDBOX_ROOT_PATH)) != 0) {
    return APPSPAWN_SANDBOX_INVALID;
}
```

#### 5. 信号处理竞态

**证据**: `standard/appspawn_service.c:126-188`

```c
// 信号处理中访问appInfo
HandleDiedPid(pid, siginfo->ssi_uid, status);
```

**问题**: SIGCHLD处理与主循环可能存在竞态

**影响**: 可能访问已释放的内存

**修复建议**: 使用锁保护appInfo列表

### 低风险

#### 6. 日志信息泄露

**评估**: 代码中使用 `APPSPAWN_LOGE` 打印错误日志

**潜在问题**: 日志可能泄露敏感信息（UID、路径等）

**建议**: 敏感信息脱敏后打印

#### 7. 调试接口暴露

**证据**: `modules/common/appspawn_dfx_dump.cpp`

**潜在问题**: 调试功能可能泄露系统信息

**建议**: release版本禁用调试接口

## 权限依赖分析

### 需要的能力 (Capabilities)

appspawn 进程需要以下Linux capabilities:

| Capability | 用途 | 风险 |
|------------|------|------|
| CAP_SYS_ADMIN | 创建namespace, mount | 高 |
| CAP_SETUID | 设置进程UID | 高 |
| CAP_SETGID | 设置进程GID | 高 |
| CAP_SETPCAP | 修改capabilities | 中 |
| CAP_NET_BIND_SERVICE | 绑定低端口 | 低 |

### 最小权限原则

appspawn 应尽可能以最小权限运行：

1. fork后立即降权
2. 只保留必要的capability
3. 使用seccomp过滤系统调用

## 安全加固建议

### 立即建议

1. **完善Socket路径验证**
   - 添加socket目录的写保护
   - 使用抽象socket地址

2. **加强消息解析**
   - 添加完整的TLV长度校验
   - 实现深度检查防止嵌套攻击

3. **增加seccomp过滤**
   - 只允许必要的系统调用
   - 阻止危险操作

### 中期建议

4. **实施沙箱配置签名验证**
   - 对JSON配置文件进行签名验证
   - 防止配置被篡改

5. **增加模糊测试**
   - 对消息解析器进行模糊测试
   - 发现潜在溢出漏洞

6. **代码签名验证**
   - 验证子进程可执行文件签名
   - 防止恶意代码执行

### 长期建议

7. **考虑使用硬件安全特性**
   - SELinux强制模式
   - eBPF filter
   - Intel CET

## 安全检查清单

### 发布前检查

- [ ] 所有输入已校验
- [ ] 无缓冲区溢出
- [ ] 竞态条件已处理
- [ ] 敏感信息已脱敏
- [ ] seccomp已配置
- [ ] SELinux策略已验证
- [ ] 调试代码已移除
- [ ] 日志级别正确

### CI/CD检查

- [ ] 静态分析通过
- [ ] 模糊测试通过
- [ ] 权限扫描通过
- [ ] 依赖扫描通过

## 相关安全文档

- OpenHarmony 安全规范
- SELinux 配置指南
- Linux Capability 指南
- Seccomp BPF 指南

## 附录：检查范围说明

本安全评审基于以下范围：

- **已检查文件**: 
  - `standard/appspawn_service.c`
  - `modules/common/appspawn_common.c`
  - `modules/sandbox/modern/*.c`
  - `interfaces/innerkits/client/*.c`

- **未检查**:
  - 测试代码 (`test/`)
  - 第三方库 (`cJSON`等内置库除外)
  - 内核实现

- **限制**:
  - 未进行实际渗透测试
  - 未验证所有配置场景
  - 基于代码静态分析
