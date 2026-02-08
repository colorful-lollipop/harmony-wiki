# SecurityGuard 错误码速查

**文档版本**: 3.1.0  
**最后更新**: 2026-02-06

---

## 快速索引

### 按错误码值快速查找

| 错误码 | 名称 | 常见原因 | 解决方案 |
|--------|------|----------|----------|
| 0 | SUCCESS | - | 无需处理 |
| 1 | FAILED | 通用失败 | 查看日志 |
| 2 | NO_PERMISSION | 权限不足 | 申请权限 |
| 6 | BAD_PARAM | 参数错误 | 检查参数 |
| 7 | JSON_ERR | JSON 解析失败 | 验证 JSON |
| 9 | TIME_OUT | 操作超时 | 重试或检查服务 |
| 10 | NOT_FOUND | 资源未找到 | 检查配置 |
| 201 | JS_ERR_NO_PERMISSION | 权限校验失败 | 动态申请 |
| 401 | JS_ERR_BAD_PARAM | 参数错误 | 检查类型和范围 |
| 801 | JS_ERR_API_SUPPORT | API 不支持 | 检查版本 |
| 1006 | FILTER_EXCEED_LIMIT | 过滤器超限 | 减少数量 |
| 21200001 | JS_ERR_SYS_ERR | 系统错误 | 重启设备 |

---

## JS 层错误码 (JavaScript/TypeScript)

**定义位置**: `frameworks/js/napi/security_guard_napi.cpp:134-141`

### JS_ERR_SUCCESS (0)

**说明**: 操作成功

**处理**: 正常流程，无需特殊处理

---

### JS_ERR_NO_PERMISSION (201)

**说明**: 权限校验失败

**常见原因**:
- 应用未声明所需权限
- 权限未被授予
- APL 等级不足

**解决方案**:
```json
// 1. 在 module.json5 中声明权限
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.QUERY_SECURITY_EVENT"
      }
    ]
  }
}
```

```javascript
// 2. 动态申请权限
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';

async function requestPermission() {
    const atManager = abilityAccessCtrl.createAtManager();
    const bundleInfo = await bundleManager.getBundleInfoForSelf(
        bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );
    atManager.requestPermissionsFromUser(
        bundleInfo.appInfo.accessTokenId,
        ['ohos.permission.QUERY_SECURITY_EVENT']
    );
}
```

**相关权限**:
| API | 所需权限 |
|-----|----------|
| getModelResult | 无 (标准应用可用) |
| querySecurityEvent | `QUERY_SECURITY_EVENT` |
| reportSecurityEvent | `COLLECT_SECURITY_EVENT` |
| updatePolicyFile | `MANAGE_SECURITY_GUARD_CONFIG` |

---

### JS_ERR_NO_SYSTEMCALL (202)

**说明**: 非系统应用使用了系统 API

**原因**: 当前应用不是系统应用

**解决方案**:
1. 使用系统签名重新签名应用
2. 或联系设备厂商获取系统签名

---

### JS_ERR_BAD_PARAM (401)

**说明**: 参数错误

**常见原因**:
- 参数类型错误
- 参数值超出范围
- 缺少必需参数
- JSON 格式错误

**排查步骤**:
```javascript
// 1. 检查参数类型
console.log(typeof params.eventId);  // 应为 number
console.log(typeof params.version);     // 应为 string

// 2. 检查参数范围
if (params.version.length > 50) {
    console.error('version 超出长度限制 (最大50字符)');
}

// 3. 检查 JSON 格式
try {
    JSON.parse(params.content);
} catch (e) {
    console.error('JSON 格式错误:', e.message);
}
```

**参数限制表**:
| 参数 | 最大长度 | 类型 |
|------|----------|------|
| version | 50 | string |
| content | 10240 | string |
| param | 900 | string |
| fileName | 64 | string |
| modelName | 64 | string |

---

### JS_ERR_API_SUPPORT_ERROR (801)

**说明**: API 不支持

**常见原因**:
- 系统版本过低
- 功能被配置关闭
- 设备类型不支持

**解决方案**:
```javascript
// 检查系统版本
import deviceInfo from '@ohos.deviceInfo';

const sdkApiVersion = deviceInfo.sdkApiVersion;
if (sdkApiVersion < 9) {
    console.error('系统版本过低，需要 OpenHarmony 3.1+');
}

// 检查 API 是否存在
if (typeof securityGuard.getModelResult === 'function') {
    await securityGuard.getModelResult('SecurityGuard_JailbreakCheck');
} else {
    console.error('API 不支持');
}
```

---

### JS_ERR_SYS_ERR (21200001)

**说明**: 系统级未知错误

**常见原因**:
- SA 服务异常
- 内存不足
- 系统资源耗尽

**解决方案**:
```bash
# 1. 查看系统日志
hilog | grep -E "ERROR|Error|error"

# 2. 检查内存
cat /proc/meminfo

# 3. 重启设备
reboot
```

---

## 内部错误码 (C++)

**定义位置**: `frameworks/common/constants/include/security_guard_define.h`

### 基础错误码

| 错误码 | 名称 | 说明 | 处理建议 |
|--------|------|------|----------|
| 0 | SUCCESS | 成功 | - |
| 1 | FAILED | 通用失败 | 查看详细日志 |
| 2 | NO_PERMISSION | 无权限 | 检查权限配置 |
| 3 | NO_SYSTEMCALL | 非系统调用 | 确认系统应用 |
| 4 | STREAM_ERROR | 流错误 | 检查文件描述符 |
| 5 | FILE_ERR | 文件错误 | 检查文件路径 |
| 6 | BAD_PARAM | 参数错误 | 验证参数 |
| 7 | JSON_ERR | JSON 解析错误 | 验证 JSON 格式 |
| 8 | NULL_OBJECT | 空对象 | 检查对象初始化 |
| 9 | TIME_OUT | 超时 | 检查网络或重试 |
| 10 | NOT_FOUND | 未找到 | 检查配置存在 |

### 数据库错误码

| 错误码 | 名称 | 说明 | 处理建议 |
|--------|------|------|----------|
| 14 | DB_CHECK_ERR | 数据库检查错误 | 检查数据库完整性 |
| 15 | DB_LOAD_ERR | 数据库加载错误 | 检查存储空间 |
| 16 | DB_OPT_ERR | 数据库操作错误 | 检查 SQL 语句 |
| 17 | DB_INFO_ERR | 数据库信息错误 | 检查数据库配置 |
| 18 | DUPLICATE | 重复数据 | 检查唯一约束 |

### 资源限制错误码

| 错误码 | 名称 | 说明 | 处理建议 |
|--------|------|------|----------|
| 1005 | FILTER_UNSUPPORTED | 过滤器不支持 | 检查过滤器类型 |
| 1006 | FILTER_EXCEED_LIMIT | 过滤器超出限制 | 减少过滤器数量 |
| 1007 | CLIENT_EXCEED_PROCESS_LIMIT | 进程级客户端超限 | 减少客户端数 |
| 1008 | CLIENT_EXCEED_GLOBAL_LIMIT | 全局客户端超限 | 减少全局连接 |

---

## 错误码映射表

### 内部错误码 → JS 错误码

| 内部错误码 | 内部名称 | JS 错误码 | JS 名称 |
|-----------|----------|-----------|----------|
| 0 | SUCCESS | 0 | SUCCESS |
| 2 | NO_PERMISSION | 201 | NO_PERMISSION |
| 3 | NO_SYSTEMCALL | 202 | NO_SYSTEMCALL |
| 6 | BAD_PARAM | 401 | BAD_PARAM |
| 801 | API_SUPPORT_ERROR | 801 | API_SUPPORT_ERROR |
| 其他 | - | 21200001 | SYS_ERR |

**代码位置**: `security_guard_napi.cpp:75-81`

```cpp
static const std::unordered_map<int32_t, std::pair<int32_t, std::string>> g_errorStringMap = {
    { SUCCESS, { JS_ERR_SUCCESS, "The operation was successful" }},
    { NO_PERMISSION, { JS_ERR_NO_PERMISSION, "check permission fail"} },
    { BAD_PARAM, { JS_ERR_BAD_PARAM, "Parameter error..."} },
    { NO_SYSTEMCALL, { JS_ERR_NO_SYSTEMCALL, "non-system application..."} },
    { API_SUPPORT_ERROR, { JS_ERR_API_SUPPORT_ERROR, "API is not supported"} },
};
```

---

## 按场景分类的错误码

### 权限相关错误

| 场景 | 错误码 | 消息 | 解决方案 |
|------|--------|------|----------|
| 权限未声明 | 201 | check permission fail | 添加权限声明 |
| 权限被拒绝 | 201 | check permission fail | 动态申请权限 |
| 非系统应用 | 202 | non-system application | 使用系统签名 |
| APL 不足 | 201 | check permission fail | 提升 APL 等级 |

### 参数相关错误

| 场景 | 错误码 | 消息 | 解决方案 |
|------|--------|------|----------|
| 类型错误 | 401 | Parameter error | 检查 typeof |
| 长度超限 | 401 | Parameter error | 检查字符串长度 |
| JSON 格式错误 | 401 | Parameter error | 验证 JSON |
| 值为空 | 401 | Parameter error | 检查可选参数 |

### 服务相关错误

| 场景 | 错误码 | 消息 | 解决方案 |
|------|--------|------|----------|
| 服务未启动 | 21200001 | Unknown error | 等待服务注册 |
| 调用超时 | 9 | Timeout | 重试或检查服务 |
| 服务崩溃 | 21200001 | Unknown error | 查看日志重启 |

### 配置相关错误

| 场景 | 错误码 | 消息 | 解决方案 |
|------|--------|------|----------|
| 配置不存在 | 10 | Not found | 检查配置路径 |
| 配置解析错误 | 7 | JSON error | 验证 JSON |
| 配置值无效 | 6 | Bad param | 检查配置值 |

### 资源相关错误

| 场景 | 错误码 | 消息 | 解决方案 |
|------|--------|------|----------|
| 过滤器超限 | 1006 | Filter exceed limit | 减少过滤器 |
| 客户端超限 | 1007/1008 | Client exceed limit | 减少连接数 |

---

## 错误处理最佳实践

### 1. 统一错误处理

```javascript
class SecurityGuardError extends Error {
    constructor(code, message, details = {}) {
        super(message);
        this.name = 'SecurityGuardError';
        this.code = code;
        this.details = details;
    }
}

async function safeApiCall(apiCall) {
    try {
        const result = await apiCall();
        return { success: true, data: result };
    } catch (error) {
        return {
            success: false,
            error: {
                code: error.code || 21200001,
                message: error.message || 'Unknown error',
                details: error
            }
        };
    }
}

// 使用
const result = await safeApiCall(() =>
    securityGuard.getModelResult('SecurityGuard_JailbreakCheck')
);

if (!result.success) {
    console.error(`错误码: ${result.error.code}`);
    console.error(`错误信息: ${result.error.message}`);
}
```

### 2. 权限检查

```javascript
async function checkPermission(permission) {
    const atManager = abilityAccessCtrl.createAtManager();
    const bundleInfo = await bundleManager.getBundleInfoForSelf(
        bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );
    const tokenId = bundleInfo.appInfo.accessTokenId;

    return atManager.checkAccessTokenSync(
        tokenId,
        permission
    ) === abilityAccessCtrl.GrantResult.PERMISSION_GRANTED;
}

// 使用
const hasPermission = await checkPermission('ohos.permission.QUERY_SECURITY_EVENT');
if (!hasPermission) {
    // 请求权限或提示用户
}
```

### 3. 参数验证

```javascript
function validateEventInfo(info) {
    const errors = [];

    // 必填字段
    if (typeof info.eventId !== 'number') {
        errors.push('eventId 必须是 number 类型');
    }

    // 版本检查
    if (typeof info.version !== 'string') {
        errors.push('version 必须是 string 类型');
    } else if (info.version.length > 50) {
        errors.push(`version 超出长度限制: ${info.version.length}/50`);
    }

    // 内容检查
    if (typeof info.content !== 'string') {
        errors.push('content 必须是 string 类型');
    } else if (info.content.length > 10240) {
        errors.push(`content 超出长度限制: ${info.content.length}/10240`);
    }

    return {
        valid: errors.length === 0,
        errors
    };
}

// 使用
const validation = validateEventInfo(eventInfo);
if (!validation.valid) {
    console.error('参数错误:', validation.errors);
    return;
}
```

---

## 日志关键词

根据错误码查找相关日志：

| 错误码 | 日志标签 | 日志关键词 |
|--------|----------|------------|
| 201 | SG_Service | "permission", "PERMISSION" |
| 401 | SG_Service | "param", "BAD_PARAM", "Parameter" |
| 801 | SG_Service | "support", "SUPPORT" |
| 1001 | SG_Service | "dlopen", "LOAD" |
| 1006 | SG_Service | "filter", "FILTER", "limit" |
| 数据库错误 | SG_Service | "DB", "database", "SQL" |
| 权限相关 | S_COLLCTOR | "permission", "ACCESS" |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [API 参考](../../02_NAPI_Reference.md) | 完整 API 文档 |
| [调试指南](../../08_Debugging.md) | 日志与排错 |
| [术语表](./99_Glossary.md) | 术语定义 |
| [架构详解](../../03_Architecture.md) | 系统架构 |
