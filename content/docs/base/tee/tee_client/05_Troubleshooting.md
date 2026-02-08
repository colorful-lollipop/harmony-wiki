# TEE Client 故障排查指南

## 1. 概述

本指南帮助开发者定位和解决 TEE Client 常见问题。

**排查原则**：
1. 从下到上排查（硬件 → 驱动 → 服务 → 应用）
2. 查看日志确认问题边界
3. 使用最小复现步骤验证

## 2. 常见问题与解决方案

### 2.1 CA 无法连接到 TEE

**问题现象**：
```
TEEC_InitializeContext 返回 TEEC_ERROR_GENERIC
```

**排查步骤**：

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 检查 TEE 设备节点是否存在 | `/dev/tee0` 或 `/dev/teepriv0` 存在 |
| 2 | 检查设备权限 | `ls -la /dev/tee*` 显示正确权限 |
| 3 | 检查 teecd 进程状态 | `ps -A | grep teecd` 显示运行中 |
| 4 | 检查 cadaemon 服务状态 | `ps -A | grep cadaemon` 显示运行中 |
| 5 | 查看内核日志 | `dmesg | grep tee` 显示驱动加载成功 |

**诊断命令**：
```bash
# 1. 检查设备节点
ls -la /dev/tee*
# 预期输出：
# crw------- 1 root root 245, 0 2024-01-01 00:00 /dev/tee0
# crw------- 1 root root 245, 1 2024-01-01 00:00 /dev/teepriv0

# 2. 检查进程状态
ps -A | grep -E "cadaemon|teecd"
# 预期输出：
# 1234 ? 00:00:01 cadaemon
# 5678 ? 00:00:02 teecd

# 3. 查看内核日志
dmesg | grep -E "tee|TEE"
# 预期输出包含驱动初始化信息

# 4. 检查 SA 注册
hdc shell sa dump 8001
```

**可能原因与解决方案**：

| 原因 | 解决方案 |
|------|----------|
| TEE 驱动未加载 | 加载 `tee_tzdriver` 内核模块 |
| 设备权限不足 | 检查 udev 规则或手动 chmod |
| teecd 未启动 | 检查启动配置和日志 |
| Socket 连接失败 | 检查 `#tc_ns_socket` 是否被占用 |

---

### 2.2 OpenSession 失败

**问题现象**：
```
TEEC_OpenSession 返回 TEEC_ERROR_TRUSTED_APP_LOAD_ERROR
```

**排查步骤**：

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 确认 TA 文件路径 | `context.ta_path` 设置正确 |
| 2 | 检查 TA 文件是否存在 | `ls -la /data/*.sec` 或 `/vendor/bin/*.sec` |
| 3 | 检查 TA 文件权限 | TA 文件对运行用户可读 |
| 4 | 查看 cadaemon 日志 | 确认加载过程中的具体错误 |
| 5 | 确认 UUID 格式正确 | 十六进制格式，无多余字符 |

**诊断命令**：
```bash
# 1. 检查 TA 文件
ls -la /data/*.sec /system/bin/*.sec /vendor/bin/*.sec

# 2. 检查 TA 文件格式（十六进制）
xxd /data/58dbb3b9-4a0c-42d2-a84d-7c7ab17539fc.sec | head -20
# 预期： magic number 0xA5A55A5A

# 3. 查看详细日志
hilog | grep -i "tee"
hilog | grep -i "ta"
```

**TA 文件格式验证**：
```c
// TA 文件头结构
struct TaImageHdr {
    uint32_t magicNum1;      // 0xA5A55A5A
    uint32_t magicNum2;      // 0x55AA (LE) / 0xAAAA (BE)
    uint32_t versionNum;     // 版本号
    uint32_t contextLen;     // 上下文长度
    uint32_t taKeyVersion;   // TA 密钥版本
};
```

**可能原因与解决方案**：

| 原因 | 解决方案 |
|------|----------|
| TA 文件不存在 | 确认 TA 已安装到正确路径 |
| TA 文件损坏 | 重新安装 TA |
| TA 路径错误 | 检查 `context.ta_path` 设置 |
| UUID 不匹配 | 确认 TA 的 UUID 与代码一致 |
| TA 签名无效 | 检查 TA 签名证书 |

---

### 2.3 InvokeCommand 超时

**问题现象**：
```
TEEC_InvokeCommand 返回 TEEC_ERROR_TARGET_DEAD
或长时间无响应
```

**排查步骤**：

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 检查 TA 是否崩溃 | 查看 TEE 日志 |
| 2 | 检查命令 ID 是否正确 | 确认 TA 支持该命令 |
| 3 | 检查参数是否正确 | 验证参数类型和大小 |
| 4 | 检查共享内存大小 | 确保缓冲区足够 |
| 5 | 查看系统资源 | 检查内存、CPU 占用 |

**诊断命令**：
```bash
# 1. 检查系统资源
free -h
top -b -n 1 | head -20

# 2. 检查 TEE 驱动状态
cat /sys/kernel/tee/*/state 2>/dev/null

# 3. 查看详细日志
hilog | grep -i "invoke\|command\|dead"
```

**TA 调试建议**：
```c
// 在 TA 中添加调试日志
TEE_Result TA_InvokeCommandEntryPoint(
    uint32_t nParamType,
    uint32_t nParamValue,
    TEE_Param pstParam,
    void *pContext)
{
    TEE_Result ret;
    
    // 添加入口日志
    EMSG("Enter TA, cmd=%u", (unsigned int)nParamValue);
    
    ret = DoCommand(nParamType, nParamValue, pstParam);
    
    // 添加出口日志
    EMSG("Exit TA, ret=0x%08x", ret);
    
    return ret;
}
```

---

### 2.4 共享内存问题

**问题现象**：
```
TEEC_AllocateSharedMemory 返回 TEEC_ERROR_OUT_OF_MEMORY
或数据传输不正确
```

**排查步骤**：

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 检查共享内存大小 | 不超过 `MAX_SHAREDMEM_LEN` (0x10000000) |
| 2 | 检查 Ashmem 状态 | 确认共享内存正确映射 |
| 3 | 检查内存泄漏 | 确认正确调用 `TEEC_ReleaseSharedMemory` |
| 4 | 检查参数类型 | 确认使用正确的 `TEEC_MEMREF_*` 类型 |

**代码验证**：
```c
// 正确使用共享内存
TEEC_SharedMemory shm = {0};

// 设置参数
shm.buffer = malloc(BUFFER_SIZE);
shm.size = BUFFER_SIZE;
shm.flags = TEEC_MEM_INPUT | TEEC_MEM_OUTPUT;

// 注册共享内存
TEEC_Result ret = TEEC_RegisterSharedMemory(&context, &shm);
if (ret != TEEC_SUCCESS) {
    printf("RegisterSharedMemory failed: 0x%x\n", ret);
    free(shm.buffer);
    return ret;
}

// ... 使用共享内存 ...

// 释放共享内存
TEEC_ReleaseSharedMemory(&shm);
free(shm.buffer);
```

**可能原因与解决方案**：

| 原因 | 解决方案 |
|------|----------|
| 大小超过限制 | 检查 `MAX_SHAREDMEM_LEN` (256MB) |
| 重复释放 | 确保 `TEEC_ReleaseSharedMemory` 只调用一次 |
| 未注册就使用 | 先注册再使用 |
| 缓冲区越界 | 确保操作在 `size` 范围内 |

---

### 2.5 权限问题

**问题现象**：
```
TEEC_ERROR_ACCESS_DENIED
```

**排查步骤**：

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 检查 CA 的用户 ID | 确认在允许的 UID 范围内 |
| 2 | 检查签名证书 | 确认 CA 签名有效 |
| 3 | 检查权限声明 | 确认 `syscap` 正确声明 |
| 4 | 查看访问日志 | 确认被哪个模块拒绝 |

**权限检查点**：

| 模块 | 检查项 | 配置文件 |
|------|--------|----------|
| CA | 签名 | bundle.json |
| CA | UID | system |
| CA | Token | IPC Token |

---

### 2.6 日志服务问题

**问题现象**：
```
tlogcat 无输出
或 TEE 日志无法读取
```

**排查步骤**：

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 检查 tlogcat 进程 | `ps -A | grep tlogcat` |
| 2 | 检查日志路径 | `/data/log/` 目录存在 |
| 3 | 检查日志权限 | 对运行用户可读 |
| 4 | 查看 TEE 驱动日志 | `cat /sys/kernel/tee/*/log` |

**诊断命令**：
```bash
# 1. 检查日志目录
ls -la /data/log/

# 2. 运行 tlogcat
tlogcat -h
tlogcat -v

# 3. 检查 TEE 日志
cat /sys/kernel/tee/tzdriver/log 2>/dev/null

# 4. 查看内核日志
dmesg | grep -i "tee\|log"
```

---

## 3. 日志查看

### 3.1 TEE 相关日志标签

| 标签 | 说明 |
|------|------|
| `TEE` | TEE Client 主日志 |
| `teecd` | TEE 代理服务日志 |
| `cadaemon` | CA 守护进程日志 |
| `tee_client` | libteec 日志 |
| `tee_vendor` | libteec_vendor 日志 |

### 3.2 日志级别

| 级别 | 说明 | 使用场景 |
|------|------|----------|
| `ERROR` | 错误 | 失败和异常 |
| `WARN` | 警告 | 可恢复问题 |
| `INFO` | 信息 | 主要流程 |
| `DEBUG` | 调试 | 开发调试 |

### 3.3 日志过滤命令

```bash
# 查看所有 TEE 相关日志
hilog | grep -E "TEE|tee|TEEC"

# 查看 cadaemon 详细日志
hilog | grep "cadaemon"

# 查看错误级别日志
hilog | grep -E "Error|ERROR|error"

# 实时查看 TEE 日志
hilog -T | grep -E "TEE|tee"
```

---

## 4. 调试工具

### 4.1 系统能力查询

```bash
# 查询 SA 8001 状态
hdc shell sa dump 8001

# 查询系统能力
hdc shell dumpsys capability

# 查看进程间通信状态
hdc shell ipc debug
```

### 4.2 设备节点调试

```bash
# 查看 TEE 设备详情
ls -la /dev/tee*

# 读取设备属性
cat /sys/class/tee/tee0/uevent

# 查看驱动状态
cat /sys/kernel/tee/*/info 2>/dev/null
```

### 4.3 内存和进程调试

```bash
# 查看进程内存使用
cat /proc/<cadaemon_pid>/status
cat /proc/<teecd_pid>/status

# 查看文件描述符
ls -la /proc/<pid>/fd/

# 查看调用栈（如果支持）
cat /proc/<pid>/stack
```

---

## 5. 常见错误码速查

### 5.1 错误码汇总表

| 错误码 | 值 | 说明 | 排查方向 |
|--------|------|------|----------|
| `TEEC_SUCCESS` | 0x0 | 成功 | - |
| `TEEC_ERROR_GENERIC` | 0xFFFF0000 | 通用错误 | 查看详细日志 |
| `TEEC_ERROR_ACCESS_DENIED` | 0xFFFF0001 | 访问被拒绝 | 检查权限 |
| `TEEC_ERROR_BAD_PARAMETERS` | 0xFFFF0006 | 参数错误 | 检查 API 参数 |
| `TEEC_ERROR_OUT_OF_MEMORY` | 0xFFFF000C | 内存不足 | 检查内存使用 |
| `TEEC_ERROR_COMMUNICATION` | 0xFFFF000E | 通信错误 | 检查 IPC/Socket |
| `TEEC_ERROR_TRUSTED_APP_LOAD_ERROR` | - | TA 加载失败 | 检查 TA 文件 |
| `TEEC_ERROR_TARGET_DEAD` | 0xFFFF3024 | TA 崩溃 | 检查 TA 状态 |

### 5.2 错误来源 (returnOrigin)

| 值 | 说明 | 来源 |
|------|------|------|
| `TEEC_ORIGIN_API` | 0x1 | TEE Client API 本身 |
| `TEEC_ORIGIN_COMMS` | 0x2 | REE/TEE 通信层 |
| `TEEC_ORIGIN_TEE` | 0x3 | TEE 内核 |
| `TEEC_ORIGIN_TRUSTED_APP` | 0x4 | TA 内部 |

---

## 6. 调试最佳实践

### 6.1 调试流程

```
问题报告
    │
    ▼
复现问题
    │
    ▼
收集日志（hilog, dmesg）
    │
    ▼
定位错误模块
    │
    ▼
检查配置
    │
    ▼
检查权限
    │
    ▼
检查资源（内存、FD）
    │
    ▼
检查代码逻辑
```

### 6.2 最小复现步骤模板

```markdown
## 问题描述
[简要描述问题]

## 环境信息
- 设备型号：[设备]
- 系统版本：[版本]
- TEE 版本：[版本]

## 复现步骤
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 预期结果
[描述期望的行为]

## 实际结果
[描述实际的行为]

## 日志信息
```
[粘贴相关日志]
```

## 代码片段
```c
[相关代码]
```
```

### 6.3 性能问题排查

```bash
# 检查 CPU 使用率
top -b -n 1 | grep -E "cadaemon|teecd"

# 检查内存使用
cat /proc/<pid>/status | grep -E "VmRSS|VmSize"

# 检查调用延迟
time TEEC_InitializeContext(NULL, &context)

# 检查系统负载
cat /proc/loadavg
```

---

## 7. 联系方式

### 7.1 相关仓库

- [tee_tzdriver](https://gitee.com/openharmony-sig/tee_tee_tzdriver) - TEE 驱动
- [tee_os](https://gitee.com/openharmony-sig/tee_tee_os) - TEE OS

### 7.2 日志上报

如需上报问题，请包含以下信息：

1. `hilog` 完整日志
2. `dmesg | grep -i tee` 输出
3. `cat /proc/<pid>/status`（相关进程）
4. 复现步骤和环境信息
