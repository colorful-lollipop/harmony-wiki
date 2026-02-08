# 06_SecurityReview - 安全风险评估

本文档对 SOC 统一调频部件进行系统性的安全风险评估，分析潜在漏洞、攻击路径和修复建议。

## 评估范围

| 范围 | 说明 |
|------|------|
| **代码范围** | `services/`, `interfaces/` 目录（排除 test/） |
| **配置范围** | `profile/` 目录下的 XML 配置文件 |
| **接口范围** | ISocPerf.idl 定义的所有 IPC 接口 |
| **排除范围** | 测试代码、内核驱动 |

## 风险评估方法

### 评估标准

| 风险等级 | 定义 | 响应要求 |
|----------|------|----------|
| **高危** | 可直接导致权限提升或系统不稳定 | 立即修复 |
| **中危** | 可能被利用，需要特定条件 | 尽快修复 |
| **低危** | 影响有限或难以利用 | 计划修复 |
| **信息** | 潜在风险点，无直接利用可能 | 记录观察 |

## 风险清单

### R1: 输入验证缺失 - cmdId 整数范围

**风险等级**：中危

**位置**：`services/core/src/socperf.cpp:45-89`

**证据**：

```cpp
void SocPerf::PerfRequest(int32_t cmdId, const std::string& msg)
{
    // 无 cmdId 范围检查
    std::shared_ptr<Actions> actions = GetActionsInfo(cmdId);
    if (actions == nullptr) {
        SOC_PERF_LOGI("cmdId: %{public}d no actions", cmdId);
        return;
    }
    // ...
}
```

**触发路径**：

```
HAP 应用 → SocPerfClient::PerfRequest(cmdId=-1)
    → IPC 调用
    → SocPerfServer::PerfRequest()
    → SocPerf::PerfRequest()
    → GetActionsInfo(-1)  // 无范围检查
    → 返回空指针
```

**影响**：

- 恶意应用可传递任意整数值的 cmdId
- 可能导致空指针解引用（DoS）
- 绕过配置的 cmdId 白名单机制

**修复建议**：

```cpp
void SocPerf::PerfRequest(int32_t cmdId, const std::string& msg)
{
    // 添加 cmdId 范围检查
    if (cmdId < 1000 || cmdId > 65535) {
        SOC_PERF_LOGE("Invalid cmdId: %{public}d", cmdId);
        return;
    }
    // ...
}
```

### R2: 数组越界风险 - LimitRequest 参数

**风险等级**：中危

**位置**：`services/core/src/socperf.cpp:136-173`

**证据**：

```cpp
void SocPerf::LimitRequest(int32_t clientId,
    const std::vector<int32_t>& tags, const std::vector<int64_t>& configs, const std::string& msg)
{
    // tags 和 configs 长度未校验
    for (size_t i = 0; i < tags.size(); i++) {
        SendLimitRequestEvent(clientId, tags[i], configs[i]);
    }
}
```

**触发路径**：

```
HAP 应用 → SocPerfClient::LimitRequest()
    → IPC 调用（tags 和 configs 数组）
    → SocPerf::LimitRequest()
    → 遍历 tags 和 configs
    → tags.size() != configs.size() 时越界访问
```

**影响**：

- 数组长度不匹配导致越界访问
- 可能读取无效内存或写入错误配置
- 崩溃风险（DoS）

**修复建议**：

```cpp
void SocPerf::LimitRequest(int32_t clientId,
    const std::vector<int32_t>& tags, const std::vector<int64_t>& configs, const std::string& msg)
{
    // 校验数组长度
    if (tags.size() != configs.size()) {
        SOC_PERF_LOGE("tags size %{public}zu != configs size %{public}zu",
            tags.size(), configs.size());
        return;
    }
    if (tags.empty() || tags.size() > MAX_LIMIT_TAGS) {
        SOC_PERF_LOGE("Invalid tags size: %{public}zu", tags.size());
        return;
    }
    // ...
}
```

### R3: 权限缓存投毒

**风险等级**：低危

**位置**：`services/server/include/socperf_server.h:124`

**证据**：

```cpp
SocPerfLRUCache<AccessToken::AccessTokenID, int32_t> permissionCache_;

// 缓存使用（socperf_server.cpp:196）
int32_t result = permissionCache_.Get(callerToken);
if (result != -1) {
    return result == PERM_OK;
}
```

**触发路径**：

```
1. 正常应用 A 获取权限缓存 [tokenA] = PERM_OK
2. 应用 B 尝试复用 tokenA 的缓存结果
3. 权限检查被绕过
```

**影响**：

- Token ID 被缓存在客户端进程
- 恶意应用可能复用其他应用的 Token 权限
- 需要进程间共享内存条件，利用难度高

**缓解措施**：

- 缓存有效期有限（LRU 机制）
- 权限验证仅跳过 HAP Token 检查

### R4: 竞态条件 - 多客户端并发

**风险等级**：低危

**位置**：`services/core/include/socperf.h:57-60`

**证据**：

```cpp
std::mutex mutex_;              // 通用互斥锁
std::mutex mutexDeviceMode_;    // 设备模式锁
std::mutex mutexBoostCmdCount_; // 提频命令计数锁
std::mutex mutexBoostTime_;     // 提频时间锁
```

**触发路径**：

```
客户端 A → PerfRequest(cmdId=1001)
    ↓ 时间片耗尽
客户端 B → PerfRequest(cmdId=1002)
    ↓
竞态窗口 → 数据不一致
```

**影响**：

- 多线程环境下 `limitRequest_` 数据结构竞争
- 统计计数不准确
- 调频策略执行顺序不确定

**缓解措施**：

- 已使用多把互斥锁保护不同数据
- FFRT 任务队列串行化执行

### R5: XML 配置解析风险

**风险等级**：中危

**位置**：`services/core/src/socperf_config.cpp:141-181`

**证据**：

```cpp
xmlDocPtr doc = xmlReadFile(filePath.c_str(), nullptr, 0);
if (doc == nullptr) {
    SOC_PERF_LOGE("Load config file failed: %{public}s", filePath.c_str());
    return false;
}
```

**触发路径**：

```
1. 恶意 XML 配置文件注入
2. 配置文件解析
3. XXE 攻击或路径遍历
```

**风险点**：

| 风险 | 说明 | 当前状态 |
|------|------|----------|
| XXE 攻击 | XML 外部实体注入 | 未验证 |
| 路径遍历 | `../` 注入到路径属性 | 未验证 |
| 整数溢出 | resId、cmdId 超出范围 | 部分验证 |

**修复建议**：

```cpp
// 禁用 XML 外部实体
xmlParserOption options = XML_PARSE_NOENT | XML_PARSE_DTDLOAD;
xmlDocPtr doc = xmlReadFile(filePath.c_str(), nullptr, options);

// 路径验证
if (filePath.find("../") != std::string::npos) {
    SOC_PERF_LOGE("Path traversal detected: %{public}s", filePath.c_str());
    return false;
}
```

### R6: 设备节点路径未验证

**风险等级**：高危

**位置**：`services/core/src/socperf_thread_wrap.cpp:156-178`

**证据**：

```cpp
ResNode* resNode = static_cast<ResNode*>(resNodeBase.get());
int32_t fd = open(resNode->path.c_str(), O_RDWR);
if (fd < 0) {
    SOC_PERF_LOGE("Open path failed: %{public}s", resNode->path.c_str());
    return false;
}
write(fd, &value, sizeof(value));
```

**触发路径**：

```
1. 恶意配置文件定义路径 "/etc/passwd"
2. SocPerfConfig 解析路径
3. 写入操作覆盖 /etc/passwd
```

**影响**：

- 配置文件中的恶意路径可导致任意文件写入
- 可用于提权或系统破坏
- 风险等级：高危

**缓解措施**：

- 配置文件只有系统管理员可修改
- SELinux/AppArmor 限制进程文件访问权限
- 内核驱动白名单验证路径

### R7: 热等级无范围检查

**风险等级**：低危

**位置**：`services/core/src/socperf.cpp:154`

**证据**：

```cpp
void SocPerf::SetThermalLevel(int32_t level)
{
    thermalLvl_ = level;  // 无范围检查
    SOC_PERF_LOGI("Set thermal level: %{public}d", thermalLvl_);
}
```

**触发路径**：

```
HAP 应用 → SetThermalLevel(level=-1000000)
    → thermalLvl_ 被设置为负值
    → 后续提频逻辑异常
```

**影响**：

- 负值热等级导致逻辑异常
- 可能绕过热限频机制
- 利用难度：高（需权限）

## 安全机制评估

### 已有安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| 权限验证 | `HasPerfPermission()` | ✅ 有效 |
| 权限缓存 | `permissionCache_` | ⚠️ 需改进 TTL |
| 互斥锁 | `mutex_` 等 | ✅ 有效 |
| SELinux | 内核层 | ✅ 依赖系统配置 |

### 缺失安全机制

| 机制 | 建议 |
|------|------|
| 输入参数范围检查 | cmdId、level、tags 范围验证 |
| XML 解析安全 | 禁用 XXE、路径白名单 |
| 路径验证 | 设备节点白名单 |
| 审计日志 | 记录所有调频操作 |

## 风险统计

| 风险等级 | 数量 | 占比 |
|----------|------|------|
| 高危 | 1 | 14% |
| 中危 | 3 | 43% |
| 低危 | 3 | 43% |
| 信息 | 0 | 0% |

## 修复优先级

| 优先级 | 风险 | 建议修复时间 |
|--------|------|--------------|
| P0 | R6 | 立即修复 |
| P1 | R1, R2, R5 | 1 周内 |
| P2 | R3, R4, R7 | 1 个月内 |

---

## 相关文档

- 攻击面分析：[05_AttackSurface](05_AttackSurface.md)
- 接口文档：[04_Interface](04_Interface.md)
- 架构设计：[02_Architecture](02_Architecture.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
