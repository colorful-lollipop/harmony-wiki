# 内部实现细节

## 1. 核心类职责

### 1.1 格式化类

| 类名 | 文件 | 职责 | 生命周期 |
|------|------|------|---------|
| `DateTimeFormat` | `date_time_format.cpp` | 日期时间格式化 | 绑定到 JS 对象 |
| `NumberFormat` | `number_format.cpp` | 数字/货币格式化 | 绑定到 JS 对象 |
| `SimpleDateTimeFormat` | `simple_date_time_format.cpp` | 简单日期时间格式化 | 临时对象 |
| `SimpleNumberFormat` | `simple_number_format.cpp` | 简单数字格式化 | 临时对象 |
| `StyledDateTimeFormat` | `styled_date_time_format.cpp` | 样式化日期时间格式化 | 临时对象 |
| `StyledNumberFormat` | `styled_number_format.cpp` | 样式化数字格式化 | 临时对象 |
| `RelativeTimeFormat` | `relative_time_format.cpp` | 相对时间格式化 | 绑定到 JS 对象 |
| `Collator` | `collator.cpp` | 字符串比较排序 | 绑定到 JS 对象 |
| `PluralRules` | `plural_rules.cpp` | 复数规则计算 | 绑定到 JS 对象 |

### 1.2 区域与配置类

| 类名 | 文件 | 职责 |
|------|------|------|
| `LocaleInfo` | `locale_info.cpp` | Locale 信息封装 |
| `LocaleConfig` | `locale_config.cpp` | 系统区域配置管理 |
| `LocaleMatcher` | `locale_matcher.cpp` | Locale 匹配与解析 |
| `LocaleData` | `locale_data.cpp` | 区域数据访问 |

### 1.3 时区与日历类

| 类名 | 文件 | 职责 |
|------|------|------|
| `I18nTimeZone` | `i18n_timezone.cpp` | 时区管理 |
| `I18nCalendar` | `i18n_calendar.cpp` | 日历计算 |
| `LunarCalendar` | `lunar_calendar.cpp` | 农历计算 |
| `ZoneRules` | `zone_rules.cpp` | 时区规则 |
| `ZoneOffsetTransition` | `zone_offset_transition.cpp` | 时区偏移转换 |

### 1.4 工具类

| 类名 | 文件 | 职责 |
|------|------|------|
| `IndexUtil` | `index_util.cpp` | 索引生成 |
| `I18nNormalizer` | - | Unicode 归一化 (ICU 包装) |
| `PhoneNumberFormat` | `phone_number_format.cpp` | 电话号码格式化 |
| `EntityRecognizer` | `entity_recognizer.cpp` | 实体识别 |
| `HolidayManager` | `holiday_manager.cpp` | 节假日管理 |

### 1.5 系统服务类

| 类名 | 文件 | 职责 |
|------|------|------|
| `I18nServiceAbility` | `i18n_service_ability.cpp` | 系统服务实现 |
| `I18nServiceAbilityClient` | `i18n_service_ability_client.cpp` | SA 客户端 |
| `I18nServiceAbilityLoadManager` | `i18n_service_ability_load_manager.cpp` | SA 加载管理 |
| `I18nServiceEvent` | `i18n_service_event.cpp` | 事件订阅处理 |

## 2. N-API 对象绑定机制

### 2.1 对象创建流程

```cpp
// 以 DateTimeFormat 为例

// 1. 构造函数定义
static napi_value DateTimeFormatConstructor(napi_env env, napi_callback_info info) {
    // 解析参数 (locale, options)
    std::vector<std::string> localeTags = GetLocaleTags(env, argv[0]);
    
    // 2. 创建 C++ 对象
    DateTimeFormat* formatter = new DateTimeFormat(localeTags, options);
    
    // 3. 绑定到 JS 对象
    napi_status status = napi_wrap_s(
        env,
        thisVar,                                    // JS 对象
        reinterpret_cast<void*>(formatter),         // C++ 对象
        DateTimeFormatAddon::Destructor,            // 析构回调
        nullptr,
        &TYPE_TAG                                   // 类型标签
    );
    return thisVar;
}
```

**关键代码位置**:
- 构造: `interfaces/js/innerkits/intl/src/intl_date_time_format_addon.cpp:82`
- 绑定: `interfaces/js/innerkits/intl/src/intl_date_time_format_addon.cpp:118`
- 析构: `interfaces/js/innerkits/intl/src/intl_date_time_format_addon.cpp:39`

### 2.2 对象解包机制

```cpp
// 方法调用时的解包
napi_value DateTimeFormatAddon::Format(napi_env env, napi_callback_info info) {
    // 获取 this 对象
    napi_value thisVar = nullptr;
    napi_get_cb_info(env, info, nullptr, nullptr, &thisVar, nullptr);
    
    // 解包 C++ 对象
    DateTimeFormatAddon* obj = nullptr;
    napi_status status = napi_unwrap_s(
        env,
        thisVar,
        &TYPE_TAG,                                  // 类型标签校验
        reinterpret_cast<void**>(&obj)
    );
    
    // 使用 C++ 对象
    std::string result = obj->intlDateTimeFormat->Format(date);
    return CreateString(env, result);
}
```

**关键代码位置**:
- 解包: `interfaces/js/innerkits/intl/src/intl_date_time_format_addon.cpp:223`
- 校验: `interfaces/js/innerkits/intl/src/intl_date_time_format_addon.cpp:224`

### 2.3 类型标签安全

```cpp
// 类型标签定义
static const char DATE_TIME_FORMAT_CLASS_NAME[] = "DateTimeFormat";
static napi_type_tag TYPE_TAG = {
    0x7F2A3B4C5D6E7F8A,  // 高 64 位 (随机)
    0x9F8A7B6C5D4E3F2A   // 低 64 位 (随机)
};
```

**说明**: 类型标签用于防止类型混淆攻击，确保解包的对象类型正确。

## 3. 资源生命周期管理

### 3.1 N-API 对象生命周期

```
JS 对象创建 (new DateTimeFormat())
        ↓
C++ 对象分配 (new DateTimeFormat)
        ↓
napi_wrap_s 绑定
        ↓
JS 对象使用期间
        ↓
JS 对象被 GC → 触发 Destructor
        ↓
C++ 对象释放 (delete)
```

**析构函数示例**:

```cpp
void DateTimeFormatAddon::Destructor(napi_env env, void* nativeObject, void* /* finalize_hint */) {
    DateTimeFormatAddon* obj = reinterpret_cast<DateTimeFormatAddon*>(nativeObject);
    delete obj;  // 释放 C++ 对象
}
```

### 3.2 ICU 资源管理

ICU 资源通过智能指针管理:

```cpp
class DateTimeFormat {
private:
    std::unique_ptr<icu::DateFormat> dateFormat_;  // ICU 对象
};
```

**位置**: `frameworks/intl/include/date_time_format.h:67`

### 3.3 SA 连接生命周期

```
首次调用配置接口
        ↓
I18nServiceAbilityClient::GetProxy()
        ↓
Load SA (如果未加载)
        ↓
建立 IPC 连接
        ↓
连接缓存复用
        ↓
进程结束 / 超时卸载
```

**延迟卸载机制**:

```cpp
// services/src/i18n_service_ability.cpp:537
void I18nServiceAbility::DelayUnloadI18nServiceAbility() {
    auto i18nSaLoadManager = DelayedSingleton<I18nServiceAbilityLoadManager>::GetInstance();
    i18nSaLoadManager->UnloadI18nService(I18N_SA_ID);
}
```

## 4. 内部 API 契约

### 4.1 稳定接口 (Inner API)

供其他系统模块调用的稳定接口:

| 接口 | 文件 | 稳定性 |
|------|------|--------|
| `LocaleConfig::GetSystemLocale()` | `locale_config.cpp` | 稳定 |
| `LocaleConfig::SetSystemLocale()` | `locale_config.cpp` | 稳定 |
| `ZoneUtil::GetZoneInfo()` | `zone_util.cpp` | 稳定 |
| `I18nServiceAbilityClient::*` | `i18n_service_ability_client.cpp` | 稳定 |

### 4.2 内部实现细节

可能变更的内部实现:

| 接口 | 文件 | 说明 |
|------|------|------|
| `DateTimeFormat::CreateInstance()` | `date_time_format.cpp` | 实现细节 |
| `NumberFormat::Format()` | `number_format.cpp` | 实现细节 |
| `LocaleData::GetData()` | `locale_data.cpp` | 实现细节 |

### 4.3 ICU 封装契约

```cpp
// ICU 对象封装示例
class DateTimeFormat {
public:
    // 创建 ICU DateFormat 对象
    static std::unique_ptr<DateTimeFormat> CreateInstance(
        const std::vector<std::string>& localeTags,
        const std::map<std::string, std::string>& options);
    
    // 使用 ICU 进行格式化
    std::string Format(double milliseconds);
    
private:
    std::unique_ptr<icu::DateFormat> dateFormat_;
    icu::Locale locale_;
};
```

## 5. 数据流内部细节

### 5.1 格式化数据流

```
JS: format(date)
  ↓
N-API: 提取毫秒时间戳 (double)
  ↓
C++: DateTimeFormat::Format(milliseconds)
  ↓
ICU: icu::DateFormat::format(UTimestamp, UnicodeString)
  ↓
ICU: 查表获取本地化字符串
  ↓
C++: UnicodeString → std::string (UTF-8)
  ↓
N-API: napi_create_string_utf8()
  ↓
JS: 返回格式化字符串
```

### 5.2 配置数据流

```
JS: setSystemLanguage("zh-CN")
  ↓
N-API: 参数校验
  ↓
IPC: SendRequest(SET_SYSTEM_LANGUAGE)
  ↓
SA: CheckPermission()
  ↓
SA: Preferences::PutString("system_language", "zh-CN")
  ↓
SA: 广播配置变更事件
  ↓
系统: 更新全局配置
```

## 6. 并发模型

### 6.1 线程安全

| 组件 | 线程安全 | 说明 |
|------|---------|------|
| `DateTimeFormat` | ❌ 否 | 每个实例绑定到创建线程 |
| `NumberFormat` | ❌ 否 | 每个实例绑定到创建线程 |
| `LocaleConfig` | ✅ 是 | 使用 Preferences 服务 |
| `I18nServiceAbility` | ✅ 是 | 线程安全设计 |

### 6.2 ICU 线程安全

ICU 对象一般不是线程安全的，需要在同一线程使用:

```cpp
// 错误: 跨线程使用
DateTimeFormat* fmt = CreateFormat();  // Thread A
fmt->Format(date);                      // Thread B - 危险!
```

**建议**: 每个线程创建独立的格式化对象。

## 7. 内存布局

### 7.1 DateTimeFormat 对象布局

```
DateTimeFormat (C++ 对象)
├── vptr
├── locale_              (icu::Locale)
├── dateFormat_          (unique_ptr<icu::DateFormat>)
│   └── icu::DateFormat  (ICU 对象, ~500 bytes)
├── timeZone_            (TimeZone)
├── calendarType_        (enum)
└── hour12_              (bool)
```

**典型大小**: ~800-1200 字节 (取决于 ICU 对象)

### 7.2 N-API 对象开销

```
JS DateTimeFormat 对象
├── V8 对象头          (~32 bytes)
├── Internal fields    (存储 native 指针)
└── C++ 对象指针       → DateTimeFormat (~1000 bytes)
```

---

*内部实现版本: 1.0*
*更新日期: 2026-02-07*
