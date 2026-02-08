# 安全风险评审

## 7.1 攻击面分析

### 7.1.1 外部输入点

| 输入点 | 类型 | 描述 | 代码证据 |
|--------|------|------|----------|
| **DataShare API** | IPC | 外部应用通过 URI 调用 CRUD 操作 | `common/include/telephony_datashare_stub_impl.h` |
| **JSON 配置文件** | 文件 | 系统预置的 APN、紧急号码等配置 | `etc/*.json` |
| **APN 敏感字段** | 用户输入 | PDP 配置中的用户名/密码 | `pdp_profile/include/apn_encryption_util.h` |
| **查询谓词** | IPC | DataSharePredicates 过滤条件 | `04_DataShare_API.md` |

### 7.1.2 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                     不可信区域                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           第三方应用 (调用 DataShare API)             │  │
│  └───────────────────────────────────────────────────────┘  │
│                              ▲                               │
│                              │ IPC                           │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │         信任边界: SystemAbilityManager                │  │
│  │         - 验证调用方身份                              │  │
│  │         - 验证权限                                    │  │
│  └───────────────────────────────────────────────────────┘  │
│                              ▲                               │
│                              │ 认证通过                       │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           可信区域: Telephony Data Storage            │  │
│  │         - RDB 数据库操作                             │  │
│  │         - APN 加密                                   │  │
│  │         - 配置文件读取                               │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 7.2 安全机制

### 7.2.1 权限检查机制

**代码证据**: `common/src/permission_util.cpp`

```cpp
bool PermissionUtil::CheckPermission(const std::string &permissionName) {
    // 1. 获取调用方 Token ID
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    
    // 2. 验证权限
    int result = AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    
    // 3. 返回结果
    return result == PermissionState::PERMISSION_GRANTED;
}
```

| 权限 | 保护级别 | 用途 |
|------|----------|------|
| `ohos.permission.GET_TELEPHONY_STATE` | system_basic | 查询电话数据 |
| `ohos.permission.SET_TELEPHONY_STATE` | system_basic | 修改电话数据 |
| `ohos.permission.READ_MESSAGES` | system_basic | 读取短信 |

### 7.2.2 APN 加密机制

**代码证据**: `pdp_profile/include/apn_encryption_util.h`

```cpp
class ApnEncryptionUtil {
public:
    static std::string EncryptApnData(const std::string &rawData);
    static std::string DecryptApnData(const std::string &encryptedData);
};
```

| 加密字段 | 用途 | 保护措施 |
|----------|------|----------|
| USER_NAME | APN 用户名 | 加密存储 |
| PASSWORD | APN 密码 | 加密存储 |

---

## 7.3 安全风险清单

### 风险 1: URI 注入攻击

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-001 |
| **风险名称** | URI 注入攻击 |
| **风险等级** | 中 |
| **证据** | `telephony_datashare_stub_impl.cpp` 中 `GetOwner()` 方法 |

**问题描述**:
`TelephonyDataShareStubImpl::GetOwner()` 根据 URI 路径判断数据归属。攻击者可能构造特殊 URI 访问未授权数据。

**触发条件**:
```cpp
// 恶意 URI 示例
Uri malicious("datashare:///com.ohos.simability/../pdp_profile");
```

**影响**:
- 可能绕过权限检查访问其他模块数据

**当前防护**:
- URI 解析基于路径前缀匹配
- 权限检查独立于 URI 路由

**修复建议**:
1. 对 URI 进行规范化处理（移除 `..` 等特殊字符）
2. 使用白名单机制验证 URI 前缀
3. 在 Stub 层添加额外的权限校验

**代码证据**: `common/src/telephony_datashare_stub_impl.cpp`

---

### 风险 2: SQL 注入漏洞

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-002 |
| **风险名称** | SQL 注入攻击 |
| **风险等级** | 高 |
| **证据** | `rdb_base_helper.cpp` 中 RDB 操作使用 |

**问题描述**:
`DataSharePredicates` 中的条件可能被拼接成 SQL 语句，存在 SQL 注入风险。

**触发条件**:
```cpp
// 恶意谓词示例
DataSharePredicates predicates;
predicates.EqualTo("name", "test'; DROP TABLE sim; --");
```

**影响**:
- 非法访问、篡改或删除数据库数据
- 可能导致服务拒绝

**当前防护**:
- RDB 使用参数化查询
- 输入验证在框架层处理

**修复建议**:
1. 在 `RdbPredicates` 构建时对用户输入进行转义
2. 添加 SQL 关键字黑名单检查
3. 使用白名单验证列名

**代码证据**: `common/include/rdb_base_helper.h`

---

### 风险 3: 路径遍历攻击

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-003 |
| **风险名称** | 路径遍历攻击 |
| **风险等级** | 中 |
| **证据** | `parser_util.cpp` 中 JSON 文件读取 |

**问题描述**:
JSON 配置文件读取时，路径可能包含 `../` 进行目录遍历。

**触发条件**:
```cpp
// 恶意配置文件路径
parserUtil.ParseJson("../etc/passwd");
```

**影响**:
- 读取系统敏感文件
- 泄露系统信息

**当前防护**:
- 配置文件路径硬编码
- 使用相对路径

**修复建议**:
1. 对所有文件路径进行规范化处理
2. 使用白名单验证文件路径
3. 在 chroot 环境中读取配置文件

**代码证据**: `common/src/parser_util.cpp`

---

### 风险 4: APN 加密强度不足

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-004 |
| **风险名称** | APN 加密算法可能过时 |
| **风险等级** | 中 |
| **证据** | `apn_encryption_util.cpp` 中加密实现 |

**问题描述**:
APN 敏感数据（用户名、密码）使用加密存储，但加密算法强度未知，可能存在已知漏洞。

**触发条件**:
- 获取加密后的 APN 数据
- 进行密码分析攻击

**影响**:
- 泄露网络认证凭据
- 造成财务损失（漫游费用）

**当前防护**:
- 使用加密存储而非明文
- 密钥由系统管理

**修复建议**:
1. 评估当前加密算法强度（建议 AES-256）
2. 使用硬件安全模块（SE/TEE）存储密钥
3. 定期轮换加密密钥
4. 添加消息认证码（MAC）防篡改

**代码证据**: `pdp_profile/src/apn_encryption_util.cpp`

---

### 风险 5: 权限升级攻击

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-005 |
| **风险名称** | 权限检查绕过 |
| **风险等级** | 高 |
| **证据** | `permission_util.cpp` 权限验证逻辑 |

**问题描述**:
`PermissionUtil::CheckPermission()` 可能存在时序问题或竞态条件，导致权限检查结果不可靠。

**触发条件**:
```cpp
// 时序攻击示例
auto callerToken = IPCSkeleton::GetCallingTokenID();
// 中间可能有其他线程修改 Token
int result = AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
```

**影响**:
- 未授权应用获取敏感数据
- 篡改电话相关配置

**当前防护**:
- 使用系统级 AccessTokenKit 验证
- TokenID 获取在权限检查前

**修复建议**:
1. 在单次操作中完成 Token 获取和验证
2. 添加操作审计日志
3. 使用原子操作保护关键步骤

**代码证据**: `common/src/permission_util.cpp`

---

### 风险 6: 信息泄露

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-006 |
| **风险名称** | 敏感信息日志泄露 |
| **风险等级** | 低 |
| **证据** | `data_storage_log_wrapper.cpp` 日志实现 |

**问题描述**:
调试日志可能输出敏感信息（如电话号码、APN 密码）。

**触发条件**:
```cpp
// 敏感信息日志示例
DATA_STORAGE_LOGD("APN password: %{public}s", password.c_str());
```

**影响**:
- 泄露用户隐私数据
- 泄露系统配置信息

**当前防护**:
- 日志级别控制（DEBUG/INFO/WARN/ERROR）
- 敏感字段使用 `%{private}` 格式

**修复建议**:
1. 强制敏感字段使用隐私格式输出
2. 在生产版本禁用 DEBUG 日志
3. 添加日志审计机制

**代码证据**: `common/src/data_storage_log_wrapper.cpp`

---

### 风险 7: 拒绝服务攻击

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-007 |
| **风险名称** | 资源耗尽 DoS |
| **风险等级** | 中 |
| **证据** | `sms_mms_ability.cpp` 批量操作实现 |

**问题描述**:
`BatchInsert()` 或大量数据查询可能导致内存耗尽或数据库锁死。

**触发条件**:
```cpp
// 大量数据插入
std::vector<DataShareValuesBucket> values(10000);
helper->BatchInsert(uri, values);
```

**影响**:
- 服务响应变慢
- 系统资源耗尽
- 其他操作阻塞

**当前防护**:
- 无明确限制

**修复建议**:
1. 限制单次批量操作的最大数量
2. 添加超时机制
3. 使用流式处理代替全量加载
4. 监控资源使用并触发告警

**代码证据**: `sms_mms/src/sms_mms_ability.cpp`

---

## 7.4 安全加固建议

### 7.4.1 短期加固 (高优先级)

| 建议 | 优先级 | 预计工时 |
|------|--------|----------|
| 修复 URI 注入风险 (SEC-001) | 高 | 1-2 天 |
| 增强 SQL 注入防护 (SEC-002) | 高 | 2-3 天 |
| 添加批量操作限制 (SEC-007) | 高 | 1 天 |

### 7.4.2 中期加固 (中优先级)

| 建议 | 优先级 | 预计工时 |
|------|--------|----------|
| 升级 APN 加密算法 (SEC-004) | 中 | 1 周 |
| 添加路径遍历防护 (SEC-003) | 中 | 2-3 天 |
| 完善日志隐私保护 (SEC-006) | 中 | 1-2 天 |

### 7.4.3 长期加固 (低优先级)

| 建议 | 优先级 | 预计工时 |
|------|--------|----------|
| 权限检查原子化 (SEC-005) | 低 | 2-3 天 |
| 集成硬件安全模块 | 低 | 2-4 周 |
| 添加安全审计日志 | 低 | 1 周 |

---

## 7.5 安全检查清单

### 部署前检查

- [ ] 所有 DataShare API 调用都有权限检查
- [ ] APN 敏感数据使用加密存储
- [ ] JSON 配置文件路径经过验证
- [ ] 日志不包含敏感信息
- [ ] 批量操作有限制和超时

### 运行时监控

- [ ] 监控异常权限访问尝试
- [ ] 监控大量数据查询模式
- [ ] 监控数据库操作错误
- [ ] 监控配置文件加载异常

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [01_Overview](01_Overview.md) |
| 架构说明 | [03_Architecture](03_Architecture.md) |
| 对外 API | [04_DataShare_API](04_DataShare_API.md) |

---

*最后更新: 2024-02-06*
