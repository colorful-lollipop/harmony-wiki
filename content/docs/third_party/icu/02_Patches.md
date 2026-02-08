# 02 - ICU OpenHarmony 适配分析

## 2.1 适配方式概述

与传统第三方库使用 `.patch` 文件进行适配不同，ICU 在 OpenHarmony 中采用了**源码级直接修改 + OH 专用扩展**的方式。

### 为什么选择这种方式

| 对比项 | Patch 方式 | 源码级修改 |
|--------|-----------|-----------|
| 代码清晰度 | 分散在 patch 文件中 | 直接可见 |
| 升级维护 | 需要 rebase patch | 手动合并修改 |
| 代码审查 | 需对比 patch | 直接审查源码 |
| OH 特有扩展 | 通过 patch 添加 | 独立目录组织 |

ICU 作为基础库，其 OH 适配主要是**新增功能**（如农历）而非**修改原有逻辑**，因此采用扩展方式更合适。

---

## 2.2 OH 专用扩展模块

### 2.2.1 初始化与数据路径适配

**文件位置**: `icu4c/source/ohos/init_data.h`, `icu4c/source/ohos/init_data.cpp`

#### 修改目的

ICU 默认从文件系统加载数据文件，OpenHarmony 需要：
1. 指定系统预装数据文件路径 (`/system/usr/icu`)
2. 支持不同产品形态的数据路径
3. 提供线程安全的初始化机制

#### 代码分析

```cpp
// init_data.cpp
const char* g_hwDirectory = "/system/usr/icu";

void SetHwIcuDirectory() {
    std::lock_guard<std::mutex> lock(dataMutex);
    if (status != 0) { return; }
    u_setDataDirectory(g_hwDirectory);
    status = 1;
}

void SetArkuiXIcuDirectory(const char* dir) {
    std::lock_guard<std::mutex> lock(dataMutex);
    if (status != 0) { return; }
    u_setDataDirectory(dir);
    status = 1;
}

void SetOhosIcuDirectory() {
    std::lock_guard<std::mutex> lock(dataMutex);
    const char* currDir = u_getDataDirectory();
    if (strncmp(currDir, g_hwDirectory, strlen(g_hwDirectory)) == 0) {
        return;
    }
    u_setDataDirectory(g_hwDirectory);
}

const char* GetIcuVersion() {
    return U_ICU_VERSION_SHORT;  // "74"
}
```

#### 关键点说明

| 函数 | 用途 | 调用场景 |
|------|------|----------|
| `SetHwIcuDirectory` | 设置 OH 系统数据目录 | 系统启动时 |
| `SetArkuiXIcuDirectory` | 设置自定义数据目录 | ArkUI-X 跨平台 |
| `SetOhosIcuDirectory` | 带检查的设置 | 重复调用保护 |
| `GetIcuVersion` | 获取版本号 | 版本检查 |

**线程安全**: 使用 `std::mutex` 保护 `status` 变量，确保初始化只执行一次

#### OH 价值

- **系统一致性**: 所有应用使用统一的 ICU 数据
- **安全性**: 数据文件放在 `/system` 分区，防止篡改
- **灵活性**: 支持不同产品形态的数据路径配置

### 2.2.2 中国农历日历支持

**文件位置**: `icu4c/source/ohos/lunar_calendar.h`, `icu4c/source/ohos/lunar_calendar.cpp`

#### 修改目的

为满足中国用户的传统日历需求，OpenHarmony 在 ICU 基础上新增了中国农历计算功能。

#### 功能范围

| 功能 | 支持范围 | 说明 |
|------|----------|------|
| 年份范围 | 1900 - 2100 | 200 年农历数据 |
| 公历转农历 | 支持 | 输入公历日期，输出农历日期 |
| 闰月处理 | 支持 | 正确识别闰月 |
| 天干地支 | 支持 | 计算干支纪年 |

#### 核心数据结构

```cpp
// 1900-2100 年农历编码数据 (共 200 个)
// 每个编码包含: 闰月位置(低 4 位) + 每月天数(位掩码) + 闰月天数(第 17 位)
static const std::vector<uint32_t> lunarDateInfo {
    0x04bd8, 0x04ae0, 0x0a570, 0x054d5, 0x0d260,  // 1900-1904
    // ... (共 200 个值)
    0x0d520,  // 2100
};

// 公历每月天数
static const std::vector<int32_t> daysOfMonth { 
    31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 
};

// 公历每月累计天数 (非闰年)
static const std::vector<int32_t> accDaysOfMonth {
    0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334
};
```

#### 农历编码解析

以编码 `0x04bd8` (1900 年) 为例：

```
二进制: 0000 0100 1011 1101 1000

位 0-3  (低 4 位): 1000 = 8 → 闰 8 月
位 4-16 (13 位):  每个位代表一个月的天数 (1=30天, 0=29天)
位 17: 0 → 闰月为小月 (29天)
```

#### 关键算法

**1. 公历转农历**

```cpp
bool LunarCalendar::SetDaysFrom1970(int32_t year, int32_t month, int32_t daysFrom1970) {
    // 计算公历日期
    int32_t day = daysFrom1970 + CalcDaysFromBaseDate(1970, 1, 1) 
                  - CalcDaysFromBaseDate(year, month, 1) + 1;
    
    // 验证日期有效性
    isValidDate = VerifyDate(year, month, day);
    if (!isValidDate) { return false; }
    
    solarYear = year;
    solarMonth = month;
    solarDay = day;
    
    // 计算从基准日期(1900-01-01)的总天数
    daysCounts = CalcDaysFromBaseDate(solarYear, solarMonth, solarDay);
    
    // 转换为农历
    SolarDateToLunarDate();
    return true;
}
```

**2. 计算农历天数**

```cpp
void LunarCalendar::SolarDateToLunarDate() {
    int32_t tempDaysCounts = daysCounts - DAYS_FROM_SOLAR_TO_LUNAR; // 减去偏移
    
    // 逐年减去，确定农历年
    for (i = START_YEAR; (tempDaysCounts > 0) && (i < END_YEAR); i++) {
        daysInPerLunarYear = GetDaysPerLunarYear(i);
        tempDaysCounts -= daysInPerLunarYear;
    }
    lunarYear = i;
    
    // 逐月减去，确定农历月和日
    leapMonth = GetLeapMonthInYear(lunarYear);
    // ... 处理闰月逻辑
}
```

**3. 闰月判断**

```cpp
int32_t LunarCalendar::GetLeapMonthInYear(int32_t year) {
    // 取编码低 4 位
    return lunarDateInfo[year - START_YEAR] & 0xf;
}

int32_t LunarCalendar::GetLeapDaysInYear(int32_t year) {
    if (GetLeapMonthInYear(year) != 0) {
        // 检查第 17 位
        return (lunarDateInfo[year - START_YEAR] & 0x10000) 
               == 0x10000 ? 30 : 29;
    }
    return 0;
}
```

#### 静态辅助函数

```cpp
// 计算某年春节在 1970 年基准的偏移天数
int32_t LunarCalendar::NewYear(int32_t eyear);

// 计算距离某天最近的朔日
int32_t LunarCalendar::NewMoonNear(int32_t days);
```

#### OH 价值

- **原生支持**: 应用无需自行实现复杂的农历算法
- **系统一致**: 日历、时钟等应用使用统一的农历数据源
- **准确性**: 使用标准天文算法生成的农历数据表

#### 升级风险

| 风险项 | 说明 | 缓解措施 |
|--------|------|----------|
| 数据表大小 | 新增 200 个整数数据 | 体积影响可忽略 |
| 范围限制 | 仅支持 1900-2100 | 满足绝大多数使用场景 |
| 算法正确性 | 需验证闰月计算 | 需对比标准农历数据测试 |

---

## 2.3 数据裁剪配置

**文件位置**: `data_filter.json`

### 裁剪策略

采用 **subtractive** (减法) 策略：默认包含所有数据，只排除不需要的部分。

### 语言/地区裁剪

支持的语言从 ICU 默认的 700+ 减少到约 20 个：

| 语言组 | 包含 |
|--------|------|
| 中文系列 | zh, zh_CN, zh_Hans, zh_Hant, zh_Hant_HK, zh_Hant_TW |
| 英语系列 | en, en_US, en_GB, en_001, en_IN |
| 欧洲主要 | fr, de, es, it, pt |
| 亚洲主要 | ja, ko, th, ar |
| 其他 | id, tr, lt, sv, ug, bo |

### 功能树裁剪

对每种数据类型都进行了裁剪：

- `localeFilter`: 本地化数据
- `lang_tree`: 语言数据
- `region_tree`: 地区数据
- `zone_tree`: 时区数据
- `curr_tree`: 货币数据
- `unit_tree`: 单位数据
- `coll_tree`: 排序规则
- `brkitr_tree`: 断词规则
- `rbnf_tree`: 数字格式化规则

### 编码映射裁剪

仅保留中文相关编码：

```json
"conversion_mappings": {
    "includelist": [
        "gb18030",
        "ibm-1386_P100-2001",
        "windows-936-2000",
        "ibm-1383_P110-1999",
        "ibm-5478_P100-1995"
    ]
}
```

---

## 2.4 适配清单总结

### 源码修改汇总

| 类型 | 文件路径 | 说明 |
|------|----------|------|
| 新增 | `icu4c/source/ohos/init_data.h` | 数据路径初始化接口 |
| 新增 | `icu4c/source/ohos/init_data.cpp` | 数据路径初始化实现 |
| 新增 | `icu4c/source/ohos/lunar_calendar.h` | 农历日历接口 |
| 新增 | `icu4c/source/ohos/lunar_calendar.cpp` | 农历日历实现 |
| 修改 | `icu4c/source/BUILD.gn` | 添加 OHOS 源文件到构建 |

### 配置/脚本修改汇总

| 类型 | 文件路径 | 说明 |
|------|----------|------|
| 新增 | `icu.gni` | 全局 ICU 构建参数 |
| 新增 | `icu4c/icu4c_config.gni` | ICU4C 子系统配置 |
| 修改 | `icu4c/BUILD.gn` | 主构建脚本 |
| 修改 | `icu4c/source/BUILD.gn` | 主机工具构建脚本 |
| 新增 | `data_filter.json` | 数据裁剪配置 |

### NDK/Java 封装

| 类型 | 文件路径 | 说明 |
|------|----------|------|
| 新增 | `ohos_icu4c/BUILD.gn` | NDK 库构建 |
| 新增 | `ohos_icu4c/src/icu_addon.cpp` | NDK 扩展实现 |
| 新增 | `ohos_icu4c/libicu.map` | 符号导出控制 |
| 新增 | `ohos_icu4j/BUILD.gn` | Java 库构建 |
| 新增 | `ohos_icu4j/src/main/java/ohos/global/icu/` | Java API 封装 |

---

## 2.5 升级建议

### 向上游合并的可行性

| 修改项 | 可合并性 | 说明 |
|--------|----------|------|
| 农历支持 | 低 | OH 特有需求，ICU 已有 ChineseCalendar |
| 数据路径初始化 | 中 | 可通过环境变量或配置实现 |
| 数据裁剪配置 | 高 | 可作为示例配置提供 |

### 版本升级检查清单

- [ ] 检查新版本的 API 兼容性
- [ ] 更新 `data_filter.json` 中的数据版本
- [ ] 重新生成裁剪后的数据文件
- [ ] 验证农历数据在新版本下的正确性
- [ ] 检查 `libicu.map` 是否需要更新符号
- [ ] 更新 `icu4c/source/common/unicode/uvernum.h` 引用
- [ ] 测试 NDK 和 Java 封装接口

---

## 2.6 回归测试建议

### 功能测试

| 测试项 | 测试内容 |
|--------|----------|
| 数据加载 | 验证数据文件能从 `/system/usr/icu` 正确加载 |
| 农历计算 | 验证 1900-2100 年的农历转换正确 |
| 多语言 | 验证裁剪后的语言数据完整 |
| 时区 | 验证时区转换正确 |
| 编码转换 | 验证 GB18030 等中文编码转换正确 |

### 性能测试

| 测试项 | 指标 |
|--------|------|
| 启动时间 | ICU 初始化时间 < 50ms |
| 内存占用 | 数据文件加载后内存增量 < 20MB |
| 农历计算 | 单次计算 < 1ms |
