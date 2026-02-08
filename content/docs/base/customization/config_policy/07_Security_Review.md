# 安全风险评审

## 评审范围

本评审覆盖 config_policy 组件的以下代码：

| 模块 | 路径 | 评审状态 |
|------|------|----------|
| C++ 核心实现 | `frameworks/config_policy/src/config_policy_utils.c` | ✅ 已评审 |
| N-API 绑定 | `interfaces/kits/js/src/config_policy_napi.cpp` | ✅ 已评审 |
| 内部头文件 | `interfaces/inner_api/include/` | ✅ 已评审 |

**未评审范围**：测试代码、模糊测试代码

---

## 攻击面分析

### 输入源

| 输入类型 | 来源 | 风险等级 |
|----------|------|----------|
| JS API 参数（relPath） | 上层应用 | 中 |
| JS API 参数（followMode） | 上层应用 | 低 |
| JS API 参数（extra） | 上层应用 | 中 |
| 系统参数（SystemGetParameter） | init 模块 | 低 |
| 文件系统（access） | VFS | 低 |

### 敏感操作

| 操作 | 描述 | 风险等级 |
|------|------|----------|
| 路径拼接 | 配置文件路径构建 | 中 |
| 文件访问检查 | access() 系统调用 | 低 |
| 系统参数读取 | SystemGetParameter() | 低 |
| 内存分配 | malloc/calloc | 低 |

---

## 可利用点分析

### 1. 路径遍历风险（中等风险）

**证据代码**：`config_policy_utils.c:463-467`

```c
if (snprintf_s(buf, bufLength, bufLength - 1, "%s/%s", dirs->paths[i - 1], pathSuffix) > 0 &&
    access(buf, F_OK) == 0) {
    break;
}
```

**触发条件**：
1. 攻击者控制 `pathSuffix` 参数（如 JS API 的 `relPath`）
2. `pathSuffix` 包含 `../` 等路径遍历序列

**影响**：
- 可能访问预期外的配置文件
- 可能泄露系统目录结构信息

**修复建议**：
1. 在 N-API 层添加路径遍历字符检测
2. 对 `relPath` 参数进行规范化处理
3. 限制可访问的路径前缀

**当前防护措施**：
- 使用 `snprintf_s` 防止缓冲区溢出
- 仅用于配置文件查询，权限受限

---

### 2. 空指针解引用风险（低风险）

**证据代码**：`config_policy_utils.c:440-443`

```c
CfgDir *dirs = GetCfgDirList();
if (dirs == NULL) {
    return NULL;
}
```

**触发条件**：
1. `GetCfgDirList()` 返回 NULL
2. 调用者未检查返回值

**影响**：
- 进程崩溃（空指针解引用）

**当前防护措施**：
- API 文档要求调用者检查返回值
- 所有公共 API 内部都有 NULL 检查

**修复建议**：
- 考虑返回空数组而非 NULL（但可能破坏现有 API 兼容性）

---

### 3. 缓冲区大小验证不严格（中风险）

**证据代码**：`config_policy_utils.c:434-438`

```c
char *GetOneCfgFileEx(const char *pathSuffix, char *buf, unsigned int bufLength, int followMode, const char *extra)
{
    if (pathSuffix == NULL || buf == NULL || bufLength < MAX_PATH_LEN) {
        return NULL;
    }
```

**触发条件**：
1. 传入的 `bufLength` 小于 `MAX_PATH_LEN`（256）
2. 实际路径长度超过传入缓冲区大小

**影响**：
- 静默失败，返回 NULL
- 可能导致应用逻辑错误

**当前防护措施**：
- 检查 `bufLength < MAX_PATH_LEN`
- 文档建议使用 `MAX_PATH_LEN` 缓冲区

**修复建议**：
- 返回实际需要的长度，让调用者分配足够空间
- 或截断过长的路径

---

### 4. 字符串截断风险（低风险）

**证据代码**：`config_policy_utils.c:140-145`

```c
size_t bufSize = strlen(opKeyDir) + strlen(opKeyValue) + 1;
bufSize = Min(bufSize, MAX_PATH_LEN);
result = (char *)calloc(bufSize, sizeof(char));
if (result != NULL && sprintf_s(result, bufSize, "%s%s", opKeyDir, opKeyValue) > 0) {
```

**触发条件**：
1. `opKeyValue` 超长
2. `sprintf_s` 被截断

**影响**：
- 路径被截断，可能导致找不到配置文件

**当前防护措施**：
- `sprintf_s` 返回截断后的长度
- MIN 运算限制最大长度

---

### 5. 整数溢出风险（低风险）

**证据代码**：`config_policy_utils.c:107-111`

```c
unsigned int len = 0;
if (SystemGetParameter(name, NULL, &len) != 0 || len <= 0 || len > PARAM_CONST_VALUE_LEN_MAX) {
    return NULL;
}
value = (char *)calloc(len, sizeof(char));
```

**触发条件**：
1. `SystemGetParameter` 返回异常 len 值
2. `len` 为 0 或超过 `PARAM_CONST_VALUE_LEN_MAX`

**影响**：
- 拒绝服务（返回 NULL）

**当前防护措施**：
- 有 `len <= 0` 和 `len > PARAM_CONST_VALUE_LEN_MAX` 检查
- 拒绝异常的参数值

---

### 6. 内存泄漏风险（低风险）

**证据代码**：`config_policy_utils.c:489-493`

```c
CfgFiles *files = (CfgFiles *)(malloc(sizeof(CfgFiles)));
if (files == NULL) {
    FreeCfgDirList(dirs);  // 注意：释放了 dirs 但没有返回
    return NULL;
}
```

**触发条件**：
1. `malloc` 失败返回 NULL
2. 代码释放了 `dirs` 但直接返回
3. 如果 `malloc` 成功但后续失败，可能泄漏 `files`

**影响**：
- 内存泄漏

**当前防护措施**：
- NULL 检查后释放 `dirs`
- 但如果 `files` 分配成功后的路径有泄漏风险

**修复建议**：
- 添加 goto 模式的错误处理，统一释放资源

---

### 7. FollowX 规则注入风险（低风险）

**证据代码**：`config_policy_utils.c:231`

```c
char *result = (addPath && *addPath) ? strdup(addPath) : NULL;
```

**触发条件**：
1. FollowX 规则中包含恶意路径
2. 规则通过 `CUST_FOLLOW_X_RULES` 系统参数注入

**影响**：
- 可能访问非预期的配置文件

**当前防护措施**：
- FollowX 规则存储在系统参数中，需系统权限才能修改
- 规则解析在系统进程中进行

**修复建议**：
- 添加规则白名单验证
- 限制 FollowX 可访问的路径范围

---

## 安全最佳实践

### 1. 参数验证

所有 N-API 接口都已添加参数类型检查：

```cpp
// 类型检查
bool matchFlag = MatchValueType(env, args, napi_string);
if (!matchFlag) {
    return ThrowNapiError(env, PARAM_ERROR, "Parameter error. The type of relPath must be string.");
}
```

**代码证据**: `config_policy_napi.cpp:466-469`

### 2. 错误码统一

使用统一的错误码体系：

```cpp
static constexpr int32_t PARAM_ERROR = 401;
```

**代码证据**: `config_policy_napi.cpp:46`

### 3. 安全字符串函数

使用安全变体函数：

| 危险函数 | 安全替代 | 代码证据 |
|----------|----------|----------|
| `sprintf` | `snprintf_s` / `sprintf_s` | `config_policy_utils.c:142` |
| `strcpy` | `memcpy_s` / `strcat_s` | `config_policy_utils.c:64` |
| `strtok` | `strtok_s` | `config_policy_utils.c:166` |

### 4. 内存安全

使用 `calloc` 分配并初始化内存：

```c
value = (char *)calloc(len, sizeof(char));
```

**代码证据**: `config_policy_utils.c:110`

---

## DFX 监控

### HiSysEvent 配置

组件定义了以下监控事件：

```yaml
domain: CUST_CONFIG

CONFIG_POLICY_FAILED:
  __BASE: {type: FAULT, level: CRITICAL, tag: cfgFail}
  APINAME: {type: STRING}
  MSG: {type: STRING}

CONFIG_POLICY_EVENT:
  __BASE: {type: STATISTIC, level: MINOR, tag: cfgEvent}
  APINAME: {type: STRING}
```

**代码证据**: `frameworks/dfx/hisysevent.yaml:14-28`

### 事件上报

```cpp
ReportConfigPolicyEvent(ReportType::CONFIG_POLICY_FAILED, "getCfgDirList", "CfgDirList is nullptr.");
```

**代码证据**: `config_policy_napi.c:384`

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      不可信区域                              │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ 上层应用 JS/ArkTS 代码                               │  │
│  │ - relPath 参数                                      │  │
│  │ - followMode 参数                                   │  │
│  │ - extra 参数                                        │  │
│  └─────────────────────────────────────────────────────┘  │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ N-API 边界（参数校验层）                            │  │
│  │ - 类型检查                                           │  │
│  │ - 范围验证                                           │  │
│  │ - 错误码统一                                         │  │
│  └─────────────────────────────────────────────────────┘  │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ 可信区域                                             │  │
│  │ - C++ 核心逻辑                                       │  │
│  │ - 系统参数读取（init 模块）                          │  │
│  │ - 文件系统访问                                       │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 修复建议优先级

| 优先级 | 问题 | 影响 | 建议 |
|--------|------|------|------|
| P1 | 路径遍历风险 | 中 | N-API 层添加路径规范化验证 |
| P2 | 缓冲区大小验证 | 中 | 返回实际需要长度 |
| P3 | 内存泄漏 | 低 | 使用 goto 错误处理模式 |
| P4 | FollowX 规则注入 | 低 | 添加规则白名单 |

---

## 相关文档

- [N-API 接口文档](./04_N-API_Reference.md)
- [C++ 内部 API](./05_Cpp_Inner_API.md)
- [GN 构建配置](./06_GN_Build.md)
