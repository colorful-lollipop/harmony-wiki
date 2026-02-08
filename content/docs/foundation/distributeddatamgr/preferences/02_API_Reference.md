# API 参考

本文档提供 Preferences 模块的完整 API 清单，包括 JS API、NDK API 和 Inner API。

## API 汇总表

### JS API（4 个模块）

| 模块 | 模块名 | 入口文件 | 主要 API 数量 |
|------|----------|----------|---------------|
| preferences | `data.preferences` | `frameworks/js/napi/preferences/src/entry_point.cpp:49` | 10+ |
| sendable_preferences | - | `frameworks/js/napi/sendable_preferences/src/entry_point.cpp:50` | 10+ |
| storage | - | `frameworks/js/napi/storage/src/entry_point_storage.cpp:50` | 1 |
| system_storage | - | `frameworks/js/napi/system_storage/src/entry_point_system_storage.cpp:45` | 1 |

### NDK C API

| 类别 | 函数数量 | 头文件 | 库文件 |
|------|----------|---------|---------|
| 实例管理 | 3 | `interfaces/ndk/include/oh_preferences.h` | `libohpreferences.so` |
| 数据操作 | 6 | `interfaces/ndk/include/oh_preferences.h` | `libohpreferences.so` |
| 观察者 | 2 | `interfaces/ndk/include/oh_preferences.h` | `libohpreferences.so` |
| **总计** | **11** | - | - |

### Inner API（C++）

| 类别 | 方法数量 | 头文件 |
|------|----------|---------|
| Preferences 接口 | 10+ | `interfaces/inner_api/include/preferences.h` |
| PreferencesHelper | 4 | `interfaces/inner_api/include/preferences_helper.h` |
| **总计** | **14+** | - |

---

## 目录

- [JS API](#js-api)
  - [Preferences 模块](#preferences-模块)
  - [Storage 模块](#storage-模块)
  - [System Storage 模块](#system-storage-模块)
- [NDK API](#ndk-api)
  - [C 接口](#c-接口)
- [Inner API](#inner-api)
  - [C++ 接口](#c-接口-1)
- [错误码汇总](#错误码汇总)

---

## JS API

JS API 通过 `@ohos.data.preferences` 模块提供。

### Preferences 模块

**模块名**：`data.preferences`

**注册位置**：`frameworks/js/napi/preferences/src/entry_point.cpp:49`

#### 常量

| 常量名 | 类型 | 值 | 说明 |
|--------|------|-----|------|
| `MAX_KEY_LENGTH` | number | 1024 | Key 最大长度 |
| `MAX_VALUE_LENGTH` | number | 16777216 | Value 最大长度（16MB） |
| `StorageType.XML` | enum | 0 | XML 存储类型 |
| `StorageType.GSKV` | enum | 1 | GSKV 存储类型 |

#### getPreferences

获取指定名称的 Preferences 实例。

**签名**：
```typescript
function getPreferences(context: Context, name: string, options?: Options): Promise<Preferences>;
```

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| context | Context | 是 | 应用上下文 |
| name | string | 是 | Preferences 名称 |
| options | Options | 否 | 可选配置 |

**Options**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| dataGroupId | string | 否 | 数据组 ID |
| storageType | StorageType | 否 | 存储类型，默认为 XML |

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<Preferences\> | Preferences 实例 |

**异步模式**：Promise

**C++ 实现**：`frameworks/js/napi/preferences/src/napi_preferences_helper.cpp:112-135`

**调用链**：
```
JS → GetPreferences → HelperAsyncContext → PreferencesHelper::GetPreferences → PreferencesImpl
```

---

#### getPreferencesSync

同步获取指定名称的 Preferences 实例。

**签名**：
```typescript
function getPreferencesSync(context: Context, name: string, options?: Options): Preferences;
```

**参数**：同 `getPreferences`

**返回值**：

| 类型 | 说明 |
|------|------|
| Preferences | Preferences 实例 |

**同步模式**：同步阻塞

---

#### deletePreferences

删除指定名称的 Preferences 实例及其文件。

**签名**：
```typescript
function deletePreferences(context: Context, name: string, options?: Options): Promise<void>;
```

**参数**：同 `getPreferences`

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<void\> | 删除完成后 resolve |

**C++ 实现**：`napi_preferences_helper.cpp:137-156`

---

#### deletePreferencesSync

同步删除指定名称的 Preferences 实例及其文件。

**签名**：
```typescript
function deletePreferencesSync(context: Context, name: string, options?: Options): void;
```

**参数**：同 `getPreferences`

**返回值**：void

---

#### removePreferencesFromCache

从内存中移除指定名称的 Preferences 实例（不删除文件）。

**签名**：
```typescript
function removePreferencesFromCache(context: Context, name: string, options?: Options): Promise<void>;
```

**参数**：同 `getPreferences`

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<void\> | 移除完成后 resolve |

**C++ 实现**：`napi_preferences_helper.cpp:158-177`

---

#### removePreferencesFromCacheSync

同步从内存中移除指定名称的 Preferences 实例。

**签名**：
```typescript
function removePreferencesFromCacheSync(context: Context, name: string, options?: Options): void;
```

**参数**：同 `getPreferences`

**返回值**：void

---

#### isStorageTypeSupported

检查指定的存储类型是否支持。

**签名**：
```typescript
function isStorageTypeSupported(storageType: StorageType): boolean;
```

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| storageType | StorageType | 是 | 存储类型 |

**返回值**：

| 类型 | 说明 |
|------|------|
| boolean | true=支持，false=不支持 |

**C++ 实现**：`napi_preferences_helper.cpp:179-211`

---

### Preferences 实例方法

getPreferences 返回的 Preferences 对象具有以下方法：

#### put

写入键值对。

**签名**：
```typescript
function put(key: string, value: string | number | boolean): Promise<void>;
```

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| key | string | 是 | 键，非空，最大 1024 字符 |
| value | string \| number \| boolean | 是 | 值，String 类型最大 16MB |

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<void\> | 写入完成后 resolve |

**C++ 实现**：`frameworks/js/napi/preferences/src/napi_preferences.cpp:76-93`（`putSync` 方法）

---

#### get

读取键值。

**签名**：
```typescript
function get(key: string, defValue: string | number | boolean): Promise<string | number | boolean>;
```

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| key | string | 是 | 键 |
| defValue | string \| number \| boolean | 是 | 默认值 |

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<string \| number \| boolean\> | 值或默认值 |

---

#### getAll

获取所有键值对。

**签名**：
```typescript
function getAll(): Promise<Object>;
```

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<Object\> | 所有键值对对象 |

---

#### hasKey

检查键是否存在。

**签名**：
```typescript
function hasKey(key: string): Promise<boolean>;
```

**返回值**：

| 类型 | 说明 |
|------|------|
| Promise\<boolean\> | true=存在，false=不存在 |

---

#### delete

删除键。

**签名**：
```typescript
function delete(key: string): Promise<void>;
```

---

#### clear

清空所有数据。

**签名**：
```typescript
function clear(): Promise<void>;
```

---

#### flush

异步持久化数据到磁盘。

**签名**：
```typescript
function flush(): Promise<void>;
```

**C++ 实现**：`frameworks/js/napi/preferences/src/napi_preferences.cpp:76-93`（`flush` 方法）

---

#### flushSync

同步持久化数据到磁盘。

**签名**：
```typescript
function flushSync(): void;
```

**C++ 实现**：`frameworks/js/napi/preferences/src/napi_preferences.cpp:76-93`（`flushSync` 方法）

---

#### on('change')

监听数据变化。

**签名**：
```typescript
function on(type: 'change', listener: (preferences: Preferences) => void): void;
```

**参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| type | string | 是 | 固定值 'change' |
| listener | function | 是 | 回调函数 |

---

#### off('change')

取消监听。

**签名**：
```typescript
function off(type: 'change', listener?: (preferences: Preferences) => void): void;
```

---

### Storage 模块

**模块名**：由 `frameworks/js/napi/storage/src/entry_point_storage.cpp:50` 注册

**导出功能**：

| API | 说明 |
|-----|------|
| `getStorageFromPath(path)` | 从指定路径获取存储对象 |

### System Storage 模块

**模块名**：由 `frameworks/js/napi/system_storage/src/entry_point_system_storage.cpp:45` 注册

**导出功能**：

| API | 说明 |
|-----|------|
| `getSystemPreferences()` | 获取系统级首选项 |

---

## NDK API

NDK API 提供 C 语言接口，供 Native 应用调用。

### C 接口

**头文件**：`interfaces/ndk/include/oh_preferences.h`

**库文件**：`libohpreferences.so`

#### OH_Preferences_Open

打开 Preferences 实例。

**签名**：
```c
OH_Preferences *OH_Preferences_Open(OH_PreferencesOption *option, int *errCode);
```

**参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| option | OH_PreferencesOption* | 选项对象 |
| errCode | int* | 错误码输出 |

**返回值**：

| 类型 | 说明 |
|------|------|
| OH_Preferences* | 成功返回实例指针，失败返回 nullptr |

**错误码**：

| 错误码 | 说明 |
|--------|------|
| `PREFERENCES_OK` | 成功 |
| `PREFERENCES_ERROR_INVALID_PARAM` | 无效参数 |
| `PREFERENCES_ERROR_NOT_SUPPORTED` | 不支持 |
| `PREFERENCES_ERROR_DELETE_FILE` | 删除文件失败 |
| `PREFERENCES_ERROR_STORAGE` | 存储错误 |
| `PREFERENCES_ERROR_MALLOC` | 内存分配失败 |

**C++ 实现**：`frameworks/ndk/src/oh_preferences.cpp:68-100`

---

#### OH_Preferences_Close

关闭 Preferences 实例。

**签名**：
```c
int OH_Preferences_Close(OH_Preferences *preference);
```

---

#### OH_Preferences_DeletePreferences

删除 Preferences。

**签名**：
```c
int OH_Preferences_DeletePreferences(OH_PreferencesOption *option);
```

---

#### OH_Preferences_GetInt

获取 int 类型值。

**签名**：
```c
int OH_Preferences_GetInt(OH_Preferences *preference, const char *key, int *value);
```

---

#### OH_Preferences_GetString

获取 string 类型值。

**签名**：
```c
int OH_Preferences_GetString(OH_Preferences *preference, const char *key, char **value, uint32_t *valueLen);
```

**注意**：调用完成后需要调用 `OH_Preferences_FreeString` 释放内存

---

#### OH_Preferences_FreeString

释放字符串内存。

**签名**：
```c
void OH_Preferences_FreeString(char *string);
```

---

#### OH_Preferences_SetInt

设置 int 类型值。

**签名**：
```c
int OH_Preferences_SetInt(OH_Preferences *preference, const char *key, int value);
```

---

#### OH_Preferences_SetString

设置 string 类型值。

**签名**：
```c
int OH_Preferences_SetString(OH_Preferences *preference, const char *key, const char *value);
```

---

#### OH_Preferences_Delete

删除键。

**签名**：
```c
int OH_Preferences_Delete(OH_Preferences *preference, const char *key);
```

---

#### OH_Preferences_Clear

清空所有数据。

**签名**：
```c
int OH_Preferences_Clear(OH_Preferences *preference);
```

---

#### OH_Preferences_Flush

持久化数据。

**签名**：
```c
int OH_Preferences_Flush(OH_Preferences *preference);
```

---

#### OH_Preferences_RegisterObserver

注册观察者。

**签名**：
```c
int OH_Preferences_RegisterObserver(OH_Preferences *preference, OH_PreferencesDataObserver observer, void *context);
```

---

#### OH_Preferences_UnRegisterObserver

取消注册观察者。

**签名**：
```c
int OH_Preferences_UnRegisterObserver(OH_Preferences *preference, OH_PreferencesDataObserver observer);
```

---

### 错误码定义

**头文件**：`interfaces/ndk/include/oh_preferences_err_code.h`

| 错误码 | 宏名 | 说明 |
|--------|------|------|
| 0 | `PREFERENCES_OK` | 成功 |
| 401 | `PREFERENCES_ERROR_INVALID_PARAM` | 无效参数 |
| 801 | `PREFERENCES_ERROR_NOT_SUPPORTED` | 能力不支持 |
| 801 | `PREFERENCES_ERROR_DELETE_FILE` | 删除文件失败 |
| 801 | `PREFERENCES_ERROR_STORAGE` | 存储错误 |
| 801 | `PREFERENCES_ERROR_MALLOC` | 内存分配失败 |
| 801 | `PREFERENCES_ERROR_INVALID_PARAM` | 无效参数 |
| 801 | `PREFERENCES_ERROR_KEY_NOT_FOUND` | Key 不存在 |

---

## Inner API

Inner API 提供 C++ 原生接口，供系统级应用使用。

### C++ 接口

**头文件**：`interfaces/inner_api/include/preferences.h`

**库文件**：`libnative_preferences.so`

#### Preferences 类

**抽象基类**，提供核心 CRUD 操作。

**头文件位置**：`interfaces/inner_api/include/preferences.h:66`

```cpp
class PREF_API_EXPORT Preferences {
public:
    virtual ~Preferences() {}

    // 读取操作
    virtual PreferencesValue Get(const std::string &key,
                                 const PreferencesValue &defValue) = 0;
    virtual int GetInt(const std::string &key, const int &defValue = {}) = 0;
    virtual std::string GetString(const std::string &key,
                                   const std::string &defValue = {}) = 0;
    virtual bool GetBool(const std::string &key, const bool &defValue = {}) = 0;
    virtual float GetFloat(const std::string &key,
                           const float &defValue = {}) = 0;
    virtual double GetDouble(const std::string &key,
                              const double &defValue = {}) = 0;
    virtual int64_t GetLong(const std::string &key,
                            const int64_t &defValue = {}) = 0;

    // 写入操作
    virtual int Put(const std::string &key,
                    const PreferencesValue &value) = 0;
    virtual int PutInt(const std::string &key, int value) = 0;
    virtual int PutString(const std::string &key,
                          const std::string &value) = 0;
    virtual int PutBool(const std::string &key, bool value) = 0;
    virtual int PutFloat(const std::string &key, float value) = 0;
    virtual int PutDouble(const std::string &key, double value) = 0;
    virtual int PutLong(const std::string &key, int64_t value) = 0;

    // 查询操作
    virtual std::map<std::string, PreferencesValue> GetAll() = 0;
    virtual bool HasKey(const std::string &key) = 0;

    // 修改操作
    virtual int Delete(const std::string &key) = 0;
    virtual int Clear() = 0;

    // 持久化操作
    virtual void Flush() = 0;
    virtual int FlushSync() = 0;

    // 观察者操作
    virtual int RegisterObserver(
        std::shared_ptr<PreferencesObserver> observer) = 0;
    virtual int UnRegisterObserver(
        std::shared_ptr<PreferencesObserver> observer) = 0;

    // 生命周期操作
    virtual int Close() = 0;
};
```

---

#### PreferencesHelper 类

提供实例管理工具方法。

**头文件**：`interfaces/inner_api/include/preferences_helper.h`

**关键方法**：

| 方法 | 说明 |
|------|------|
| `GetPreferences(options, &errCode)` | 获取 Preferences 实例 |
| `DeletePreferences(path)` | 删除 Preferences |
| `RemovePreferencesFromCache(path)` | 从缓存移除 |
| `IsStorageTypeSupported(type)` | 检查存储类型支持 |

---

#### PreferencesObserver 类

数据变化观察者接口。

**头文件**：`interfaces/inner_api/include/preferences_observer.h`

```cpp
class PreferencesObserver : public RefBase {
public:
    virtual void OnChange(
        std::shared_ptr<std::unordered_set<std::string>> keys) = 0;
};
```

---

#### Options 结构体

Preferences 打开选项。

**头文件**：`preferences.h:36-62`

```cpp
struct Options {
    std::string filePath{ "" };      // 文件路径
    std::string bundleName{ "" };   // Bundle 名称
    std::string dataGroupId{ "" };  // 数据组 ID
    bool isEnhance = false;         // 是否增强模式
};
```

---

#### StorageType 枚举

```cpp
enum StorageType {
    XML = 0,   // XML 存储
    GSKV       // GSKV 存储
};
```

---

#### PreferencesValue 类

通用值类型，支持多种数据类型。

**头文件**：`interfaces/inner_api/include/preferences_value.h`

---

## 错误码汇总

**头文件**：`interfaces/inner_api/include/preferences_errno.h`

| 错误码 | 宏名 | 说明 |
|--------|------|------|
| 0 | `E_OK` | 成功 |
| 1 | `E_STALE` | 资源已停止/销毁 |
| 2 | `E_INVALID_ARGS` | 输入参数无效 |
| 3 | `E_OUT_OF_MEMORY` | 内存不足 |
| 4 | `E_NOT_PERMIT` | 操作不允许 |
| 5 | `E_KEY_EMPTY` | Key 为空 |
| 6 | `E_KEY_EXCEED_MAX_LENGTH` | Key 超过最大长度 |
| 7 | `E_PTR_EXIST_ANOTHER_HOLDER` | 指针被其他线程持有 |
| 8 | `E_RELATIVE_PATH` | 相对路径 |
| 9 | `E_EMPTY_FILE_PATH` | 空文件路径 |
| 10 | `E_DELETE_FILE_FAIL` | 删除文件失败 |
| 11 | `E_EMPTY_FILE_NAME` | 空文件名 |
| 12 | `E_INVALID_FILE_PATH` | 无效文件路径 |
| 13 | `E_PATH_EXCEED_MAX_LENGTH` | 路径超过最大长度 |
| 14 | `E_VALUE_EXCEED_MAX_LENGTH` | Value 超过最大长度 |
| 15 | `E_KEY_EXCEED_LENGTH_LIMIT` | Key 超过长度限制 |
| 16 | `E_VALUE_EXCEED_LENGTH_LIMIT` | Value 超过长度限制 |
| 17 | `E_DEFAULT_EXCEED_LENGTH_LIMIT` | 默认值超过长度限制 |
| 18 | `PERMISSION_DENIED` | 权限拒绝 |
| 19 | `E_GET_DATAOBSMGRCLIENT_FAIL` | 获取 DataObsMgrClient 失败 |
| 20 | `E_OBSERVER_RESERVE` | 观察者保留 |
| 21 | `E_ALREADY_CLOSED` | 已关闭 |
| 22 | `E_NO_DATA` | 数据不存在 |
| 23 | `E_XML_RESTORED_FROM_BACKUP_FILE` | 从备份恢复 |
| 24 | `E_OPERAT_IS_LOCKED` | 文件被锁定 |
| 25 | `E_OPERAT_IS_CROSS_PROESS` | 跨进程操作 |
| 26 | `E_SUBSCRIBE_FAILED` | 订阅失败 |
| 27 | `E_OBJECT_NOT_ACTIVE` | 对象未激活 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](./00_Overview.md) | 模块定位和能力边界 |
| [架构设计](./01_Architecture.md) | 内部架构详解 |
| [构建配置](./03_Build.md) | 构建流程说明 |
| [安全评审](./04_Security_Review.md) | 安全风险分析 |
