# 常见问题与附录

## 6.1 关键调用链

### 6.1.1 日期时间格式化调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DateTimeFormat::Format() 调用链                     │
└─────────────────────────────────────────────────────────────────────┘

应用层
  │
  ▼
DateTimeFormat::Format(time_t cal, string zoneInfo, string& appendTo, I18nStatus& status)
  │
  ├─▶ DateTimeFormat::Init() ──▶ DateTimeFormatImpl::Init()
  │         │
  │         ▼
  │    DataResource::DataResource(localeInfo)
  │         │
  │         ▼
  │    LocaleInfo::GetMask()
  │         │
  │         ▼
  │    DataResource::GetFallbackMask(locale)  ◀── 区域回退
  │         │
  │         ▼
  │    Load data from i18n.dat
  │
  └─▶ DateTimeFormatImpl::Format(cal, zoneInfo, out, status)
             │
             ├─▶ DateTimeData::GetTimeSeparator()
             │         │
             │         ▼
             │    DataResource::GetString(TIME_SEPARATOR)
             │
             ├─▶ DateTimeData::GetAmPmMarker(index, type)
             │         │
             │         ▼
             │    DataResource::GetString(AM_PM_MARKER)
             │
             └─▶ Format using pattern from i18n.dat
```

**关键文件**：
- `interfaces/kits/i18n/include/date_time_format.h` - 公开 API
- `frameworks/i18n/src/date_time_format.cpp` - 入口实现
- `frameworks/i18n/src/date_time_format_impl.cpp` - 核心实现
- `frameworks/i18n/src/data_resource.cpp` - 数据加载

### 6.1.2 数字格式化调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                      NumberFormat::Format() 调用链                     │
└─────────────────────────────────────────────────────────────────────┘

应用层
  │
  ▼
NumberFormat::Format(double/int num, NumberFormatType type, int& status)
  │
  ├─▶ NumberFormat::Init() ──▶ NumberFormatImpl::Init()
  │         │
  │         ▼
  │    DataResource::DataResource(localeInfo)
  │         │
  │         ▼
  │    Load number format data from i18n.dat
  │
  └─▶ NumberFormatImpl::Format(num, type, status)
             │
             ├─▶ Get decimal separator
             │         │
             │         ▼
             │    DataResource::GetString(DECIMAL_SEPARATOR)
             │
             ├─▶ Get grouping separator
             │         │
             │         ▼
             │    DataResource::GetString(GROUPING_SEPARATOR)
             │
             └─▶ Format number string
```

### 6.1.3 JS API 调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                      getLocale() JS API 调用链                        │
└─────────────────────────────────────────────────────────────────────┘

JavaScript 应用
  │
  ▼
import getLocale from '@ohos.i18n';
getLocale()
  │
  ▼
LocaleModule::GetLocale(JSIValue thisVal, JSIValue* args, uint8_t argsNum)
  │
  ├─▶ GetLanguage() ──▶ ace_malloc() ──▶ GLOBAL_GetLanguage()
  │                                                        │
  │   ◀────────────────────────────────────────────────────┘
  │
  ├─▶ GetRegion() ──▶ ace_malloc() ──▶ GLOBAL_GetRegion()
  │                                                │
  │   ◀────────────────────────────────────────────┘
  │
  └─▶ JSI::CreateObject()
             │
             ├─▶ JSI::SetStringProperty("language", lang)
             │         │
             │         ▼
             │    ace_free(lang)
             │
             ├─▶ JSI::SetStringProperty("countryOrRegion", region)
             │         │
             │         ▼
             │    ace_free(region)
             │
             └─▶ JSI::SetStringProperty("dir", TEXT_DIRECTION_LTR)
```

## 6.2 配置标志参考

### 6.2.1 GN 构建标志

| 标志 | 文件 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `i18n_lite_support_i18n_product` | `i18n_lite.gni` | bool | false | 产品级支持开关 |

**使用示例**：

```gni
# 在产品配置中启用
declare_args() {
  i18n_lite_support_i18n_product = true
}
```

### 6.2.2 编译宏定义

| 宏 | 定义位置 | 条件 | 说明 |
|----|----------|------|------|
| `I18N_PRODUCT` | `BUILD.gn:66` | `i18n_lite_support_i18n_product=true` | 产品级编译 |
| `_INC_STRING_S` | `BUILD.gn:17` | always | 使用安全字符串函数 |
| `_INC_WCHAR_S` | `BUILD.gn:18` | always | 使用安全宽字符函数 |

### 6.2.3 运行时配置

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `DATA_RESOURCE_PATH` | 路径字符串 | `system/i18n/i18n.dat` 或 `/storage/data/i18n.dat` | i18n.dat 文件路径 |

## 6.3 常见构建问题与解决方案

### 6.3.1 编译错误：找不到头文件

**错误信息**：
```
fatal error: 'date_time_format.h' file not found
```

**原因**：include_dirs 配置缺失

**解决方案**：
```bash
# 1. 检查 GN 配置
cat frameworks/i18n/BUILD.gn | grep -A5 "include_dirs"

# 2. 确保依赖正确
# 在目标 BUILD.gn 中添加：
deps = [
  "//base/global/i18n_lite/frameworks/i18n:global_i18n",
]
```

### 6.3.2 链接错误：未定义引用

**错误信息**：
```
undefined reference to `DateTimeFormat::Format(...)'
```

**原因**：未链接 i18n_lite 库

**解决方案**：
```bash
# 1. 检查链接库
# 在产品配置中添加：
libs = [
  "libglobal_i18n.a",
  "libsec_shared.a",
]

# 2. 检查链接顺序（依赖在前）
ld foo.o -lglobal_i18n -lsec
```

### 6.3.3 运行时崩溃：i18n.dat 找不到

**错误信息**：
```
i18n: cannot open data file
```

**原因**：i18n.dat 文件未部署到正确路径

**解决方案**：
```bash
# 1. 检查文件是否存在
ls -la system/i18n/i18n.dat

# 2. 手动复制文件
cp frameworks/i18n/i18n.dat system/i18n/

# 3. 检查构建配置
# 确保 i18n_dat target 被包含在构建中
```

### 6.3.4 区域回退异常

**问题**：请求的区域数据未正确回退到默认值

**诊断步骤**：
```cpp
// 1. 检查区域掩码
LocaleInfo locale("zh", "Hans", "CN");
uint32_t mask = locale.GetMask();
printf("Locale mask: %u\n", mask);

// 2. 检查回退机制
DataResource resource(&locale);
// 验证 resource.localeMask 是否正确

// 3. 检查 i18n.dat 是否包含目标区域
// 查看 tools/i18n-dat-tool/resources/locales.json
```

### 6.3.5 JS API 返回 undefined

**问题**：`getLocale()` 返回 undefined

**诊断步骤**：
```cpp
// 1. 检查 JS 引擎初始化
// 确保调用了 InitLocaleModule()

// 2. 检查系统 API
char lang[MAX_LANGUAGE_LENGTH];
if (GLOBAL_GetLanguage(lang, MAX_LANGUAGE_LENGTH) != 0) {
    printf("Failed to get system language\n");
}

// 3. 检查内存分配
char *lang = GetLanguage();
if (lang == nullptr) {
    printf("Memory allocation failed\n");
}
```

## 6.4 调试技巧

### 6.4.1 启用调试日志

在产品级编译中启用日志：

```gni
# BUILD.gn
if (i18n_lite_support_i18n_product) {
  defines = [ "I18N_PRODUCT" ]
}
```

```cpp
// 代码中使用
#ifdef I18N_PRODUCT
#include <log.h>
#define I18N_LOGI(...) HILOG_INFO(LOG_CORE, ##__VA_ARGS__)
#else
#define I18N_LOGI(...)
#endif
```

### 6.4.2 区域数据调试

```cpp
// 打印区域信息
void PrintLocaleInfo(const LocaleInfo& locale) {
    printf("ID: %s\n", locale.GetId());
    printf("Language: %s\n", locale.GetLanguage());
    printf("Script: %s\n", locale.GetScript() ? locale.GetScript() : "N/A");
    printf("Region: %s\n", locale.GetRegion());
    printf("Mask: %u\n", locale.GetMask());
    printf("Is Default: %s\n", locale.IsDefaultLocale() ? "Yes" : "No");
}

// 检查数据加载
void CheckDataResource(const LocaleInfo& locale) {
    DataResource resource(&locale);
    
    const char* timeSep = resource.GetString(DataResourceType::RESOURCE_TYPE_TIME_SEPARATOR);
    printf("Time separator: %s\n", timeSep ? timeSep : "N/A");
    
    const char* decimalSep = resource.GetString(DataResourceType::RESOURCE_TYPE_DECIMAL_SEPARATOR);
    printf("Decimal separator: %s\n", decimalSep ? decimalSep : "N/A");
}
```

### 6.4.3 格式化调试

```cpp
// 详细的格式化过程调试
void DebugDateTimeFormat() {
    LocaleInfo locale("zh", "CN");
    DateTimeFormat formatter(AvailableDateTimeFormatPattern::FULL, locale);
    
    time_t now = time(nullptr);
    std::string result;
    I18nStatus status = I18nStatus::ISUCCESS;
    
    printf("Formatting time: %ld\n", now);
    printf("Pattern: %d\n", AvailableDateTimeFormatPattern::FULL);
    
    formatter.Format(now, "+8:00", result, status);
    
    printf("Result: %s\n", result.c_str());
    printf("Status: %d\n", status);
}
```

## 6.5 支持的语言列表

### 6.5.1 完整语言列表（部分）

| 代码 | 语言 | 代码 | 语言 | 代码 | 语言 |
|------|------|------|------|------|------|
| `am_ET` | 阿姆哈拉语 | `hr_HR` | 克罗地亚语 | `or_IN` | 奥里亚语 |
| `ar_EG` | 阿拉伯语 | `hu_HU` | 匈牙利语 | `pa_IN` | 旁遮普语 |
| `as_IN` | 阿萨姆语 | `in_ID` | 印尼语 | `pl_PL` | 波兰语 |
| `az_AZ` | 阿塞拜疆语 | `it_IT` | 意大利语 | `pt_BR` | 巴西葡萄牙语 |
| `be_BY` | 白俄罗斯语 | `iw_IL` | 希伯来语 | `pt_PT` | 欧洲葡萄牙语 |
| `bg_BG` | 保加利亚语 | `ja_JP` | 日语 | `ro_RO` | 罗马尼亚语 |
| `bn_BD` | 孟加拉语 | `jv_ID` | 爪哇语 | `ru_RU` | 俄语 |
| `bo_CN` | 藏语 | `ka_GE` | 格鲁吉亚语 | `si_LK` | 僧伽罗语 |
| `bs_BA` | 波斯尼亚语 | `kk_KZ` | 哈萨克语 | `sk_SK` | 斯洛伐克语 |
| `ca_ES` | 加泰罗尼亚语 | `km_KH` | 高棉语 | `sl_SI` | 斯洛文尼亚语 |
| `cs_CZ` | 捷克语 | `kn_IN` | 坎纳达语 | `sr_Latn_RS` | 塞尔维亚语（拉丁） |
| `da_DK` | 丹麦语 | `ko_KR` | 韩语 | `sv_SE` | 瑞典语 |
| `de_DE` | 德语 | `lo_LA` | 老挝语 | `sw_TZ` | 斯瓦希里语 |
| `el_GR` | 希腊语 | `lt_LT` | 立陶宛语 | `ta_IN` | 泰米尔语 |
| `en_GB` | 英语（英国） | `lv_LV` | 拉脱维亚语 | `te_IN` | 泰卢固语 |
| `en_US` | 英语（美国） | `mk_MK` | 马其顿语 | `th_TH` | 泰语 |
| `es_ES` | 西班牙语（欧洲） | `ml_IN` | 马拉雅拉姆语 | `tl_PH` | 他加禄语 |
| `es_US` | 西班牙语（拉美） | `mn_MN` | 蒙古语 | `tr_TR` | 土耳其语 |
| `et_EE` | 爱沙尼亚语 | `mr_IN` | 马拉地语 | `uk_UA` | 乌克兰语 |
| `eu_ES` | 巴斯克语 | `ms_MY` | 马来语 | `ur_PK` | 乌尔都语 |
| `fa_IR` | 波斯语 | `my_MM` | 缅甸语 | `uz_UZ` | 乌兹别克语 |
| `fi_FI` | 芬兰语 | `nb_NO` | 挪威语（博克马尔） | `vi_VN` | 越南语 |
| `fr_FR` | 法语 | `ne_NP` | 尼泊尔语 | `zh_CN` | 简体中文 |
| `gl_ES` | 加利西亚语 | `nl_NL` | 荷兰语 | `zh_HK` | 繁体中文（香港） |
| `gu_IN` | 古吉拉特语 | `zh_TW` | 繁体中文（台湾） |

### 6.5.2 特殊语言

| 代码 | 语言 | 说明 |
|------|------|------|
| `zh_Hans` | 简体中文（无地区） | |
| `zh_Hant` | 繁体中文（无地区） | |
| `en` | 英语（无地区） | 默认英语 |
| `bqi` | 俾路支语 | 右到左书写 |

## 6.6 版本历史

### 6.6.1 i18n_lite 版本

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| 1.0.0 | 2021-2022 | 初始版本，支持核心 i18n 功能 |

### 6.6.2 API 兼容性

| 版本 | API 稳定性 | 变更类型 |
|------|------------|----------|
| 1.0.0 | 稳定 | 初始发布 |

## 6.7 参考资料

### 6.7.1 内部文档

| 文档 | 位置 | 说明 |
|------|------|------|
| 项目 README | `README.md` | 项目概述 |
| bundle 配置 | `bundle.json` | 组件元数据 |
| GN 配置 | `i18n_lite.gni` | 构建标志 |

### 6.7.2 外部标准

| 标准 | 说明 | URL |
|------|------|-----|
| BCP 47 | 语言标签 | https://tools.ietf.org/html/bcp47 |
| Unicode CLDR | 语言环境数据 | http://cldr.unicode.org/ |
| ISO 639 | 语言代码 | https://www.iso.org/iso-639-language-codes |
| ISO 15924 | 脚本代码 | https://www.iso.org/iso-15924-script-codes |
| ISO 3166 | 国家/地区代码 | https://www.iso.org/iso-3166-country-codes |

### 6.7.3 相关仓库

| 仓库 | 说明 |
|------|------|
| `global_resmgr_lite` | 资源管理 |
| `bounds_checking_function` | 内存安全函数 |
| `ace_engine_lite` | ACELite JS 引擎 |

---

*最后更新：2026-02-06*
