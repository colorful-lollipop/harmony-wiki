# 05_API_Differences.md - API/接口差异

## 概述

**tzdata 在 OpenHarmony 中没有 API 差异**，因为：

1. **纯数据组件**：tzdata 仅提供时区数据文件，不提供可链接的 API
2. **无代码修改**：没有 Patch 文件，也没有 OH 特有的代码修改
3. **数据格式一致**：使用标准的 IANA 时区数据格式

本文档说明 tzdata 的 API/数据提供方式，以及与其他系统的对比。

---

## 1. tzdata 的"接口"

### 1.1 数据文件作为接口

tzdata 的核心"接口"是其提供的数据文件：

| "接口" | 类型 | 路径 | 说明 |
|-------|------|------|------|
| `tzdata` | 二进制数据 | `/system/etc/zoneinfo/tzdata` | IANA 时区数据库 |
| `timezone_list.cfg` | 文本配置 | `/system/etc/zoneinfo/timezone_list.cfg` | 时区名称列表 |

### 1.2 与标准 API 的关系

在标准 Linux 系统中，tzdata 通过以下方式被使用：

```c
// 标准 C 库方式 - 通过 TZ 环境变量
setenv("TZ", "Asia/Shanghai", 1);
tzset();
localtime_r(&time, &tm);

// 直接读取时区文件
// 使用 tzfile.h 定义的结构解析 /usr/share/zoneinfo/Asia/Shanghai
```

在 OpenHarmony 中，使用方式类似，只是路径不同：

```c
// OH 系统中
// 时区数据位于 /system/etc/zoneinfo/
// 应用通常通过 ICU 而非直接访问
```

---

## 2. OH 与上游的差异

### 2.1 无 API 差异

| 方面 | 上游 | OpenHarmony | 差异 |
|-----|------|-------------|------|
| **数据格式** | IANA 二进制格式 | IANA 二进制格式 | **无差异** |
| **文件结构** | 按大陆/国家组织 | 合并为单个 tzdata 文件 | 组织方式不同，格式相同 |
| **访问方式** | 直接文件系统访问 | 通过 ICU 或直接访问 | 使用方式不同 |

### 2.2 数据组织差异

#### 上游 Linux 系统

```
/usr/share/zoneinfo/
├── Africa/
│   ├── Abidjan
│   ├── Accra
│   └── ...
├── Asia/
│   ├── Shanghai
│   ├── Tokyo
│   └── ...
├── Europe/
├── America/
└── ...
```

#### OpenHarmony 系统

```
/system/etc/zoneinfo/
├── tzdata              # 合并后的二进制数据
└── timezone_list.cfg   # 时区名称列表
```

**说明**：
- OH 使用单个合并的 `tzdata` 文件
- 上游通常保留按地区组织的多个文件
- 数据格式（tzfile）是相同的

---

## 3. 与其他时区数据提供方式的对比

### 3.1 三种时区数据来源

| 来源 | 路径 | 格式 | 主要使用者 |
|-----|------|------|-----------|
| **tzdata (本库)** | `/system/etc/zoneinfo/` | IANA 原生 | 系统底层、回退 |
| **icu_tzdata** | `/system/etc/icu_tzdata/` | ICU 格式 | ICU 库（主要） |
| **tzdata_distro** | `/system/etc/tzdata_distro/` | 未确认 | i18n 模块 |

### 3.2 ICU 格式的差异

ICU 使用自己的时区数据格式（`.res` 文件）：

```
base/global/timezone/data/prebuild/icu/
└── zoneinfo64.res      # ICU 二进制格式
```

**差异说明**：
- ICU 格式是 ICU 库专有的资源格式
- 包含与 IANA tzdata 相同的信息，但编码方式不同
- ICU 库优先使用自己的格式

---

## 4. OH 特有的"扩展"

### 4.1 timezone_list.cfg

这是 OH 特有的配置文件，上游 tzdata 不包含此文件：

```
Africa/Abidjan
Africa/Accra
...
Asia/Shanghai
...
```

**用途**：
- 列出系统中可用的时区
- 供系统设置应用展示时区列表
- 验证时区名称的有效性

### 4.2 平台差异化

OH 提供了不同平台的时区列表：

| 文件 | 路径 | 适用平台 |
|-----|------|---------|
| 标准版 | `data/prebuild/posix/timezone_list.cfg` | 手机、平板、TV 等 |
| 穿戴版 | `data/prebuild/wearable/timezone_list.cfg` | 智能手表等 |

**差异**：
- 穿戴设备版本包含更少的时区
- 节省存储空间
- 满足穿戴设备的资源限制

---

## 5. 使用建议

### 5.1 应用开发者

**推荐方式**：通过 ICU 库使用时区功能

```cpp
#include "unicode/timezone.h"
#include "unicode/calendar.h"

// 获取时区
TimeZone* tz = TimeZone::createTimeZone("Asia/Shanghai");

// 使用时区
Calendar* cal = Calendar::createInstance(tz, status);
```

**优势**：
- 跨平台一致
- 丰富的 API
- 自动处理时区数据

### 5.2 系统开发者

如需直接访问 tzdata：

```c
#include "tzfile.h"  // tzdata 提供的头文件

// 直接读取 /system/etc/zoneinfo/tzdata
// 解析时区规则
```

**注意事项**：
- 确保路径正确（`/system/etc/zoneinfo/`）
- 处理数据格式版本兼容性
- 考虑通过 ICU 间接访问

---

## 6. 无 API 差异的原因分析

### 6.1 设计哲学

tzdata 作为基础数据组件，其设计目标是：
1. **数据中立**：仅提供原始数据，不规定使用方式
2. **格式标准**：使用行业标准的 IANA 格式
3. **平台无关**：同样的数据可用于任何平台

### 6.2 OH 的适配策略

OH 通过以下方式适配，而非修改数据格式：
1. **预构建数据**：使用标准 zic 工具生成
2. **路径配置**：通过 ICU 宏指定数据路径
3. **包装层**：通过 ICU 库提供高级 API

这种设计保持了与上游的兼容性，同时满足 OH 的需求。

---

## 7. 总结

| 项目 | 状态 |
|-----|------|
| **API 修改** | 无 |
| **数据格式修改** | 无 |
| **接口新增** | 无 |
| **OH 特有扩展** | `timezone_list.cfg`（时区列表配置） |
| **平台适配** | 差异化时区列表（standard vs wearable） |

### 关键结论

1. **无 API 差异**：tzdata 是纯数据组件，没有 API 层
2. **数据格式一致**：使用标准的 IANA 格式
3. **OH 特有文件**：`timezone_list.cfg` 是 OH 添加的辅助文件
4. **推荐使用 ICU**：应用应通过 ICU 库使用时区功能，而非直接访问 tzdata

---

## 8. 相关文档

- [01_Overview.md](./01_Overview.md) - tzdata 概述和在 OH 中的作用
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系和使用方式
- [ICU 时区文档](https://unicode-org.github.io/icu/userguide/datetime/timezone/)
