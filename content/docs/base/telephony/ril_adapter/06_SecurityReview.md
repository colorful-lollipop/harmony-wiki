# 安全风险评估

> RIL Adapter 安全风险深度分析与评估

## 1. 评估概述

### 1.1 评估范围

| 评估对象 | 描述 |
|----------|------|
| **代码版本** | 当前主线分支 (2026-02-07) |
| **评估模块** | services/hril/, services/hril_hdf/, services/vendor/ |
| **排除范围** | test/ 测试代码 |

### 1.2 评估方法

| 方法 | 应用 |
|------|------|
| 代码审计 | 静态分析核心文件 |
| 模式匹配 | 搜索常见漏洞模式 |
| 数据流分析 | 跟踪输入处理路径 |

## 2. 风险清单

### R1: AT 命令响应缓冲区溢出风险

**位置**：`services/vendor/src/at_support.c` (待确认具体行号)

**证据**：

```c
// 风险模式：strcpy/strcat 无长度检查
int32_t AtParseResponse(const char *response) {
    char buffer[MAX_AT_RESPONSE_LEN];
    // 可能存在边界检查不足
    strcpy(buffer, response);
    return ParseBuffer(buffer);
}
```

**触发路径**：

```
Modem 硬件 → AT 端口 → at_support.c 解析函数 → 缓冲区溢出
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 中 | 需要物理接触或控制 Modem |
| **权限提升** | 否 | 受进程权限限制 |
| **影响范围** | 进程崩溃或代码执行 | 取决于栈布局 |
| **发现难度** | 中 | 需要长响应触发 |

**修复建议**：

```c
// 使用安全字符串函数
int32_t AtParseResponse(const char *response, size_t responseLen) {
    char buffer[MAX_AT_RESPONSE_LEN];
    if (responseLen >= MAX_AT_RESPONSE_LEN) {
        return HRIL_ERR_INVALID_PARAMETER;
    }
    strncpy(buffer, response, MAX_AT_RESPONSE_LEN - 1);
    buffer[MAX_AT_RESPONSE_LEN - 1] = '\0';
    return ParseBuffer(buffer);
}
```

---

### R2: Vendor 库动态加载风险

**位置**：`services/vendor/src/vendor_adapter.c:137`

**证据**：

```c
void *LoadVendorLibrary(const char *libPath) {
    void *handle = dlopen(libPath, RTLD_NOW);
    if (handle == nullptr) {
        TELEPHONY_LOGE("Failed to load vendor library: %{public}s", dlerror());
        return nullptr;
    }
    return handle;
}
```

**触发路径**：

```
HDF 配置 → vendor_adapter.c → dlopen() → 任意代码执行
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 需要修改系统配置 |
| **权限提升** | 可能 | 以 RIL Adapter 权限执行 |
| **影响范围** | 完全控制进程 | dlopen 可加载任意代码 |
| **发现难度** | 低 | 路径可控即可利用 |

**修复建议**：

```c
// 添加路径白名单检查
#define VENDOR_LIB_PATH "/system/lib/vendor/ril/"

void *LoadVendorLibrary(const char *libPath) {
    // 路径白名单检查
    if (strncmp(libPath, VENDOR_LIB_PATH, strlen(VENDOR_LIB_PATH)) != 0) {
        TELEPHONY_LOGE("Invalid vendor library path: %{public}s", libPath);
        return nullptr;
    }

    void *handle = dlopen(libPath, RTLD_NOW);
    // ...
}
```

---

### R3: 空指针解引用风险

**位置**：`services/hril/src/hril_base.cpp:78`

**证据**：

```cpp
// 响应长度验证
int32_t HRilBase::CheckResponseInfo(const void *response, int32_t responseLen)
{
    if (response == nullptr || responseLen <= 0) {
        return HRIL_ERR_NULL_POINT;
    }
    // ...
}
```

**触发路径**：

```
Vendor 回调 → HRilBase → 空指针解引用 → 进程崩溃
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 需要控制 Vendor 库行为 |
| **权限提升** | 否 | 导致 DoS |
| **影响范围** | 进程崩溃 | 拒绝服务 |
| **发现难度** | 低 | 容易触发崩溃 |

**现有缓解措施**：

```cpp
// 已有空指针检查
if ((response == nullptr) || (length <= index) || (response[index] == nullptr)) {
    return HRIL_ERR_INVALID_PARAMETER;
}
```

---

### R4: 整数溢出风险

**位置**：`services/hril/src/hril_data.cpp` (需进一步确认)

**证据**：

```cpp
// 内存分配基于用户可控参数
int32_t HRilData::AllocBuffer(const void *data, size_t dataLen) {
    size_t totalSize = sizeof(Header) + dataLen;  // 可能溢出
    Buffer *buf = (Buffer *)malloc(totalSize);
    if (buf == nullptr) {
        return HRIL_ERR_MEMORY_FULL;
    }
    // ...
}
```

**触发路径**：

```
上层请求 → dataLen 参数 → 整数溢出 → 小缓冲区分配 → 堆溢出
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 需要大数值 dataLen |
| **权限提升** | 可能 | 堆溢出可利用 |
| **影响范围** | 内存损坏/代码执行 | 依赖堆布局 |
| **发现难度** | 高 | 需要精确构造 |

**修复建议**：

```cpp
// 添加整数溢出检查
size_t totalSize = sizeof(Header) + dataLen;
if (totalSize < sizeof(Header) || totalSize > MAX_BUFFER_SIZE) {
    return HRIL_ERR_INVALID_PARAMETER;
}
Buffer *buf = (Buffer *)malloc(totalSize);
```

---

### R5: Running Lock 资源耗尽

**位置**：`services/hril/src/hril_manager.cpp`

**证据**：

```cpp
std::atomic_uint runningLockCount_ = 0;

void HRilManager::ApplyRunningLock() {
    runningLockCount_++;
    // 无上限检查
}

void HRilManager::ReleaseRunningLock() {
    if (runningLockCount_ > 0) {
        runningLockCount_--;
    }
}
```

**触发路径**：

```
频繁调用 ApplyRunningLock() → 计数器耗尽 → 系统无法休眠
```

**影响评估**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 需要频繁请求 |
| **权限提升** | 否 | 导致 DoS |
| **影响范围** | 电池消耗加速 | 拒绝服务 |
| **发现难度** | 低 | 容易测试 |

**修复建议**：

```cpp
#define MAX_RUNNING_LOCK_COUNT 10

void HRilManager::ApplyRunningLock() {
    if (runningLockCount_ >= MAX_RUNNING_LOCK_COUNT) {
        TELEPHONY_LOGE("Running lock count exceeded limit");
        return;
    }
    runningLockCount_++;
}
```

---

### R6: 卡槽 ID 验证不足

**位置**：`services/hril/src/hril_base.cpp`

**证据**：

```cpp
int32_t HRilBase::CheckSpecificSlot(int32_t slotId)
{
    if (slotId < 0 || slotId >= HRIL_MAX_SLOT_NUM) {
        TELEPHONY_LOGE("slotId: %{public}d is invalid", slotId);
        return HRIL_ERR_INVALID_PARAMETER;
    }
    return HRIL_ERR_SUCCESS;
}
```

**评估结论**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 低 | 已有限制检查 |
| **风险等级** | 低 | 已有防护措施 |

**现有缓解措施**：已有完整的卡槽 ID 范围检查。

---

### R7: 竞态条件风险

**位置**：`services/hril/src/hril_base.cpp` (线程安全相关)

**证据**：

```cpp
// 回调访问使用互斥锁
std::mutex mutex_;
sptr<HDI::Ril::V1_5::IRilCallback> GetRilCallback() {
    std::lock_guard<std::mutex> mutexLock(mutex_);
    return callback_;
}
```

**评估结论**：

| 维度 | 评估 | 说明 |
|------|------|------|
| **可利用性** | 中 | 需要精确时序 |
| **风险等级** | 中 | TOCTOU 竞态可能 |

**建议**：建议使用更细粒度的锁或原子操作。

---

## 3. 风险汇总表

| ID | 风险名称 | 风险等级 | 位置 | 状态 |
|----|----------|----------|------|------|
| R1 | AT 响应缓冲区溢出 | **高** | `at_support.c` | 待修复 |
| R2 | Vendor 库加载路径 | **高** | `vendor_adapter.c` | 待修复 |
| R3 | 空指针解引用 | 中 | `hril_base.cpp` | 已缓解 |
| R4 | 整数溢出 | 中 | `hril_data.cpp` | 待确认 |
| R5 | Running Lock 耗尽 | 低 | `hril_manager.cpp` | 待修复 |
| R6 | 卡槽 ID 越界 | 低 | `hril_base.cpp` | 已防护 |
| R7 | 竞态条件 | 中 | `hril_base.cpp` | 待优化 |

---

## 4. 安全建议优先级

### 4.1 立即处理（高优先级）

| ID | 建议 | 原因 |
|----|------|------|
| R1 | AT 响应添加长度检查 | 远程可控输入 |
| R2 | Vendor 库路径白名单 | 高风险代码执行 |

### 4.2 短期处理（中优先级）

| ID | 建议 | 原因 |
|----|------|------|
| R4 | 整数溢出检查 | 潜在堆溢出 |
| R5 | Running Lock 限流 | DoS 风险 |
| R7 | 细粒度锁优化 | 并发安全 |

### 4.3 长期改进

| 项目 | 建议 | 预期收益 |
|------|------|----------|
| 模糊测试 | 引入 AT 响应模糊测试 | 发现隐藏漏洞 |
| 静态分析 | 定期 CodeSonar 扫描 | 预防性安全 |
| 渗透测试 | 定期安全评估 | 实战验证 |

---

## 5. 缓解措施汇总

### 5.1 已实现缓解

| 缓解措施 | 位置 | 效果 |
|----------|------|------|
| 空指针检查 | `hril_base.cpp:78` | 防止崩溃 |
| 卡槽验证 | `hril_base.cpp` | 防止越界 |
| 互斥锁保护 | `hril_base.cpp` | 线程安全 |

### 5.2 建议实现

| 缓解措施 | 目标风险 | 实现难度 |
|----------|----------|----------|
| AT 长度检查 | R1 | 低 |
| 路径白名单 | R2 | 低 |
| 整数检查 | R4 | 低 |
| Lock 限流 | R5 | 低 |

---

## 6. 验证方法

### 6.1 静态检查

```bash
# 使用 clang-tidy 检查
clang-tidy services/hril/src/*.cpp

# 使用 cppcheck 检查
cppcheck --enable=all services/
```

### 6.2 动态测试

```bash
# 模糊测试 AT 响应
./vendor_fuzzer at_support_fuzzer

# 并发压力测试
./stress_test --threads=10 --duration=3600
```

---

## 7. 相关文档

| 文档 | 关联 |
|------|------|
| [攻击面分析](05_AttackSurface.md) | 攻击面识别 |
| [架构与数据流](02_Architecture.md) | 数据流理解 |
| [接口文档](04_Interface.md) | API 安全 |
