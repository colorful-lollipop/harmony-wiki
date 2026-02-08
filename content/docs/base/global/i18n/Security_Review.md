# 安全风险评审

## 1. 攻击面清单

### 1.1 N-API 接口

| 接口 | 文件 | 暴露方法 | 风险等级 |
|------|------|----------|----------|
| `@ohos/i18n` | `interfaces/js/kits/src/i18n_addon.cpp` | 系统配置、时区、单位转换 | 中 |
| `@ohos/intl` | `interfaces/js/innerkits/intl/src/intl_addon.cpp` | 格式化、区域敏感比较 | 低 |

### 1.2 SA 服务

| 服务 | System Ability ID | 敏感操作 | 风险等级 |
|------|-------------------|----------|----------|
| I18nServiceAbility | `I18N_SA_ID` | 系统语言/区域配置修改 | 高 |

### 1.3 文件 I/O

| 操作 | 路径 | 风险等级 |
|------|------|----------|
| 时区数据读取 | `frameworks/intl/etc/timezone/*.xml` | 低 |
| 语言配置读取 | `frameworks/intl/etc/lang/*.xml` | 低 |

## 2. 信任边界

```
+-------------------+     IPC      +-------------------+
|   JS/ArkTS 应用   | <---------> | I18nServiceAbility|
+--------+----------+             +--------+----------+
         |                               |
         | N-API                        | Permission
         v                               v
+--------+-------------------------------+----------+
|                    libi18n.so / libintl.so          |
+----------------------------------------------------+
         |                               |
         v                               v
+--------+----------+             +--------+----------+
|  ICU 库 (数据)    |             | Preferences     |
+-------------------+             +-----------------+
```

**信任边界**:
- **边界 1**: JS 层到 N-API 层 (参数校验在 N-API 完成)
- **边界 2**: App 进程到 SA 进程 (权限检查在 SA 完成)

## 3. 可被利用点

### 3.1 潜在 DoS: 数组长度未限制 (中风险)

**证据**:
- 文件: `interfaces/js/innerkits/intl/src/js_utils.cpp:311`
- 代码: `result.resize(arrayLength);`

**触发条件**:
1. 攻击者构造超大数组传入 N-API
2. `napi_get_array_length` 返回超大值
3. `resize` 导致内存分配失败或 OOM

**影响**:
- 应用进程崩溃 (Out of Memory)
- 系统资源耗尽

**修复建议**:
```cpp
// 添加最大长度限制
constexpr uint32_t MAX_ARRAY_LENGTH = 1024;
if (arrayLength > MAX_ARRAY_LENGTH) {
    HILOG_ERROR_I18N("GetStringArray: Array length exceeds limit");
    code = 1;
    return result;
}
result.resize(arrayLength);
```

### 3.2 潜在路径遍历: Unicode 文件路径处理 (低风险)

**证据**:
- 文件: `interfaces/js/kits/src/i18n_addon.cpp:130`
- 代码: `GetUnicodeWrappedFilePath`

**触发条件**:
1. 输入包含 `..` 或绝对路径的 Unicode 字符串
2. 路径被用于文件操作 (如果存在此类操作)

**影响**:
- 访问非预期文件

**修复建议**:
- 对输入路径进行规范化
- 验证路径是否在预期目录内

### 3.3 敏感操作权限控制不足风险 (已缓解)

**证据**:
- 文件: `services/src/i18n_service_ability.cpp:634-669`
- 代码: `CheckPermission()` -> `CheckSystemPermission()` + `CheckUpdatePermission()`

**缓解措施**:
```cpp
// CheckSystemPermission: 验证调用者身份
bool isSystemApp = TokenIdKit::IsSystemAppByFullTokenID(callerFullToken);
bool isShell = (tokenType == TOKEN_SHELL);
bool isNative = (tokenType == TOKEN_NATIVE);
if (!isSystemApp && !isShell && !isNative) {
    return NOT_SYSTEM_APP;
}

// CheckUpdatePermission: 验证权限
int result = AccessTokenKit::VerifyAccessToken(callerToken, "ohos.permission.UPDATE_CONFIGURATION");
```

**风险等级**: 低 (已正确实现权限检查)

### 3.4 N-API 参数类型混淆 (低风险)

**证据**:
- 文件: `interfaces/js/innerkits/intl/src/intl_addon.cpp:143-173`
- 代码: `GetLocaleTag()` 只检查 `napi_string`

**触发条件**:
1. 传入非字符串类型作为区域标签
2. `napi_typeof` 返回非 string 类型

**当前处理**:
```cpp
if (valueType != napi_valuetype::napi_string) {
    HILOG_ERROR_I18N("GetLocaleTag: Parameter type does not match");
    return "";  // 返回空字符串
}
```

**影响**: 返回空/错误结果，非安全风险

### 3.5 ICU 库安全 (外部依赖)

**证据**:
- 文件: `frameworks/intl/BUILD.gn`
- 依赖: `icu:shared_icui18n`, `icu:shared_icuuc`

**说明**:
- ICU 库由 OpenHarmony 基础设施管理
- i18n 模块不直接处理 ICU 内部数据
- 需关注 ICU 库的安全公告

**风险等级**: 低 (依赖基础设施)

## 4. 安全机制总结

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| 权限检查 | `I18nServiceAbility::CheckPermission` | ✅ 有效 |
| 参数类型校验 | `JSUtils::GetLocaleArray`, `CheckNapiIsNull` | ✅ 有效 |
| 空值处理 | `CheckNapiIsNull` | ✅ 有效 |
| 字符串长度校验 | `JSUtils::GetString` | ✅ 有效 |
| 输入日志记录 | `HILOG_ERROR_I18N` | ✅ 有效 |

## 5. 限制与未覆盖范围

**本次评审未覆盖**:
- ICU 内部实现安全
- Binder/IPC 底层传输安全
- 系统启动阶段初始化安全
- NDK 接口安全 (`ndk/`)

**评审方法**:
- 静态代码分析
- 依赖代码扫描
- 架构威胁建模

**评审时间**: 2026-02-06
