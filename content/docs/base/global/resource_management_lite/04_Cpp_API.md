# C++ API 接口文档

## 1. 概述

本文档描述 `resource_management_lite` 组件对外暴露的 **C++ 接口**，供 Native 应用调用。

**接口特点**：
- 面向对象设计，提供丰富的资源类型支持
- 支持多种资源类型（字符串、数组、布尔、整数、颜色等）
- 提供资源配置（ResConfig）用于多语言/多设备适配
- 资源引用解析支持

**适用场景**：
- 需要访问多种资源类型的应用
- 需要动态切换资源配置的场景
- 需要解析资源引用（@ref）的场景

---

## 2. 头文件与命名空间

### 2.1 头文件位置

```
frameworks/resmgr_lite/include/resource_manager.h
frameworks/resmgr_lite/include/res_config.h
frameworks/resmgr_lite/include/rstate.h
frameworks/resmgr_lite/include/res_common.h
```

### 2.2 命名空间

```cpp
namespace OHOS {
namespace Global {
namespace Resource {
    // 所有接口在此命名空间下
}
}
}
```

---

## 3. ResourceManager 类

### 3.1 类概述

`ResourceManager` 是资源管理的抽象基类，提供统一的资源访问接口。

```cpp
namespace OHOS {
namespace Global {
namespace Resource {
class ResourceManager {
public:
    virtual ~ResourceManager() = 0;
    // ... 虚方法
};
}
}
}
```

### 3.2 工厂函数

```cpp
ResourceManager *CreateResourceManager();
```

**功能**：创建 `ResourceManagerImpl` 实例。

**返回值**：`ResourceManager*` 指针，调用方负责释放。

**使用示例**：

```cpp
#include "resource_manager.h"

using namespace OHOS::Global::Resource;

ResourceManager *mgr = CreateResourceManager();
if (mgr != nullptr) {
    // 使用资源管理器
    delete mgr;  // 使用完毕后释放
}
```

---

### 3.3 资源管理方法

#### AddResource

```cpp
virtual bool AddResource(const char *path) = 0;
```

**功能**：添加 HAP 资源路径。

| 参数 | 类型 | 说明 |
|------|------|------|
| `path` | `const char *` | HAP 包中资源目录路径 |

**返回值**：`true` 添加成功，`false` 添加失败。

---

#### UpdateResConfig

```cpp
virtual RState UpdateResConfig(ResConfig &resConfig) = 0;
```

**功能**：更新资源配置（语言、区域、设备类型等）。

| 参数 | 类型 | 说明 |
|------|------|------|
| `resConfig` | `ResConfig &` | 新的资源配置 |

**返回值**：`RState` 状态码。

---

#### GetResConfig

```cpp
virtual void GetResConfig(ResConfig &resConfig) = 0;
```

**功能**：获取当前资源配置。

| 参数 | 类型 | 说明 |
|------|------|------|
| `resConfig` | `ResConfig &` | 输出当前配置 |

---

### 3.4 字符串资源方法

#### GetStringById

```cpp
virtual RState GetStringById(uint32_t id, std::string &outValue) = 0;
```

**功能**：根据资源 ID 获取字符串资源。

---

#### GetStringByName

```cpp
virtual RState GetStringByName(const char *name, std::string &outValue) = 0;
```

**功能**：根据资源名称获取字符串资源。

---

#### GetStringFormatById

```cpp
virtual RState GetStringFormatById(std::string &outValue, uint32_t id, ...) = 0;
```

**功能**：根据资源 ID 获取格式化字符串（支持参数）。

---

#### GetStringFormatByName

```cpp
virtual RState GetStringFormatByName(std::string &outValue, const char *name, ...) = 0;
```

**功能**：根据资源名称获取格式化字符串（支持参数）。

---

#### GetStringArrayById

```cpp
virtual RState GetStringArrayById(uint32_t id, std::vector<std::string> &outValue) = 0;
```

**功能**：根据资源 ID 获取字符串数组。

---

#### GetStringArrayByName

```cpp
virtual RState GetStringArrayByName(const char *name, std::vector<std::string> &outValue) = 0;
```

**功能**：根据资源名称获取字符串数组。

---

### 3.5 图案资源方法

#### GetPatternById

```cpp
virtual RState GetPatternById(uint32_t id, std::map<std::string, std::string> &outValue) = 0;
```

**功能**：根据资源 ID 获取图案资源（键值对集合）。

---

#### GetPatternByName

```cpp
virtual RState GetPatternByName(const char *name, std::map<std::string, std::string> &outValue) = 0;
```

**功能**：根据资源名称获取图案资源。

---

### 3.6 复数字符串方法

#### GetPluralStringById

```cpp
virtual RState GetPluralStringById(uint32_t id, int quantity, std::string &outValue) = 0;
```

**功能**：根据资源 ID 获取复数字符串（根据数量选择正确形式）。

| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | `uint32_t` | 资源 ID |
| `quantity` | `int` | 数量（用于选择复数形式） |
| `outValue` | `std::string &` | 输出字符串 |

---

#### GetPluralStringByName

```cpp
virtual RState GetPluralStringByName(const char *name, int quantity, std::string &outValue) = 0;
```

**功能**：根据资源名称获取复数字符串。

---

#### GetPluralStringByIdFormat

```cpp
virtual RState GetPluralStringByIdFormat(std::string &outValue, uint32_t id, int quantity, ...) = 0;
```

**功能**：根据资源 ID 获取格式化复数字符串。

---

#### GetPluralStringByNameFormat

```cpp
virtual RState GetPluralStringByNameFormat(std::string &outValue, const char *name, int quantity, ...) = 0;
```

**功能**：根据资源名称获取格式化复数字符串。

---

### 3.7 主题资源方法

#### GetThemeById

```cpp
virtual RState GetThemeById(uint32_t id, std::map<std::string, std::string> &outValue) = 0;
```

**功能**：根据资源 ID 获取主题资源。

---

#### GetThemeByName

```cpp
virtual RState GetThemeByName(const char *name, std::map<std::string, std::string> &outValue) = 0;
```

**功能**：根据资源名称获取主题资源。

---

### 3.8 布尔值资源方法

#### GetBooleanById

```cpp
virtual RState GetBooleanById(uint32_t id, bool &outValue) = 0;
```

**功能**：根据资源 ID 获取布尔值。

---

#### GetBooleanByName

```cpp
virtual RState GetBooleanByName(const char *name, bool &outValue) = 0;
```

**功能**：根据资源名称获取布尔值。

---

### 3.9 整数资源方法

#### GetIntegerById

```cpp
virtual RState GetIntegerById(uint32_t id, int &outValue) = 0;
```

**功能**：根据资源 ID 获取整数值。

---

#### GetIntegerByName

```cpp
virtual RState GetIntegerByName(const char *name, int &outValue) = 0;
```

**功能**：根据资源名称获取整数值。

---

### 3.10 浮点数资源方法

#### GetFloatById

```cpp
virtual RState GetFloatById(uint32_t id, float &outValue) = 0;
```

**功能**：根据资源 ID 获取浮点值。

---

#### GetFloatByName

```cpp
virtual RState GetFloatByName(const char *name, float &outValue) = 0;
```

**功能**：根据资源名称获取浮点值。

---

### 3.11 整数数组方法

#### GetIntArrayById

```cpp
virtual RState GetIntArrayById(uint32_t id, std::vector<int> &outValue) = 0;
```

**功能**：根据资源 ID 获取整数数组。

---

#### GetIntArrayByName

```cpp
virtual RState GetIntArrayByName(const char *name, std::vector<int> &outValue) = 0;
```

**功能**：根据资源名称获取整数数组。

---

### 3.12 颜色资源方法

#### GetColorById

```cpp
virtual RState GetColorById(uint32_t id, uint32_t &outValue) = 0;
```

**功能**：根据资源 ID 获取颜色值（ARGB 格式）。

---

#### GetColorByName

```cpp
virtual RState GetColorByName(const char *name, uint32_t &outValue) = 0;
```

**功能**：根据资源名称获取颜色值。

---

### 3.13 配置资源方法

#### GetProfileById

```cpp
virtual RState GetProfileById(uint32_t id, std::string &outValue) = 0;
```

**功能**：根据资源 ID 获取配置资源路径。

---

#### GetProfileByName

```cpp
virtual RState GetProfileByName(const char *name, std::string &outValue) = 0;
```

**功能**：根据资源名称获取配置资源路径。

---

### 3.14 媒体资源方法

#### GetMediaById

```cpp
virtual RState GetMediaById(uint32_t id, std::string &outValue) = 0;
```

**功能**：根据资源 ID 获取媒体资源路径。

---

#### GetMediaByName

```cpp
virtual RState GetMediaByName(const char *name, std::string &outValue) = 0;
```

**功能**：根据资源名称获取媒体资源路径。

---

**证据来源**：
- `frameworks/resmgr_lite/include/resource_manager.h` (lines 26-92)

---

## 4. ResConfig 类

### 4.1 类概述

`ResConfig` 是资源配置的抽象基类，用于配置语言、区域、设备类型等参数。

```cpp
namespace OHOS {
namespace Global {
namespace Resource {
class ResConfig {
public:
    virtual RState SetLocaleInfo(const char *language, const char *script, const char *region) = 0;
    virtual RState SetLocaleInfo(LocaleInfo &localeInfo) = 0;
    virtual void SetDeviceType(DeviceType deviceType) = 0;
    virtual void SetDirection(Direction direction) = 0;
    virtual void SetScreenDensity(ScreenDensity screenDensity) = 0;
    virtual const LocaleInfo *GetLocaleInfo() const = 0;
    virtual Direction GetDirection() const = 0;
    virtual ScreenDensity GetScreenDensity() const = 0;
    virtual DeviceType GetDeviceType() const = 0;
    virtual bool Copy(ResConfig &other) = 0;
    virtual ~ResConfig() {}
};
}
}
}
```

### 4.2 工厂函数

```cpp
ResConfig *CreateResConfig();
```

**功能**：创建 `ResConfigImpl` 实例。

---

### 4.3 区域配置方法

#### SetLocaleInfo（组件形式）

```cpp
virtual RState SetLocaleInfo(const char *language, const char *script, const char *region) = 0;
```

**功能**：设置区域信息（语言、脚本、地区）。

| 参数 | 类型 | 说明 |
|------|------|------|
| `language` | `const char *` | 语言代码（如 "zh"、"en"） |
| `script` | `const char *` | 脚本代码（如 "Hans"、"Latn"） |
| `region` | `const char *` | 地区代码（如 "CN"、"US"） |

**返回值**：`RState` 状态码。

---

#### SetLocaleInfo（LocaleInfo 形式）

```cpp
virtual RState SetLocaleInfo(LocaleInfo &localeInfo) = 0;
```

**功能**：从 i18n_lite 的 LocaleInfo 设置区域信息。

---

### 4.4 设备配置方法

#### SetDeviceType

```cpp
virtual void SetDeviceType(DeviceType deviceType) = 0;
```

**功能**：设置设备类型。

| DeviceType | 说明 |
|------------|------|
| `DEVICE_PHONE` | 手机 |
| `DEVICE_TABLET` | 平板 |
| `DEVICE_CAR` | 车载 |
| `DEVICE_PAD` | 平板设备 |
| `DEVICE_TV` | 电视 |
| `DEVICE_WEARABLE` | 可穿戴设备 |

---

#### SetDirection

```cpp
virtual void SetDirection(Direction direction) = 0;
```

**功能**：设置屏幕方向。

| Direction | 说明 |
|-----------|------|
| `DIRECTION_VERTICAL` | 垂直 |
| `DIRECTION_HORIZONTAL` | 水平 |

---

#### SetScreenDensity

```cpp
virtual void SetScreenDensity(ScreenDensity screenDensity) = 0;
```

**功能**：设置屏幕密度。

| ScreenDensity | 值 | 说明 |
|----------------|-----|------|
| `SCREEN_DENSITY_SDPI` | 120 | sdpi |
| `SCREEN_DENSITY_MDPI` | 160 | mdpi（基准） |
| `SCREEN_DENSITY_LDPI` | 240 | ldpi |
| `SCREEN_DENSITY_XLDPI` | 320 | xldpi |
| `SCREEN_DENSITY_XXLDPI` | 480 | xxldpi |
| `SCREEN_DENSITY_XXXLDPI` | 640 | xxxldpi |

---

### 4.5 获取配置方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `GetLocaleInfo()` | `const LocaleInfo *` | 获取区域信息 |
| `GetDirection()` | `Direction` | 获取屏幕方向 |
| `GetScreenDensity()` | `ScreenDensity` | 获取屏幕密度 |
| `GetDeviceType()` | `DeviceType` | 获取设备类型 |

---

### 4.6 辅助函数

```cpp
const LocaleInfo *GetSysDefault();
void UpdateSysDefault(const LocaleInfo &localeInfo, bool needNotify);
LocaleInfo *BuildFromString(const char *str, char sep, RState &rState);
LocaleInfo *BuildFromParts(const char *language, const char *script, const char *region, RState &rState);
void FindAndSort(const std::string localeStr, std::vector<std::string> &candidateLocale,
    std::vector<std::string> &outValue);
```

**证据来源**：
- `frameworks/resmgr_lite/include/res_config.h` (lines 27-63)

---

## 5. RState 错误码

### 5.1 错误码枚举

```cpp
enum RState {
    SUCCESS = 0,                      // 成功
    NOT_SUPPORT_SEP = 1,              // 不支持的分隔符
    INVALID_BCP47_STR_LEN_TOO_SHORT = 2,  // BCP 47 字符串过短
    INVALID_BCP47_LANGUAGE_SUBTAG = 3,     // 无效语言子标签
    INVALID_BCP47_SCRIPT_SUBTAG = 4,       // 无效脚本子标签
    INVALID_BCP47_REGION_SUBTAG = 5,       // 无效区域子标签
    HAP_INIT_FAILED = 6,             // HAP 初始化失败
    NOT_FOUND = 7,                   // 未找到资源
    INVALID_FORMAT = 8,              // 无效格式
    LOCALEINFO_IS_NULL = 9,          // LocaleInfo 为空
    NOT_ENOUGH_MEM = 10,             // 内存不足
    ERROR = 10000                    // 通用错误
};
```

### 5.2 错误码速查

| 错误码 | 场景 |
|--------|------|
| `SUCCESS` | 操作成功 |
| `NOT_FOUND` | 资源未找到 |
| `INVALID_FORMAT` | 资源格式错误 |
| `NOT_ENOUGH_MEM` | 内存分配失败 |
| `HAP_INIT_FAILED` | HAP 文件解析失败 |

**证据来源**：
- `frameworks/resmgr_lite/include/rstate.h` (lines 22-35)

---

## 6. 使用示例

### 6.1 基础使用

```cpp
#include "resource_manager.h"
#include "res_config.h"

using namespace OHOS::Global::Resource;

int main() {
    // 1. 创建资源管理器
    ResourceManager *mgr = CreateResourceManager();
    if (mgr == nullptr) {
        return -1;
    }
    
    // 2. 添加资源路径
    mgr->AddResource("/data/resource");
    
    // 3. 获取字符串资源
    std::string value;
    RState state = mgr->GetStringById(0x16777216, value);
    if (state == SUCCESS) {
        printf("String: %s\n", value.c_str());
    }
    
    // 4. 释放资源管理器
    delete mgr;
    return 0;
}
```

### 6.2 多语言配置

```cpp
#include "resource_manager.h"
#include "res_config.h"

using namespace OHOS::Global::Resource;

int main() {
    ResourceManager *mgr = CreateResourceManager();
    ResConfig *config = CreateResConfig();
    
    // 添加多个语言的资源
    mgr->AddResource("/data/resource/zh");
    mgr->AddResource("/data/resource/en");
    
    // 配置为英文
    config->SetLocaleInfo("en", "Latn", "US");
    mgr->UpdateResConfig(*config);
    
    std::string value;
    mgr->GetStringByName("greeting", value);
    printf("English greeting: %s\n", value.c_str());
    
    // 切换为中文
    config->SetLocaleInfo("zh", "Hans", "CN");
    mgr->UpdateResConfig(*config);
    
    mgr->GetStringByName("greeting", value);
    printf("Chinese greeting: %s\n", value.c_str());
    
    delete mgr;
    delete config;
    return 0;
}
```

### 6.3 获取不同类型资源

```cpp
#include "resource_manager.h"
#include <vector>
#include <map>

using namespace OHOS::Global::Resource;

void GetVariousResources(ResourceManager *mgr) {
    RState state;
    
    // 字符串
    std::string str;
    state = mgr->GetStringById(0x10000, str);
    
    // 布尔值
    bool boolVal;
    state = mgr->GetBooleanByName("is_enabled", boolVal);
    
    // 整数
    int intVal;
    state = mgr->GetIntegerById(0x20000, intVal);
    
    // 颜色
    uint32_t color;
    state = mgr->GetColorByName("primary_color", color);
    
    // 字符串数组
    std::vector<std::string> arr;
    state = mgr->GetStringArrayByName("weekdays", arr);
    
    // 主题（键值对）
    std::map<std::string, std::string> theme;
    state = mgr->GetThemeByName("app_theme", theme);
}
```

---

## 7. 文档链接

| 主题 | 文档 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](./01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| C API | [03_C_API.md](./03_C_API.md) |
| 构建系统 | [05_Build_System.md](./05_Build_System.md) |
| 安全分析 | [06_Security_Analysis.md](./06_Security_Analysis.md) |
