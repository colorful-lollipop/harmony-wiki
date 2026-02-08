# 问题定位

## 常见问题

### 1. 服务启动失败

**现象**: 服务进程启动后立即退出

**排查步骤**:

```bash
# 1. 查看 init 日志
hilog | grep -E "init|service"

# 2. 查看 dmesg
dmesg | grep -E "init|SIGCHLD"

# 3. 检查服务配置
cat /etc/init.cfg | jq '.services[] | select(.name=="xxx")'

# 4. 手动测试服务
/system/bin/xxx &
```

**常见原因**:

| 原因 | 解决方案 |
|------|----------|
| 路径错误 | 检查 path 配置 |
| 权限不足 | 检查 uid/gid/caps |
| 依赖服务未启动 | 检查依赖顺序 |
| 配置文件错误 | 验证 JSON 格式 |

---

### 2. init.cfg 解析错误

**现象**: init 进程启动失败或部分命令不执行

**排查步骤**:

```bash
# 1. 验证 JSON 格式
cat /etc/init.cfg | python3 -m json.tool

# 2. 检查文件大小
ls -la /etc/init.cfg
# 限制: ≤ 100KB

# 3. 检查权限
ls -la /etc/init.cfg
```

**JSON 格式要求**:

```json
{
    "jobs": [
        {
            "name": "pre-init",
            "cmds": ["mkdir /test", "chmod 0755 /test"]
        }
    ],
    "services": [
        {
            "name": "service_name",
            "path": "/system/bin/service",
            "uid": 0,
            "gid": 0,
            "once": 0,
            "importance": 0,
            "caps": []
        }
    ]
}
```

---

### 3. 进程重启循环

**现象**: 服务在 4 分钟内重启 5 次后不再重启

**排查步骤**:

```bash
# 1. 查看重启日志
hilog | grep -E "restart|exit|SIGCHLD"

# 2. 检查服务崩溃原因
cat /proc/<pid>/statm
dmesg | tail

# 3. 检查 importance 配置
# importance=1 的服务退出将触发系统重置
```

**重启策略**:

| 配置 | 行为 |
|------|------|
| `once: 0` | 退出后自动重启 |
| `once: 1` | 仅启动一次 |
| `importance: 1` | 关键服务，退出触发系统重置 |
| 5 次/4 分钟 | 停止重启 (防重启循环) |

---

### 4. 参数服务异常

**现象**: `systemparameter.get/set` 返回错误

**排查步骤**:

```bash
# 1. 检查 param 服务状态
hilog | grep -E "param|Parameter"

# 2. 测试参数读写
setprop const.product.type test
getprop const.product.type

# 3. 检查参数守护进程
ps -A | grep param
```

---

### 5. ueventd 问题

**现象**: 设备节点权限不正确

**排查步骤**:

```bash
# 1. 检查 ueventd 配置
cat /ueventd.cfg

# 2. 查看 ueventd 日志
hilog | grep ueventd

# 3. 检查设备节点
ls -la /dev/binder
ls -la /dev/hwbinder
```

---

## 日志查看

### 关键日志标签

| 标签 | 模块 | 说明 |
|------|------|------|
| `INIT` | init 主进程 | 启动日志 |
| `BEGETCTL` | 控制工具 | 命令日志 |
| `UEVENTD` | uevent 守护 | 设备事件 |
| `DEVICEINFO` | 设备信息 | 设备信息服务 |
| `PARAM` | 参数服务 | 参数操作 |

### 日志级别

| 级别 | 说明 |
|------|------|
| `LOGV` | 详细 (Verbose) |
| `LOGI` | 信息 (Info) |
| `LOGW` | 警告 (Warning) |
| `LOGE` | 错误 (Error) |

---

## 调试工具

### begetctl 命令

```bash
# 查看服务状态
begetctl services

# 查看进程信息
begetctl pid <service_name>

# 查看系统参数
begetctl getparam <key>

# 查看启动阶段
begetctl bootstatus
```

### hilog 过滤

```bash
# 过滤 init 相关日志
hilog | grep -E "INIT|BEGETCTL|service"

# 实时监控
hilog -T | grep init
```

---

## 配置文件位置

| 配置文件 | 路径 | 说明 |
|----------|------|------|
| init.cfg | `/etc/init.cfg` | 主配置 |
| ueventd.cfg | `/etc/ueventd.cfg` | 设备权限 |
| seccomp | `/system/etc/seccomp/` | 系统调用策略 |
| selinux | `/system/etc/selinux/` | SELinux 策略 |

---

## 相关跳转

- [概览](01_Overview.md)
- [安全机制](06_Security.md)
- [N-API](03_NAPI.md)
