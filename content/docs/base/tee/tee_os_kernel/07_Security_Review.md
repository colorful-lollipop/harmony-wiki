# 安全评审

> **文档版本**: 1.0
> **生成时间**: 2026-02-06
> **适用范围**: OpenHarmony TEE OS Kernel 安全机制

---

## 文档目的

本文档基于代码证据进行安全风险评审，包括攻击面、信任边界、安全机制和可被利用点分析。

---

## 目录

- [威胁模型](#威胁模型)
- [安全机制概述](#安全机制概述)
- [攻击面分析](#攻击面分析)
- [信任边界](#信任边界)
- [可被利用点](#可被利用点)
- [安全建议](#安全建议)

---

## 威胁模型

### 1.1 安全架构层次

```
┌──────────────────────────────────────────┐
│        应用层（Applications）         │
│  - TEE Applications (TA)          │
│  - Normal World Apps              │
│                                     │
│         ↕ API 调用               │
│                                     │
└──────────────────────────────────────────┘
            ↕ 能力系统（Capability System）
            ↕ 系统调用接口（256 个）
            ↕ IPC 通信（Connection + Channel）
            ↕ TrustZone SMC
            ↕ 地址空间隔离（VMSpace + PMO）
            ↕ Badge 授权
```

### 1.2 攻击者模型

| 攻击者类型 | 能力 | 典型场景 |
|------------|------|----------|
| **本地应用** | 有限权限 | 恶意 TA、提权尝试 |
| **远程应用** | 网络通信 | 通过 normal world 的 RPC 调用注入恶意数据 |
| **内核级** | 无权限 | 内核漏洞利用 |
| **硬件级** | 物理访问 | 侧信道攻击、硬件漏洞 |

---

## 安全机制概述

### 2.1 能力（Capability）系统

**核心机制**：所有内核对象访问通过 Capability 控制

**实现位置**：`kernel/object/capability.c`

**关键结构**（`kernel/include/object/object.h:20-38`）：
```c
struct object {
    u64 type;              // 对象类型
    u64 size;              // 对象大小
    unsigned long refcount;  // 引用计数
    u64 opaque[];          // 对象数据
};
```

**Capability 特性**：
- ✅ 类型安全：Capability 指向特定类型的对象
- ✅ 权限分离：通过 badge 区分不同权限级别
- ✅ 撤销机制：`sys_revoke_cap()` 可撤销能力
- ✅ 转移限制：`MAX_CAP_TRANSFER` 限制批量转移

### 2.2 Badge 授权系统

**固定 Badge**（`kernel/include/object/cap_group.h:131-138`）：
```c
#define ROOT_CAP_GROUP_BADGE (1)  // PROCMGR（根进程）
#define FSM_BADGE            (2)  // 文件系统管理器
#define LWIP_BADGE           (3)  // 网络栈
#define TMPFS_BADGE          (4)  // 临时文件系统
#define SERVER_BADGE_START   (5)  // 动态服务起点
#define DRIVER_BADGE_START   (100) // 设备驱动起点
#define APP_BADGE_START      (200) // 应用程序起点
```

**权限检查**（`kernel/syscall/syscall_hooks.c`）：
- ✅ `hook_sys_create_device_pmo()` - 仅驱动允许
- ✅ `hook_sys_get_phys_addr()` - Badge 范围检查（APP/DRIVER）
- ✅ `hook_sys_create_cap_group()` - 仅 ROOT/FSM/PROCMGR 允许

### 2.3 内存保护

**权限位**（`kernel/include/arch/aarch64/arch/mmu.h`）：
```c
#define VMR_READ    (1 << 0)   // 读权限
#define VMR_WRITE   (1 << 1)   // 写权限
#define VMR_EXEC    (1 << 2)   // 执行权限
#define VMR_DEVICE  (1 << 3)   // 设备内存
#define VMR_NOCACHE (1 << 4)   // 不可缓存
#define VMR_COW     (1 << 5)   // 写时复制
#define VMR_TZ_NS   (1 << 6)   // TrustZone 非安全内存
```

**地址空间隔离**（`kernel/include/mm/uaccess.h`）：
```c
// 用户空间：0x0000_0000_0000 - 0x7FFF_FFFF_FFFF
// 内核空间：0xFFFF_FF00_0000_0000+ (KBASE)

int check_user_addr_range(vaddr_t addr, size_t len);
```

### 2.4 TrustZone SMC

**SMC 接口**（`kernel/arch/aarch64/trustzone/spd/teed/smc.c`）：
- ✅ `sys_tee_switch_req()` - 切换到安全世界
- ✅ `sys_tee_wait_switch_req()` - 等待 SMC 请求
- ✅ `sys_tee_pull_kernel_var()` - 获取内核变量

**安全监控调用**（`kernel/arch/aarch64/trustzone/spd/opteed/smc_num.h`）：
```c
#define SMC_STD_REQUEST  0x3e000008  // 标准请求
#define SMC_STD_RESPONSE 0xbe000005  // 标准响应
```

---

## 攻击面分析

### 3.1 用户态 API

| API 类型 | 风险 | 证据 |
|----------|------|------|
| **256 个系统调用** | 权限提升、参数注入 | `kernel/syscall/syscall.c` |
| **IPC 接口** | 越界读写、Capability 篡改 | `kernel/ipc/connection.c`, `kernel/ipc/channel.c` |
| **共享内存（PMO_SHM）** | 竞争条件、数据泄露 | `kernel/object/memory.c` |

### 3.2 IPC 通信

**标准 ChCore IPC**：
```c
// kernel/ipc/connection.c
// 共享内存边界检查
static int check_ipc_msg_in_shm(struct ipc_msg *user_ipc_msg_ptr,
                                vaddr_t shm_start, size_t shm_size)
{
    // 检查 ipc_msg 是否在共享内存范围内
    if (!(shm_start_ptr <= ipc_msg_start && ipc_msg_end <= shm_end)) {
        return -1;  // ❌ 越界访问将被拒绝
    }
    // ... 验证 ipc_msg 的 data 和 cap 数组
}
```

**TEE IPC Channel**：
```c
// kernel/ipc/channel.c
// Channel 创建者验证
int sys_tee_msg_stop_channel(int channel_cap)
{
    struct channel *channel = get_channel(cap);
    // 仅创建者可以停止通道
    if (channel->creater != current_cap_group) {
        return -EPERM;  // ❌ 权限验证
    }
}
```

### 3.3 内存操作

**PMO 类型**（`kernel/include/object/memory.h:22-31`）：
| PMO 类型 | 安全风险 | 说明 |
|----------|----------|------|
| `PMO_ANONYM` | 懒分配 | 可能导致按需分页延迟 |
| `PMO_SHM` | 共享内存 | 需要 Capability + 权限验证 |
| `PMO_DEVICE` | 设备映射 | 需要 Badge 验证（仅驱动）|
| `PMO_TZ_NS` | 非安全内存 | 跨 TrustZone 边界 |

### 3.4 进程管理

**Badge 验证**（`user/system-services/system-servers/procmgr/procmgr.c`）：
```c
// 进程创建时的 Badge 分配
badge = pid_start++;
// 系统服务（badge 1-4） vs 应用（badge 200+）
```

---

## 信任边界

### 4.1 内核态隔离

**边界**：
```
用户态（EL0）          内核态（EL1）
    ↓ SVC
    ↓
    地址空间隔离（0x0000...0x7FFF）
    ↓
    Capability 验证
```

**保护机制**：
- ✅ 用户态地址验证（`check_user_addr_range()`）
- ✅ Capability 类型检查
- ✅ 页表权限检查（VMR_READ/WRITE/EXEC）

### 4.2 进程隔离

**边界**：
```
进程 A（VMSpace A，Badge X）
进程 B（VMSpace B，Badge Y）
进程 C（VMSpace C，Badge Z）
```

**保护机制**：
- ✅ 每个 Capability Group 独立的 VMSpace
- ✅ Capability 不能跨进程共享（除非显式转移）
- ✅ 堆大小限制（`cap_group->heap_size_limit`）

### 4.3 TrustZone 边界

**边界**：
```
非安全世界（Normal World）     安全世界（Secure World - TEE）
    ↓ SMC 接口
    ↓
    硬件强制隔离（NS bit）
```

**保护机制**：
- ✅ SMC 调用需要特定参数（`sys_tee_switch_req()`）
- ✅ TrustZone 监控器验证 SMC 参数
- ✅ 非安全 PMO 仅在 NS 世界访问

---

## 可被利用点

### 5.1 可被利用点清单

| # | 漏洞类型 | 严重性 | 证据 | 可利用路径 | 修复建议 |
|---|----------|--------|------|----------|----------|
| **1** | **共享内存越界** | 🔴 高 | `kernel/ipc/connection.c:check_ipc_msg_in_shm()` | 增强边界检查、整数溢出防护 |
| **2** | **IPC 数据注入** | 🟠 中 | `kernel/ipc/connection.c:copy_from_user()` | 严格验证 `ipc_msg` 结构，拒绝恶意参数 |
| **3** | **Capability 伪造** | 🟠 中 | `kernel/object/capability.c:cap_copy()` | 增强 Capability 转移验证，限制跨进程转移 |
| **4** | **Badge 欺骗** | 🟠 中 | `kernel/syscall/syscall_hooks.c` | 严格检查 Badge 来源，限制敏感操作 |
| **5** | **PMO 权限提升** | 🟠 中 | `kernel/object/memory.c:sys_map_pmo()` | 严格验证 Badge 权限，拒绝未授权 PMO 映射 |

### 5.2 详细漏洞分析

#### 漏洞 1：共享内存越界

**证据**：
```c
// kernel/ipc/connection.c:87-118
static int check_ipc_msg_in_shm(...)
{
    // ❌ 问题：可能存在整数溢出或边界检查绕过
    if (!(shm_start_ptr <= ipc_msg_start && ipc_msg_end <= shm_end)) {
        return -1;
    }
    // ... 验证 ipc_data 和 ipc_caps 数组
}
```

**可利用路径**：
```
1. 恶意 TA 通过精心构造的 ipc_msg 结构
2. 绕过边界检查（整数溢出或符号问题）
3. 在共享内存中越界写入
4. 导致内核信息泄露或代码执行
```

**修复建议**：
- 使用安全的整数运算（`size_t` 而非 `int`）
- 添加额外的长度验证
- 使用 `add_overflow(a, b)` 类安全宏
- 限制单个 IPC 消息的最大大小

#### 漏洞 2：Capability 伪造/泄露

**证据**：
```c
// kernel/object/capability.c
cap_t cap_copy(struct cap_group *src_cap_group,
               struct cap_group *dest_cap_group, cap_t src_slot_id)
{
    // ❌ 问题：如果 src_slot_id 已泄露或伪造
    src_slot = get_slot(src_cap_group, src_slot_id);
    // 复制到目标进程
    alloc_slot_id(dest_cap_group, &dst_slot_id);
    dest_slot->isvalid = 1;
    dest_slot->object = src_slot->object;
}
```

**可利用路径**：
```
1. 攻击者通过漏洞泄露一个高权限 Capability
2. 使用伪造的 Capability 创建新进程
3. 获得对敏感内核对象的访问权
4. 读取其他进程内存、修改配置等
```

**修复建议**：
- 限制 Capability 转移仅限父进程到子进程
- 验证调用者是否有权限转移 Capability
- 记录 Capability 转移审计日志
- 实现 Capability 引用计数验证

#### 漏洞 3：Badge 欺骗

**证据**：
```c
// kernel/syscall/syscall_hooks.c:44-64
// ❌ 问题：部分敏感系统调用仅检查 Badge 范围
int hook_sys_get_phys_addr(vaddr_t va, paddr_t *pa_buf)
{
    // 仅检查 Badge 是否在 APP 或 DRIVER 范围
    if ((badge >= APP_BADGE_START) || (badge < DRIVER_BADGE_START))
        return -EPERM;
}
```

**可利用路径**：
```
1. 恶意进程通过 Badge 欺骗通过检查
2. 伪装成系统服务（Badge 2-4）
3. 调用受限系统调用（如获取物理地址）
4. 泄露敏感信息或提升权限
```

**修复建议**：
- 使用 Capability 而非 Badge 进行敏感操作
- 引入 Capability 类型检查（ Capability 对象本身有类型）
- 实现 Capability 传递深度限制
- 记录所有敏感操作的调用者

#### 漏洞 4：PMO 权限提升

**证据**：
```c
// kernel/object/memory.c:72-89
int sys_map_pmo(cap_t target_cap_group, cap_t pmo_cap,
               unsigned long addr, unsigned long perm, unsigned long len)
{
    // ❌ 问题：可能绕过 Badge 检查
    // 获取 PMO 对象
    pmo = get_opaque(current_cap_group, pmo_cap, TYPE_PMO);
    // 验证权限（但 PMO 本身可能属于其他进程）
    // ... 映射到目标进程
}
```

**可利用路径**：
```
1. 攻击者创建共享 PMO（PMO_SHM）
2. 通过漏洞获得 PMO 的 Capability
3. 将 PMO 映射到目标进程
4. 绕过地址隔离，读写其他进程内存
5. 可能实现任意代码执行
```

**修复建议**：
- 严格验证 Capability 所有者和 PMO 所有者匹配
- 在映射前验证 PMO 的 `cap_group` 所有者
- 限制 PMO 转移和跨进程共享
- 实现 PMO 使用审计日志

#### 漏洞 5：TrustZone SMC 注入

**证据**：
```c
// kernel/arch/aarch64/trustzone/spd/teed/smc.c:34-56
void handle_yield_smc(unsigned long x0, unsigned long x1,
                      unsigned long x2, unsigned long x3,
                      unsigned long x4, unsigned long x5)
{
    // ❌ 问题：SMC 参数验证不足
    // 直接使用寄存器值进行内核操作
    kernel_var.params_stack[0] = x0;
    kernel_var.params_stack[1] = x1;
    // ... 缺少足够的验证
}
```

**可利用路径**：
```
1. Normal World 恶意应用发起 SMC 调用
2. 通过精心构造的参数（如 x2）影响内核状态
3. 导致 TEE 内核逻辑错误
4. 可能绕过安全检查或导致信息泄露
```

**修复建议**：
- 严格验证 SMC 参数范围和格式
- 实现 SMC 调用频率限制
- 记录所有 SMC 调用和来源
- 使用 TrustZone 监控器参数验证

---

## 安全建议

### 6.1 短期改进

| 改进项 | 优先级 | 说明 |
|----------|--------|------|
| **参数校验增强** | 🔴 高 | 在所有系统调用中添加严格的参数验证，防止整数溢出和类型混淆 |
| **Capability 审计** | 🔴 高 | 实现 Capability 转移和使用的审计日志，支持事后追溯 |
| **SMC 参数验证** | 🔴 高 | 增强 TrustZone SMC 调用的参数验证，添加调用频率限制 |
| **边界检查强化** | 🟠 中 | 使用安全的整数运算库，防止边界检查绕过 |
| **内存隔离加固** | 🟠 中 | 严格验证 Capability 所有者权限，限制跨进程 PMO 共享 |

### 6.2 长期改进

| 改进项 | 优先级 | 说明 |
|----------|--------|------|
| **形式化验证** | 🟠 中 | 引入形式化的 Capability 验证逻辑，使用数学模型证明安全性 |
| **攻击面最小化** | 🟠 中 | 减少特权系统调用暴露，限制不必要的 IPC 接口 |
| **TrustZone 强化** | 🟠 中 | 实现更严格的 SMC 协议，支持密钥协商和认证 |
| **动态分析** | 🟠 中 | 集成运行时安全分析工具，检测异常行为 |

---

## 相关跳转

- [系统调用接口](03_Syscall_Interfaces.md) - 完整的系统调用清单与参数校验
- [架构设计](02_Architecture.md) - 安全架构层次与信任边界
- [目录结构](01_Directory_Structure.md) - 安全相关代码位置

---

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2026-02-06
