# libphonenumber - Patch 详细分析

## 概述

libphonenumber 在 OpenHarmony 中进行了**最小化适配**，主要通过**条件编译**实现 OHOS 特性，而非传统的源代码 Patch。本文档详细分析所有 Patch 和 OHOS 特有的代码修改。

---

## Patch 清单

| Patch 文件 | 修改文件 | 修改目的 | OH 关联需求 | 类型 |
|-----------|-----------|-----------|-------------|------|
| `0003-maven-exclude-demo.patch` | `java/pom.xml` | 排除 Java demo 模块构建 | Build Config |

**Patch 数量**: 1 个

**主要适配方式**: 条件编译 (`is_ohos`, `LIBPHONENUMBER_UPGRADE`)

---

## 1. Maven Build Patch

### Patch: 0003-maven-exclude-demo.patch

**文件路径**: `debian/patches/0003-maven-exclude-demo.patch`

**修改文件**: `java/pom.xml`

**修改摘要**:
```diff
--- a/java/pom.xml
+++ b/java/pom.xml
@@ -76,7 +76,7 @@
     <module>internal/prefixmapper</module>
     <module>carrier</module>
     <module>geocoder</module>
-    <module>demo</module>
+    <!--<module>demo</module>-->
   </modules>
```

**原始问题**: 无实际问题，仅为构建优化

**修改内容**: 注释掉 Maven 构建中的 `demo` 模块

**OH 需求**:
- OpenHarmony 仅使用 C++ 版本的 libphonenumber
- 不需要 Java demo 模块
- 减少不必要的构建时间和产物

**OH 价值**:
- 减少构建时间
- 减少产物体积
- 避免构建不需要的代码

**回归风险**: 极低
- 仅为构建配置修改，不影响任何核心功能
- Java 模块在 OH 中本就不使用

**升级建议**: 此 Patch **可推向上游**
- 建议使用 Maven profile 方式更优雅地控制 demo 构建
- 示例：`<profiles><profile><id>without-demo</id><modules>...</modules></profile></profiles>`

---

## 2. OHOS 条件编译适配

OpenHarmony 通过 `BUILD.gn` 中的条件编译实现适配，而非传统 Patch。

### 2.1 LIBPHONENUMBER_UPGRADE 宏

这是 OHOS 对 libphonenumber 的**核心创新**，允许运行时更新元数据。

#### 宏定义

**位置**: `cpp/BUILD.gn` (第 110-112 行)

```gn
if (is_ohos) {
  phonenumber_defines += [ "LIBPHONENUMBER_UPGRADE" ]
  geocoding_defines += [ "LIBPHONENUMBER_UPGRADE" ]
}
```

#### 启用的代码路径

**a) phonenumberutil.cc**

**头文件引入** (第 37-40 行):
```cpp
#ifdef LIBPHONENUMBER_UPGRADE
#include "phonenumbers/ohos/update_metadata.h"
#include "phonenumbers/ohos/update_libphonenumber.h"
#endif
```

**构造函数修改** (第 920-923 行):
```cpp
#ifdef LIBPHONENUMBER_UPGRADE
  // 加载外部更新文件
  UpdateLibphonenumber::LoadUpdateData();
  // 合并更新到内置元数据
  UpdateMetadata::UpdatePhoneNumber(country_code_to_non_geographical_metadata_map_,
                                    region_to_metadata_map_);
#endif
```

**原始问题**:
- 上游版本仅支持编译时元数据
- 更新电话号码规则需要重新编译整个库
- 对于系统库更新困难

**修改内容**:
1. 新增 `ohos/update_libphonenumber.cc` - 数据加载入口
2. 新增 `ohos/update_metadata.cc` - 核心更新逻辑
3. 在初始化时加载外部元数据文件并合并

**OH 需求**:
- 支持系统运行时更新电话号码规则
- 无需重新编译库即可更新
- 便于 OTA 升级

**OH 价值**:
- 支持动态元数据更新
- 降低系统升级成本
- 提高维护灵活性

**回归风险**: 中等
- 需要确保元数据格式兼容性
- 更新文件格式变化需要同步修改解析代码

**升级建议**:
- 此功能为 OH 特有，无法直接推向上游
- 升级上游版本时需保留 `LIBPHONENUMBER_UPGRADE` 宏的代码路径
- 建议上游考虑支持类似机制

---

**b) phonenumbermatcher.cc**

**头文件引入** (第 46-49 行):
```cpp
#ifdef LIBPHONENUMBER_UPGRADE
#include "phonenumbers/ohos/update_metadata.h"
#include "phonenumbers/ohos/update_libphonenumber.h"
#endif
```

**AlternateFormats 构造函数修改** (第 385-388 行):
```cpp
#ifdef LIBPHONENUMBER_UPGRADE
  UpdateLibphonenumber::LoadUpdateData();
  UpdateMetadata::UpdateAlternateFormat(calling_code_to_alternate_formats_map_);
#endif
```

**修改内容**: 更新备用格式元数据

---

**c) shortnumberinfo.cc**

**头文件引入** (第 24-27 行):
```cpp
#ifdef LIBPHONENUMBER_UPGRADE
#include "phonenumbers/ohos/update_metadata.h"
#include "phonenumbers/ohos/update_libphonenumber.h"
#endif
```

**ShortNumberInfo 构造函数修改** (第 67-70 行):
```cpp
#ifdef LIBPHONENUMBER_UPGRADE
  UpdateLibphonenumber::LoadUpdateData();
  UpdateMetadata::UpdateShortNumber(region_to_short_metadata_map_);
#endif
```

**修改内容**: 更新短号码元数据（紧急号码等）

---

### 2.2 OHOS 特有源文件

#### 文件清单

| 文件 | 行数 | 功能 |
|------|------|------|
| `cpp/src/phonenumbers/ohos/update_metadata.cc` | 120 | 核心元数据更新逻辑 |
| `cpp/src/phonenumbers/ohos/update_metadata.h` | 49 | 元数据更新类定义 |
| `cpp/src/phonenumbers/ohos/update_libphonenumber.cc` | 34 | 电话号码库初始化 |
| `cpp/src/phonenumbers/ohos/update_libphonenumber.h` | 31 | 初始化类定义 |
| `cpp/src/phonenumbers/ohos/update_geocoding.cc` | 379 | 地理编码数据更新 |
| `cpp/src/phonenumbers/ohos/update_geocoding.h` | - | 地理编码更新类定义 |
| `cpp/src/phonenumbers/ohos/update_libgeocoding.cc` | 34 | 地理编码库初始化 |
| `cpp/src/phonenumbers/ohos/update_libgeocoding.h` | 31 | 地理编码初始化类定义 |
| `cpp/src/phonenumbers/ohos/geocoding_data.proto` | 52 | 地理编码数据 Protobuf 定义 |
| `cpp/src/phonenumbers/ohos/geocoding_data.pb.h` | 48 | 生成的 Protobuf 头 |
| `cpp/src/phonenumbers/ohos/geocoding_data.pb.cc` | 1889 | 生成的 Protobuf 实现 |

**总计**: 约 2,637 行 OHOS 特有代码

---

### 2.3 元数据更新机制详解

#### 核心类

**UpdateMetadata 类**

**位置**: `cpp/src/phonenumbers/ohos/update_metadata.cc`

**关键方法**:

| 方法 | 功能 |
|------|------|
| `LoadUpdatedMetadata(int fd)` | 从文件描述符解析 Protobuf 数据 |
| `UpdateShortNumber()` | 更新短号码元数据映射 |
| `UpdatePhoneNumber()` | 更新电话号码元数据映射 |
| `UpdateAlternateFormat()` | 更新备用格式元数据映射 |

**实现细节**:

```cpp
void UpdateMetadata::UpdatePhoneNumber(
    scoped_ptr<std::map<int, PhoneMetadata>>& countryMetadataMap,
    scoped_ptr<std::map<std::string, PhoneMetadata>>& regionMetadataMap) {

    // 遍历更新的元数据
    for (auto it = regionMetadata->begin(); it != regionMetadata->end(); it++) {
        std::string code = it->first;
        PhoneMetadata phoneMetadata = it->second;

        // 合并策略：如果存在则更新，否则插入
        if (regionMetadataMap->find(code) != regionMetadataMap->end()) {
            regionMetadataMap->at(code) = phoneMetadata;
        } else {
            regionMetadataMap->insert(std::make_pair(code, phoneMetadata));
        }
    }
}
```

---

#### 更新文件路径

**元数据文件**: `/system/etc/icu_tzdata/i18n/MetadataInfo`

**地理编码文件**: `/system/etc/icu_tzdata/i18n/GeocodingInfo`

---

#### 数据格式

**元数据类型前缀**:
- `"short"` - 短号码元数据
- `"phone"` - 电话号码元数据
- `"alter"` - 备用格式元数据
- `"001"` - 非地理实体区域代码

**ID 编码**: `type + region_code`
- 示例: `shortUS`, `phoneCN`, `alter1`

---

### 2.4 地理编码更新机制

#### UpdateGeocoding 类

**位置**: `cpp/src/phonenumbers/ohos/update_geocoding.cc`

**关键方法**:

| 方法 | 功能 |
|------|------|
| `LoadGeocodingData(int fd)` | 从文件描述符解析地理编码数据 |
| `UpdatePrefixDescriptions()` | 更新前缀到描述的映射 |
| `UpdateLanguageCodes()` | 更新语言代码数组 |
| `UpdateCountryLanguages()` | 更新国家到语言的映射 |
| `UpdateCountryCodes()` | 更新国家代码数组 |

**数据结构** (geocoding_data.proto):

```protobuf
message GeocodingInfo {
    repeated PrefixesInfo prefixes_info = 1;    // 前缀信息
    repeated string languages = 2;               // 支持的语言
    required LanguageCodeInfo language_code_info = 3;
    repeated CountriesInfo countries_info = 4;   // 国家信息
    repeated int32 countries = 5;                // 国家代码
    required CountryCodeInfo country_code_info = 6;
}

message PrefixesInfo {
    required int32 prefixes_num = 1;
    repeated int32 prefixes = 2;                 // 前缀数组
    repeated string descriptions = 3;            // 描述数组
    required int32 lengths_num = 4;
    repeated int32 lengths = 5;                 // 长度数组
}
```

---

### 2.5 初始化调用流程

#### 电话号码库初始化

```
PhoneNumberUtil 构造函数
  │
  ├── 加载编译内置元数据 (原始行为)
  │   ├── metadata_get()
  │   └── ParseFromArray()
  │
  ├── #ifdef LIBPHONENUMBER_UPGRADE
  │   ├── UpdateLibphonenumber::LoadUpdateData()
  │   │   └── open("/system/etc/icu_tzdata/i18n/MetadataInfo")
  │   │       └── UpdateMetadata::LoadUpdatedMetadata(fd)
  │   │           └── 解析 Protobuf 数据
  │   │           └── 存储到静态变量
  │   │
  │   └── UpdateMetadata::UpdatePhoneNumber()
  │       └── 合并更新数据
  │
  └── #endif
```

#### 地理编码库初始化

```
exposeLocationName() (C 接口)
  │
  └── UpdateLibgeocoding::LoadUpdateData()
      └── open("/system/etc/icu_tzdata/i18n/GeocodingInfo")
          └── UpdateGeocoding::LoadGeocodingData(fd)
              └── 解析 Protobuf 数据
              └── 存储到静态变量
```

---

## 3. Patch 总结与维护建议

### Patch 分类统计

| 类型 | 数量 | 说明 |
|-----|------|------|
| Build Config | 1 | Maven 配置调整 |
| Runtime Update | 1 | LIBPHONENUMBER_UPGRADE 机制 |
| Geocoding Support | 1 | 地理编码更新机制 |
| **总计** | 3 | 实际 Patch 数量较少 |

### 维护建议

#### 升级上游版本

**必须保留**:
1. `LIBPHONENUMBER_UPGRADE` 宏及其相关代码
2. `ohos/` 目录下的所有源文件
3. `is_ohos` 条件编译逻辑

**风险点**:
1. 元数据格式变化
   - `PhoneMetadata` Protobuf 结构可能变化
   - 需要更新 `update_metadata.cc` 解析逻辑

2. API 变化
   - `PhoneNumberUtil` 构造函数签名可能变化
   - 需要调整初始化调用

3. 依赖库版本
   - abseil-cpp, protobuf, ICU 版本兼容性

**验证步骤**:
1. 编译 `phonenumber_standard` 和 `geocoding`
2. 运行单元测试
3. 测试元数据更新功能
4. 验证与 SMS/MMS 模块集成

#### 更新元数据

**流程**:
1. 从上游获取最新元数据（`resources/` 目录）
2. 使用 `generate_metadata` 工具生成 Protobuf 二进制
3. 使用前缀编码（type + region_code）
4. 放置到 `/system/etc/icu_tzdata/i18n/MetadataInfo`

**注意事项**:
- 确保元数据类型前缀正确
- 验证 Protobuf 格式兼容性
- 测试加载和合并逻辑

---

## 4. 证据清单

### 代码证据

- [x] BUILD.gn 第 110-112 行 - LIBPHONENUMBER_UPGRADE 定义
- [x] phonenumberutil.cc 第 37-40 行 - 头文件引入
- [x] phonenumberutil.cc 第 920-923 行 - 初始化修改
- [x] phonenumbermatcher.cc 第 46-49 行 - 头文件引入
- [x] phonenumbermatcher.cc 第 385-388 行 - 备用格式更新
- [x] shortnumberinfo.cc 第 24-27 行 - 头文件引入
- [x] shortnumberinfo.cc 第 67-70 行 - 短号码更新

### 文件证据

- [x] debian/patches/0003-maven-exclude-demo.patch - 实际 Patch 文件
- [x] cpp/src/phonenumbers/ohos/ 目录 - OHOS 特有文件
- [x] ohos/update_metadata.cc - 元数据更新实现
- [x] ohos/update_libphonenumber.cc - 初始化实现
- [x] ohos/update_geocoding.cc - 地理编码更新实现

### 配置证据

- [x] bundle.json - OH 组件信息
- [x] cpp/BUILD.gn - GN 构建配置
- [x] phonenumber_defines - 编译选项

---

**最后更新**: 2026-02-08
