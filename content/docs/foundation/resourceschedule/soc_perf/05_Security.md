# 安全风险评审

## 威胁模型

### 外部输入 → 敏感操作

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          攻击面图                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐            │
│  │ IPC 调用者   │────▶│ SocPerfServer │────▶│ Linux Kernel │            │
│  │ (跨进程)     │     │ (权限校验)    │     │ (cpufreq)   │            │
│  └──────────────┘     └──────────────┘     └──────────────┘            │
│         ↑                      ↑                      ↑                 │
│         │                      │                      │                 │
│    恶意应用            权限绕过              内核接口漏洞                │
│    越权请求            配置篡改              竞态条件                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 信任边界

| 边界 | 信任级别 | 说明 |
|------|---------|------|
| IPC 接口 | 不可信 | 任何进程都可发起 IPC 调用 |
| 配置目录 | 可信 | 需系统签名才能修改 |
| 内核接口 | 可信 | 内核层有独立安全机制 |
| SA 服务 | 核心 | 运行在特权进程 |

## 攻击面清单

| 攻击面 | 类型 | 位置 |
|--------|------|------|
| IPC 接口 | 远程 | `ISocPerf.idl` 定义的所有方法 |
| 配置解析 | 本地 | `socperf_config.cpp` XML 解析 |
| 权限校验 | 远程 | `socperf_server.cpp:HasPerfPermission()` |
| 服务死亡回调 | 远程 | `socperf_client.cpp:DeathRecipient` |

## 可被利用点分析

### 1. 权限绕过漏洞（高风险）

**证据**：`services/server/src/socperf_server.cpp:183-212`

```cpp
bool SocPerfServer::HasPerfPermission()
{
    uint32_t accessToken = IPCSkeleton::GetCallingTokenID();
    auto tokenType = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(accessToken);
    if (int(tokenType) == OHOS::Security::AccessToken::ATokenTypeEnum::TOKEN_HAP) {
        uint64_t fullTokenId = IPCSkeleton::GetCallingFullTokenID();
        if (!Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(fullTokenId)) {
            SOC_PERF_LOGE("Invalid Permission to SocPerf");
            return false;
        }
    }
    // ... 后续权限检查
}
```

**触发条件**：
- 非系统 HAP 应用调用任何 PerfRequest 接口
- TokenType 获取失败导致 bypass 检查

**影响**：
- 恶意应用可触发 CPU 频率调整
- 耗电增加、设备发热

**修复建议**：
- 增加 TokenType 获取失败的默认拒绝逻辑
- 添加完整的审计日志

---

### 2. 权限缓存投毒（中风险）

**证据**：`services/server/include/socperf_server.h:124`

```cpp
SocPerfLRUCache<AccessToken::AccessTokenID, int32_t> permissionCache_;
```

```cpp
// socperf_server.cpp:196-200
if (permissionCache_.get(accessToken, hasPermission)) {
    return hasPermission == 0;
}
hasPermission = AccessToken::AccessTokenKit::VerifyAccessToken(accessToken, NEEDED_PERMISSION);
permissionCache_.put(accessToken, hasPermission);
```

**触发条件**：
- TokenID 在缓存期间权限被撤销
- LRU 缓存条目未设置过期时间

**影响**：
- 应用权限变更后仍可使用旧权限访问
- 权限回收延迟

**修复建议**：
- 缓存添加 TTL（Time To Live）
- 监听权限变更事件刷新缓存

---

### 3. 整数溢出风险（低风险）

**证据**：`interfaces/inner_api/socperf_client/src/socperf_client.cpp:186`

```cpp
if (!CheckClientValid() || mode.length() > MAX_MODE_LEN) {
    return;
}
```

**触发条件**：
- `mode.length()` 返回负值（理论上的整数溢出场景）

**影响**：
- 边界检查失效
- 潜在的缓冲区溢出

**修复建议**：
- 转换为 size_t 类型后比较

---

### 4. 竞态条件（中风险）

**证据**：`services/core/src/socperf.cpp:47-52`

```cpp
private:
    bool enabled_ = false;
    std::mutex mutex_;
    std::mutex mutexDeviceMode_;
```

**触发条件**：
- 多线程并发调用 PerfRequest
- 状态检查与使用之间存在窗口

**影响**：
- 状态不一致
- 配置覆盖

**修复建议**：
- 关键操作使用原子操作
- 增加锁粒度控制

---

### 5. 配置文件注入（中风险）

**证据**：`services/core/include/socperf_config.h:49-51`

```cpp
std::string GetRealConfigPath(const std::string& configFile);
std::vector<std::string> GetAllRealConfigPath(const std::string& configFile);
bool LoadAllConfigXmlFile(const std::string& configFile);
```

**触发条件**：
- XML 解析器存在漏洞（libxml2）
- 恶意构造的配置文件

**影响**：
- 拒绝服务（解析崩溃）
- 潜在的代码执行（XML 外部实体）

**修复建议**：
- 禁用 XML 外部实体
- 配置文件签名校验

---

## 已有的安全机制

| 机制 | 位置 | 效果 |
|------|------|------|
| 权限校验 | `HasPerfPermission()` | 阻止非授权调用 |
| System App 检查 | `IsSystemAppByFullTokenID()` | 限制 HAP 应用 |
| 权限缓存 | `LRUCache` | 性能优化 |
| ENG_MODE 检查 | `AllowDump()` | 限制调试接口 |
| Stack Protector | `BUILD.gn:83` | 栈溢出防护 |
| ASLR/RELRO | `BUILD.gn:81` | 内存保护 |

**证据**：`services/BUILD.gn:81-83`

```gn
asmflags = [ "-Wl,-z,relro,-z,now" ]
cflags_cc = [ "-fstack-protector-strong" ]
```

---

## 检查范围声明

### 已检查范围

| 范围 | 文件数 |
|------|--------|
| services/server | 2 |
| interfaces/inner_api | 2 |
| services/core | 3 |
| profile/*.xml | 2 |
| sa_profile/*.json | 1 |

### 未检查范围

| 范围 | 原因 |
|------|------|
| test/ 目录 | 测试代码，不属于安全边界 |
| IDL 生成代码 | 由编译工具自动生成 |
| 第三方库 (libxml2, ffrt) | 依赖组件独立审计 |

---

## 安全建议

### 高优先级

1. **权限校验加固**
   - TokenType 失败时默认拒绝
   - 增加审计日志

2. **缓存安全管理**
   - 实现 TTL 机制
   - 监听权限变更

### 中优先级

3. **XML 解析硬化**
   - 禁用外部实体
   - 添加配置文件签名

4. **竞态修复**
   - 关键操作原子化
   - 减少锁粒度

### 低优先级

5. **整数安全**
   - 使用 size_t 类型
   - 添加溢出检查
