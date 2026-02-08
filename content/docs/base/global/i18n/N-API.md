# N-API 接口文档

## 模块列表

| 模块名 | 命名空间 | 注册文件 | 注册函数 |
|--------|----------|----------|----------|
| @ohos/i18n | global | `interfaces/js/kits/src/i18n_addon.cpp` | `I18nAddon::Init` |
| @ohos/intl | Intl | `interfaces/js/innerkits/intl/src/intl_addon.cpp` | `IntlAddon::Init` |

## @ohos/i18n 模块

### 静态函数

| JS API | C++ 实现 | 参数 | 返回值 | 说明 |
|--------|----------|------|--------|------|
| `getDisplayLanguage(lang: string, script?: string, region?: string): string` | `I18nSystemAddon::GetDisplayLanguage` | lang, script, region | string | 获取语言显示名 |
| `getDisplayCountry(country: string, lang?: string, script?: string): string` | `I18nSystemAddon::GetDisplayCountry` | country, lang, script | string | 获取国家显示名 |
| `getSystemLanguage(): string` | `I18nSystemAddon::GetSystemLanguage` | - | string | 系统语言 |
| `getSystemRegion(): string` | `I18nSystemAddon::GetSystemRegion` | - | string | 系统区域 |
| `getSystemLocale(): string` | `I18nSystemAddon::GetSystemLocale` | - | string | 系统区域标签 |
| `getCalendar(lang: string, region?: string): Calendar` | `I18nCalendarAddon::GetCalendar` | lang, region | Calendar | 获取日历实例 |
| `isRTL(lang: string): boolean` | `IsRTL` | lang | boolean | 是否从右到左语言 |
| `getLineInstance(lang: string): I18nBreakIterator` | `GetLineInstance` | lang | I18nBreakIterator | 获取断行器 |
| `getInstance(type: IndexType): IndexUtil` | `GetIndexUtil` | type | IndexUtil | 获取索引工具 |
| `addPreferredLanguage(language: string, index?: number): boolean` | `I18nSystemAddon::AddPreferredLanguage` | language, index | boolean | 添加首选语言 |
| `removePreferredLanguage(index: number): boolean` | `I18nSystemAddon::RemovePreferredLanguage` | index | boolean | 移除首选语言 |
| `getPreferredLanguageList(): Array<PreferredLanguage>` | `I18nSystemAddon::GetPreferredLanguageList` | - | Array | 获取首选语言列表 |
| `getFirstPreferredLanguage(): string` | `I18nSystemAddon::GetFirstPreferredLanguage` | - | string | 获取首个首选语言 |
| `getSimpleNumberFormatBySkeleton(skeleton: string, locale?: string): string` | `SimpleNumberFormatAddon::GetSimpleNumberFormatBySkeleton` | skeleton, locale | string | 按骨架格式化数字 |
| `getSimpleDateTimeFormatByPattern(pattern: string, locale?: string): string` | `SimpleDateTimeFormatAddon::GetSimpleDateTimeFormatByPattern` | pattern, locale | string | 按模式格式化日期 |
| `getSimpleDateTimeFormatBySkeleton(skeleton: string, locale?: string): string` | `SimpleDateTimeFormatAddon::GetSimpleDateTimeFormatBySkeleton` | skeleton, locale | string | 按骨架格式化日期 |
| `is24HourClock(): boolean` | `I18nSystemAddon::Is24HourClock` | - | boolean | 是否 24 小时制 |
| `set24HourClock(is24Hour: boolean): boolean` | `I18nSystemAddon::Set24HourClock` | is24Hour | boolean | 设置 24 小时制 |
| `getTimeZone(zoneID?: string): I18nTimeZone` | `I18nTimeZoneAddon::GetI18nTimeZone` | zoneID | I18nTimeZone | 获取时区 |

### 枚举

| 枚举名 | 值 | 说明 |
|--------|-----|------|
| `NormalizerMode` | `NFC`, `NFD`, `NFKC`, `NFKD` | Unicode 归一化模式 |
| `TemperatureType` | `CELSIUS`, `FAHRENHEIT`, `KELVIN` | 温度单位 |
| `WeekDay` | `MON`~`SUN` | 星期 |
| `UnitUsage` | `AREA_LAND_AGRICULT`... | 单位用途 |

### 类: I18NUtil

| 方法 | C++ 实现 | 说明 |
|------|----------|------|
| `unitConvert(from: number, fromUnit: string, toUnit: string, usage?: UnitUsage): number` | `UnitConvert` | 单位转换 |
| `getDateOrder(locale?: string): string` | `GetDateOrder` | 获取日期顺序 |
| `getTimePeriodName(hour: number, locale?: string): string` | `GetTimePeriodName` | 获取时段名 |
| `getBestMatchLocale(): string` | `GetBestMatchLocale` | 获取最佳匹配区域 |
| `getThreeLetterLanguage(lang: string): string` | `GetThreeLetterLanguage` | 获取三字母语言码 |
| `getThreeLetterRegion(region: string): string` | `GetThreeLetterRegion` | 获取三字母国家码 |
| `getUnicodeWrappedFilePath(path: string): string` | `GetUnicodeWrappedFilePath` | 获取 Unicode 包装路径 |

## @ohos/intl 模块

### 类: DateTimeFormat

| 方法 | C++ 实现 | 说明 |
|------|----------|------|
| `constructor(locale?: string, options?: DateTimeFormatOptions)` | `DateTimeFormatConstructor` | 构造函数 |
| `format(date: Date \| number): string` | `FormatDateTime` | 格式化日期 |
| `formatRange(start: Date \| number, end: Date \| number): string` | `FormatDateTimeRange` | 格式化日期范围 |
| `resolvedOptions(): DateTimeResolvedOptions` | `GetDateTimeResolvedOptions` | 获取解析选项 |

### 类: Collator

| 方法 | C++ 实现 | 说明 |
|------|----------|------|
| `constructor(locale?: string, options?: CollatorOptions)` | `CollatorConstructor` | 构造函数 |
| `compare(first: string, second: string): number` | `CollatorCompare` | 比较字符串 |
| `resolvedOptions(): CollatorResolvedOptions` | `GetCollatorResolvedOptions` | 获取解析选项 |

### 类: RelativeTimeFormat

| 方法 | C++ 实现 | 说明 |
|------|----------|------|
| `constructor(locale?: string, options?: RelativeTimeFormatOptions)` | `RelativeTimeFormatConstructor` | 构造函数 |
| `format(value: number, unit: string): string` | `FormatRelativeTime` | 格式化相对时间 |
| `formatToParts(value: number, unit: string): Array<RelativeTimeFormatPart>` | `FormatToParts` | 格式化相对时间 (部件) |
| `resolvedOptions(): RelativeTimeResolvedOptions` | `GetRelativeTimeResolvedOptions` | 获取解析选项 |

### 类: PluralRules

| 方法 | C++ 实现 | 说明 |
|------|----------|------|
| `constructor(locale?: string, options?: PluralRulesOptions)` | `PluralRulesConstructor` | 构造函数 |
| `select(n: number \| string): string` | `PluralRulesSelect` | 选择复数形式 |
| `selectRange(start: number, end: number): string` | `PluralRulesSelectRange` | 选择范围复数形式 |
| `resolvedOptions(): PluralRulesResolvedOptions` | `GetPluralRulesResolvedOptions` | 获取解析选项 |

## 错误码

| 错误码 | 说明 |
|--------|------|
| `I18nErrorCode::SUCCESS` | 成功 |
| `I18nErrorCode::NOT_SYSTEM_APP` | 非系统应用 |
| `I18nErrorCode::NO_PERMISSION` | 无权限 |

## 参数校验

### 区域标签校验

- 遵循 ISO 639 (语言), ISO 15924 (文字), ISO 3166 (国家) 标准
- 支持 BCP 47 格式 (e.g., `zh-Hans-CN`)

### 字符串参数

- UTF-8 编码
- 长度限制: 依赖具体 API

### 数字参数

- JavaScript `number` 类型
- 日期: `Date` 对象或毫秒时间戳
