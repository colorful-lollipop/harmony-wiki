# 攻击面分析

本文档从安全研究员视角，分析 init 模块的外部输入入口、敏感操作和信任边界。

## 1. 外部输入清单

### 1.1 配置文件输入

| 输入源 | 文件路径 | 说明 | 信任级别 |
|--------|----------|------|----------|
| **init.cfg** | `/vendor/etc/init/init.cfg` | 主配置文件，JSON 格式 | 高（厂商签名） |
| **额外 .cfg** | `/etc/patch.cfg`, `/patch/fstab.cfg` | loadcfg 命令加载 | 中 |
| **参数文件** | `/system/etc/param/`, `/vendor/etc/param` | 系统参数 | 中 |

**证据**（`services/init/lite/init.c:53-58`）：
```c
int ParseCfgByPriority(const char *filePath)
{
    ReadFileInDir(OTHER_CFG_PATH, ".cfg", ParseInitCfg, NULL);
    ReadFileInDir("/vendor/etc/init", ".cfg", ParseInitCfg, NULL);
}
```

**证据**（`services/init/lite/init_cmds.c:53-71`）：
```c
static bool CheckValidCfg(const char *path)
{
    // 仅允许特定路径
    static const char *supportCfg[] = {
        "/etc/patch.cfg",
        "/patch/fstab.cfg",
    };
    // ... 路径白名单检查
}
```

### 1.2 Jobs 命令输入

Jobs 中的命令参数来自 init.cfg：

| 命令 | 参数来源 | 处理位置 |
|------|----------|----------|
| `mkdir` | 路径参数 | `services/init/lite/init_cmds.c` |
| `chmod` | 权限数值 | `services/init/lite/init_cmds.c` |
| `chown` | uid/gid | `services/init/lite/init_cmds.c` |
| `mount` | 设备/挂载点/类型 | `services/init/lite/init_cmds.c` |
| `start` | 服务名 | `services/init/lite/init_service.c` |
| `loadcfg` | 配置文件路径 | `services/init/lite/init_cmds.c:73` |
| `exec` | 可执行文件路径 | `services/init/lite/init_cmds.c:31` |

### 1.3 服务配置输入

| 配置项 | 用途 | 风险说明 |
|--------|------|----------|
| `path` | 服务可执行文件路径 | **路径劫持风险** |
| `uid` | 用户 ID | 权限配置风险 |
| `gid` | 组 ID | 权限配置风险 |
| `caps` | Linux Capabilities | 特权提升风险 |
| `importance` | 关键进程标志 | 系统复位风险 |
| `once` | 单次运行标志 | 进程管理风险 |

**证据**（`services/init/lite/init_service.c:81-101`）：
```c
int ServiceExec(Service *service, const ServiceArgs *pathArgs)
{
    // 直接使用配置文件中的 path 执行服务
    INIT_ERROR_CHECK(execv(pathArgs->argv[0], pathArgs->argv) == 0,
        return errno, "[startup_failed]failed to execv %s", service->name);
}
```

### 1.4 系统参数输入

| 输入源 | 接口 | 说明 |
|--------|------|------|
| 参数文件 | `LoadDefaultParams()` | 加载持久化参数 |
| 运行时参数 | `SetParameter()` | 运行时设置参数 |

---

## 2. 敏感操作清单

### 2.1 特权系统调用

| 系统调用 | 用途 | 风险等级 |
|----------|------|----------|
| `execv()` | 执行服务 | **高** - 可执行任意程序 |
| `fork()` | 创建子进程 | **高** - 资源消耗 |
| `setuid()` | 设置用户 ID | **高** - 权限变更 |
| `setgid()` | 设置组 ID | **高** - 权限变更 |
| `setpriority()` | 设置进程优先级 | **中** - 资源管理 |
| `mount()` | 挂载文件系统 | **高** - 文件系统访问 |
| `reboot()` | 系统重启 | **高** - 系统可用性 |
| `mknod()` | 创建设备节点 | **高** - 设备访问 |

**证据**（`services/init/lite/init_cmds.c:31-51`）：
```c
static void DoExec(const struct CmdArgs *ctx)
{
    pid_t pid = fork();
    if (pid == 0) {
        int ret = execve(ctx->argv[0], ctx->argv, NULL);  // 直接执行用户可控路径
        if (ret == -1) {
            INIT_LOGE("DoExec: execute \"%s\" failed", ctx->argv[0]);
        }
        _exit(0x7f);
    }
}
```

### 2.2 文件系统操作

| 操作 | 命令 | 风险 |
|------|------|------|
| 创建目录 | `mkdir` | 路径遍历 |
| 修改权限 | `chmod` | 权限配置错误 |
| 修改所有者 | `chown` | 所有权配置错误 |
| 挂载文件系统 | `mount` | 挂载点劫持 |

### 2.3 进程管理操作

| 操作 | 触发条件 | 影响 |
|------|----------|------|
| 服务启动 | `start` 命令 | 执行任意程序 |
| 服务重启 | SIGCHLD | 进程循环 |
| 系统复位 | importance=1 进程退出 | 系统不可用 |

**证据**（`services/init/lite/init_signal_handler.c:33-45`）：
```c
void ReapService(Service *service)
{
    if (service->attribute & SERVICE_ATTR_IMPORTANT) {
        // 关键进程退出 → 系统复位
        service->pid = -1;
        StopAllServices(0, NULL, 0, NULL);
        RebootSystem();  // 系统重启
    }
    ServiceReap(service);
}
```

---

## 3. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         信任边界划分                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────┐     ┌─────────────────────────────┐      │
│  │     高信任区域 (内核)       │     │    中信任区域 (init.cfg)     │      │
│  │                            │     │                             │      │
│  │  - Kernel                 │     │  - /vendor/etc/init/        │      │
│  │  - Hardware               │     │  - 厂商签名配置              │      │
│  │                            │     │                             │      │
│  └─────────────────────────────┘     └─────────────────────────────┘      │
│            │                                    │                        │
│            │ 启动                                │ 解析                   │
│            ▼                                    ▼                        │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                     init 进程 (UID=0, Root)                      │    │
│  │                                                                  │    │
│  │  入口: SystemConfig() → ReadConfig() → ParseInitCfg()           │    │
│  │                                                                  │    │
│  │  敏感操作:                                                        │    │
│  │  - execv() 服务执行                                               │    │
│  │  - mount() 文件系统挂载                                           │    │
│  │  - reboot() 系统重启                                              │    │
│  │                                                                  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                    │                                     │
│            │ 启动服务                         │ 配置                    │
│            ▼                                  ▼                         │
│  ┌─────────────────────────────┐     ┌─────────────────────────────┐      │
│  │   中信任区域 (系统服务)      │     │   低信任区域 (应用层)        │      │
│  │                             │     │                             │      │
│  │  - foundation              │     │  - N-API 调用               │      │
│  │  - samgr                    │     │  - 参数读写                 │      │
│  │  - appspawn                 │     │  - 事件监听                │      │
│  │                             │     │                             │      │
│  └─────────────────────────────┘     └─────────────────────────────┘      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.1 边界跨越点

| 跨越方向 | 跨越点 | 验证机制 |
|----------|--------|----------|
| vendor.cfg → init | `ParseInitCfg()` | JSON 解析 |
| init → 服务进程 | `ServiceExec()` | uid/gid/caps 配置 |
| init → 文件系统 | `mount()` | 挂载点验证（TODO） |
| 子进程 → init | `SIGCHLD` | waitpid() 收集状态 |

---

## 4. 攻击向量分析

### 4.1 配置注入攻击

**向量 1：恶意 init.cfg**

```
攻击路径：
1. 攻击者替换 /vendor/etc/init/init.cfg
2. init 解析恶意 JSON 配置
3. 启动恶意服务或执行任意命令
```

**缓解因素**：
- 厂商分区通常只读
- 可能存在签名验证

### 4.2 路径遍历攻击

**向量 2：恶意 mkdir/mount 路径**

```
攻击路径：
1. init.cfg 中配置: mkdir /data/../../etc/malicious
2. init 执行 mkdir 命令
3. 写入恶意配置文件到受保护目录
```

**证据**（TODO: 需确认路径规范化逻辑）

### 4.3 命令注入攻击

**向量 3：exec 命令参数注入**

```
攻击路径：
1. init.cfg 中配置: exec /bin/sh -c "malicious command"
2. init 执行 exec 命令
3. 执行任意 shell 命令
```

**证据**（`services/init/lite/init_cmds.c:31-51`）：
```c
static void DoExec(const struct CmdArgs *ctx)
{
    // 直接使用用户配置的参数执行 execve
    int ret = execve(ctx->argv[0], ctx->argv, NULL);
    // ...
}
```

### 4.4 服务路径劫持

**向量 4：服务路径替换**

```
攻击路径：
1. init.cfg 中配置: path: "/tmp/malicious_service"
2. init 执行 execv("/tmp/malicious_service")
3. 以 root 权限执行恶意程序
```

**证据**（`services/init/lite/init_service.c:97`）：
```c
INIT_ERROR_CHECK(execv(pathArgs->argv[0], pathArgs->argv) == 0,
    return errno, "[startup_failed]failed to execv %s", service->name);
```

### 4.5 拒绝服务攻击

**向量 5：关键进程崩溃触发系统复位**

```
攻击路径：
1. init.cfg 配置 importance=1 的服务
2. 攻击者使该服务崩溃
3. init 执行 RebootSystem() 导致系统重启
```

**证据**（`services/init/lite/init_signal_handler.c:39-42`）：
```c
if (service->attribute & SERVICE_ATTR_IMPORTANT) {
    StopAllServices(0, NULL, 0, NULL);
    RebootSystem();  // 关键进程退出 → 系统复位
}
```

---

## 5. 安全机制现状

### 5.1 已实现的安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| `/bin/sh` 禁止 | `services/init/lite/init_service.c:38` | 禁止直接执行 shell |
| SELinux 支持 | `services/modules/selinux/` | 强制访问控制 |
| Seccomp 支持 | `services/modules/seccomp/` | 系统调用过滤 |
| 进程优先级 | `services/init/lite/init_service.c:60` | 重要进程优先级设置 |
| 信号处理 | `services/init/lite/init_signal_handler.c` | 安全的 SIGCHLD 处理 |

**证据**（`services/init/lite/init_service.c:38-58`）：
```c
int IsForbidden(const char *fieldStr)
{
    size_t forbidStrLen = strlen(BIN_SH_NOT_ALLOWED);
    // 检查是否包含 "/bin/sh"
    if (strncmp(fieldStr, BIN_SH_NOT_ALLOWED, forbidStrLen) == 0) {
        return 1;  // 禁止执行
    }
    return 0;
}
```

### 5.2 缺失的安全机制

| 机制 | 风险 | 建议 |
|------|------|------|
| 路径规范化 | 路径遍历 | 实现 realpath() 验证 |
| 路径白名单 | 服务劫持 | 只允许特定目录下的服务 |
| 配置文件签名 | 配置篡改 | 使用 dm-verity 或签名 |
| 参数长度检查 | 缓冲区溢出 | 添加参数长度限制 |

---

## 6. 风险等级总结

| 风险类型 | 风险等级 | 可利用性 | 影响范围 |
|----------|----------|----------|----------|
| 服务路径劫持 | **高危** | 高 | 完全系统入侵 |
| 命令注入 | **高危** | 高 | 任意代码执行 |
| 配置注入 | **高危** | 中 | 系统被控 |
| 路径遍历 | **中危** | 中 | 文件写入越权 |
| 拒绝服务 | **中危** | 中 | 系统重启 |
| 权限配置错误 | **低危** | 低 | 权限提升 |

---

## 7. 相关文档

- [安全风险评估](06_SecurityReview.md)
- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [代码地图](03_CodeMap.md)
- [构建配置](05_Build.md)
