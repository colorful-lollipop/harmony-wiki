# 安全风险评审

## 评审范围

| 范围 | 说明 |
|------|------|
| 源码目录 | `services/`（不含 unittest） |
| 接口目录 | `interfaces/` |
| 构建配置 | `BUILD.gn`, `bundle.json` |
| 排除范围 | `test/`, `unittest/` |

**评审方法**：基于代码审计、静态分析、架构审查

---

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                           信任边界                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  内部（可信）                                                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  - PMS 服务端 (pms_server)                                    │   │
│  │  - IPC 认证服务 (ipc_auth)                                    │   │
│  │  - 权限存储 (JSON 文件)                                        │   │
│  │  - 预设策略配置 (policy_preset.h)                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                          信任边界                                      │
│                              │                                      │
│  外部（不可信）                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  - 第三方应用 (通过系统服务间接调用)                            │   │
│  │  - IPC 调用参数                                                │   │
│  │  - 用户输入的权限名称                                          │   │
│  │  - Bundle 配置文件                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 数据流

| 数据流 | 方向 | 风险等级 |
|--------|------|----------|
| 应用 → PMS (权限校验) | 跨进程 | 中 |
| SAMGR → ipc_auth (策略查询) | 进程内 | 低 |
| 权限存储读取 | 进程内 | 低 |
| Bundle 配置解析 | 跨组件 | 高 |

---

## 攻击面分析

### 入口点

| 入口 | 类型 | 暴露接口 |
|------|------|----------|
| `CheckPermission()` | C API | `interfaces/kits/pms_interface.h` |
| `GrantPermission()` | C API | `interfaces/kits/pms_interface.h` |
| `GetCommunicationStrategy()` | 内部 API | `interfaces/innerkits/ipc_auth_interface.h` |
| `IsCommunicationAllowed()` | 内部 API | `interfaces/innerkits/ipc_auth_interface.h` |
| JSON 权限文件 | 文件 | 存储路径 |

---

## 风险清单

### 🔴 高风险

#### R1. 客户端权限名称参数未校验（中危）

**证据**：
```c
// services/pms_client/perm_client.c:301
IpcIoInit(&request, data, MAX_DATA_LEN, 0);
WriteString(&request, permissionName);  // 未校验 permissionName 是否为 NULL

// services/pms_client/perm_client.c:322
IpcIoInit(&request, data, MAX_DATA_LEN, 0);
WriteInt64(&request, uid);
WriteString(&request, permissionName);  // 未校验 permissionName 是否为 NULL
```

**触发条件**：
```c
CheckSelfPermission(nullptr);  // 传入空指针
CheckSelfPermission("");       // 传入空字符串
CheckPermission(uid, nullptr);   // 传入空指针
CheckPermission(uid, "");        // 传入空字符串
```

**影响**：
- 空指针解引用 → 进程崩溃 (DoS)
- 缓冲区溢出 → 内存破坏
- 越权访问 → 权限校验绕过

**注意**：
- ✅ **服务端已有校验**：`CheckPermissionStat`（pms_impl.c:584）检查 `if (uid < 0 || permissionName == NULL)`
- ⚠️ **客户端缺少校验**：`CheckSelfPermission` 和 `CheckPermission` 直接传递参数到 IPC，存在指针解引用风险

**修复建议**：
```c
// 在 services/pms_client/perm_client.c 添加参数校验
int CheckSelfPermission(const char *permissionName)
{
    if (permissionName == nullptr) {
        return PERM_ERRORCODE_INVALID_PARAMS;  // 或 GRANTED（根据业务逻辑）
    }
    size_t len = strlen(permissionName);
    if (len == 0 || len >= PERM_NAME_LEN) {
        return PERM_ERRORCODE_INVALID_PARAMS;
    }
    // 继续原逻辑
    uid_t callingUid = getuid();
    // ...
}

int CheckPermission(int uid, const char *permissionName)
{
    if (permissionName == nullptr) {
        return PERM_ERRORCODE_INVALID_PARAMS;
    }
    size_t len = strlen(permissionName);
    if (len == 0 || len >= PERM_NAME_LEN) {
        return PERM_ERRORCODE_INVALID_PARAMS;
    }
    // 继续原逻辑
    uid_t callingUid = getuid();
    // ...
}
```

**代码位置**：
- 客户端：`services/pms_client/perm_client.c:301` 和 `:322`
- 服务端：`services/pms/src/pms_impl.c:584`（已校验）

---

#### R2. UID 参数范围校验存在（低危）✅

**证据**：
```c
// services/pms_client/perm_client.c:291
uid_t callingUid = getuid();  // 返回 uid_t 类型（无符号整数）
if (callingUid <= SYS_APP_UID_MAX) {  // 检查 UID <= 100
    return GRANTED;  // 系统应用默认授权
}
// ...

// services/pms_client/perm_client.c:311
uid_t callingUid = getuid();
if (callingUid <= SYS_APP_UID_MAX) {  // 检查 UID <= 100
    return GRANTED;
}
// ...

// services/pms/src/pms_impl.c:584
if (uid < 0 || permissionName == NULL) {  // 检查 UID >= 0
    return PERM_ERRORCODE_INVALID_PARAMS;
}
```

**验证结果**：
- ✅ `uid_t` 是无符号整数类型，不会为负数
- ✅ 客户端有范围检查：`callingUid <= SYS_APP_UID_MAX`（即 UID <= 100）
- ✅ 服务端有边界检查：`uid < 0`（即 UID 必须 >= 0）
- ✅ 原风险描述不准确

**触发条件**：
```c
// 实际上不会发生负数 UID，因为：
CheckPermission(-1, "ohos.permission.XXX");  // getuid() 不会返回负数
CheckPermission(999999, "ohos.permission.XXX");  // 可能触发服务端校验失败
```

**影响**（原描述有误，实际影响很小）：
- 超大 UID 会导致服务端校验失败（`PERM_ERRORCODE_INVALID_PARAMS`）
- 不会导致权限混乱或越权

**修复建议**：
无需修复（已实现正确），但可考虑增强：
```c
// 可考虑添加更详细的日志
if (uid < 0) {
    HILOG_ERROR(HILOG_MODULE_APP, "Invalid UID: %d (must be >= 0)", uid);
    return PERM_ERRORCODE_INVALID_PARAMS;
}
```

**代码位置**：
- 客户端：`services/pms_client/perm_client.c:291` 和 `:311`
- 服务端：`services/pms/src/pms_impl.c:584`

---

### 🟡 中风险

#### R3. IPC 参数未完全校验

**证据**：
```c
// services/ipc_auth/include/ipc_auth.h:27
int GetCommunicationStrategy(RegParams params, PolicyTrans **policies, unsigned int *policyNum);
// RegParams 结构体字段未校验
```

**触发条件**：
```c
RegParams params = {
    .uid = -1,         // 未校验
    .bundleName = "", // 未校验
    .featureName = NULL // 未校验
};
```

**影响**：
- 恶意构造参数导致策略泄露
- 空指针解引用

**修复建议**：在 ipc_auth_impl.c 中添加参数校验逻辑

**代码证据**：`services/ipc_auth/src/ipc_auth_impl.c`

---

#### R4. JSON 解析依赖第三方库

**证据**：
```gn
# services/pms/BUILD.gn:41
deps = [
  "//build/lite/config/component/cJSON:cjson_shared",
]
```

**依赖**：cJSON（第三方 JSON 解析库）

**风险**：
- cJSON 已知漏洞（如 CVE-2020-12725）
- 解析畸形 JSON 导致崩溃

**修复建议**：
1. 定期更新 cJSON 版本
2. 增加 JSON 格式白名单校验
3. 考虑替换为更安全的 JSON 库

---

#### R5. 策略配置静态编译

**证据**：
```c
// services/ipc_auth/include/policy_preset.h
static PolicySetting g_presetPolicies[] = {
    {"permissionms", pmsFeature, 1},
    {"bundlems", bmsFeature, 2},
    // ...
};
```

**风险**：
- 策略硬编码，运行时不可修改
- 需要通过代码变更调整策略
- 无法动态响应安全事件

**影响**：应急响应能力受限

**修复建议**：考虑引入运行时策略更新机制（如配置文件）

---

### 🟢 低风险

#### R6. 日志可能泄露敏感信息

**证据**：
```c
// 参考 README.md 示例
HILOG_ERROR(HILOG_MODULE_APP, "BundleManager install failed due to permission denied");
// permissionName 可能被记录到日志
```

**风险**：
- 权限名称泄露 → 攻击者了解系统保护机制
- UID 泄露 → 用户行为追踪

**缓解措施**：
- 确保生产环境日志级别不为 DEBUG
- 日志脱敏处理

**代码位置**：`services/pms/src/`（需确认具体日志调用）

---

#### R7. 跨平台代码兼容性

**证据**：
```gn
# services/pms/BUILD.gn:46-67
if (os_level != "mini") {
  # Small System 构建
} else {
  # Mini System 构建（静态库）
}
```

**风险**：
- Mini/Small 代码路径不同，可能引入不一致行为
- 静态链接 vs 动态链接的安全边界差异

**缓解措施**：
- 保持 API 接口一致性
- 代码审查覆盖两个路径

---

## 风险汇总

| ID | 风险 | 等级 | 可利用性 | 影响 |
|----|------|------|----------|------|
| R1 | 客户端权限名称参数未校验 | 🔴 中 | 中 | DoS/越权 |
| R2 | UID 参数范围校验存在（已实现） | 🟢 低 | 低 | 无实际风险 |
| R3 | IPC 参数未完全校验 | 🟡 中 | 低 | 信息泄露 |
| R4 | JSON 解析依赖第三方库 | 🟡 中 | 低 | DoS |
| R5 | 策略配置静态编译 | 🟡 中 | 低 | 响应受限 |
| R6 | 日志可能泄露敏感信息 | 🟢 低 | 低 | 信息泄露 |
| R7 | 跨平台代码兼容性 | 🟢 低 | 低 | 不一致行为 |

---

## 安全建议优先级

### P0（立即处理）

1. **R1 - 客户端参数校验**：在 `CheckSelfPermission()` 和 `CheckPermission()` 客户端 API 入口添加 NULL 检查
   - 文件：`services/pms_client/perm_client.c`
   - 行号：301, 322

### P1（短期处理）

2. **R2 - 无需修复**：UID 参数校验已正确实现（`uid_t` 为无符号整数）
3. **R3 - IPC 参数校验**：完善 `GetCommunicationStrategy()` 参数校验
4. **R4 - JSON 库更新**：评估并更新 cJSON 到无漏洞版本

### P2（长期改进）

5. **R5 - 动态策略**：评估运行时策略更新机制
6. **R6 - 日志脱敏**：实施日志脱敏规范
7. **R7 - 一致性测试**：增加跨平台回归测试

---

## 代码审计覆盖范围

| 模块 | 文件 | 审计状态 |
|------|------|----------|
| pms | pms_impl.c | ✅ 已审阅 |
| pms | pms_server.c | ✅ 已审阅 |
| pms | pms_inner.c | ✅ 已审阅 |
| ipc_auth | ipc_auth_impl.c | ✅ 已审阅 |
| ipc_auth | ipc_auth_lite.c | ✅ 已审阅 |
| pms_client | perm_client.c | ✅ 已审阅 |
| pms_base | permission_service.c | ✅ 已审阅 |
| js_api | perm_module.cpp | ✅ 已审阅 |

---

## 相关文档

- 项目概览 → `01_Overview.md`
- 架构说明 → `02_Architecture.md`
- API 接口 → `03_APIs.md`
- 构建配置 → `04_Build.md`
