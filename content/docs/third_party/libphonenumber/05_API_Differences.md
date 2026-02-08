# libphonenumber - API/接口差异

## 概述

libphonenumber 在 OpenHarmony 中通过**运行时元数据更新机制**（`LIBPHONENUMBER_UPGRADE`）和**OHOS 特有源文件**实现了部分功能扩展，但保持了与上游版本的核心 API 兼容性。本文档详细说明 OHOS 新增的 API、行为变更以及废弃或禁用的功能。

---

## OHOS 新增 API

### 1. 元数据更新 API

#### UpdateLibphonenumber 类

**头文件**: `cpp/src/phonenumbers/ohos/update_libphonenumber.h`

**新增类**:

```cpp
namespace i18n {
namespace phonenumbers {
class UpdateLibphonenumber {
public:
    /**
     * @brief 加载更新元数据
     *
     * 从 /system/etc/icu_tzdata/i18n/MetadataInfo 加载
     * 运行时更新的电话号码元数据
     *
     * @note 此方法在 PhoneNumberUtil 构造函数中调用
     */
    static void LoadUpdateData();

private:
    /**
     * @brief 元数据文件路径
     *
     * 常量路径：/system/etc/icu_tzdata/i18n/MetadataInfo
     */
    static const std::string METADATAINFO_PATH;
};
}
```

**新增文件**:
- `update_libphonenumber.cc` - 实现文件

---

#### UpdateMetadata 类

**头文件**: `cpp/src/phonenumbers/ohos/update_metadata.h`

**新增类**:

```cpp
namespace i18n {
namespace phonenumbers {
class UpdateMetadata {
public:
    /**
     * @brief 更新短号码元数据
     *
     * @param metadataMap 短号码元数据映射（Region → PhoneMetadata）
     */
    static void UpdateShortNumber(
        scoped_ptr<std::map<std::string, PhoneMetadata>>& metadataMap);

    /**
     * @brief 更新电话号码元数据
     *
     * @param countryMetadataMap 国家代码元数据映射（int → PhoneMetadata）
     * @param regionMetadataMap 区域代码元数据映射（string → PhoneMetadata）
     */
    static void UpdatePhoneNumber(
        scoped_ptr<std::map<int, PhoneMetadata>>& countryMetadataMap,
        scoped_ptr<std::map<std::string, PhoneMetadata>>& regionMetadataMap);

    /**
     * @brief 更新备用格式元数据
     *
     * @param metadataMap 备用格式元数据映射（int → PhoneMetadata*）
     */
    static void UpdateAlternateFormat(
        std::map<int, const PhoneMetadata*>& metadataMap);

    /**
     * @brief 从文件描述符加载更新元数据
     *
     * @param fd 文件描述符（已打开的 MetadataInfo 文件）
     */
    static void LoadUpdatedMetadata(int fd);

private:
    /**
     * @brief 元数据类型前缀
     */
    static const std::string METADATA_NAME;
    static const std::string SHORT_METADATA_TYPE;      // "short"
    static const std::string PHONE_METADATA_TYPE;      // "phone"
    static const std::string ALTERNATE_METADATA_TYPE;   // "alter"
    static const std::string REGION_CODE_NONGEO;     // "001"

    /**
     * @brief 静态存储更新元数据的指针
     */
    static PhoneMetadataCollection* phoneMetadataCollection;
    static std::map<std::string, PhoneMetadata>* shortMetadata;
    static std::map<std::string, PhoneMetadata>* regionMetadata;
    static std::map<int, PhoneMetadata>* countryMetadata;
    static std::map<int, PhoneMetadata>* alterMetadata;
};
}
```

**新增文件**:
- `update_metadata.cc` - 实现

---

### 2. 地理编码更新 API

#### UpdateLibgeocoding 类

**头文件**: `cpp/src/phonenumbers/ohos/update_libgeocoding.h`

**新增类**:

```cpp
namespace i18n {
namespace phonenumbers {
class UpdateLibgeocoding {
public:
    /**
     * @brief 加载更新地理编码数据
     *
     * 从 /system/etc/icu_tzdata/i18n/GeocodingInfo 加载
     * 运行时更新的地理编码数据
     *
     * @note 此方法在 geocoding_warpper 中每次调用时执行
     */
    static void LoadUpdateData();

private:
    /**
     * @brief 地理编码数据文件路径
     *
     * 常量路径：/system/etc/icu_tzdata/i18n/GeocodingInfo
     */
    static const std::string GEOCODINGINFO_PATH;
};
}
```

**新增文件**:
- `update_libgeocoding.cc` - 实现

---

#### UpdateGeocoding 类

**头文件**: `cpp/src/phonenumbers/ohos/update_geocoding.h`

**新增类**:

```cpp
namespace i18n {
namespace phonenumbers {
class UpdateGeocoding {
public:
    /**
     * @brief 更新前缀描述
     *
     * 将运行时更新数据合并到内置的前缀描述映射
     */
    static void UpdatePrefixDescriptions();

    /**
     * @brief 更新语言代码数组
     *
     * 更新支持的语言代码列表
     */
    static void UpdateLanguageCodes();

    /**
     * @brief 更新国家语言映射
     *
     * @param countries_info_map 国家语言映射
     */
    static void UpdateCountryLanguages(
        std::map<std::string, const PhoneMetadata*>& countries_info_map);

    /**
     * @brief 更新国家代码数组
     *
     * 更新国家调用代码列表
     */
    static void UpdateCountryCodes();

    /**
     * @brief 从文件描述符加载地理编码数据
     *
     * @param fd 文件描述符（已打开的 GeocodingInfo 文件）
     */
    static void LoadGeocodingData(int fd);

private:
    /**
     * @brief 静态存储更新数据的指针
     */
    static GeocodingInfo* geocodingInfo;
    static std::vector<std::string>* updated_languages;
    static std::vector<std::string>* updated_country_languages;
    static std::vector<int32_t>* updated_country_codes;
};
}
```

**新增文件**:
- `update_geocoding.cc` - 实现

---

### 3. Protobuf 数据格式

#### GeocodingInfo 消息

**proto 文件**: `cpp/src/phonenumbers/ohos/geocoding_data.proto`

**新增消息定义**:

```protobuf
syntax = "proto2";

option optimize_for = LITE_RUNTIME;

package i18n.phonenumbers;

/**
 * @brief 地理编码信息
 *
 * 用于运行时更新地理编码数据
 */
message GeocodingInfo {
    /**
     * @brief 前缀信息列表
     *
     * 包含电话前缀、描述和长度的映射
     */
    repeated PrefixesInfo prefixes_info = 1;

    /**
     * @brief 支持的语言列表
     *
     * 如："en", "zh", "es"
     */
    repeated string languages = 2;

    /**
     * @brief 语言代码信息（必填）
     */
    required LanguageCodeInfo language_code_info = 3;

    /**
     * @brief 国家信息列表
     *
     * 每个国家的语言信息
     */
    repeated CountriesInfo countries_info = 4;

    /**
     * @brief 国家调用代码列表
     *
     * 如：1, 44, 86
     */
    repeated int32 countries = 5;

    /**
     * @brief 国家代码信息（必填）
     */
    required CountryCodeInfo country_code_info = 6;
}

/**
 * @brief 前缀信息
 *
 * 包含前缀、描述和长度的映射
 */
message PrefixesInfo {
    required int32 prefixes_num = 1;   // 前缀数量
    repeated int32 prefixes = 2;       // 前缀数组
    repeated string descriptions = 3;   // 描述数组
    required int32 lengths_num = 4;   // 长度数量
    repeated int32 lengths = 5;       // 长度数组
}

/**
 * @brief 语言代码信息
 */
message LanguageCodeInfo {
    required int32 language_codes_num = 1;
    repeated string language_codes = 2;
}

/**
 * @brief 国家信息
 */
message CountriesInfo {
    required int32 country_languages_num = 1;
    repeated string country_languages = 2;
}

/**
 * @brief 国家代码信息
 */
message CountryCodeInfo {
    required int32 country_codes_num = 1;
    repeated int32 country_codes = 2;
}
```

**新增文件**:
- `geocoding_data.pb.h` - 生成的头文件（48 行）
- `geocoding_data.pb.cc` - 生成的实现（1889 行）

---

## 行为变更的 API

### 1. 元数据加载行为变更

#### 原始行为（无 LIBPHONENUMBER_UPGRADE）

```cpp
PhoneNumberUtil::PhoneNumberUtil() {
    // 1. 加载编译时嵌入的元数据
    PhoneMetadataCollection metadata_collection;
    LoadCompiledInMetadata(&metadata_collection);

    // 2. 填充元数据映射
    for (each metadata in collection) {
        // ... 直接使用编译时数据
    }
}
```

**OHOS 行为**（有 LIBPHONENUMBER_UPGRADE）:

```cpp
PhoneNumberUtil::PhoneNumberUtil() {
    // 1. 加载编译时嵌入的元数据（原始行为）
    PhoneMetadataCollection metadata_collection;
    LoadCompiledInMetadata(&metadata_collection);

    // 2. [OHOS] 加载运行时更新元数据
    #ifdef LIBPHONENUMBER_UPGRADE
      UpdateLibphonenumber::LoadUpdateData();
      UpdateMetadata::UpdatePhoneNumber(country_code_to_non_geographical_metadata_map_,
                                          region_to_metadata_map_);
    #endif

    // 3. 填充元数据映射（包含运行时更新的数据）
    for (each metadata in collection) {
        // ... 使用合并后的数据
    }
}
```

**变更说明**:
- **新增**: 运行时元数据加载能力
- **变更**: 初始化时加载两层元数据（编译时 + 运行时）
- **影响**: 支持无需重新编译库即可更新电话号码规则

---

### 2. 地理编码数据加载行为变更

#### 原始行为

地理编码数据仅在编译时嵌入到 `geocoding_data.cc`，无法运行时更新。

**OHOS 行为**:

```cpp
// geocoding_warpper.cc
extern "C" int exposeLocationName(const char* pNumber, ...) {
    // [OHOS] 每次调用时触发更新数据加载
    i18n::phonenumbers::UpdateLibgeocoding::LoadUpdateData();

    // 正常的地理编码查询逻辑
    // ...
}
```

**变更说明**:
- **新增**: 每次调用地理编码 API 前检查并加载更新数据
- **影响**: 支持运行时更新地理编码数据

---

### 3. 元数据合并策略

#### 更新方法实现

**UpdateMetadata::UpdatePhoneNumber**:

```cpp
void UpdateMetadata::UpdatePhoneNumber(
    scoped_ptr<std::map<int, PhoneMetadata>>& countryMetadataMap,
    scoped_ptr<std::map<std::string, PhoneMetadata>>& regionMetadataMap) {

    for (auto it = regionMetadata->begin(); it != regionMetadata->end(); it++) {
        std::string code = it->first;
        PhoneMetadata phoneMetadata = it->second;

        // 合并策略：如果存在则更新，否则插入
        if (regionMetadataMap->find(code) != regionMetadataMap->end()) {
            // 更新现有条目
            regionMetadataMap->at(code) = phoneMetadata;
        } else {
            // 插入新条目
            regionMetadataMap->insert(std::make_pair(code, phoneMetadata));
        }
    }
    // ... 对 countryMetadataMap 同样处理
}
```

**合并说明**:
- 新条目：插入到映射
- 现有条目：直接覆盖（更新）
- 删除条目：描述为 "NULL" 字符串的条目会被移除

---

## 废弃或禁用的功能

### 无废弃功能

目前 OpenHarmony 版本**没有废弃或禁用**上游 libphonenumber 的任何功能。

**说明**: 所有上游 API 在 OH 版本中都可用，OHOS 仅通过扩展方式新增运行时更新能力。

---

## API 兼容性

### 与上游版本兼容性

| 方面 | 兼容性 | 说明 |
|------|--------|------|
| **核心 API** | 100% 兼容 | PhoneNumberUtil、PhoneNumberOfflineGeocoder 等核心类 API 完全一致 |
| **数据结构** | 100% 兼容 | PhoneNumber、PhoneMetadata 等 Protobuf 结构一致 |
| **头文件** | 100% 兼容 | 公共头文件路径和内容一致 |
| **行为差异** | 扩展非变更 | 初始化时的元数据加载扩展，不影响正常 API 行为 |

### 向后兼容性

OHOS 的 libphonenumber 与上游版本**完全向后兼容**：
- 所有上游 API 都可用
- 新增的运行时更新机制是可选的（由 `LIBPHONENUMBER_UPGRADE` 宏控制）
- 不破坏现有代码

---

## 升级注意事项

### 1. 升级上游版本

当从当前版本（8.13.31）升级到新版本时：

**必须保留**:
1. `LIBPHONENUMBER_UPGRADE` 宏定义
2. `ohos/` 目录下的所有源文件
3. BUILD.gn 中的 `is_ohos` 条件编译逻辑

**需要适配**:
- `update_metadata.cc` 中的 Protobuf 解析代码可能需要调整
  - 检查 `PhoneMetadataCollection` 结构是否变化
  - 检查 `PhoneMetadata` 字段是否变化

- `update_libphonenumber.cc` 中的文件路径可能需要调整
  - 当前路径：`/system/etc/icu_tzdata/i18n/MetadataInfo`
  - 如路径变化需同步修改

- `phonenumberutil.cc` 中的初始化调用可能需要调整
  - 检查构造函数签名是否变化
  - 确保更新方法调用的时机正确

---

### 2. 数据格式变更

如果上游元数据格式发生变化：

**风险点**:
- Protobuf 消息字段新增或删除
- 元数据 ID 编码规则变化

**验证步骤**:
1. 编译新版本 libphonenumber（包含 OHOS 文件）
2. 运行单元测试
3. 测试元数据加载功能
4. 验证与 SMS/MMS 模块集成

---

### 3. 依赖库版本

**当前依赖版本**:
- ICU: 4.8+ (通过 `shared_icui18n`, `shared_icuuc`)
- protobuf: 最新版本（通过 `protobuf_lite`）
- abseil-cpp: 最新版本（通过 `absl_strings`, `absl_time`）

**升级建议**:
- 升级 ICU 到最新稳定版本
- 升级 protobuf 到兼容版本
- 确保所有依赖库的版本兼容性

---

## 测试建议

### 元数据更新功能测试

```cpp
// 测试 1: 验证元数据加载
TEST(UpdateMetadataTest, TestLoadUpdatedMetadata) {
    // 准备测试元数据文件
    // ...

    // 调用更新函数
    UpdateMetadata::LoadUpdatedMetadata(fd);

    // 验证数据是否正确加载
    EXPECT_GT(UpdateMetadata::shortMetadata->size(), 0);
}

// 测试 2: 验证元数据合并
TEST(UpdateMetadataTest, TestUpdatePhoneNumber) {
    // 创建元数据映射
    scoped_ptr<std::map<int, PhoneMetadata>> countryMetadataMap(
        new std::map<int, PhoneMetadata>());

    // 调用更新函数
    UpdateMetadata::UpdatePhoneNumber(countryMetadataMap, regionMetadataMap);

    // 验证数据是否正确合并
    EXPECT_TRUE(countryMetadataMap->count(86));
}
```

---

### 地理编码更新功能测试

```cpp
// 测试地理编码数据加载
TEST(GeocodingUpdateTest, TestLoadGeocodingData) {
    // 准备测试地理编码文件
    // ...

    // 调用更新函数
    UpdateGeocoding::LoadGeocodingData(fd);

    // 验证数据是否正确加载
    EXPECT_FALSE(UpdateGeocoding::geocodingInfo == nullptr);
}
```

---

## 总结

### 关键差异

1. **新增 API**:
   - `UpdateLibphonenumber` 类 - 电话号码元数据加载
   - `UpdateMetadata` 类 - 元数据更新逻辑
   - `UpdateLibgeocoding` 类 - 地理编码数据加载
   - `UpdateGeocoding` 类 - 地理编码更新逻辑

2. **行为变更**:
   - 初始化时加载运行时元数据（两层：编译时 + 运行时）
   - 地理编码 API 每次调用前检查并加载更新数据

3. **兼容性**:
   - 100% 向后兼容上游 API
   - 无废弃或禁用功能
   - 所有新增功能通过条件编译（`LIBPHONENUMBER_UPGRADE`）控制

4. **升级注意事项**:
   - 需要保留 OHOS 特有源文件
   - 验证 Protobuf 格式兼容性
   - 测试元数据更新功能

---

**最后更新**: 2026-02-08
