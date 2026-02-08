# 对外 API（N-API/ANI/CJ FFI）

> Settings 应用的完整对外 API 文档

---

## 目的

本文档详细说明 Settings 应用提供的所有对外 API，包括 N-API、ANI 和 CJ FFI。

## 适用范围

- 目标读者：应用开发者
- 项目：@ohos/settings (Settings 3.1)
- API 版本：API 23

---

## API 模块概览

| API 类型 | 模块名 | 文档位置 |
|----------|--------|----------|
| N-API | settings | 本节 |
| N-API | intelligentscene | [智能场景](#智能场景-intelligentscene) |
| ANI | settings_ani | 本节 |
| ANI | intelligentscene_ani | [智能场景](#智能场景-intelligentscene) |
| CJ FFI | cj_settings_ffi | [CJ FFI](#cj-ffi) |

---

## N-API 模块：settings

### 模块信息

- **模块名**：settings
- **JS 命名空间**：@ohos.settings
- **注册点**：napi/settings/native_module.cpp:77
- **版本**：3.1
- **权限**：ohos.permission.UPDATE_CONFIGURATION（应用层）

### API 清单表

| API 名称 | 同步/异步 | 参数 | 返回值 | 实现函数 | 文件位置 |
|----------|-----------|--------|--------|-----------|----------|
| getURI | 异步 | name?, domainName? | Promise&lt;string&gt; | napi_get_uri | napi_settings.h:125 |
| getValue | 异步 | name, domainName? | Promise&lt;string&gt; | napi_get_value | napi_settings.h:141 |
| setValue | 异步 | name, value, domainName? | Promise&lt;void&gt; | napi_set_value | napi_settings.h:158 |
| getUriSync | 同步 | name?, domainName? | string | napi_get_uri_sync | napi_settings.h:133 |
| getValueSync | 同步 | name, domainName? | string | napi_get_value_sync | napi_settings.h:150 |
| setValueSync | 同步 | name, value, domainName? | void | napi_set_value_sync | napi_settings.h:167 |
| enableAirplaneMode | 异步 | enable (boolean) | Promise&lt;void&gt; | napi_enable_airplane_mode | napi_settings.h:183 |
| canShowFloating | 异步 | - | Promise&lt;boolean&gt; | napi_can_show_floating | napi_settings.h:191 |
| registerKeyObserver | 异步 | name, domainName?, observer (Object) | Promise&lt;void&gt; | napi_register_key_observer | napi_settings.h:200 |
| unregisterKeyObserver | 异步 | name, domainName? | Promise&lt;void&gt; | napi_unregister_key_observer | napi_settings.h:201 |
| openNetworkManagerSettings | 异步 | - | Promise&lt;void&gt; | opne_manager_settings | native_module.cpp:46 |
| openInputMethodSettings | 异步 | - | Promise&lt;void&gt; | openInputMethodSettings | native_module.cpp:47 |
| openInputMethodDetail | 异步 | - | Promise&lt;void&gt; | openInputMethodDetail | native_module.cpp:48 |

### 调用链：getValue

```
JS 应用
  ↓
settings.getValue("brightness", "system")
  ↓
napi_get_value() [napi/settings/napi_settings.cpp]
  ↓
unwrap 参数（name: "brightness", domainName: "system"）
  ↓
调用 DataShareHelper 查询
  ↓
DataAbility: SELECT KEYWORD, VALUE FROM global WHERE KEYWORD = "brightness"
  ↓
返回值（Promise）
```

### 调用链：setValue

```
JS 应用
  ↓
settings.setValue("brightness", "100", "system")
  ↓
napi_set_value() [napi/settings/napi_settings.cpp]
  ↓
unwrap 参数
  ↓
调用 DataShareHelper 更新
  ↓
DataAbility: UPDATE SET VALUE = "100" WHERE KEYWORD = "brightness"
  ↓
触发观察者
  ↓
返回状态（Promise）
```

### 错误处理

| 错误码 | 说明 | 触发条件 |
|----------|------|-----------|
| 201 | 权限被拒绝 | 权限不足时 |
| -1 | 查询失败 | DataShare 返回失败 |
| -2 | 权限拒绝 | 检查权限失败 |
| 29189, 32 | DataShare 死亡 | DataShare 服务不可用 |

---

## N-API 模块：intelligentscene

### 模块信息

- **模块名**：intelligentscene
- **JS 命名空间**：@ohos.intelligentscene
- **注册点**：napi/intelligentscene/native_module.cpp:59
- **权限**：无（依赖系统权限）

### API 清单表

| API 名称 | 同步/异步 | 参数 | 返回值 | 实现函数 | 文件位置 |
|----------|-----------|--------|--------|-----------|----------|
| isDoNotDisturbEnabled | 同步 | - | boolean | napi_is_do_not_disturb_enabled | native_module.cpp:31 |
| isNotifyAllowedInDoNotDisturb | 同步 | - | boolean | napi_is_notify_allowed | native_module.cpp:32 |

### 调用链

```
JS 应用
  ↓
intelligentscene.isDoNotDisturbEnabled()
  ↓
调用通知服务（ANS Innerkits）
  ↓
查询免打扰状态
  ↓
返回 boolean
```

### 错误处理

| 错误码 | 说明 |
|----------|------|
| 201 | 权限被拒绝 |
| 801 | SystemCapability 未找到 |
| 35200001 | 内部错误 |
| 1600002 | 序列化/反序列化错误 |
| 1600003 | 连接服务失败 |

---

## ANI 模块：settings_ani

### 模块信息

- **模块名**：settings（ANI）
- **ArkTS 文件**：ani/settings/ets/@ohos.settings.ets
- **注册点**：ani/settings/ani_settings.cpp:606
- **产物**：libsettings_ani.z.so, settings.abc

### API 清单表

| ANI 函数 | 同步/异步 | 参数 | 返回值 | 文件位置 |
|-----------|-----------|--------|--------|----------|
| ani_get_value | 异步 | env, context, name, domainName | ani_string | ani_settings.h:73 |
| ani_get_value_ext | 异步 | env, context, name, domainName | ani_string | ani_settings.h:74 |
| ani_set_value | 异步 | env, context, name, value, domainName | ani_boolean | ani_settings.h:75 |
| ani_set_value_ext | 异步 | env, context, name, value, domainName | ani_boolean | ani_settings.h:76 |
| ani_get_value_sync | 同步 | env, context, key, defaultValue, domainName | ani_string | ani_settings.h:80 |
| ani_get_value_sync_ext | 同步 | env, context, key, defaultValue, domainName | ani_string | ani_settings.h:88 |
| ani_set_value_sync | 同步 | env, context, key, value, domainName | ani_boolean | ani_settings.h:89 |
| ani_set_value_sync_ext | 同步 | env, context, key, value, domainName | ani_boolean | ani_settings.h:91 |
| ani_get_uri_sync | 同步 | env, key | ani_string | ani_settings.h:94 |
| ani_register_key_observer | 异步 | env, context, name, domainName, observer | ani_boolean | ani_settings.h:95 |
| ani_unregister_key_observer | 异步 | env, context, name, domainName | ani_boolean | ani_settings.h:97 |
| ani_enable_airplane_mode | 异步 | env, enable | void | ani_settings.h:78 |
| ani_can_show_floating | 同步 | env | ani_boolean | ani_settings.h:79 |

---

## ANI 模块：intelligentscene_ani

### 模块信息

- **模块名**：intelligentscene（ANI）
- **ArkTS 文件**：ani/intelligentscene/ets/@ohos.intelligentscene.ets
- **注册点**：ani/intelligentscene/ani_init_module.cpp（待确认）
- **产物**：libintelligentscene_ani.z.so, intelligentscene.abc

### API 清单表

| ANI 函数 | 同步/异步 | 参数 | 返回值 | 文件位置 |
|-----------|-----------|--------|--------|----------|
| ani_is_do_not_disturb_enabled | 同步 | env | ani_boolean | ani_nodisturb.h:23 |
| ani_is_notify_allowed | 同步 | env | ani_boolean | ani_nodisturb.h:25 |

---

## CJ FFI 模块

### 模块信息

- **模块名**：cj_settings_ffi
- **C ABI 导出**：extern "C"
- **稳定性**：platformsdk（innerapi_tags）
- **产物**：libcj_settings_ffi.z.so

### API 清单表

| FFI 函数 | 同步/异步 | 参数 | 返回值 | 文件位置 |
|-----------|-----------|--------|--------|----------|
| FfiSettingsSetValue | 同步 | context, name, value, domainName, ret (out) | bool | settings_ffi.h:24 |
| FfiSettingsGetValue | 同步 | context, name, value (out), domainName, ret (out) | char* | settings_ffi.h:27 |
| FfiSettingsRegisterKeyObserver | 同步 | context, name, domainName, observer | bool | settings_ffi.h:30 |
| FfiSettingsUnregisterKeyObserver | 同步 | context, name, domainName | bool | settings_ffi.h:32 |
| FfiSettingsGetUriSync | 同步 | name, tableName | char* | settings_ffi.h:34 |

### 内部 API

详见 [05_Inner_API.md](05_Inner_API.md)。

---

## 参数规范

### 通用参数

| 参数 | 类型 | 说明 | 必填 | 默认值 |
|------|------|------|--------|--------|
| name | string | 设置键名 | 是 | - |
| value | string | 设置值 | 否 | - |
| domainName | string | 域名（表名） | 否 | "global" |
| defaultValue | string | 默认值 | 否 | - |
| observer | Object | 观察者对象 | 否 | - |
| enable | boolean | 启用/禁用 | 是 | - |

### 域名（domainName）规范

| 域名 | 说明 | 访问权限 |
|--------|------|----------|
| "global" | 全局设置（默认） | 所有应用可读，部分可写需权限 |
| "system" | 系统设置 | 仅系统应用可写 |
| "secure" | 安全设置 | 需特殊权限 |

---

## 相关跳转

- **[00_Overview.md](00_Overview.md)** - 项目概览
- **[01_Positioning.md](01_Positioning.md)** - 项目定位与边界
- **[03_Architecture.md](03_Architecture.md)** - 架构说明
- **[05_Inner_API.md](05_Inner_API.md)** - 内部 API 文档

---

**最后更新**：2026-02-06 00:11:23
