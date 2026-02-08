# API 接口文档

## 2.1 概述

本文档详细描述 i18n_lite 模块对外暴露的所有 API，包括：

- **N-API (JavaScript API)**：为 ACELite 轻量级 JS 引擎提供接口
- **C++ API**：为 native 应用提供国际化能力

### 2.1.1 API 清单概览

| API 类型 | 模块 | 主要类/函数 | 稳定状态 |
|----------|------|------------|----------|
| **N-API** | locale | `getLocale()` | 稳定 |
| **C++** | DateTimeFormat | `DateTimeFormat` | 稳定 |
| **C++** | NumberFormat | `NumberFormat` | 稳定 |
| **C++** | PluralFormat | `PluralFormat` | 稳定 |
| **C++** | LocaleInfo | `LocaleInfo` | 稳定 |
| **C++** | MeasureFormat | `MeasureFormat` | 稳定 |
| **C++** | WeekInfo | `WeekInfo` | 稳定 |

## 2.2 N-API (JavaScript API)

### 2.2.1 Locale 模块

#### 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | `@ohos.i18n` (JavaScript) |
| **头文件** | `interfaces/kits/js/builtin/include/locale_module.h` |
| **实现文件** | `interfaces/kits/js/builtin/src/locale_module.cpp` |
| **注册函数** | `InitLocaleModule()` |

#### API 清单

| JS 方法 | C++ 实现 | 同步/异步 | 返回类型 |
|---------|----------|----------|----------|
| `getLocale()` | `LocaleModule::GetLocale()` | 同步 | Object |

#### getLocale()

获取当前设备的区域设置。

**JS 调用示例**：

```javascript
import getLocale from '@ohos.i18n';
const locale = getLocale();
console.log(locale.language);        // 如 "zh"
console.log(locale.countryOrRegion); // 如 "CN"
console.log(locale.dir);            // 如 "ltr"
```

**C++ 实现** (`interfaces/kits/js/builtin/src/locale_module.cpp:63-89`)：

```cpp
JSIValue LocaleModule::GetLocale(const JSIValue thisVal, 
                                  const JSIValue* args, 
                                  uint8_t argsNum)
{
    JSIValue result = JSI::CreateObject();
    
    // 获取语言
    char *lang = GetLanguage();
    if (lang == nullptr) {
        JSI::ReleaseValue(result);
        return JSI::CreateUndefined();
    }
    JSI::SetStringProperty(result, "language", lang);
    ace_free(lang);
    
    // 获取地区
    char *region = GetRegion();
    if (region == nullptr) {
        JSI::ReleaseValue(result);
        return JSI::CreateUndefined();
    }
    JSI::SetStringProperty(result, "countryOrRegion", region);
    ace_free(region);
    
    // 获取文本方向
    const char *dir = GetTextDirection();
    JSI::SetStringProperty(result, "dir", dir);
    return result;
}
```

**返回值对象结构**：

| 属性 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `language` | string | ISO 639 语言代码 | `"zh"`, `"en"` |
| `countryOrRegion` | string | ISO 3166 国家/地区代码 | `"CN"`, `"US"` |
| `dir` | string | 文本方向 (`"ltr"` 或 `"rtl"`) | `"ltr"` |

**错误处理**：

- 内存分配失败时返回 `JSI::CreateUndefined()`
- 内部调用 `GLOBAL_GetLanguage()` 和 `GLOBAL_GetRegion()` 失败时返回 `undefined`

#### TEXT_DIRECTION_LTR

文本方向常量。

| 属性 | 类型 | 值 |
|------|------|-----|
| `TEXT_DIRECTION_LTR` | const char* | `"ltr"` |

**源码位置**：`interfaces/kits/js/builtin/include/locale_module.h:28`

```cpp
static const char * const TEXT_DIRECTION_LTR;
```

**实现**：`interfaces/kits/js/builtin/src/locale_module.cpp:22`

```cpp
const char * const LocaleModule::TEXT_DIRECTION_LTR = "ltr";
```

#### InitLocaleModule()

初始化 locale 模块，将 `getLocale` 方法注册到 exports 对象。

**函数签名**：

```cpp
void InitLocaleModule(JSIValue exports);
```

**调用位置**：`interfaces/kits/js/builtin/src/locale_module.cpp:58-61`

```cpp
void InitLocaleModule(JSIValue exports)
{
    JSI::SetModuleAPI(exports, "getLocale", LocaleModule::GetLocale);
}
```

## 2.3 C++ API

### 2.3.1 LocaleInfo 类

区域信息类，用于管理语言、脚本和地区。

#### 类信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/i18n/include/locale_info.h` |
| **命名空间** | `OHOS::I18N` |
| **稳定性** | 稳定 |

#### 构造函数

**LocaleInfo(const char *lang, const char *script, const char *region)**

创建完整的区域信息实例。

```cpp
LocaleInfo(const char *lang, const char *script, const char *region);
```

**参数**：

| 参数 | 类型 | 说明 | 约束 |
|------|------|------|------|
| `lang` | const char* | 语言代码 (ISO 639) | 必填，2-3 字符 |
| `script` | const char* | 脚本代码 (ISO 15924) | 可选，4 字符 |
| `region` | const char* | 地区代码 (ISO 3166) | 可选，2 字符 |

**使用示例**：

```cpp
// 简体中文 (中国)
LocaleInfo locale1("zh", "Hans", "CN");

// 英语 (美国)
LocaleInfo locale2("en", "US");

// 英语 (英国)
LocaleInfo locale3("en", "Latn", "GB");
```

**源码位置**：`frameworks/i18n/src/locale_info.cpp:71-78`

#### LocaleInfo(const char *lang, const char *region)

创建简化的区域信息实例（无脚本）。

```cpp
LocaleInfo(const char *lang, const char *region);
```

**使用示例**：

```cpp
LocaleInfo locale("zh", "CN");
```

**源码位置**：`frameworks/i18n/src/locale_info.cpp:88-95`

#### LocaleInfo()

默认构造函数。

```cpp
LocaleInfo();
```

**效果**：创建无效的区域信息实例，`IsDefaultLocale()` 返回 `false`。

**源码位置**：`frameworks/i18n/src/locale_info.cpp:97-100`

#### GetId()

获取区域 ID（语言_脚本_地区的组合）。

```cpp
const char *GetId() const;
```

**返回值**：

| 返回值 | 说明 | 示例 |
|--------|------|------|
| `"zh-Hans-CN"` | 完整 ID | |
| `"en-US"` | 无脚本 | |

**源码位置**：`interfaces/kits/i18n/include/locale_info.h:128`

#### GetLanguage()

获取语言代码。

```cpp
const char *GetLanguage() const;
```

**返回值**：ISO 639 语言代码，如 `"zh"`、`"en"`

**源码位置**：`interfaces/kits/i18n/include/locale_info.h:137`

#### GetScript()

获取脚本代码。

```cpp
const char *GetScript() const;
```

**返回值**：ISO 15924 脚本代码，如 `"Hans"`、`"Latn"`（可能返回 `nullptr`）

**源码位置**：`interfaces/kits/i18n/include/locale_info.h:146`

#### GetRegion()

获取地区代码。

```cpp
const char *GetRegion() const;
```

**返回值**：ISO 3166 国家/地区代码，如 `"CN"`、`"US"`

**源码位置**：`interfaces/kits/i18n/include/locale_info.h:155`

#### IsDefaultLocale()

检查是否为默认区域 (en-US)。

```cpp
bool IsDefaultLocale() const;
```

**返回值**：

| 返回值 | 说明 |
|--------|------|
| `true` | 默认区域 (en-US) |
| `false` | 其他区域 |

**源码位置**：`frameworks/i18n/src/locale_info.cpp:80-86`

#### ForLanguageTag()

解析 BCP 47 语言标签。

```cpp
static LocaleInfo ForLanguageTag(const char *languageTag, I18nStatus &status);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `languageTag` | const char* | BCP 47 语言标签 |
| `status` | I18nStatus& | 输出状态 |

**返回值**：`LocaleInfo` 实例

**使用示例**：

```cpp
I18nStatus status = I18nStatus::ISUCCESS;
LocaleInfo locale = LocaleInfo::ForLanguageTag("zh-Hans-CN", status);
```

**支持的标签格式**：
- `zh-CN`
- `zh-Hans-CN`
- `en-US`
- `en-US-u-co-narrow`

### 2.3.2 DateTimeFormat 类

日期时间格式化类。

#### 类信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/i18n/include/date_time_format.h` |
| **命名空间** | `OHOS::I18N` |
| **稳定性** | 稳定 |

#### 构造函数

**DateTimeFormat(AvailableDateTimeFormatPattern pattern, const LocaleInfo &locale)**

```cpp
DateTimeFormat(AvailableDateTimeFormatPattern requestPattern, const LocaleInfo &locale);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `requestPattern` | AvailableDateTimeFormatPattern | 格式化模式 |
| `locale` | const LocaleInfo& | 区域信息 |

**使用示例**：

```cpp
LocaleInfo locale("zh", "Hans", "CN");
DateTimeFormat formatter(AvailableDateTimeFormatPattern::HOUR_MINUTE, locale);
```

#### Format()

格式化时间值为字符串。

```cpp
void Format(const time_t &cal, const std::string &zoneInfo, 
            std::string &appendTo, I18nStatus &status);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `cal` | const time_t& | Unix 时间戳 |
| `zoneInfo` | const std::string& | 时区信息 (`+/-HH:MM`) |
| `appendTo` | std::string& | 输出字符串 |
| `status` | I18nStatus& | 格式化状态 |

**时区格式示例**：
- `"+1:00"` - UTC+1
- `"+8:00"` - UTC+8
- `"-5:00"` - UTC-5

**使用示例**：

```cpp
LocaleInfo locale("zh", "Hans", "CN");
DateTimeFormat formatter(AvailableDateTimeFormatPattern::HOUR_MINUTE, locale);

time_t time = 3600 * 3;  // 3小时后
std::string zoneInfo = "+1:00";
std::string out;
Ii8nStatus status = Ii8nStatus::ISUCCESS;
formatter.Format(time, zoneInfo, out, status);
// 输出: "4:00"
```

#### GetWeekName()

获取星期名称。

```cpp
std::string GetWeekName(const int32_t &index, DateTimeDataType type);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `index` | int32_t | 星期索引 (0=Sunday, 6=Saturday) |
| `type` | DateTimeDataType | 名称类型 |

**DateTimeDataType 枚举**：

| 枚举值 | 说明 | 示例 |
|--------|------|------|
| `FORMAT_ABBR` | 缩写格式 | "Sun" |
| `FORMAT_WIDE` | 全称格式 | "Sunday" |
| `STANDALONE_ABBR` | 独立缩写 | "Sun" |
| `STANDALONE_WIDE` | 独立全称 | "Sunday" |

#### GetMonthName()

获取月份名称。

```cpp
std::string GetMonthName(const int32_t &index, DateTimeDataType type);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `index` | int32_t | 月份索引 (0=January, 11=December) |
| `type` | DateTimeDataType | 名称类型 |

**使用示例**：

```cpp
DateTimeFormat formatter(AvailableDateTimeFormatPattern::HOUR_MINUTE, locale);
std::string month = formatter.GetMonthName(0, DateTimeDataType::FORMAT_WIDE);
// 输出: "January"
```

#### FormatElapsedDuration()

格式化逝去时间。

```cpp
std::string FormatElapsedDuration(int32_t milliseconds, 
                                   ElapsedPatternType type, 
                                   I18nStatus &status);
```

**ElapsedPatternType 枚举**：

| 枚举值 | 格式示例 |
|--------|----------|
| `ELAPSED_MINUTE_SECOND` | "5:30" |
| `ELAPSED_MINUTE_SECOND_MILLISECOND` | "5:30.123" |
| `ELAPSED_HOUR_MINUTE_SECOND` | "1:05:30" |
| `ELAPSED_HOUR_MINUTE` | "1:05" |

#### GetTimeSeparator()

获取时间分隔符。

```cpp
std::string GetTimeSeparator();
```

**返回值示例**：

| 区域 | 分隔符 |
|------|--------|
| en_US | `:` |
| zh_CN | `:` |

### 2.3.3 NumberFormat 类

数字格式化类。

#### 类信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/i18n/include/number_format.h` |
| **命名空间** | `OHOS::I18N` |
| **稳定性** | 稳定 |

#### 构造函数

**NumberFormat(LocaleInfo &locale, int &status)**

```cpp
NumberFormat(LocaleInfo &locale, int &status);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `locale` | LocaleInfo& | 区域信息 |
| `status` | int& | 输出状态 (0=成功, 1=失败) |

**使用示例**：

```cpp
LocaleInfo locale("en", "US");
int status = 0;
NumberFormat formatter(locale, status);
if (status != 0) {
    // 初始化失败
}
```

#### Format(double, NumberFormatType, int&)

格式化双精度浮点数。

```cpp
std::string Format(double num, NumberFormatType type, int &status);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `num` | double | 要格式化的数字 |
| `type` | NumberFormatType | 格式化类型 |
| `status` | int& | 输出状态 |

**NumberFormatType 枚举**：

| 枚举值 | 说明 | en_US 示例 |
|--------|------|------------|
| `DECIMAL` | 十进制 | "1,234.56" |
| `PERCENT` | 百分比 | "50%" |

**使用示例**：

```cpp
int status = 0;
NumberFormat formatter(locale, status);
std::string result = formatter.Format(1234.56, NumberFormatType::DECIMAL, status);
// 输出: "1,234.56"
```

#### Format(int, int&)

格式化整数。

```cpp
std::string Format(int num, int &status);
```

**使用示例**：

```cpp
int status = 0;
NumberFormat formatter(locale, status);
std::string result = formatter.Format(1234567, status);
// 输出: "1,234,567"
```

#### SetMaxDecimalLength()

设置小数部分最大长度。

```cpp
bool SetMaxDecimalLength(int length);
```

#### SetMinDecimalLength()

设置小数部分最小长度（不足补零）。

```cpp
bool SetMinDecimalLength(int length);
```

### 2.3.4 PluralFormat 类

复数格式化类。

#### 类信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/i18n/include/plural_format.h` |
| **命名空间** | `OHOS::I18N` |
| **稳定性** | 稳定 |

#### 构造函数

**PluralFormat(LocaleInfo &locale, I18nStatus &status)**

```cpp
PluralFormat(LocaleInfo &locale, I18nStatus &status);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `locale` | LocaleInfo& | 区域信息 |
| `status` | I18nStatus& | 输出状态 |

#### GetPluralRuleIndex(int, I18nStatus)

获取整数对应的复数规则索引。

```cpp
int GetPluralRuleIndex(int number, I18nStatus status);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `number` | int | 数字 |
| `status` | I18nStatus& | 输出状态 |

**返回值**：`PluralRuleType` 枚举值 (0-5)

**使用示例**：

```cpp
I18nStatus status = I18nStatus::ISUCCESS;
PluralFormat formatter(locale, status);
int rule = formatter.GetPluralRuleIndex(1, status);
// 英语: rule = ONE (1)
// 英语: rule = OTHER (2)
```

#### GetPluralRuleIndex(double, I18nStatus)

获取小数对应的复数规则索引。

```cpp
int GetPluralRuleIndex(double number, I18nStatus status);
```

### 2.3.5 MeasureFormat 类

度量单位格式化类。

#### 类信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/i18n/include/measure_format.h` |
| **命名空间** | `OHOS::I18N` |
| **稳定性** | 稳定 |

#### 构造函数

**MeasureFormat(LocaleInfo &locale, MeasureFormatType type, I18nStatus &status)**

```cpp
MeasureFormat(LocaleInfo &locale, MeasureFormatType type, I18nStatus &status);
```

**MeasureFormatType 枚举**：

| 枚举值 | 说明 |
|--------|------|
| `MEASURE_SHORT` | 短格式 |
| `MEASURE_MEDIUM` | 中等格式 |
| `MEASURE_LONG` | 长格式 |
| `MEASURE_FULL` | 完整格式 |

### 2.3.6 WeekInfo 类

周信息类。

#### 类信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/i18n/include/week_info.h` |
| **命名空间** | `OHOS::I18N` |
| **稳定性** | 稳定 |

#### GetFirstDayOfWeek()

获取一周的第一天。

```cpp
int GetFirstDayOfWeek();
```

**返回值**：

| 返回值 | 说明 |
|--------|------|
| 1 | Monday |
| 7 | Sunday |

#### GetMinimalDaysInFirstWeek()

获取第一周的最少天数。

```cpp
int GetMinimalDaysInFirstWeek();
```

## 2.4 类型定义

### 2.4.1 I18nStatus 枚举

格式化状态枚举。

```cpp
enum I18nStatus {
    ISUCCESS = 0,  // 成功
    IERROR         // 错误
};
```

**使用场景**：
- `LocaleInfo::ForLanguageTag()`
- `PluralFormat` 构造
- `DateTimeFormat::Format()`
- `NumberFormat::Format()`

### 2.4.2 AvailableDateTimeFormatPattern 枚举

日期时间格式化模式枚举（完整列表见 `types.h:79-154`）。

| 模式 | 格式示例 |
|------|----------|
| `HOUR12_MINUTE_SECOND` | "03:30:45 PM" |
| `HOUR24_MINUTE_SECOND` | "15:30:45" |
| `FULL` | "Friday December 18, 2020" |
| `MEDIUM` | "Dec 18, 2020" |
| `SHORT` | "12/18/2020" |

### 2.4.3 PluralRuleType 枚举

复数规则类型枚举。

```cpp
enum PluralRuleType {
    ZERO = 0,   // 零
    ONE = 1,    // 一
    TWO = 2,    // 二
    FEW = 3,    // 少量
    MANY = 4,   // 大量
    OTHER = 5   // 其他
};
```

## 2.5 错误码处理

### 2.5.1 常见错误码

| 错误码 | 来源 | 说明 |
|--------|------|------|
| `I18nStatus::IERROR` | `types.h:51-57` | 国际化操作失败 |
| `int status = 1` | `number_format.h:71` | NumberFormat 初始化失败 |
| `nullptr` 返回 | `locale_info.h` | 无效的区域参数 |

### 2.5.2 错误处理建议

```cpp
// 示例：安全的 LocaleInfo 创建
I18nStatus status = I18nStatus::ISUCCESS;
LocaleInfo locale = LocaleInfo::ForLanguageTag("zh-CN", status);
if (status != I18nStatus::ISUCCESS) {
    // 使用默认区域
    locale = LocaleInfo("en", "US");
}

// 示例：安全的 DateTimeFormat 使用
DateTimeFormat formatter(pattern, locale);
if (!formatter.Init()) {
    // 初始化失败
}
```

## 2.6 API 使用示例

### 2.6.1 完整使用示例

```cpp
#include "date_time_format.h"
#include "number_format.h"
#include "plural_format.h"
#include "locale_info.h"
#include "types.h"

using namespace OHOS::I18N;

void Example()
{
    // 1. 创建区域信息
    LocaleInfo locale("zh", "Hans", "CN");
    
    // 2. 日期时间格式化
    DateTimeFormat dateFormatter(AvailableDateTimeFormatPattern::HOUR_MINUTE, locale);
    time_t now = time(nullptr);
    std::string timeStr;
    I18nStatus status = I18nStatus::ISUCCESS;
    dateFormatter.Format(now, "+8:00", timeStr, status);
    
    // 3. 数字格式化
    int status = 0;
    NumberFormat numFormatter(locale, status);
    std::string numStr = numFormatter.Format(1234.56, NumberFormatType::DECIMAL, status);
    
    // 4. 复数规则
    PluralFormat pluralFormatter(locale, status);
    int rule = pluralFormatter.GetPluralRuleIndex(1, status);
}
```

---

*最后更新：2026-02-06*
