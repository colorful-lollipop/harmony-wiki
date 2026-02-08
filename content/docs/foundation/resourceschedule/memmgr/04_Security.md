# 安全风险评审 (Security)

> MemMgr 组件安全分析与风险评估

## 1. 评审范围

### 1.1 覆盖范围

| 范围 | 描述 |
|------|------|
| **代码目录** | `common/`, `interface/innerkits/`, `services/memmgrservice/` |
| **配置文件** | `memmgr_config.xml`, `1909.json` |
| **IPC 接口** | `IMemMgr` 定义的全部方法 |
| **内核交互** | `kernel_interface.cpp` 全部操作 |

### 1.2 未覆盖范围

| 范围 | 原因 |
|------|------|
| `test/` | 按规范忽略测试代码 |
| 运行时动态行为 | 需要实际运行环境验证 |
| 依赖子系统安全 | 依赖组件 (`ipc`, `safwk` 等) 由各自评审覆盖 |

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 攻击面 | 类型 | 风险等级 | 证据 |
|--------|------|----------|------|
| **IPC/System Ability** | 远程调用 | **高** | `i_mem_mgr.h:34-64` |
| **配置文件解析** | XML 注入/路径遍历 | **中** | `xml_helper.cpp` |
| **内核接口** | 权限提升/拒绝服务 | **中** | `kernel_interface.cpp` |
| **进程查杀** | DoS/误杀关键进程 | **高** | `low_memory_killer.cpp` |
| **回调接口** | 回调劫持 | **低** | `*Observer` 类 |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                              │
├─────────────────────────────────────────────────────────────┤
│  内部 (可信)                                                 │
│  ├── MemMgrService (SA 1909)                               │
│  ├── MemMgrClient (单例)                                    │
│  └── KernelInterface (特权操作)                              │
├─────────────────────────────────────────────────────────────┤
│  外部 (不可信)                                               │
│  ├── IPC 调用方 (其他 SA/应用)                              │
│  ├── XML 配置文件 (/etc/memmgr/)                             │
│  └── 内核接口 (/proc/, /dev/)                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 可利用风险点

### 3.1 风险 1: IPC 参数校验不足

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-001 |
| **风险等级** | 高 |
| **攻击面** | IPC/System Ability |

**证据** (`interface/innerkits/include/i_mem_mgr.h:40`):

```cpp
virtual int32_t NotifyDistDevStatus(int32_t pid, int32_t uid,
    const std::string &name, bool connected) = 0;
```

**问题描述**:
- `NotifyDistDevStatus` 接收 `name` (string) 参数但未校验长度和内容
- 恶意调用方可构造超长字符串导致栈溢出或堆溢出

**触发条件**:
```cpp
// 恶意调用
memMgr->NotifyDistDevStatus(pid, uid,
    "超长字符串...（>10KB）", true);
```

**潜在影响**:
- 堆溢出可能导致代码执行
- 拒绝服务 (DoS)

**修复建议**:
```cpp
// 添加参数校验
if (name.length() > MAX_DEVICE_NAME_LEN) {
    return ERR_INVALID_VALUE;
}
```

---

### 3.2 风险 2: 进程查杀无权限校验

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-002 |
| **风险等级** | 高 |
| **攻击面** | 查杀策略 |

**证据** (`low_memory_killer.cpp`, `kill_strategy_manager/low_memory_killer.h`):

```cpp
class LowMemoryKiller {
public:
    bool KillProcessByPriority(int minPriority);  // 无权限校验
};
```

**问题描述**:
- `SetCritical`/`NotifyProcessStatus` 等接口可能被滥用
- 恶意进程可标记自身或他人为"关键进程" (不可被杀)
- 或触发非必要的进程查杀

**触发条件**:
```cpp
// 恶意进程调用
memMgr->SetCritical(maliciousPid, true, -1);  // 标记自己为关键进程
```

**潜在影响**:
- 关键进程无法被回收
- 内存耗尽导致系统不稳定

**修复建议**:
```cpp
// 添加调用方权限校验
if (!CheckPermission(callerUid, "ohos.permission.MANAGE_MEMMGMT")) {
    return ERR_PERMISSION_DENIED;
}
```

---

### 3.3 风险 3: 配置文件路径遍历

| 属性 | 值 |
|| **风险 ID------|-----|
** | SEC-003 |
| **风险等级** | 中 |
| **攻击面** | XML 配置解析 |

**证据** (`common/src/xml_helper.cpp`, `common/include/xml_helper.h`):

```cpp
class XMLHelper {
public:
    bool LoadConfig(const std::string &configPath);
};
```

**问题描述**:
- 配置路径硬编码或可控，可能导致路径遍历
- 攻击者替换配置文件注入恶意参数

**触发条件**:
```cpp
// 可能的攻击场景
// 1. 恶意配置文件注入
// 2. 符号链接攻击
```

**潜在影响**:
- 恶意配置注入
- 权限提升

**修复建议**:
```cpp
// 验证配置文件路径
if (!IsPathSafe(configPath)) {
    return ERR_INVALID_PATH;
}
```

---

### 3.4 风险 4: OOM Score 注入

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-004 |
| **风险等级** | 中 |
| **攻击面** | 内核接口 |

**证据** (`reclaim_priority_manager/oom_score_adj_utils.h`):

```cpp
class OomScoreAdjUtils {
public:
    static bool SetOomScoreAdj(int pid, int oomScoreAdj);
};
```

**问题描述**:
- `SetOomScoreAdj` 直接写入内核参数
- 未校验 pid 是否属于调用方

**触发条件**:
```cpp
// 恶意进程修改其他进程 OOM score
OomScoreAdjUtils::SetOomScoreAdj(victimPid, -1000);  // 标记为不可杀
```

**潜在影响**:
- 进程逃避查杀
- 系统内存管理失效

**修复建议**:
```cpp
// 校验 pid 归属
if (getuid() != getPidUid(pid)) {
    return ERR_PERMISSION_DENIED;
}
```

---

### 3.5 风险 5: 回调注册无校验

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-005 |
| **风险等级** | 低 |
| **攻击面** | 事件订阅 |

**证据** (`purgeable_mem_manager/iapp_state_subscriber.h`):

```cpp
virtual int32_t SubscribeAppState(const sptr<IAppStateSubscriber> &subscriber) = 0;
```

**问题描述**:
- 任意进程可注册应用状态回调
- 回调可能被用于信息收集

**触发条件**:
```cpp
// 恶意进程注册回调
memMgr->SubscribeAppState(maliciousSubscriber);
```

**潜在影响**:
- 用户行为追踪 (低风险)
- 信息泄露 (低风险)

**修复建议**:
```cpp
// 添加权限校验
if (!CheckPermission(callerUid, "ohos.permission.MANAGE_MEMMGMT")) {
    return ERR_PERMISSION_DENIED;
}
```

---

## 4. 安全机制

### 4.1 现有安全机制

| 机制 | 实现 | 证据 |
|------|------|------|
| SA 权限管控 | SAFwk 框架 | `bundle.json:24` (safwk 依赖) |
| Binder IPC | IPC 框架权限校验 | `BUILD.gn:122` (ipc 依赖) |
| DAC 权限配置 | `memmgr.para.dac` | `bundle.json:47` |

### 4.2 缺失安全机制

| 机制 | 缺失原因 |
|------|----------|
| API 权限校验 | `IMemMgr` 方法未检查调用者权限 |
| 参数范围校验 | 多数接口缺少输入验证 |
| PID 归属校验 | `SetOomScoreAdj` 未校验 pid 归属 |

---

## 5. 运行时配置安全

### 5.1 配置文件权限

| 文件 | 默认权限 | 建议 |
|------|----------|------|
| `/etc/memmgr/memmgr_config.xml` | 系统只读 | 应限制仅 memmgrservice 可写 |

### 5.2 配置项风险

| 配置项 | 风险 | 建议 |
|--------|------|------|
| `killLevel` | 误配置可导致系统不稳定 | 限制取值范围 |
| `availBuffer` | 错误值可触发 DoS | 校验与内存总量关系 |
| `ZswapdParam` | 极端值影响性能 | 添加边界检查 |

---

## 6. 总结

### 6.1 风险统计

| 等级 | 数量 | 详情 |
|------|------|------|
| **高** | 2 | SEC-001 (IPC 参数), SEC-002 (查杀权限) |
| **中** | 2 | SEC-003 (路径遍历), SEC-004 (OOM 注入) |
| **低** | 1 | SEC-005 (回调注册) |

### 6.2 修复优先级

| 优先级 | 风险 | 建议 |
|--------|------|------|
| **P0** | SEC-001, SEC-002 | 添加权限校验和参数验证 |
| **P1** | SEC-003, SEC-004 | 路径和 PID 归属校验 |
| **P2** | SEC-005 | 回调权限控制 |

---

## 7. 相关跳转

- **概览**: [00_Overview.md](./00_Overview.md)
- **架构**: [01_Architecture.md](./01_Architecture.md)
- **Inner API**: [02_Inner_API.md](./02_Inner_API.md)
- **构建配置**: [03_Build.md](./03_Build.md)
- **导航**: [SUMMARY.md](./SUMMARY.md)

---

*文档版本: 3.1.0 | 最后更新: 2026-02-06*
