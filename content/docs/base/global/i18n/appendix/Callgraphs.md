# 关键调用链

## 1. N-API 调用入口

### @ohos/i18n 模块

```mermaid
sequenceDiagram
    participant JS as "JS 应用"
    participant NAPI as "libi18n.so<br/>(i18n_addon.cpp)"
    participant Core as "libintl_util.so"

    JS->>NAPI: import i18n from '@ohos/i18n'
    JS->>NAPI: getSystemLocale()
    NAPI->>Core: LocaleConfig::GetSystemLocale()
    Core-->>NAPI: locale string
    NAPI-->>JS: "zh-CN"

    JS->>NAPI: set24HourClock(true)
    Note right of NAPI: 权限检查在 SA 侧
    NAPI->>SA: IPC: Set24HourClock()
    SA->>Core: LocaleConfig::Set24HourClock()
    SA-->>NAPI: result
    NAPI-->>JS: true
```

### @ohos/intl 模块

```mermaid
sequenceDiagram
    participant JS as "JS 应用"
    participant NAPI as "libintl.so<br/>(intl_addon.cpp)"
    participant Core as "libintl_util.so"

    JS->>NAPI: new DateTimeFormat('zh-CN', {...})
    NAPI->>Core: DateTimeFormat::CreateInstance(localeTags, options)
    Core-->>NAPI: DateTimeFormat instance
    NAPI-->>JS: DateTimeFormat object

    JS->>NAPI: format(date)
    NAPI->>Core: DateTimeFormat::Format(date)
    Core-->>NAPI: formatted string
    NAPI-->>JS: "2024年1月1日"
```

## 2. SA 服务调用链

### I18nServiceAbility 初始化

```mermaid
sequenceDiagram
    participant Client as "libi18n_sa_client.so"
    participant SM as "SAMgr"
    participant SA as "I18nServiceAbility"

    Client->>SM: GetSystemAbility(I18N_SA_ID)
    SM->>SA: OnStart()
    SA->>SA: Register to SAMgr
    SA-->>Client: SA proxy
```

### 配置修改流程

```mermaid
sequenceDiagram
    participant JS as "JS 应用"
    participant NAPI as "libi18n.so"
    participant IPC as "IPC/Binder"
    participant SA as "I18nServiceAbility"

    JS->>NAPI: set24HourClock(true)
    NAPI->>IPC: SendRequest(CODE_SET_24_HOUR_CLOCK)
    IPC->>SA: OnRemoteRequest()
    SA->>SA: CheckPermission()
    Note right of SA: Verify UPDATE_CONFIGURATION
    SA->>SA: LocaleConfig::Set24HourClock(true)
    SA-->>IPC: ERR_OK
    IPC-->>NAPI: result
    NAPI-->>JS: true
```

## 3. 核心模块调用

### LocaleConfig 读写

```mermaid
graph LR
    A["LocaleConfig::GetSystemLocale()<br/>(frameworks/intl/src/locale_config.cpp)"]
    B["Preferences<br/>(preferences 服务)"]
    C["ICU API<br/>(libicui18n)"]

    A --> B
    A --> C
```

### DateTimeFormat 格式化

```mermaid
graph TD
    A["DateTimeFormat::Format()<br/>(frameworks/intl/src/date_time_format.cpp)"]
    B["icu::DateFormat::format()<br/>(ICU)"]
    C["LocaleData<br/>(frameworks/intl/src/locale_data.cpp)"]
    D["TimeZone<br/>(frameworks/intl/src/i18n_timezone.cpp)"]

    A --> B
    A --> C
    A --> D
```

## 4. 实体识别调用

### 日期时间识别

```mermaid
graph TD
    A["EntityRecognizer::Recognize()<br/>(entity_recognition/src/entity_recognizer.cpp)"]
    B["DateTimeRecognition<br/>(date_time_recognition/)"]
    C["RulesEngine<br/>(date_time_recognition/src/rules_engine.cpp)"]
    D["DateTimeFilter<br/>(date_time_recognition/src/date_time_filter.cpp)"]

    A --> B
    B --> C
    B --> D
```

## 5. NDK 接口调用

```mermaid
sequenceDiagram
    participant NDK as "Native 应用"
    participant NativeAPI as "libnative_i18n.so<br/>(ndk/src/)"
    participant Core as "libintl_util.so"

    NDK->>NativeAPI: OH_I18n_GetSystemLocale()
    NativeAPI->>Core: LocaleConfig::GetSystemLocale()
    Core-->>NativeAPI: locale string
    NativeAPI-->>NDK: const char*
```
