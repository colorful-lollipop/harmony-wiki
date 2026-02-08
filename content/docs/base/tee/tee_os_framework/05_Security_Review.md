# 安全风险评审

## 1. 威胁模型

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界层次                                     │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ 边界 0: REE ↔ TEE                                                    │  │
│  │   - 入口: SMC 指令                                                   │  │
│  │   - 防护: ATF/EL3 安全监控器                                         │  │
│  │   - 数据: 共享内存 (由内核控制)                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ 边界 1: gtask ↔ Permission Service                                   │  │
│  │   - 入口: IPC 消息                                                   │  │
│  │   - 标识: GLOBAL_HANDLE (gtask 专用)                                 │  │
│  │   - 校验: 只有 gtask 可请求 ELF 验证                                 │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ 边界 2: gtask ↔ TA                                                  │  │
│  │   - 入口: IPC 命令                                                   │  │
│  │   - 隔离: 每个 TA 独立进程                                           │  │
│  │   - 机制: 会话上下文隔离                                             │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ 边界 3: TA ↔ Driver                                                 │  │
│  │   - 入口: drvmgr IPC                                                │  │
│  │   - 防护: MAC 访问控制列表                                           │  │
│  │   - 校验: 权限检查                                                   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ 边界 4: TA ↔ SSA (安全存储)                                          │  │
│  │   - 入口: Agent IPC                                                  │  │
│  │   - 隔离: TA Root Key 隔离                                           │  │
│  │   - 加密: AES-256-XTS + HMAC-SHA256                                 │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 外部输入点

| 输入点 | 来源 | 数据类型 | 校验机制 |
|--------|------|----------|----------|
| SMC 调用 | REE CA | smc_cmd_t | 消息队列边界检查 |
| ELF 文件 | REE | TA 二进制 | SEC 签名验签 |
| 配置文件 | REE | 属性列表 | RSA/ECC 签名验证 |
| 存储文件 | REE | 用户数据 | AES-XTS 加密 |

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 组件 | 接口 | 风险等级 | 描述 |
|------|------|----------|------|
| **SMC 接口** | smc_cmd_t | 高 | REE → TEE 唯一入口 |
| **ELF 加载** | tarunner | 高 | 未验证二进制执行 |
| **权限服务** | perm_srv_elf_verify | 高 | 签名验证绕过 |
| **安全存储** | ssa_fs.c | 中 | 路径遍历 |
| **驱动接口** | drvmgr | 中 | 权限检查绕过 |
| **IPC 消息** | ipc_msg_* | 中 | 消息注入 |
| **密钥派生** | huk_derive_takey | 高 | 密钥泄露 |
| **随机数** | crypto_syscall_random | 中 | 熵不足 |

---

## 3. 安全风险点

### 3.1 高风险项

#### 风险 1: SMC 命令队列溢出

**证据**：`framework/gtask/src/include/gtask_inner.h:60-73`
```c
typedef struct {
    DECLEAR_BITMAP(in_bitmap, MAX_SMC_CMD);  // MAX_SMC_CMD = 18
    smc_cmd_t in[MAX_SMC_CMD];               // 固定大小队列
    // ...
} nwd_cmd_t;
```

**触发条件**：
- 攻击者发送超过 `MAX_SMC_CMD` (18) 个命令
- 队列满时新命令覆盖旧命令

**影响**：
- 命令丢失或混淆
- 可能导致状态不一致

**修复建议**：
- 增加队列大小或使用循环队列
- 添加队列满时的拒绝机制

---

#### 风险 2: ELF 签名验证绕过

**证据**：`services/permission_service/src/perm_srv_elf_verify/perm_srv_elf_verify_cmd.c:78-80`
```c
// 只有 gtask 可以请求 ELF 验证
if (sender_taskid != GLOBAL_HANDLE) {
    return TEE_ERROR_ACCESS_DENIED;
}
```

**触发条件**：
- `GLOBAL_HANDLE` 标识被伪造
- IPC 消息发送者验证不严格

**影响**：
- 加载未签名恶意 TA
- 提权到 TEE 环境

**修复建议**：
- 强化 `GLOBAL_HANDLE` 验证机制
- 使用硬件绑定的任务标识

---

#### 风险 3: 路径遍历攻击

**证据**：`services/ssa/src/secure_storage_agent/ssa_fs.c:141-160`
```c
TEE_Result check_file_name(const char *name)
{
    // 使用黑名单方式
    if (strstr(name, "../")) {
        tloge("Invalid file name: %s\n", name);
        return TEE_ERROR_BAD_PARAMETERS;
    }
}
```

**触发条件**：
- 文件名包含 `../` 序列
- 使用黑名单而非白名单

**影响**：
- 访问其他 TA 的存储文件
- 数据泄露

**修复建议**：
- 使用白名单验证文件名
- 对路径进行规范化处理后再验证

---

#### 风险 4: 密钥派生盐值固定

**证据**：`services/ssa/src/secure_storage_agent/sfs_internal.h`
```c
#define FILEKEY_SALT "0 file key salt."
#define MASTER_HMAC_SALT "master hmacsalt."
#define ENCRYPTION1_SALT "1 aes xti1 salt."
#define ENCRYPTION2_SALT "2 AES XTI2 Salt."
#define FILE_NAME_SALT "3 FileName salt."
```

**触发条件**：
- 盐值硬编码在代码中
- 攻击者可以预测派生结果

**影响**：
- 密钥派生可预测
- 降低密钥强度

**修复建议**：
- 从 HUK 派生盐值
- 使用设备唯一信息作为盐

---

### 3.2 中风险项

#### 风险 5: 会话数量限制绕过

**证据**：`framework/gtask/src/include/gtask_core.h`
```c
#define TA_SESSION_MAX 8
struct service_struct {
    uint32_t session_bitmap[TA_SESSION_MAX / UINT32_BIT_NUM + 1];
    // ...
};
```

**触发条件**：
- 恶意应用打开超过 8 个会话

**影响**：
- 资源耗尽
- 拒绝服务

**修复建议**：
- 添加会话创建速率限制
- 强制会话超时机制

---

#### 风险 6: TA2TA 嵌套深度限制

**证据**：`framework/gtask/src/include/gtask_core.h`
```c
#define MAX_TA2TA_LEVEL 1  // 限制为 1 层
```

**触发条件**：
- TA2TA 递归调用

**影响**：
- 栈溢出
- 拒绝服务

**修复建议**：
- 保持当前限制
- 添加栈大小检查

---

#### 风险 7: 随机数缓存可预测

**证据**：`drivers/crypto_mgr/src/crypto_ioctl/crypto_syscall_random.c`
```c
static uint8_t g_cached_random[CACHED_RANDOM_SIZE];  // 4KB 缓存
static uint32_t g_used_block_count = TOTAL_RANDOM_BLOCK;
```

**触发条件**：
- 缓存被部分读取
- 硬件 RNG 熵不足

**影响**：
- 随机数可预测
- 加密强度降低

**修复建议**：
- 使用后立即清除缓存
- 确保硬件 RNG 正确初始化

---

#### 风险 8: 内存安全

**证据**：全代码库搜索 `memcpy_s` 替代 `memcpy`，`memset_s` 替代 `memset`

**当前措施**：
- 使用 `memcpy_s`、`memmove_s`、`strncpy_s`
- 使用 `memset_s` 清除敏感数据

**潜在问题**：
- 编译器优化可能移除 `memset_s`
- 手动内存管理可能导致泄漏

**修复建议**：
- 使用 `volatile` 指针防止优化
- 添加内存分配审计

---

### 3.3 低风险项

#### 风险 9: 日志信息泄露

**当前情况**：代码中使用 `tloge`、`tlogd` 输出调试信息

**潜在问题**：
- 敏感信息写入日志
- 调试接口未关闭

**建议**：
- 生产版本禁用调试日志
- 日志内容脱敏

---

#### 风险 10: 时间侧信道

**当前情况**：TEE 时间 API 可能泄露操作耗时

**潜在问题**：
- 基于时间的攻击（如时序攻击）

**建议**：
- 对敏感操作添加随机延迟
- 使用恒定时间比较函数

---

## 4. 安全机制分析

### 4.1 已实现的安全机制

| 机制 | 实现位置 | 效果 |
|------|----------|------|
| **SEC 签名验签** | `perm_srv_elf_verify_cmd.c` | 确保 TA 未被篡改 |
| **X.509 证书链** | `perm_srv_ta_cert.c` | 验证 TA 签名者身份 |
| **CRL 检查** | `perm_srv_ta_crl.c` | 吊销证书检查 |
| **TA 去激活** | `perm_srv_ta_ctrl.c` | 禁用恶意 TA |
| **防回滚** | `anti_version_rollback()` | 防止版本降级 |
| **AES-256-XTS** | `ssa_crypto.c` | 数据机密性 |
| **HMAC-SHA256** | `sfs.c` | 数据完整性 |
| **TA 隔离** | tarunner | 进程级隔离 |
| **MAC 访问控制** | `drv_auth.c` | 驱动权限控制 |

### 4.2 安全特性总结

```
┌─────────────────────────────────────────────────────────────────┐
│                      安全特性矩阵                                 │
├─────────────────┬─────────┬─────────┬─────────┬─────────┬─────┤
│     机制        │ TA 加载  │ 存储    │ 通信    │ 密钥    │ API │
├─────────────────┼─────────┼─────────┼─────────┼─────────┼─────┤
│ 签名验证        │    ✓    │    -    │    -    │    -    │  -  │
│ 证书链          │    ✓    │    -    │    -    │    -    │  -  │
│ 加密            │    -    │    ✓    │    -    │    -    │  -  │
│ 完整性          │    -    │    ✓    │    -    │    -    │  -  │
│ 进程隔离        │    ✓    │    ✓    │    ✓    │    ✓    │  -  │
│ 访问控制        │    -    │    ✓    │    ✓    │    -    │  -  │
│ 密钥派生        │    -    │    ✓    │    -    │    ✓    │  -  │
│ 随机数          │    -    │    -    │    -    │    ✓    │  -  │
└─────────────────┴─────────┴─────────┴─────────┴─────────┴─────┘
```

---

## 5. 内存安全实践

### 5.1 安全函数使用

| 场景 | 安全函数 | 证据位置 |
|------|----------|----------|
| 字符串复制 | `strncpy_s` | 全代码库 |
| 内存复制 | `memcpy_s` | 全代码库 |
| 内存移动 | `memmove_s` | 全代码库 |
| 内存清零 | `memset_s` | `huk_derive_takey.c`, `sfs_internal.c` |
| 字符串比较 | `strcmp_s` | 全代码库 |

### 5.2 输入验证

```c
// 标准模式
if (src == NULL || dest == NULL || size > MAX_SIZE)
    return TEE_ERROR_BAD_PARAMETERS;
```

---

## 6. 安全审计建议

### 6.1 定期审计项

1. **证书 CRL 更新**：确保吊销列表及时更新
2. **密钥派生流程**：审计密钥生成和使用
3. **会话管理**：检查会话泄漏和资源耗尽
4. **日志审计**：检查敏感信息泄露

### 6.2 安全测试建议

1. **模糊测试**：对 SMC 接口和消息处理进行模糊测试
2. **边界测试**：测试各种边界条件和异常输入
3. **渗透测试**：模拟 REE 攻击者尝试绕过安全机制
4. **代码审计**：定期审计新增代码的安全函数使用

---

## 7. 安全相关文件索引

| 功能 | 关键文件 |
|------|----------|
| **权限服务** | |
| ELF 验证 | `services/permission_service/src/perm_srv_elf_verify/perm_srv_elf_verify_cmd.c` |
| 证书验证 | `services/permission_service/src/perm_srv_ta_cert/perm_srv_ta_cert.c` |
| CRL 管理 | `services/permission_service/src/perm_srv_ta_crl/perm_srv_ta_crl.c` |
| TA 控制 | `services/permission_service/src/perm_srv_ta_ctrl/perm_srv_ta_ctrl.c` |
| **安全存储** | |
| 加解密 | `services/ssa/src/secure_storage_agent/ssa_crypto.c` |
| 文件系统 | `services/ssa/src/secure_storage_agent/ssa_fs.c` |
| 密钥派生 | `services/ssa/src/secure_storage_agent/sfs_internal.c` |
| **HUK 服务** | |
| 密钥派生 | `services/huk_service/src/huk_derive_takey.c` |
| **加密驱动** | |
| 随机数 | `drivers/crypto_mgr/src/crypto_ioctl/crypto_syscall_random.c` |
| IOCTL | `drivers/crypto_mgr/src/crypto_ioctl/crypto_mgr_syscall.h` |

---

## 相关文档

- 架构设计 → `01_Architecture.md`
- 模块详情 → `02_Module_Detail.md`
- 构建系统 → `04_Build_System.md`
