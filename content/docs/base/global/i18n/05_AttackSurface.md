# 攻击面分析

## 1. 攻击面总览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           i18n 模块攻击面分布                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────┐                                                       │
│  │  JS/ArkTS 应用层  │                                                       │
│  └────────┬─────────┘                                                       │
│           │ N-API 接口 (~100+ API)                                          │
│           ▼                                                                 │
│  ┌──────────────────┐    ┌──────────────────┐                              │
│  │  libi18n.so      │    │  libintl.so      │                              │
│  │  @ohos/i18n      │    │  @ohos/intl      │                              │
│  └────────┬─────────┘    └────────┬─────────┘                              │
│           │                       │                                        │
│           └───────────┬───────────┘                                        │
│                       ▼                                                    │
│              ┌──────────────────┐                                          │
│              │  libintl_util.so │                                          │
│              │  核心框架实现     │                                          │
│              └────────┬─────────┘                                          │
│                       │                                                    │
│           ┌───────────┴───────────┐                                       │
│           ▼                       ▼                                       │
│  ┌──────────────────┐    ┌──────────────────┐                             │
│  │  ICU 第三方库     │    │  libi18n_sa_client.so                          │
│  │  (数据解析)       │    │  IPC 客户端      │                             │
│  └──────────────────┘    └────────┬─────────┘                             │
│                                   │ IPC/Binder                            │
│                                   ▼                                        │
│                          ┌──────────────────┐                             │
│                          │  I18nServiceAbility│                            │
│                          │  SA ID: 5296       │                            │
│                          │  (权限检查点)       │                            │
│                          └──────────────────┘                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. 外部输入入口清单

### 2.1 N-API 字符串参数入口

| API 类别 | 示例 API | 参数类型 | 处理文件 | 风险等级 |
|---------|---------|---------|---------|---------|
| **Locale 标签** | `new DateTimeFormat('zh-CN')` | string | `intl_addon.cpp:143` | 中 |
| **日期字符串** | `EntityRecognizer.recognize(text)` | string | `entity_recognizer_addon.cpp` | 中 |
| **电话号码** | `PhoneNumberFormat.format(number)` | string | `phone_number_format_addon.cpp` | 中 |
| **骨架模式** | `getSimpleDateTimeFormatBySkeleton(skeleton)` | string | `simple_date_time_format_addon.cpp` | 低 |
| **文件路径** | `getUnicodeWrappedFilePath(path)` | string | `i18n_addon.cpp:130` | 高 |
| **时区 ID** | `getTimeZone(zoneID)` | string | `i18n_timezone_addon.cpp` | 低 |
| **语言标签** | `setSystemLanguage(language)` | string | `i18n_system_addon.cpp:634` | 高 |

### 2.2 N-API 数值/对象参数入口

| API 类别 | 示例 API | 参数类型 | 处理文件 | 风险等级 |
|---------|---------|---------|---------|---------|
| **日期对象** | `format(date)` | Date/number | `intl_date_time_format_addon.cpp:203` | 低 |
| **数字** | `format(number)` | number/BigInt | `intl_number_format_addon.cpp:181` | 低 |
| **配置选项** | `new DateTimeFormat(locales, options)` | object | `intl_date_time_format_addon.cpp:82` | 中 |
| **数组** | `supportedLocalesOf(locales[])` | string[] | `js_utils.cpp:301` | 中 |
| **布尔值** | `setUsingLocalDigit(flag)` | boolean | `i18n_system_addon.cpp:900` | 低 |

### 2.3 IPC 接口入口 (I18nServiceAbility)

| 接口方法 | 参数类型 | 权限要求 | 风险等级 |
|---------|---------|---------|---------|
| `SetSystemLanguage(language)` | string | UPDATE_CONFIGURATION | 高 |
| `SetSystemRegion(region)` | string | UPDATE_CONFIGURATION | 高 |
| `SetSystemLocale(locale)` | string | UPDATE_CONFIGURATION | 高 |
| `Set24HourClock(flag)` | string | UPDATE_CONFIGURATION | 高 |
| `AddPreferredLanguage(lang, index)` | string, int | UPDATE_CONFIGURATION | 高 |
| `RemovePreferredLanguage(index)` | int | UPDATE_CONFIGURATION | 高 |
| `SetSystemCollation(identifier)` | string | UPDATE_CONFIGURATION | 高 |
| `SetSystemNumberingSystem(id)` | string | UPDATE_CONFIGURATION | 高 |
| `SetSystemNumberPattern(pattern)` | string | UPDATE_CONFIGURATION | 高 |
| `SetSystemMeasurement(id)` | string | UPDATE_CONFIGURATION | 高 |
| `SetSystemNumericalDatePattern(id)` | string | UPDATE_CONFIGURATION | 高 |
| `SetTemperatureType(type)` | int | UPDATE_CONFIGURATION | 中 |
| `SetFirstDayOfWeek(day)` | int | UPDATE_CONFIGURATION | 中 |
| `SetUsingLocalDigit(flag)` | boolean | UPDATE_CONFIGURATION | 中 |

*完整接口列表见 `services/II18nServiceAbility.idl`*

### 2.4 配置文件输入

| 配置文件 | 路径 | 解析代码 | 风险等级 |
|---------|------|---------|---------|
| 时区 XML | `frameworks/intl/etc/timezone/*.xml` | `i18n_timezone.cpp` | 低 |
| 区域 XML | `frameworks/intl/etc/locale/*.xml` | `locale_config.cpp` | 低 |
| 语言 XML | `frameworks/intl/etc/lang/*.xml` | `locale_data.cpp` | 低 |
| 数字格式 XML | `frameworks/intl/etc/number/*.xml` | `number_format.cpp` | 低 |
| SA 配置 | `sa_profile/i18n_service_ability.xml` | 系统框架 | 低 |

## 3. 敏感操作清单

### 3.1 系统配置修改

| 操作 | API | 权限检查 | 实现位置 |
|------|-----|---------|---------|
| 修改系统语言 | `setSystemLanguage()` | ✅ | `i18n_service_ability.cpp:51` |
| 修改系统区域 | `setSystemRegion()` | ✅ | `i18n_service_ability.cpp:73` |
| 修改系统 Locale | `setSystemLocale()` | ✅ | `i18n_service_ability.cpp:95` |
| 修改 24 小时制 | `set24HourClock()` | ✅ | `i18n_service_ability.cpp:117` |
| 修改本地数字 | `setUsingLocalDigit()` | ✅ | `i18n_service_ability.cpp:139` |
| 添加首选语言 | `addPreferredLanguage()` | ✅ | `i18n_service_ability.cpp:161` |
| 移除首选语言 | `removePreferredLanguage()` | ✅ | `i18n_service_ability.cpp:178` |
| 修改温度单位 | `setTemperatureType()` | ✅ | `i18n_service_ability.cpp:195` |
| 修改周首日 | `setFirstDayOfWeek()` | ✅ | `i18n_service_ability.cpp:218` |
| 修改排序规则 | `setSystemCollation()` | ✅ | `i18n_service_ability.cpp:276` |
| 修改数字系统 | `setSystemNumberingSystem()` | ✅ | `i18n_service_ability.cpp:333` |
| 修改数字格式 | `setSystemNumberPattern()` | ✅ | `i18n_service_ability.cpp:390` |
| 修改度量衡 | `setSystemMeasurement()` | ✅ | `i18n_service_ability.cpp:447` |
| 修改日期格式 | `setSystemNumericalDatePattern()` | ✅ | `i18n_service_ability.cpp:504` |

### 3.2 文件系统操作

| 操作 | API | 目标路径 | 风险等级 |
|------|-----|---------|---------|
| 时区数据读取 | `getTimeZone()` | `/system/usr/ohos_locale_config/timezone/` | 低 |
| 区域配置读取 | `getSystemLocale()` | `/system/etc/ohos_locale_config/` | 低 |
| 路径处理 | `getUnicodeWrappedFilePath()` | 用户输入路径 | 高 |

### 3.3 内存操作

| 操作 | API | 风险 | 位置 |
|------|-----|------|------|
| 数组 resize | `GetStringArray()` | OOM | `js_utils.cpp:311` |
| 字符串分配 | `GetString()` | 内存溢出 | `js_utils.cpp:253` |
| 对象 unwrap | `napi_unwrap_s()` | UAF 风险 | 各 addon 文件 |

### 3.4 第三方库调用

| 库 | 用途 | 风险 |
|---|------|------|
| ICU (libicui18n.so) | 所有格式化操作 | 依赖库安全 |
| libphonenumber | 电话号码处理 | 依赖库安全 |
| libxml2 | 配置文件解析 | 依赖库安全 |

## 4. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           信任边界模型                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                        用户应用 (非特权)                             │  │
│   │  ┌───────────────────────────────────────────────────────────────┐ │  │
│   │  │  JS/ArkTS 代码                                                 │ │  │
│   │  │  • import i18n from '@ohos/i18n'                               │ │  │
│   │  │  • new Intl.DateTimeFormat()                                   │ │  │
│   │  └───────────────────────────────────────────────────────────────┘ │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ N-API 边界                             │
│                                    ▼                                        │
│   ╔═════════════════════════════════════════════════════════════════════╗  │
│   ║ 信任边界 1: N-API 层                                               ║  │
│   ║  • 参数类型检查 (napi_typeof)                                       ║  │
│   ║  • 字符串长度限制                                                  ║  │
│   ║  • 数组长度限制                                                    ║  │
│   ╚═════════════════════════════════════════════════════════════════════╝  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │  i18n 核心框架 (应用进程)                                            │  │
│   │  • libi18n.so / libintl.so / libintl_util.so                        │  │
│   │  • ICU 数据读取                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ IPC 边界                               │
│                                    ▼                                        │
│   ╔═════════════════════════════════════════════════════════════════════╗  │
│   ║ 信任边界 2: IPC 层                                                 ║  │
│   ║  • Binder 传输                                                     ║  │
│   ║  • 序列化/反序列化                                                 ║  │
│   ╚═════════════════════════════════════════════════════════════════════╝  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │  I18nServiceAbility (系统服务进程, uid=system)                      │  │
│   │  • 权限检查 (UPDATE_CONFIGURATION)                                  │  │
│   │  • 系统配置修改                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 5. 攻击向量分析

### 5.1 高危攻击向量

| 向量 | 攻击路径 | 影响 | 缓解措施 |
|------|---------|------|---------|
| **配置修改** | 伪造系统应用 → 调用 setSystemLanguage() | 修改全局系统设置 | 权限检查 `CheckPermission()` |
| **路径遍历** | getUnicodeWrappedFilePath("../../../") | 访问非预期文件 | TODO: 路径规范化 |

### 5.2 中危攻击向量

| 向量 | 攻击路径 | 影响 | 缓解措施 |
|------|---------|------|---------|
| **DoS (OOM)** | 传入超大数组 → GetStringArray() | 内存耗尽 | TODO: 数组长度限制 |
| **ICU 解析** | 构造恶意 locale 标签 | ICU 异常 | Locale 标签校验 |
| **IPC 洪泛** | 频繁调用 SA 接口 | 服务不可用 | 调用频率限制 |

### 5.3 低危攻击向量

| 向量 | 攻击路径 | 影响 | 缓解措施 |
|------|---------|------|---------|
| **类型混淆** | 传入非预期类型 | 返回错误结果 | napi_typeof 检查 |
| **空指针** | 传入 null/undefined | 异常抛出 | 空值检查 |

## 6. 攻击面缓解总结

| 缓解层 | 机制 | 位置 | 有效性 |
|-------|------|------|--------|
| **N-API 输入校验** | 类型检查、长度限制 | 各 addon 文件 | ✅ 有效 |
| **权限检查** | UPDATE_CONFIGURATION 权限 | `i18n_service_ability.cpp:634` | ✅ 有效 |
| **调用者身份** | 系统应用/Shell/Native | `i18n_service_ability.cpp:643` | ✅ 有效 |
| **IPC 隔离** | 跨进程调用 | Binder | ✅ 有效 |
| **日志记录** | 异常操作记录 | HILOG | ✅ 有效 |

---

*攻击面分析版本: 1.0*
*更新日期: 2026-02-07*
