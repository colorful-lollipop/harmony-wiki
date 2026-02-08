# 安全风险评估

## 1. 评估概述

- **评估对象**: OpenHarmony i18n 模块 (base/global/i18n)
- **评估范围**: N-API 接口、IPC 接口、系统服务
- **评估方法**: 静态代码分析、威胁建模、攻击面分析
- **评估日期**: 2026-02-07

---

## 2. 风险分级标准

| 等级 | CVSS 范围 | 定义 | 处理优先级 |
|------|-----------|------|-----------|
| **高危** | 7.0-10.0 | 可导致权限提升、数据泄露、系统崩溃 | 立即修复 |
| **中危** | 4.0-6.9 | 可导致 DoS、有限数据泄露 | 尽快修复 |
| **低危** | 0.1-3.9 | 轻微影响，难以利用 | 计划修复 |
| **信息** | 0.0 | 非安全问题，改进建议 | 参考 |

---

## 3. 详细风险分析

### R1: 数组长度未限制导致 DoS (中危)

**位置**: `interfaces/js/innerkits/intl/src/js_utils.cpp:305-311`

**证据代码**:
```cpp
// js_utils.cpp:305-311
std::vector<std::string> JSUtils::GetStringArray(napi_env env, napi_value value, int32_t& code)
{
    uint32_t arrayLength = 0;
    napi_status status = napi_get_array_length(env, value, &arrayLength);  // [1] 获取数组长度
    // ...
    std::vector<std::string> result;
    result.resize(arrayLength);  // [2] 直接 resize，无长度检查!
```

**触发路径**:
```
JS: supportedLocalesOf([...超大数组...])
  ↓
intl_addon.cpp: GetParameterLocales()
  ↓
js_utils.cpp: GetLocaleArray() → GetStringArray()
  ↓
result.resize(超大值) → OOM → 进程崩溃
```

**攻击场景**:
1. 攻击者构造包含 2^32-1 个元素的数组
2. 传入 `supportedLocalesOf()` 或类似 API
3. `resize()` 尝试分配数 GB 内存
4. 内存分配失败或系统 OOM

**影响评估**:
- **可利用性**: 容易 (只需构造大数组)
- **影响范围**: 当前应用进程
- **危害**: DoS (应用崩溃)

**修复建议**:
```cpp
constexpr uint32_t MAX_ARRAY_LENGTH = 1024;  // 合理限制
if (arrayLength > MAX_ARRAY_LENGTH) {
    HILOG_ERROR_I18N("GetStringArray: Array length %u exceeds limit %u", 
                     arrayLength, MAX_ARRAY_LENGTH);
    code = 1;
    return {};
}
result.resize(arrayLength);
```

**缓解状态**: ❌ 未修复

---

### R2: 路径遍历风险 (中危)

**位置**: `interfaces/js/kits/src/i18n_addon.cpp` (需确认确切位置)

**证据**: 存在 `GetUnicodeWrappedFilePath` 函数处理用户输入路径

**潜在问题**:
```cpp
// 假设实现 (需确认)
std::string GetUnicodeWrappedFilePath(const std::string& path) {
    // 如果直接使用 path 拼接，可能存在路径遍历
    return BASE_PATH + path;  // 危险: 未检查 ../
}
```

**触发路径**:
```
JS: i18n.getUnicodeWrappedFilePath("../../../etc/passwd")
  ↓
i18n_addon.cpp: GetUnicodeWrappedFilePath()
  ↓
访问非预期文件 → 信息泄露
```

**影响评估**:
- **可利用性**: 中等 (需确认函数实际用途)
- **影响范围**: 文件系统访问
- **危害**: 信息泄露

**修复建议**:
```cpp
std::string GetUnicodeWrappedFilePath(const std::string& path) {
    // 1. 规范化路径
    std::string normalized = NormalizePath(path);
    
    // 2. 检查路径遍历
    if (normalized.find("..") != std::string::npos) {
        return "";
    }
    
    // 3. 验证在预期目录内
    std::string fullPath = BASE_PATH + normalized;
    if (!IsPathUnderBase(fullPath, BASE_PATH)) {
        return "";
    }
    
    return fullPath;
}
```

**缓解状态**: ⚠️ 待确认

---

### R3: 权限检查实现正确 (已缓解)

**位置**: `services/src/i18n_service_ability.cpp:634-669`

**证据代码**:
```cpp
// services/src/i18n_service_ability.cpp:634-669
I18nErrorCode I18nServiceAbility::CheckPermission()
{
    // 1. 检查系统权限
    I18nErrorCode errCode = I18nServiceAbility::CheckSystemPermission();
    if (errCode != I18N_SUCCESS) {
        return errCode;
    }
    // 2. 检查更新权限
    return I18nServiceAbility::CheckUpdatePermission();
}

I18nErrorCode I18nServiceAbility::CheckSystemPermission()
{
    // services/src/i18n_service_ability.cpp:643
    bool isSystemApp = TokenIdKit::IsSystemAppByFullTokenID(callerFullToken);
    bool isShell = (tokenType == TOKEN_SHELL);
    bool isNative = (tokenType == TOKEN_NATIVE);
    if (!isSystemApp && !isShell && !isNative) {
        HILOG_ERROR_I18N("caller process is not System app, Shell or Native.");
        return NOT_SYSTEM_APP;
    }
    return I18N_SUCCESS;
}

I18nErrorCode I18nServiceAbility::CheckUpdatePermission()
{
    // services/src/i18n_service_ability.cpp:659
    int result = AccessTokenKit::VerifyAccessToken(callerToken, 
                                                   "ohos.permission.UPDATE_CONFIGURATION");
    if (result != PERMISSION_GRANTED) {
        HILOG_ERROR_I18N("caller process doesn't have UPDATE_CONFIGURATION permission.");
        return NO_PERMISSION;
    }
    return I18N_SUCCESS;
}
```

**分析**:
- ✅ 双重权限检查 (身份 + 权限)
- ✅ 日志记录失败尝试
- ✅ 每个敏感操作都调用检查

**触发路径**:
```
JS: i18n.setSystemLanguage("en")
  ↓
i18n_system_addon.cpp: SetSystemLanguage()
  ↓
IPC → I18nServiceAbility::SetSystemLanguage()
  ↓
CheckPermission() → 失败 → 返回 NO_PERMISSION
```

**影响评估**:
- **状态**: ✅ 已正确实现
- **风险**: 无

---

### R4: 字符串长度校验缺失 (低危)

**位置**: `interfaces/js/kits/src/i18n_addon.cpp` (多处)

**证据**: Locale 标签等字符串参数未限制最大长度

**潜在问题**:
```cpp
// 假设代码
std::string localeTag = GetString(env, argv[0]);
// 未检查长度，直接传递给 ICU
DateTimeFormat* fmt = new DateTimeFormat(localeTag, options);
```

**ICU 风险**: ICU 库对超长字符串的处理可能存在性能问题或漏洞

**影响评估**:
- **可利用性**: 低 (需配合 ICU 漏洞)
- **影响范围**: 应用进程
- **危害**: DoS (CPU/内存消耗)

**修复建议**:
```cpp
constexpr size_t MAX_LOCALE_TAG_LENGTH = 256;
std::string localeTag = GetString(env, argv[0]);
if (localeTag.length() > MAX_LOCALE_TAG_LENGTH) {
    napi_throw_range_error(env, nullptr, "Locale tag too long");
    return nullptr;
}
```

**缓解状态**: ❌ 未修复

---

### R5: IPC 反序列化风险 (低危)

**位置**: `services/II18nServiceAbility.idl` (生成的 IPC 代码)

**证据**: IDL 定义的接口传输字符串和复杂类型

```idl
// services/II18nServiceAbility.idl
interface OHOS.Global.I18n.II18nServiceAbility {
    void SetSystemLanguage([in] String language, [out] int code);
    void AddPreferredLanguage([in] String language, [in] int index, [out] int code);
    void GetSystemCollations([inout] Map<String, String> systemCollations, [out] int code);
    // ...
};
```

**分析**:
- 依赖 OpenHarmony IPC 框架的序列化/反序列化
- IPC 框架通常已实现长度检查
- 风险较低

**影响评估**:
- **可利用性**: 低
- **依赖**: IPC 框架安全
- **风险**: 低

---

### R6: 第三方库风险 (信息)

**依赖库**:
| 库 | 版本 | 用途 | 风险 |
|---|------|------|------|
| ICU | 系统版本 | 国际化数据 | 依赖基础设施 |
| libphonenumber | 系统版本 | 电话号码 | 依赖基础设施 |
| libxml2 | 系统版本 | XML 解析 | 依赖基础设施 |

**建议**:
- 关注 ICU 安全公告 (CVE)
- 及时更新第三方库版本

---

## 4. 安全机制评估

### 4.1 权限检查机制

| 检查点 | 位置 | 有效性 | 备注 |
|--------|------|--------|------|
| 系统应用检查 | `CheckSystemPermission()` | ✅ | TokenIdKit::IsSystemAppByFullTokenID |
| Shell/Native 检查 | `CheckSystemPermission()` | ✅ | tokenType 检查 |
| UPDATE_CONFIGURATION | `CheckUpdatePermission()` | ✅ | AccessTokenKit::VerifyAccessToken |
| 日志记录 | 各处 | ✅ | HILOG_ERROR_I18N |

### 4.2 输入校验机制

| 校验类型 | 位置 | 有效性 | 备注 |
|---------|------|--------|------|
| 类型检查 | `napi_typeof` | ✅ | 基础类型检查 |
| 空值检查 | `CheckNapiIsNull` | ✅ | null/undefined 检查 |
| 字符串长度 | `GetString` | ⚠️ | 未限制最大长度 |
| 数组长度 | `GetStringArray` | ❌ | 无长度限制 |
| Locale 格式 | ICU 内部 | ✅ | ICU 负责 |

### 4.3 内存安全机制

| 机制 | 位置 | 有效性 | 备注 |
|------|------|--------|------|
| 智能指针 | `unique_ptr` | ✅ | C++ 现代实践 |
| napi_wrap | N-API | ✅ | JS 对象绑定 |
| 类型标签 | `napi_type_tag` | ✅ | 防止类型混淆 |

---

## 5. 修复建议汇总

### 立即修复 (高危/中危)

1. **R1 - 数组长度限制**
   - 文件: `js_utils.cpp`
   - 修改: 添加 MAX_ARRAY_LENGTH 检查

2. **R2 - 路径遍历防护**
   - 文件: `i18n_addon.cpp`
   - 修改: 添加路径规范化检查

### 建议修复 (低危)

3. **R4 - 字符串长度限制**
   - 文件: 各 addon 文件
   - 修改: 添加最大长度检查

---

## 6. 验证清单

- [x] 权限检查覆盖所有敏感操作
- [x] IPC 接口有权限校验
- [ ] 数组长度限制 (待修复)
- [ ] 字符串长度限制 (待修复)
- [ ] 路径遍历防护 (待确认)

---

## 7. 参考资料

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs)
- ICU Security: https://unicode.org/security/

---

*安全评估版本: 1.0*
*更新日期: 2026-02-07*
