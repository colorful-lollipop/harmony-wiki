# 项目概览

> **适用对象**: 新人学习者、安全研究员
> **阅读时间**: 5 分钟
> **前置知识**: OpenHarmony 基础概念、DataShare 框架

---

## 目的与适用范围

本文档提供 RingtoneLibrary 项目的核心信息，帮助读者快速理解：
- 项目定位和解决的问题
- 核心能力和边界
- 运行环境和依赖
- 快速开始使用示例

**适用场景**:
- 开发者：了解项目架构，准备开发新功能
- 安全研究员：理解系统架构，识别潜在攻击面
- 系统集成者：了解如何使用 RingtoneLibrary API

---

## 一句话定义

**RingtoneLibrary** 是 OpenHarmony 多媒体子系统中的铃音管理系统，提供铃音、振动模式和 SIM 卡设置的增删改查功能，通过 DataShare 接口为系统应用和数据提供方服务。

**证据**:
- `bundle.json:2-4` - 组件定义
- `README_zh.md:9-10` - DataShareExtension 接口

---

## 核心能力

### 1. 铃音管理

| 功能 | 说明 | 证据 |
|------|------|------|
| 读取铃音信息 | 查询系统预制和用户自定义铃音 | `README_zh.md:16` |
| 存储自定义铃音 | 添加用户上传的铃音文件 | `README_zh.md:18` |
| 删除铃音 | 移除铃音记录和文件 | `README_zh.md:18` |
| 扫描系统铃音 | 自动索引系统预制铃音目录 | `README_zh.md:17` |

**支持的类型**:
- 来电铃音（RINGTONE）
- 闹钟铃音（ALARM）
- 通知铃音（NOTIFICATION）
- 短信铃音（SHOT）

**证据**: `interfaces/inner_api/native/ringtone_type.h:25-34` - ToneType 枚举

### 2. 振动模式管理

支持标准振动和弱化振动两种模式：
- **预制铃声**：支持同步振动与非同步振动
- **自定义铃声**：仅支持非同步振动

**证据**: `README_zh.md:243-252`

### 3. SIM 卡设置

支持双卡设备的铃音设置：
- SIM 卡 1 铃音
- SIM 卡 2 铃音
- 双卡铃音

**证据**: `README_zh.md:272-277` - ring_tone_type 字段

### 4. 静默数据访问

通过 `datashareproxy://` URI 实现不拉起铃音库进程的直接数据库访问，提高性能。

**证据**: `README_zh.md:184-241`

---

## 能力边界

### ✅ 能做什么

1. **数据管理**：对铃音、振动、SIM 卡设置进行增删改查
2. **扫描索引**：自动扫描系统预制铃音目录
3. **备份恢复**：支持跨版本数据迁移
4. **权限控制**：基于权限的访问控制

### ❌ 不能做什么

1. **音频播放**：不负责铃音的播放（由 player_framework 负责）
2. **音频录制**：不提供录音功能
3. **跨用户访问**：只能访问当前用户的数据
4. **非系统应用访问**：默认仅系统应用可访问

**证据**: `services/utils/src/permission_utils.cpp:161-178` - IsSystemApp() 实现

---

## 运行环境

### 系统要求

| 属性 | 要求 | 证据 |
|------|------|------|
| **子系统** | multimedia | `bundle.json:12` |
| **适配系统** | small, standard | `bundle.json:15` |
| **最低版本** | OpenHarmony 4.0 | `bundle.json:4` |

### 依赖的系统服务

| 服务 | 用途 | 证据 |
|------|------|------|
| DataShare | 数据共享框架 | `services/BUILD.gn:107-109` |
| RelationalStore (RDB) | 关系型数据库 | `services/BUILD.gn:121-122` |
| AccessToken | 权限管理 | `services/BUILD.gn:103` |
| MediaLibrary | 媒体库集成 | `bundle.json:41-42` |
| PlayerFramework | 媒体播放器 | `services/BUILD.gn:119` |

### 权限要求

| 权限 | 用途 | 证据 |
|------|------|------|
| `ohos.permission.WRITE_RINGTONE` | 铃音写操作 | `services/ringtone_data_extension/src/ringtone_datashare_extension.cpp:200` |
| `ohos.permission.ACCESS_CUSTOM_RINGTONE` | 静默访问自定义铃音 | `README_zh.md:200` |

---

## 快速开始

### 场景 1：查询所有系统铃音

```cpp
#include "data_share_data_share_helper.h"
#include "data_share_predicates.h"
#include "ringtone_db_const.h"

using namespace OHOS::DataShare;

// 获取 DataShareHelper
auto saManager = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
auto remoteObj = saManager->GetSystemAbility(STORAGE_MANAGER_MANAGER_ID);
std::shared_ptr<DataShareHelper> helper = DataShareHelper::Creator(remoteObj,
    "datashare:///ringtone/ringtone");

// 查询系统铃音（source_type = 1）
DataSharePredicates predicates;
predicates.EqualTo(RINGTONE_COLUMN_SOURCE_TYPE, "1");
predicates.EqualTo(RINGTONE_COLUMN_TONE_TYPE, "1"); // 来电铃音

vector<string> columns = {
    RINGTONE_COLUMN_TONE_ID,
    RINGTONE_COLUMN_TITLE,
    RINGTONE_COLUMN_DATA,
    RINGTONE_COLUMN_DURATION
};

DatashareBusinessError businessError;
auto resultSet = helper->Query(
    Uri("datashare:///ringtone/ringtone"),
    predicates,
    columns,
    &businessError
);

// 遍历结果
if (resultSet != nullptr && resultSet->GoToFirstRow() == 0) {
    do {
        int32_t id;
        string title;
        string path;
        int32_t duration;

        resultSet->GetInt(0, id);
        resultSet->GetString(1, title);
        resultSet->GetString(2, path);
        resultSet->GetInt(3, duration);

        MEDIA_LOGI("Ringtone: id=%d, title=%s, path=%s, duration=%dms",
            id, title.c_str(), path.c_str(), duration);
    } while (resultSet->GoToNextRow() == 0);
}
```

**证据**: `README_zh.md:153-182`

---

### 场景 2：添加自定义铃音

```cpp
#include "data_share_values_bucket.h"

using namespace OHOS::DataShare;

// 准备铃音数据
DataShareValuesBucket values;
values.Put(RINGTONE_COLUMN_DATA, "/data/storage/el2/base/files/Ringtone/my_ringtone.ogg");
values.Put(RINGTONE_COLUMN_SIZE, static_cast<int64_t>(2048));
values.Put(RINGTONE_COLUMN_DISPLAY_NAME, "my_ringtone.ogg");
values.Put(RINGTONE_COLUMN_TITLE, "My Ringtone");
values.Put(RINGTONE_COLUMN_MEDIA_TYPE, 2); // 音频
values.Put(RINGTONE_COLUMN_TONE_TYPE, 1); // 来电铃音
values.Put(RINGTONE_COLUMN_MIME_TYPE, "ogg");
values.Put(RINGTONE_COLUMN_SOURCE_TYPE, 2); // 用户自定义
values.Put(RINGTONE_COLUMN_DURATION, 15); // 15 秒
values.Put(RINGTONE_COLUMN_DATE_ADDED, 1707260800000L);
values.Put(RINGTONE_COLUMN_RING_TONE_TYPE, 1); // 卡1来电铃音

// 插入数据库
int32_t toneId = helper->Insert(
    Uri("datashare:///ringtone/ringtone"),
    values
);

if (toneId > 0) {
    MEDIA_LOGI("Ringtone added successfully, id=%d", toneId);
}
```

**证据**: `README_zh.md:67-100`

---

### 场景 3：静默访问（高性能）

```cpp
#include "ringtone_proxy_uri.h"

// 使用静默访问 URI（要求 ACCESS_CUSTOM_RINGTONE 权限）
Uri proxyUri(RINGTONE_LIBRARY_PROXY_DATA_URI_TONE_FILES + "&user=" +
    std::to_string(GetCurrentUserId()));

DataSharePredicates predicates;
predicates.EqualTo(RINGTONE_COLUMN_SOURCE_TYPE, "2"); // 仅查询自定义

auto resultSet = helper->Query(proxyUri, predicates, columns, &businessError);

// 静默访问不拉起铃音库进程，直接访问数据库
```

**证据**: `README_zh.md:214-241`

---

## 核心数据模型

### 铃音数据（RingtoneAsset）

| 字段 | 类型 | 说明 |
|------|------|------|
| ringtone_id | int32_t | 唯一标识 |
| data | string | 铃音文件路径 |
| title | string | 铃音名称 |
| duration | int32_t | 时长（毫秒） |
| tone_type | ToneType | 铃音类型 |
| source_type | SourceType | 来源（预制/自定义） |

**证据**: `interfaces/inner_api/native/ringtone_asset.h`

### 数据库表结构

| 表名 | 说明 | 记录数限制 |
|------|------|-----------|
| ToneFiles | 铃音表 | 最多 40 个视频铃音 |
| VibrateFiles | 振动表 | 无限制 |
| SimCardSetting | SIM 卡设置 | 单条记录 |
| PreloadConfig | 预加载配置 | 系统预置 |

**证据**: `README_zh.md:254-297`

---

## 关键结论

1. **定位**：RingtoneLibrary 是 OpenHarmony 系统服务层组件，提供 DataShare 接口的铃音管理服务。

2. **核心能力**：铃音/振动/SIM 卡设置的增删改查、自动扫描、静默访问。

3. **访问方式**：
   - 标准访问：`datashare:///ringtone/ringtone`
   - 静默访问：`datashareproxy://...?Proxy=true`（需要权限）

4. **权限控制**：默认仅系统应用可访问，普通应用需申请 `ACCESS_CUSTOM_RINGTONE` 权限。

5. **数据存储**：使用 RDB（Relational Database）存储，铃音文件存放在 `/data/storage/el2/base/files/Ringtone/`。

---

## 相关链接

- [架构与数据流](./02_Architecture.md) - 深入理解系统架构
- [对外接口文档](./04_Interface.md) - 完整 API 参考
- [目录结构与代码地图](./03_CodeMap.md) - 定位关键代码

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
