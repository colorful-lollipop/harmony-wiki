# i18n 模块概览

## 项目定位

i18n (Internationalization) 模块是 OpenHarmony 的全球化子系统核心组件，提供完整的国际化支持能力。

**核心能力**:
- 日期/时间格式化 (`DateTimeFormat`)
- 数字/货币格式化 (`NumberFormat`, `SimpleNumberFormat`)
- 区域敏感比较 (`Collator`)
- 相对时间格式化 (`RelativeTimeFormat`)
- 复数规则 (`PluralRules`)
- 时区管理 (`TimeZone`)
- 节假日管理 (`HolidayManager`)
- 电话号码格式化/解析 (`PhoneNumberFormat`)
- 实体识别 (日期/时间/电话号码)
- 系统语言/区域配置管理

## 运行环境

- **系统能力**: `SystemCapability.Global.I18n`
- **依赖组件**:
  - `ability_base`, `ability_runtime` (UI 支持)
  - `icu` (Unicode 支持)
  - `ipc`, `samgr` (进程间通信)
  - `access_token` (权限管理)
  - `bundle_framework` (应用信息)

## 目录结构

```
/base/global/i18n/
├── frameworks/          # 核心框架实现
│   ├── intl/           # 主要功能实现 (日期/时间/数字/区域等)
│   └── zone/           # 时区工具
├── interfaces/         # 对外接口
│   ├── js/             # JS N-API (kits/对外, innerkits/内部)
│   ├── native/         # Native API (inner_api)
│   ├── ets/            # ETS/ANI (ArkTS)
│   └── cj/             # C-JS FFI
├── services/           # SA 服务实现
├── ndk/                # NDK 接口
├── sa_profile/         # SA 配置文件
└── tools/              # 工具脚本
```

## 关键概念

### 区域标签 (Locale Tag)

遵循 [BCP 47](https://tools.ietf.org/html/bcp47) 标准，例如：
- `zh-CN`: 简体中文，中国
- `en-US`: 美式英语，美国
- `zh-Hant-HK`: 繁体中文，香港

### I18nServiceAbility

系统能力 `I18N_SA_ID`，负责：
- 系统语言/区域配置管理
- 权限校验 (需 `ohos.permission.UPDATE_CONFIGURATION`)

### 线程模型

- **JS 主线程**: N-API 调用在 JS 线程执行
- **SA 调用**: 跨进程调用通过 IPC 完成
- **内部处理**: ICU 操作在调用线程执行

## 快速示例

```javascript
// 日期格式化
import Intl from '@ohos.intl';
const dateFmt = new Intl.DateTimeFormat('zh-CN', { dateStyle: 'full' });
console.log(dateFmt.format(new Date())); // "2024年1月1日星期一"

// 获取系统区域
import i18n from '@ohos/i18n';
const locale = i18n.getSystemLocale();
console.log(locale); // "zh-CN"
```

## 相关文档

- [N-API 接口](N-API.md)
- [架构说明](Architecture.md)
- [构建与产物](Build.md)
- [安全评审](Security_Review.md)
