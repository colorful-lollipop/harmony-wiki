# i18n_lite Wiki 首页

> OpenHarmony i18n_lite 国际化模块工程文档

## 项目简介

**i18n_lite** 是 OpenHarmony Globalization 子系统的核心轻量级国际化组件，提供日期时间格式化、数字格式化、复数规则处理、区域信息管理等能力。

### 核心能力

| 能力 | 说明 | 关键类 |
|------|------|--------|
| 日期时间格式化 | 按区域习惯格式化日期和时间 | `DateTimeFormat` |
| 数字格式化 | 按区域习惯格式化数字和百分比 | `NumberFormat` |
| 复数规则 | 处理不同语言的复数形式 | `PluralFormat` |
| 区域信息 | 管理语言/脚本/地区信息 | `LocaleInfo` |
| JavaScript API | 为 ACELite 提供 JS 接口 | `LocaleModule` |

## 快速导航

### 📚 核心文档

- [项目概述](./01_Overview.md) - 深入了解项目定位和架构
- [API 接口](./02_API.md) - 完整的 N-API 和 C++ API 参考
- [架构设计](./03_Architecture.md) - 组件结构、数据流分析
- [构建配置](./04_Build.md) - GN 构建系统详解
- [安全分析](./05_Security.md) - 安全风险评审
- [常见问题](./06_Appendix.md) - 附录和问题定位

### 🔗 关键链接

- [项目仓库](https://gitee.com/openharmony/global_i18n_lite)
- [官方文档](https://gitee.com/openharmony/docs)
- [Globalization 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/globalization.md)

## 目录结构概览

```
i18n_lite/
├── frameworks/i18n/           # 核心实现
│   ├── include/              # 内部头文件
│   ├── src/                  # 实现代码
│   └── i18n.dat             # 区域数据文件
├── interfaces/               # API 接口
│   ├── kits/i18n/           # C++ API
│   └── kits/js/builtin/     # JS API (ACELite)
└── tools/i18n-dat-tool/      # 数据生成工具
```

## 支持的语言

i18n_lite 支持超过 **150 种语言/区域**，包括：

- 简体中文 (zh_CN)
- 繁体中文 (zh_TW, zh_HK)
- 英语 (en_US, en_GB)
- 日语、韩语、阿拉伯语等

完整列表见 [附录](./06_Appendix.md#支持的语言列表)。

## 适配系统

- **Mini System**：轻量级设备
- **Small System**：小型设备

## 开始使用

### C++ 开发

```cpp
#include "date_time_format.h"
#include "locale_info.h"

using namespace OHOS::I18N;

LocaleInfo locale("zh", "Hans", "CN");
DateTimeFormat formatter(AvailableDateTimeFormatPattern::HOUR_MINUTE, locale);

time_t time = 3600 * 3;
std::string zoneInfo = "+1:00";
std::string out;
Ii8nStatus status = Ii8nStatus::ISUCCESS;
formatter.Format(time, zoneInfo, out, status);
// 输出: 4:00
```

### JavaScript 开发 (ACELite)

```javascript
import getLocale from '@ohos.i18n';
const locale = getLocale();
console.log(language);      // 如 "zh"
console.log(countryOrRegion); // 如 "CN"
console.log(dir);            // 如 "ltr"
```

## 文档贡献

本 Wiki 由 i18n_lite 维护团队维护，欢迎贡献：

- 发现错误？请提交 Issue
- 改进建议？请创建 PR
- 新人指引：参见 [README.md](./README.md)

## 版本信息

- **Wiki 版本**：1.0.0
- **组件版本**：i18n_lite 1.0.0
- **最后更新**：2026-02-06
