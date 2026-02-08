# 目录结构与代码地图

## 1. 顶层目录概览

```
/base/global/i18n/
├── frameworks/          # 核心框架实现（C++）
│   ├── intl/           # 国际化核心功能
│   └── zone/           # 时区工具
├── interfaces/         # 对外接口层
│   ├── js/             # JavaScript N-API 接口
│   │   ├── kits/       # @ohos/i18n 对外接口
│   │   └── innerkits/  # @ohos/intl 内部接口
│   ├── native/         # Native Inner API
│   ├── ets/            # ETS/ANI (ArkTS)
│   └── cj/             # Cangjie FFI
├── services/           # 系统服务 (SA)
├── ndk/                # NDK 接口
├── sa_profile/         # SA 配置文件
├── tools/              # 工具脚本
└── wiki/               # 本文档
```

## 2. 核心代码路径映射

### 2.1 N-API 接口层

| JS 模块 | 注册文件 | 注册函数 | 行号 |
|---------|---------|---------|------|
| `@ohos/i18n` | `interfaces/js/kits/src/i18n_addon.cpp` | `I18nAddon::Init` | 1900-1956 |
| `@ohos/intl` | `interfaces/js/innerkits/intl/src/intl_module.cpp` | `IntlAddon::Init` | 25-34 |

**关键 N-API 注册点**:

```cpp
// interfaces/js/kits/src/i18n_addon.cpp:1958-1971
static napi_module g_i18nModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "i18n",          // JS 模块名
    .nm_priv = nullptr,
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void I18nRegister()
{
    napi_module_register(&g_i18nModule);  // 注册点
}
```

### 2.2 功能模块映射

| 功能域 | JS API 类 | N-API Addon 文件 | 核心实现文件 |
|--------|-----------|-----------------|-------------|
| **系统配置** | `i18n.*` | `i18n_system_addon.cpp` | `locale_config.cpp` |
| **日历** | `Calendar` | `i18n_calendar_addon.cpp` | `i18n_calendar.cpp` |
| **时区** | `I18nTimeZone` | `i18n_timezone_addon.cpp` | `i18n_timezone.cpp` |
| **规范化** | `I18nNormalizer` | `i18n_normalizer_addon.cpp` | - (ICU 直接) |
| **断行** | `I18nBreakIterator` | `i18n_addon.cpp` (内联) | - (ICU 直接) |
| **索引** | `IndexUtil` | `i18n_addon.cpp` (内联) | `index_util.cpp` |
| **转换** | `I18NUtil` | `i18n_addon.cpp` (内联) | `measure_format.cpp` |
| **日期格式化** | `DateTimeFormat` | `intl_date_time_format_addon.cpp` | `date_time_format.cpp` |
| **数字格式化** | `NumberFormat` | `intl_number_format_addon.cpp` | `number_format.cpp` |
| **相对时间** | `RelativeTimeFormat` | `intl_relative_time_format_addon.cpp` | `relative_time_format.cpp` |
| **复数规则** | `PluralRules` | `intl_plural_rules_addon.cpp` | `plural_rules.cpp` |
| **排序** | `Collator` | `collator_addon.cpp` | `collator.cpp` |
| **显示名称** | `DisplayNames` | `displaynames_addon.cpp` | `display_names.cpp` |
| **Locale** | `Locale` | `intl_locale_addon.cpp` | `locale_info.cpp` |
| **电话号码** | `PhoneNumberFormat` | `phone_number_format_addon.cpp` | `phone_number_format.cpp` |
| **节假日** | `HolidayManager` | `holiday_manager_addon.cpp` | `holiday_manager.cpp` |
| **实体识别** | `EntityRecognizer` | `entity_recognizer_addon.cpp` | `entity_recognizer.cpp` |

### 2.3 核心框架文件

| 组件类别 | 头文件路径 | 实现文件路径 | 职责 |
|---------|-----------|-------------|------|
| **日期时间** | `frameworks/intl/include/date_time_format.h` | `frameworks/intl/src/date_time_format.cpp` | 日期时间格式化 |
| **数字** | `frameworks/intl/include/number_format.h` | `frameworks/intl/src/number_format.cpp` | 数字/货币格式化 |
| **区域** | `frameworks/intl/include/locale_info.h` | `frameworks/intl/src/locale_info.cpp` | Locale 信息 |
| **配置** | `frameworks/intl/include/locale_config.h` | `frameworks/intl/src/locale_config.cpp` | 系统区域配置 |
| **时区** | `frameworks/intl/include/i18n_timezone.h` | `frameworks/intl/src/i18n_timezone.cpp` | 时区管理 |
| **日历** | `frameworks/intl/include/i18n_calendar.h` | `frameworks/intl/src/i18n_calendar.cpp` | 日历计算 |
| **排序** | `frameworks/intl/include/collator.h` | `frameworks/intl/src/collator.cpp` | 字符串比较 |
| **复数** | `frameworks/intl/include/plural_rules.h` | `frameworks/intl/src/plural_rules.cpp` | 复数规则 |
| **匹配** | `frameworks/intl/include/locale_matcher.h` | `frameworks/intl/src/locale_matcher.cpp` | Locale 匹配 |
| **电话号码** | `frameworks/intl/include/phone_number_format.h` | `frameworks/intl/src/phone_number_format.cpp` | 电话号码处理 |
| **数据** | `frameworks/intl/include/locale_data.h` | `frameworks/intl/src/locale_data.cpp` | 区域数据访问 |

### 2.4 系统服务 (SA)

| 组件 | 头文件 | 实现文件 | 行数 |
|------|--------|---------|------|
| **服务接口** | `services/II18nServiceAbility.idl` | - | 42 行 (IDL 定义) |
| **服务实现** | `services/include/i18n_service_ability.h` | `services/src/i18n_service_ability.cpp` | ~640 行 |
| **客户端** | `services/include/i18n_service_ability_client.h` | `services/src/i18n_service_ability_client.cpp` | ~210 行 |
| **加载管理** | `services/include/i18n_service_ability_load_manager.h` | `services/src/i18n_service_ability_load_manager.cpp` | ~130 行 |
| **回调** | `services/include/i18n_service_ability_load_callback.h` | `services/src/i18n_service_ability_load_callback.cpp` | ~45 行 |
| **事件** | `services/include/i18n_service_event.h` | `services/src/i18n_service_event.cpp` | ~95 行 |

**SA 注册点**:

```cpp
// services/src/i18n_service_ability.cpp:35
REGISTER_SYSTEM_ABILITY_BY_ID(I18nServiceAbility, I18N_SA_ID, false);
```

### 2.5 NDK 接口

| 接口文件 | 实现文件 | 功能 |
|---------|---------|------|
| `ndk/include/oh_timezone.h` | `ndk/src/oh_timezone.cpp` | 时区查询 |
| `ndk/include/oh_errorcode.h` | - | 错误码定义 |

## 3. 功能-文件速查表

### 3.1 日期时间相关

| 功能 | 入口文件 | 核心实现 |
|------|---------|---------|
| DateTimeFormat.format() | `intl_date_time_format_addon.cpp:203` | `date_time_format.cpp:95` |
| DateTimeFormat.formatRange() | `intl_date_time_format_addon.cpp:273` | `date_time_format.cpp:120` |
| SimpleDateTimeFormat | `simple_date_time_format_addon.cpp` | `simple_date_time_format.cpp` |
| StyledDateTimeFormat | `styled_date_time_format_addon.cpp` | `styled_date_time_format.cpp` |
| I18nCalendar | `i18n_calendar_addon.cpp` | `i18n_calendar.cpp` |
| LunarCalendar | (通过 Calendar) | `lunar_calendar.cpp` |
| 时区查询 | `i18n_timezone_addon.cpp` | `i18n_timezone.cpp` |
| ZoneRules | `zone_rules_addon.cpp` | `zone_rules.cpp` |

### 3.2 数字格式化相关

| 功能 | 入口文件 | 核心实现 |
|------|---------|---------|
| NumberFormat.format() | `intl_number_format_addon.cpp:181` | `number_format.cpp:88` |
| NumberFormat.formatRange() | `intl_number_format_addon.cpp:206` | `number_format.cpp:110` |
| NumberFormat.formatToParts() | `intl_number_format_addon.cpp:343` | `number_format.cpp:140` |
| SimpleNumberFormat | `simple_number_format_addon.cpp` | `simple_number_format.cpp` |
| StyledNumberFormat | `styled_number_format_addon.cpp` | `styled_number_format.cpp` |
| AdvancedMeasureFormat | `advanced_measure_format_addon.cpp` | `advanced_measure_format.cpp` |

### 3.3 系统配置相关

| 功能 | 入口文件 | 核心实现 |
|------|---------|---------|
| getSystemLocale() | `i18n_system_addon.cpp:158` | `locale_config.cpp:175` |
| setSystemLocale() | `i18n_system_addon.cpp:707` | 通过 IPC → SA |
| getSystemLanguage() | `i18n_system_addon.cpp:126` | `locale_config.cpp:165` |
| setSystemLanguage() | `i18n_system_addon.cpp:634` | 通过 IPC → SA |
| is24HourClock() | `i18n_system_addon.cpp:742` | `locale_config.cpp:205` |
| set24HourClock() | `i18n_system_addon.cpp:760` | 通过 IPC → SA |
| getPreferredLanguageList() | `i18n_system_addon.cpp:916` | `preferred_language.cpp:85` |

### 3.4 其他功能

| 功能 | 入口文件 | 核心实现 |
|------|---------|---------|
| Collator.compare() | `collator_addon.cpp:112` | `collator.cpp:78` |
| PluralRules.select() | `intl_plural_rules_addon.cpp:129` | `plural_rules.cpp:62` |
| RelativeTimeFormat.format() | `intl_relative_time_format_addon.cpp:218` | `relative_time_format.cpp:95` |
| DisplayNames.of() | `displaynames_addon.cpp:176` | `display_names.cpp:55` |
| PhoneNumberFormat | `phone_number_format_addon.cpp` | `phone_number_format.cpp` |
| EntityRecognizer | `entity_recognizer_addon.cpp` | `entity_recognizer.cpp` |
| HolidayManager | `holiday_manager_addon.cpp` | `holiday_manager.cpp` |
| I18nNormalizer | `i18n_normalizer_addon.cpp` | (ICU 直接) |
| IndexUtil | `i18n_addon.cpp` (内联) | `index_util.cpp` |
| UnitConvert | `i18n_addon.cpp` (内联) | `measure_format.cpp` |

## 4. 配置文件路径

| 配置类型 | 源路径 | 安装路径 |
|---------|-------|---------|
| 区域支持列表 | `frameworks/intl/etc/locale/supported_locales.xml` | `/system/etc/ohos_locale_config/` |
| 时区数据 | `frameworks/intl/etc/timezone/` | `/system/usr/ohos_locale_config/timezone/` |
| 语言数据 | `frameworks/intl/etc/lang/` | `/system/usr/ohos_locale_config/lang/` |
| 地区数据 | `frameworks/intl/etc/region/` | `/system/usr/ohos_locale_config/region/` |
| 数字格式数据 | `frameworks/intl/etc/number/` | `/system/usr/ohos_locale_config/number/` |
| 度量衡数据 | `frameworks/intl/etc/measure/` | `/system/usr/ohos_locale_config/measure/` |
| 时区默认配置 | `frameworks/zone/etc/` | `/system/etc/` |
| SA 配置 | `sa_profile/i18n_service_ability.xml` | `/system/profile/` |

## 5. 工具类与辅助代码

| 工具类 | 路径 | 用途 |
|-------|------|------|
| JSUtils | `interfaces/js/innerkits/intl/src/js_utils.cpp` | N-API 工具函数 |
| ErrorUtil | `interfaces/js/kits/src/error_util.cpp` | 错误处理 |
| VariableConvertor | `interfaces/js/kits/src/variable_convertor.cpp` | 类型转换 |
| I18nHilog | `interfaces/js/kits/src/i18n_hilog.h` | 日志宏 |
| Utils | `frameworks/intl/src/utils.cpp` | 通用工具 |
| Types | `frameworks/intl/include/i18n_types.h` | 类型定义 |

## 6. 代码统计概览

```
Language       Files    Lines    Code    Comment    Blank
C/C++ Header    ~60    ~8000    ~5000     ~1500     ~1500
C/C++ Source   ~120   ~35000   ~25000     ~4000     ~6000
XML Config      ~50    ~5000    ~4500      ~100      ~400
GN Build        ~15    ~2000    ~1800       ~50      ~150
```

*统计范围：不包含 test/ 目录*

---

*代码地图生成时间: 2026-02-07*
*版本: OpenHarmony i18n 1.0.0*
