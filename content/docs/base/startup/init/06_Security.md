# 安全风险评审

## 威胁模型概述

```
┌─────────────────────────────────────────────────────────────────┐
│                        外部输入源                                 │
├─────────────────────────────────────────────────────────────────┤
│  init.cfg (JSON)        │  配置文件解析                          │
│  uevent (内核事件)       │  设备节点事件                          │
│  参数服务 (IPC)          │  跨进程通信                            │
│  shell 命令             │  begetctl 命令                        │
└─────────────────────────┬─────────────────────────────────────┘
                          │
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界                                     │
├─────────────────────────────────────────────────────────────────┤
│  • init 进程 (UID 0, root)                                       │
│  • 已签名系统服务                                                  │
│  • 配置文件 (/etc/init.cfg)                                       │
└─────────────────────────┬─────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
┌─────────────────┐ ┌──────────────┐ ┌─────────────────┐
│  敏感操作        │ │  系统调用     │ │  权限配置        │
│  - 进程启动      │ │  - mount     │ │  - uid/gid      │
│  - 权限变更      │ │  - chown     │ │  - capabilities │
│  - 系统重置      │ │  - reboot    │ │  - selinux      │
└─────────────────┘ └──────────────┘ └─────────────────┘
```

---

## 攻击面清单

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| **init.cfg 解析** | JSON 配置解析漏洞 | 高 |
| **服务启动** | 进程参数注入 | 高 |
| **uevent 处理** | 设备节点事件 | 中 |
| **参数服务 IPC** | 跨进程通信 | 中 |
| **begetctl 命令** | Shell 注入 | 中 |
| **文件操作** | 路径遍历 | 低 |

---

## 可利用点 (风险清单)

### 1. init.cfg JSON 解析溢出

**证据**: `services/init/init_config.c` 使用 `cJSON_Parse()` 解析配置

**触发条件**:
```
恶意构造的 init.cfg 文件，包含：
- 异常深的 JSON 嵌套
- 超长字符串 (>100KB)
- 特殊格式字符
```

**影响**:
- 拒绝服务 (DoS)
- 内存损坏
- 可能的代码执行

**修复建议**:
- 添加 JSON 解析深度限制
- 字符串长度校验
- 启用 ASan/UBSan 测试

---

### 2. 服务路径注入

**证据**: `services/init/init_service_manager.c` 中的 `LoadService()` 函数

**触发条件**:
```
init.cfg services 数组中：
"path": "/bin/../../../bin/sh"
或
"path": "/system/bin/ls; rm -rf /"
```

**影响**:
- 任意命令执行
- 权限提升

**修复建议**:
```c
// 路径白名单检查
if (!IsPathInWhitelist(servicePath)) {
    return -1;
}
// 路径规范化
char resolved[MAX_PATH];
realpath(path, resolved);
// 只允许 system/bin 目录
```

---

### 3. uid/gid 配置错误

**证据**: `services/init/init_service_manager.c` 中的 `DecodeUid()` / `DecodeGid()`

**触发条件**:
```
init.cfg 中配置：
"uid": 0,  // root 权限
"gid": 0
```

**影响**:
- 服务以 root 权限运行
- 提权攻击

**修复建议**:
- 限制可配置的 UID/GID 范围
- 非特权服务强制使用非零 UID
- 敏感服务使用 Capability 替代

---

### 4. Capabilities 过度授权

**证据**: `services/init/init_capability.c` 中的 `InitServiceCaps()`

**触发条件**:
```
init.cfg services.caps 配置：
"caps": [0, 1, 2, 3, 4, 5, 6, 7]  // 全部 Capability
```

**影响**:
- CAP_SETUID: 任意用户提权
- CAP_SYS_ADMIN: 系统管理权限
- CAP_NET_ADMIN: 网络配置权限

**修复建议**:
- 遵循最小权限原则
- 敏感 Capability 需要签名验证
- 记录 Capability 使用审计

---

### 5. SELinux 上下文逃逸

**证据**: `services/modules/selinux/selinux_adp.c`

**触发条件**:
```
配置文件 secon 字段缺失或不正确：
"secon": ""  // 未设置上下文
```

**影响**:
- SELinux 策略绕过
- 进程权限提升

**修复建议**:
```c
// 强制检查 secon 配置
if (cfg.secon && strlen(cfg.secon) > 0) {
    setexeccon(cfg.secon);
} else {
    // 使用默认安全上下文或拒绝启动
    return -1;
}
```

---

### 6. Seccomp 策略绕过

**证据**: `services/modules/seccomp/seccomp_policy.c`

**触发条件**:
```
seccomp 策略配置错误：
- 允许危险系统调用
- 策略文件损坏
- 策略未正确加载
```

**影响**:
- 任意系统调用
- 内核漏洞利用

**修复建议**:
```c
// 策略加载前验证
int ValidatePolicy(const char *policy) {
    const char *dangerous[] = {
        "ptrace", "kexec", "reboot", NULL
    };
    for (int i = 0; dangerous[i]; i++) {
        if (strstr(policy, dangerous[i])) {
            return -1;  // 拒绝危险调用
        }
    }
    return 0;
}
```

---

### 7. ueventd 设备节点权限配置错误

**证据**: `ueventd/ueventd_read_cfg.c` / `ueventd/etc/ueventd.config`

**触发条件**:
```
ueventd.config 中：
/dev/binder    root   root   0666    # 任意用户可访问
/dev/sda       root   root   0666    # 任意用户可读写
```

**影响**:
- 任意进程绑定 Binder
- 磁盘数据泄露

**修复建议**:
```bash
# 最小权限原则
/dev/binder    root   system   0660
/dev/sda       root   system   0640
```

---

## 安全机制说明

### 已实现的安全机制

| 机制 | 文件 | 说明 |
|------|------|------|
| **DAC** | `init_service_manager.c` | UID/GID 访问控制 |
| **Capabilities** | `init_capability.c` | 细粒度权限 |
| **SELinux** | `selinux_adp.c` | 强制访问控制 |
| **Seccomp** | `seccomp_policy.c` | 系统调用过滤 |
| **签名验证** | `bundle_framework` | 服务签名 |

### 建议增强

1. **配置签名**: init.cfg 文件签名验证
2. **运行时完整性**: 服务进程完整性检测
3. **审计日志**: 所有敏感操作记录
4. **ASLR/PIE**: 启用地址空间随机化

---

## 敏感系统调用列表

### 高危系统调用 (需谨慎授权)

| 系统调用 | 危险等级 | 说明 |
|----------|----------|------|
| `ptrace` | 极高 | 进程调试，可用于注入代码 |
| `kexec` | 极高 | 内核执行，可加载新内核 |
| `reboot` | 极高 | 系统重启 |
| `mount` | 高 | 文件系统挂载 |
| `umount` | 高 | 文件系统卸载 |
| `chown` | 高 | 改变文件所有者 |
| `chmod` | 高 | 修改文件权限 |
| `setuid` | 高 | 设置用户 ID |
| `setgid` | 高 | 设置组 ID |
| `kill` | 中 | 发送信号 |
| `mprotect` | 中 | 修改内存保护 |

### Seccomp 策略配置示例

```json
// system.seccomp.policy
{
    "default": "kill",
    "syscalls": [
        {
            "names": ["read", "write", "open", "close"],
            "action": "allow"
        },
        {
            "names": ["ptrace", "kexec", "reboot"],
            "action": "kill"
        }
    ]
}
```

---

## 权限配置最佳实践

### 最小权限原则

```json
// Good: 最小权限
{
    "name": "my_service",
    "uid": 1000,
    "gid": 1000,
    "caps": [1, 2]  // 只授予必要的能力
}

// Bad: 过度授权
{
    "name": "my_service",
    "uid": 0,
    "gid": 0,
    "caps": [0, 1, 2, 3, 4, 5, 6, 7]  // 全部能力
}
```

### UID/GID 配置建议

| 服务类型 | 建议 UID | 建议 GID | 说明 |
|----------|----------|----------|------|
| 系统服务 | 0 | 0 | 系统组件 |
| 特权服务 | < 1000 | < 1000 | 系统用户 |
| 普通服务 | >= 1000 | >= 1000 | 应用用户 |
| 访客服务 | 9999 | 9999 | 临时用户 |

---

## SELinux 上下文配置

### 常用安全上下文

| 上下文格式 | 说明 | 示例 |
|-----------|------|------|
| `u:r:类型:s0` | 标准格式 | `u:r:init:s0` |

### 常见上下文

| 服务 | 安全上下文 | 说明 |
|------|-----------|------|
| init | `u:r:init:s0` | init 进程 |
| ueventd | `u:r:ueventd:s0` | uevent 守护 |
| watchdog | `u:r:watchdog:s0` | 看门狗 |
| console | `u:r:console:s0` | 控制台 |
| shell | `u:r:shell:s0` | Shell |

---

## 相关跳转

- [架构](02_Architecture.md)
- [构建](05_Build.md)
- [问题定位](07_Troubleshooting.md)
