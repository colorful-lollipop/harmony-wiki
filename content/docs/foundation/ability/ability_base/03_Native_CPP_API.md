# Native C++ API 文档

## 目的

本文档详细说明 ability_base 组件提供的 Native C++ API，包括 Want、Configuration、URI、Base 类型等核心类的接口定义、使用方法和注意事项，供 C++ 开发者参考。

**适用范围**:
- OpenHarmony 标准系统（standard system type）
- API 版本：15+
- 组件版本：3.1

---

## 1. Want API

### 1.1 Want 类概览

**头文件**: `interfaces/kits/native/want/include/want.h`  
**实现文件**: `interfaces/kits/native/want/src/want.cpp`  
**所属库**: `libwant.so`

**类定义**:
```cpp
class Want final : public Parcelable {
    // 100+ 个公共方法
};
```

**证据位置**: `interfaces/kits/native/want/include/want.h:34`

### 1.2 核心方法清单

#### 构造与工厂方法

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `MakeMainAbility` | `static Want MakeMainAbility(const ElementName& element)` | 创建主 Ability 的 Want | want.h:208 |
| `ParseUri` | `static Want* ParseUri(const std::string& uriStr)` | 从 URI 字符串解析 Want | want.h:216 |
| `Unmarshalling` | `static Want* Unmarshalling(Parcel& parcel)` | 从 Parcel 反序列化 | want.h:794 |

#### 参数操作

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `SetParam` | `Want& SetParam(const std::string& key, bool value)` | 设置布尔参数 | want.h:443 |
| `SetParam` | `Want& SetParam(const std::string& key, int32_t value)` | 设置整型参数 | want.h:452 |
| `SetParam` | `Want& SetParam(const std::string& key, int64_t value)` | 设置长整型参数 | want.h:461 |
| `SetParam` | `Want& SetParam(const std::string& key, double value)` | 设置双精度参数 | want.h:470 |
| `SetParam` | `Want& SetParam(const std::string& key, const std::string& value)` | 设置字符串参数 | want.h:479 |
| `SetParam` | `Want& SetParam(const std::string& key, const sptr<IRemoteObject>& value)` | 设置远程对象 | want.h:488 |
| `GetBoolParam` | `bool GetBoolParam(const std::string& key, bool defaultValue) const` | 获取布尔参数 | want.h:525 |
| `GetIntParam` | `int GetIntParam(const std::string& key, int defaultValue) const` | 获取整型参数 | want.h:534 |
| `GetStringParam` | `std::string GetStringParam(const std::string& key) const` | 获取字符串参数 | want.h:551 |
| `HasParameter` | `bool HasParameter(const std::string& key) const` | 检查参数存在 | want.h:615 |

#### Operation 操作

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `SetOperation` | `Want& SetOperation(const Operation& operation)` | 设置操作 | want.h:272 |
| `GetOperation` | `const Operation& GetOperation() const` | 获取操作 | want.h:281 |
| `SetElementName` | `Want& SetElementName(const std::string& bundleName, const std::string& abilityName)` | 设置组件名 | want.h:302 |
| `GetElement` | `const ElementName& GetElement() const` | 获取组件名 | want.h:320 |
| `SetUri` | `Want& SetUri(const std::string& uri)` | 设置 URI | want.h:340 |
| `GetUri` | `std::string GetUri() const` | 获取 URI | want.h:358 |

#### 标志和实体

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `AddFlags` | `Want& AddFlags(unsigned int flags)` | 添加标志 | want.h:198 |
| `SetFlags` | `Want& SetFlags(unsigned int flags)` | 设置标志 | want.h:188 |
| `GetFlags` | `unsigned int GetFlags() const` | 获取标志 | want.h:380 |
| `AddEntity` | `Want& AddEntity(const std::string& entity)` | 添加实体 | want.h:172 |
| `GetEntities` | `const std::vector<std::string>& GetEntities() const` | 获取实体列表 | want.h:399 |

### 1.3 使用示例

```cpp
#include "want.h"
#include "element_name.h"

using namespace OHOS::AAFwk;

// 创建显式 Want
Want want;
want.SetElementName("com.example.app", "MainAbility");
want.SetParam("key1", 123);
want.SetParam("key2", std::string("value"));

// 创建隐式 Want
Want implicitWant;
implicitWant.SetAction("ohos.action.home");
implicitWant.AddEntity("entity.system.home");
implicitWant.SetUri("https://example.com");

// 从 Parcel 反序列化
Parcel parcel;
// ... 从 IPC 接收数据
Want* receivedWant = Want::Unmarshalling(parcel);
if (receivedWant != nullptr) {
    int value = receivedWant->GetIntParam("key1", 0);
    delete receivedWant;
}
```

### 1.4 Operation 类

**头文件**: `interfaces/kits/native/want/include/operation.h`

**核心字段**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `action_` | `std::string` | 操作类型（如 "ohos.action.home"） |
| `entities_` | `std::vector<std::string>` | 实体列表 |
| `flags_` | `unsigned int` | 标志位 |
| `uri_` | `Uri` | URI 目标 |
| `elementName_` | `ElementName` | 组件标识 |

**Builder 模式**:
```cpp
OperationBuilder builder;
Operation operation = builder
    .WithAction("ohos.action.view")
    .WithEntity("entity.video")
    .WithUri("https://example.com/video.mp4")
    .Build();
```

### 1.5 WantParams 类

**头文件**: `interfaces/kits/native/want/include/want_params.h`

**职责**: 类型安全的键值存储，支持嵌套结构。

**支持类型**:
- 基本类型: Boolean, Byte, Char, Short, Integer, Long, Float, Double, String
- 复杂类型: Array, WantParams (嵌套)
- 特殊类型: IRemoteObject, 文件描述符(int32_t fd)

**证据位置**: `interfaces/kits/native/want/src/want_params.cpp:204-233`

---

## 2. Configuration API

### 2.1 Configuration 类概览

**头文件**: `interfaces/kits/native/configuration/include/configuration.h`  
**实现文件**: `interfaces/kits/native/configuration/src/configuration.cpp`  
**所属库**: `libconfiguration.so`

**类定义**:
```cpp
class Configuration final : public Parcelable {
    // 系统配置容器
};
```

**证据位置**: `interfaces/kits/native/configuration/include/configuration.h:68`

### 2.2 核心方法清单

#### 配置访问

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `GetItem` | `std::string GetItem(const std::string& key) const` | 获取配置项 | configuration.h:73 |
| `GetItem` | `std::string GetItem(size_t id, const std::string& key) const` | 按 displayId 获取 | configuration.h:78 |
| `AddItem` | `void AddItem(const std::string& key, const std::string& value)` | 添加配置项 | configuration.h:83 |
| `AddItem` | `void AddItem(size_t id, const std::string& key, const std::string& value)` | 按 displayId 添加 | configuration.h:88 |
| `RemoveItem` | `void RemoveItem(const std::string& key)` | 移除配置项 | configuration.h:93 |

#### 配置比较与合并

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `CompareDifferent` | `bool CompareDifferent(std::vector<std::string>& diffKeys, const Configuration& other)` | 比较差异键 | configuration.h:118 |
| `Merge` | `bool Merge(const Configuration& other)` | 合并配置 | configuration.h:96 |
| `MergeOther` | `bool MergeOther(const std::vector<std::string>& diffKeys, const Configuration& other)` | 合并指定键 | configuration.h:108 |

#### 线程安全

Configuration 使用 `std::recursive_mutex` 保护内部数据，所有公共方法都是线程安全的。

**证据位置**: `interfaces/kits/native/configuration/include/configuration.h:234`

### 2.3 全局配置键

**头文件**: `interfaces/kits/native/configuration/include/global_configuration_key.h`

| 常量 | 值 | 说明 |
|------|-----|------|
| `SYSTEM_LANGUAGE` | `"ohos.system.language"` | 系统语言 |
| `SYSTEM_COLORMODE` | `"ohos.system.colorMode"` | 颜色模式（深色/浅色） |
| `APPLICATION_DIRECTION` | `"ohos.application.direction"` | 应用方向 |
| `APPLICATION_DENSITY` | `"ohos.application.density"` | 屏幕密度 |
| `INPUT_POINTER_DEVICE` | `"ohos.input.pointer.device"` | 指针设备 |

### 2.4 使用示例

```cpp
#include "configuration.h"
#include "global_configuration_key.h"

using namespace OHOS::AppExecFwk;

// 查询配置
Configuration config;
std::string language = config.GetItem(GlobalConfigurationKey::SYSTEM_LANGUAGE);
// 返回: "zh-CN" 或 "en-US" 等

// 多屏配置
std::string direction = config.GetItem(1, GlobalConfigurationKey::APPLICATION_DIRECTION);
// displayId=1 的应用方向

// 配置变更监听（伪代码）
void OnConfigurationChanged(const Configuration& newConfig) {
    std::vector<std::string> diffKeys;
    if (config.CompareDifferent(diffKeys, newConfig)) {
        for (const auto& key : diffKeys) {
            // 处理变更的配置项
        }
    }
}
```

---

## 3. URI API

### 3.1 Uri 类概览

**头文件**: `interfaces/kits/native/uri/include/uri.h`  
**实现文件**: `interfaces/kits/native/uri/src/uri.cpp`  
**所属库**: `libzuri.so`

**类定义**:
```cpp
class Uri : public Parcelable {
    // URI 解析和访问
};
```

**证据位置**: `interfaces/kits/native/uri/include/uri.h:24`

### 3.2 核心方法清单

#### 构造与解析

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `Uri` | `Uri(const std::string& uriString)` | 从字符串构造 | uri.h:27 |
| `Uri` | `Uri(const Uri& other)` | 拷贝构造 | uri.h:35 |
| `ParseUri` | `static std::shared_ptr<Uri> ParseUri(const std::string& uriStr)` | 静态解析方法 | uri.h:43 |

#### 组件访问

| 方法 | 签名 | 说明 | 位置 |
|------|------|------|------|
| `GetScheme` | `std::string GetScheme() const` | 获取 scheme | uri.h:52 |
| `GetAuthority` | `std::string GetAuthority() const` | 获取 authority | uri.h:61 |
| `GetHost` | `std::string GetHost() const` | 获取 host | uri.h:70 |
| `GetPort` | `int GetPort() const` | 获取 port | uri.h:79 |
| `GetPath` | `std::string GetPath() const` | 获取 path | uri.h:88 |
| `GetQuery` | `std::string GetQuery() const` | 获取 query | uri.h:97 |
| `GetFragment` | `std::string GetFragment() const` | 获取 fragment | uri.h:106 |
| `ToString` | `std::string ToString() const` | 转为字符串 | uri.h:115 |

### 3.3 使用示例

```cpp
#include "uri.h"

using namespace OHOS;

// 解析 URI
Uri uri("https://example.com:8080/path/to/resource?query=value#fragment");

// 访问各组件
std::string scheme = uri.GetScheme();      // "https"
std::string host = uri.GetHost();          // "example.com"
int port = uri.GetPort();                  // 8080
std::string path = uri.GetPath();          // "/path/to/resource"
std::string query = uri.GetQuery();        // "query=value"
std::string fragment = uri.GetFragment();  // "fragment"

// IPC 传输
Parcel parcel;
uri.Marshalling(parcel);
// ... 传输 ...
auto receivedUri = Uri::Unmarshalling(parcel);
```

---

## 4. Base 类型 API

### 4.1 IInterface 层次结构

**头文件**: `interfaces/inner_api/base/include/base_interfaces.h`  
**所属库**: `libbase.so`

**基接口定义**:
```cpp
class IInterface : public virtual Object {
public:
    virtual InterfaceID GetInterfaceID() = 0;
    virtual IInterface* Query(const InterfaceID& iid) = 0;
};
```

### 4.2 类型包装器清单

| 类 | 头文件 | Box 方法 | UnBox 方法 |
|-----|--------|----------|-----------|
| `Boolean` | `bool_wrapper.h` | `Boolean::Box(bool)` | `Boolean::UnBox(IBoolean*)` |
| `Byte` | `byte_wrapper.h` | `Byte::Box(byte)` | `Byte::UnBox(IByte*)` |
| `Char` | `zchar_wrapper.h` | `Char::Box(char)` | `Char::UnBox(IChar*)` |
| `Short` | `short_wrapper.h` | `Short::Box(short)` | `Short::UnBox(IShort*)` |
| `Integer` | `int_wrapper.h` | `Integer::Box(int)` | `Integer::UnBox(IInteger*)` |
| `Long` | `long_wrapper.h` | `Long::Box(long)` | `Long::UnBox(ILong*)` |
| `Float` | `float_wrapper.h` | `Float::Box(float)` | `Float::UnBox(IFloat*)` |
| `Double` | `double_wrapper.h` | `Double::Box(double)` | `Double::UnBox(IDouble*)` |
| `String` | `string_wrapper.h` | `String::Box(std::string)` | `String::UnBox(IString*)` |
| `Array` | `array_wrapper.h` | `Array::Box(...)` | `Array::UnBox(IArray*)` |
| `PacMap` | `pac_map.h` | `PacMap::Box(...)` | `PacMap::UnBox(IPacMap*)` |

### 4.3 使用示例

```cpp
#include "bool_wrapper.h"
#include "int_wrapper.h"
#include "string_wrapper.h"

using namespace OHOS;

// 创建包装器对象
sptr<IBoolean> boolObj = Boolean::Box(true);
sptr<IInteger> intObj = Integer::Box(42);
sptr<IString> strObj = String::Box(std::string("Hello"));

// 解包获取值
bool boolValue = Boolean::UnBox(boolObj);
int intValue = Integer::UnBox(intObj);
std::string strValue = String::UnBox(strObj);

// 类型查询
IInterface* obj = ...;
IBoolean* boolInterface = IBoolean::Query(obj);
if (boolInterface != nullptr) {
    // 对象是 Boolean 类型
}
```

---

## 5. SessionInfo API

### 5.1 SessionInfo 类概览

**头文件**: `interfaces/kits/native/session_info/include/session_info.h`  
**所属库**: `libsession_info.so`

**职责**: UI Ability 会话元数据，包含窗口、Token 等信息。

**证据位置**: `interfaces/kits/native/session_info/include/session_info.h`

### 5.2 核心字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `sessionToken` | `sptr<IRemoteObject>` | Session token |
| `callerToken` | `sptr<IRemoteObject>` | 调用者 token |
| `callerAbilityToken` | `sptr<IRemoteObject>` | 调用者 Ability token |
| `callingTokenId` | `uint32_t` | 调用者 token ID |
| `uiAbilityId` | `int32_t` | UI Ability ID |

**安全注意**: SessionInfo 中的 token 字段在系统服务端需要重新验证。

---

## 6. ViewData API

### 6.1 ViewData 类概览

**头文件**: `interfaces/kits/native/view_data/include/view_data.h`  
**所属库**: `libview_data.so`

**职责**: 自动填充相关的视图数据结构。

### 6.2 核心类

| 类 | 说明 |
|-----|------|
| `ViewData` | 视图数据容器 |
| `PageNodeInfo` | 页面节点信息 |
| `Rect` | 矩形几何（left, top, right, bottom） |
| `AutoFillType` | 自动填充类型枚举 |

---

## 7. Extractor API

### 7.1 Extractor 类概览

**头文件**: `interfaces/kits/native/extractortool/include/extractor.h`  
**所属库**: `libextractortool.so`

**职责**: HAP/ZIP 文件提取和解压。

### 7.2 核心方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `GetInstance` | `static ExtractorUtil& GetInstance()` | 获取单例 |
| `GetHapPath` | `std::string GetHapPath(const std::string& moduleName)` | 获取 HAP 路径 |
| `ExtractToBuf` | `bool ExtractToBuf(const std::string& entryName, std::unique_ptr<uint8_t[]>& buffer, size_t& len)` | 提取到缓冲区 |
| `ExtractToFile` | `bool ExtractToFile(const std::string& entryName, const std::string& targetPath)` | 提取到文件 |

**安全注意**: ZIP 文件处理存在 ZIP bomb 风险，见 [06_Security_Review.md](06_Security_Review.md)。

---

## 8. 错误处理

### 8.1 返回值约定

| 类型 | 成功 | 失败 |
|------|------|------|
| `bool` | `true` | `false` |
| `int32_t` | `0` | 负错误码 |
| 指针 | 非 nullptr | `nullptr` |
| `std::string` | 有效字符串 | 空字符串 `""` |

### 8.2 常见错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERR_OK` | 0 | 成功 |
| `ERR_INVALID_VALUE` | -1 | 无效参数 |
| `ERR_INVALID_DATA` | -2 | 无效数据 |
| `ERR_NAME_NOT_FOUND` | -3 | 名称未找到 |

---

## 9. 线程安全说明

| 类 | 线程安全 | 说明 |
|-----|----------|------|
| `Want` | ❌ 否 | 非线程安全，需外部同步 |
| `WantParams` | ❌ 否 | 非线程安全 |
| `Configuration` | ✅ 是 | 使用 recursive_mutex |
| `Uri` | ✅ 是 | 只读对象 |
| `ExtractorUtil` | ✅ 是 | 单例 + mutex |

---

## 10. 性能考虑

### 10.1 内存分配

- 使用 `new (std::nothrow)` 安全分配
- 大对象使用 `sptr` 智能指针
- 避免频繁的 WantParams 拷贝

### 10.2 IPC 传输

- Want 对象传输前先估算大小
- 避免在 IPC 中传递过大的字符串或数组
- 使用文件描述符传递大文件

### 10.3 引用计数

- 使用 `sptr<T>` 自动管理生命周期
- 避免循环引用

---

## 相关跳转

- 📁 **目录结构**：[01_Directory_Structure.md](01_Directory_Structure.md)
- 🏗️ **架构设计**：[02_Architecture.md](02_Architecture.md)
- 🔧 **C NDK API**：[04_C_NDK_API.md](04_C_NDK_API.md)
- ⚙️ **GN 构建**：[05_GN_Build.md](05_GN_Build.md)
- 🔒 **安全评审**：[06_Security_Review.md](06_Security_Review.md)

---

**返回导航**：[SUMMARY.md](SUMMARY.md)
