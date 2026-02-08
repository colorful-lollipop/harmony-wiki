# 05 - ICU OpenHarmony API 差异

## 5.1 概述

OpenHarmony 在保持 ICU 原有 API 不变的基础上，新增了一些 OpenHarmony 特有的功能和接口。

### 变更类型说明

| 类型 | 说明 | 示例 |
|------|------|------|
| 新增 API | OH 新增的功能接口 | `LunarCalendar` 类 |
| 扩展 API | 对原有功能的扩展 | `init_data` 初始化函数 |
| 行为变更 | 默认行为变化 | 数据文件路径 |
| 封装 API | Java/NDK 层封装 | `ohos.global.icu` 包 |

---

## 5.2 新增 C++ API

### 5.2.1 ICU 初始化接口

**头文件**: `icu4c/source/ohos/init_data.h`

#### 函数列表

| 函数 | 原型 | 说明 |
|------|------|------|
| `SetHwIcuDirectory` | `void SetHwIcuDirectory()` | 设置 ICU 数据目录为 `/system/usr/icu` |
| `SetArkuiXIcuDirectory` | `void SetArkuiXIcuDirectory(const char* dir)` | 设置自定义数据目录 (ArkUI-X) |
| `SetOhosIcuDirectory` | `void SetOhosIcuDirectory()` | 设置数据目录 (带检查) |
| `GetIcuVersion` | `const char* GetIcuVersion()` | 获取 ICU 版本号 |

#### 使用示例

```cpp
#include "ohos/init_data.h"

// 系统启动时初始化 ICU
void SystemInit() {
    SetHwIcuDirectory();
    
    // 验证版本
    const char* version = GetIcuVersion();
    // version = "74"
}
```

#### 注意事项

- 这些函数是**线程安全**的，内部使用 mutex 保护
- 只能初始化一次，重复调用会被忽略
- 必须在调用其他 ICU API 之前初始化

---

### 5.2.2 农历日历类

**头文件**: `icu4c/source/ohos/lunar_calendar.h`

#### 类定义

```cpp
namespace OHOS {
namespace ICU {

class LunarCalendar {
public:
    static int32_t NewYear(int32_t eyear);
    static int32_t NewMoonNear(int32_t days);
    static const std::string LunarType;
    
    LunarCalendar();
    ~LunarCalendar();
    
    // 设置日期 (通过 1970 基准天数)
    bool SetDaysFrom1970(int32_t year, int32_t month, int32_t day);
    
    // 获取公历日期
    int32_t GetSolarYear();
    int32_t GetSolarMonth();
    int32_t GetSolarDay();
    
    // 获取农历日期
    int32_t GetLunarYear();
    int32_t GetLunarMonth();
    int32_t GetLunarDay();
    
    // 获取农历扩展信息
    int32_t GetEra();           // 甲子周期 (第几个 60 年)
    int32_t GetCycleYear();     // 干支年 (1-60)
    int32_t GetDateOfYear();    // 当年第几天
    int32_t GetExtendedYear();  // 从黄帝纪年开始的年数
    
    // 判断闰月
    bool IsLeapMonth();
};

} // namespace ICU
} // namespace OHOS
```

#### 成员函数详解

##### SetDaysFrom1970

```cpp
bool SetDaysFrom1970(int32_t year, int32_t month, int32_t day)
```

设置公历日期，通过年月日和相对于 1970-01-01 的天数来转换。

**参数**:
- `year`: 公历年 (1970-2100)
- `month`: 公历月 (1-12)
- `day`: 相对于该月 1 日的偏移天数

**返回值**: 
- `true`: 日期有效，转换成功
- `false`: 日期无效

**示例**:
```cpp
OHOS::ICU::LunarCalendar lunar;

// 设置 2024年2月10日 (大年初一)
// 2月10日是 2月1日 + 9 天
lunar.SetDaysFrom1970(2024, 2, 9);
```

##### GetLunarYear / GetLunarMonth / GetLunarDay

获取转换后的农历日期。

**示例**:
```cpp
int32_t year = lunar.GetLunarYear();    // 2024 (甲辰年)
int32_t month = lunar.GetLunarMonth();  // 1 (正月)
int32_t day = lunar.GetLunarDay();      // 1 (初一)
```

##### GetEra / GetCycleYear

获取干支纪年信息。

**天干**: 甲、乙、丙、丁、戊、己、庚、辛、壬、癸
**地支**: 子、丑、寅、卯、辰、巳、午、未、申、酉、戌、亥

**示例**:
```cpp
int32_t era = lunar.GetEra();          // 第几个 60 年周期
int32_t cycleYear = lunar.GetCycleYear();  // 1-60 的干支序号

// 2024年: 甲辰年 = 第 41 个干支
// era = 36, cycleYear = 41
```

##### IsLeapMonth

判断当前农历月是否为闰月。

**示例**:
```cpp
// 2023年有闰二月
// 闰二月: IsLeapMonth() == true
// 正常二月: IsLeapMonth() == false
```

##### 静态辅助函数

```cpp
// 计算某年春节的 1970 基准天数
int32_t newYearDays = LunarCalendar::NewYear(2024);

// 计算距离某天最近的朔日
int32_t newMoonDays = LunarCalendar::NewMoonNear(days);
```

#### 使用完整示例

```cpp
#include "ohos/lunar_calendar.h"
#include <iostream>

void PrintLunarDate(int32_t solarYear, int32_t solarMonth, int32_t solarDay) {
    OHOS::ICU::LunarCalendar lunar;
    
    // 计算从该月 1 日开始的天数
    // 这里简化处理，假设输入的 day 就是实际日期
    bool valid = lunar.SetDaysFrom1970(solarYear, solarMonth, solarDay - 1);
    
    if (!valid) {
        std::cout << "Invalid date\n";
        return;
    }
    
    std::cout << "Solar: " << lunar.GetSolarYear() << "-"
              << lunar.GetSolarMonth() << "-"
              << lunar.GetSolarDay() << "\n";
    
    std::cout << "Lunar: " << lunar.GetLunarYear() << "-"
              << lunar.GetLunarMonth() << "-"
              << lunar.GetLunarDay();
    
    if (lunar.IsLeapMonth()) {
        std::cout << " (Leap)";
    }
    std::cout << "\n";
    
    std::cout << "GanZhi: Year " << lunar.GetCycleYear() << " of Era "
              << lunar.GetEra() << "\n";
}

// 输出:
// Solar: 2024-2-10
// Lunar: 2024-1-1
// GanZhi: Year 41 of Era 36
```

#### 限制与约束

| 限制项 | 说明 |
|--------|------|
| 年份范围 | 1900 - 2100 |
| 闰月计算 | 基于内置数据表，已预计算 |
| 精度 | 到日级别，不含时辰 |
| 节气 | 不直接提供节气计算 |

---

## 5.3 NDK 导出 API

### 5.3.1 符号导出控制

**文件**: `ohos_icu4c/libicu.map`

NDK 只导出部分 ICU C API，约 300 个常用符号。

#### 导出的主要 API 类别

| 类别 | 前缀 | 数量 | 说明 |
|------|------|------|------|
| 字符属性 | `u_char*` | 30+ | 字符类型、大小写转换 |
| 字符串操作 | `u_str*` | 40+ | Unicode 字符串操作 |
| 编码转换 | `ucnv_*` | 60+ | 字符编码转换 |
| 日历 | `ucal_*` | 40+ | 日历计算 |
| 日期格式化 | `udat_*` | 30+ | 日期时间格式化 |
| 排序 | `ucol_*` | 30+ | 字符串排序 |
| 数字格式化 | `unum*` | 40+ | 数字/货币格式化 |
| 文本断词 | `ubrk_*` | 20+ | 文本断词 |
| 正则表达式 | `uregex*` | 10+ | 正则表达式 |
| 规范化 | `unorm*` | 15+ | Unicode 规范化 |
| IDNA | `uidna*` | 10+ | 国际化域名 |
| 本地化 | `uloc_*` | 30+ | 本地化操作 |

#### 使用示例

```cpp
// 字符类型判断
UChar32 c = u'中';
UCharCategory cat = u_charType(c);  // U_OTHER_LETTER

// 字符串比较
UChar str1[] = { 'a', 'b', 'c', 0 };
UChar str2[] = { 'A', 'B', 'C', 0 };
int32_t result = u_strcasecmp(str1, str2, 0);  // 0 (相等)

// 编码转换
UConverter* conv = ucnv_open("gb18030", &status);
```

### 5.3.2 与标准 ICU 的差异

| 差异项 | 标准 ICU | OpenHarmony NDK |
|--------|----------|-----------------|
| 库名 | libicuuc.so | libicu.so (NDK) |
| 符号数量 | 全部导出 | 约 300 个 |
| C++ API | 支持 | 不支持 (C only) |
| 版本 | 最新 | 固定版本 (74.2) |

---

## 5.4 Java API 封装

### 5.4.1 包名映射

| 标准 ICU4J | OpenHarmony |
|------------|-------------|
| `com.ibm.icu.*` | `ohos.global.icu.*` |

### 5.4.2 主要类映射

| 功能 | 标准 ICU4J | OpenHarmony |
|------|-----------|-------------|
| 日期格式化 | `com.ibm.icu.text.DateFormat` | `ohos.global.icu.text.DateFormat` |
| 数字格式化 | `com.ibm.icu.text.NumberFormat` | `ohos.global.icu.text.NumberFormat` |
| 日历 | `com.ibm.icu.util.Calendar` | `ohos.global.icu.util.Calendar` |
| 本地化 | `com.ibm.icu.util.ULocale` | `ohos.global.icu.util.ULocale` |
| 排序 | `com.ibm.icu.text.Collator` | `ohos.global.icu.text.Collator` |
| 断词 | `com.ibm.icu.text.BreakIterator` | `ohos.global.icu.text.BreakIterator` |

### 5.4.3 使用差异

**标准 ICU4J**:
```java
import com.ibm.icu.text.DateFormat;
import com.ibm.icu.util.ULocale;

DateFormat df = DateFormat.getDateInstance(
    DateFormat.DEFAULT, 
    new ULocale("zh_CN")
);
```

**OpenHarmony**:
```java
import ohos.global.icu.text.DateFormat;
import ohos.global.icu.util.ULocale;

// 使用方式完全相同
DateFormat df = DateFormat.getDateInstance(
    DateFormat.DEFAULT, 
    new ULocale("zh_CN")
);
```

### 5.4.4 数据文件差异

| 特性 | 标准 ICU4J | OpenHarmony |
|------|-----------|-------------|
| 数据加载 | 从 jar 包加载 | 预加载到内存 |
| 数据版本 | 随库版本 | 与系统 ICU 一致 |
| 裁剪 | 无 | 根据配置裁剪 |

---

## 5.5 行为差异

### 5.5.1 数据文件路径

| 场景 | 标准 ICU | OpenHarmony |
|------|----------|-------------|
| 默认路径 | 环境变量/编译配置 | `/system/usr/icu` |
| 修改方式 | 环境变量 | `SetArkuiXIcuDirectory()` |
| 多版本 | 支持 | 支持 (新旧数据共存) |

### 5.5.2 时区数据

| 特性 | 标准 ICU | OpenHarmony |
|------|----------|-------------|
| 数据来源 | ICU 自带 | ICU + 系统 TZData |
| 更新方式 | ICU 升级 | 系统更新 |
| 路径 | 内置 | `/system/etc/icu_tzdata` |

### 5.5.3 默认本地化

| 场景 | 标准 ICU | OpenHarmony |
|------|----------|-------------|
| 默认语言 | 系统语言 | 系统设置 |
| 回退策略 | ICU 默认 | ICU 默认 |
| 可用语言 | 全部 | 裁剪后列表 |

---

## 5.6 升级兼容性

### 5.6.1 二进制兼容性

| 变更类型 | 兼容性 | 处理建议 |
|----------|--------|----------|
| 新增 API | 向后兼容 | 可直接使用 |
| 删除 API | 不兼容 | 需修改代码 |
| 修改行为 | 视情况而定 | 需测试验证 |

### 5.6.2 数据兼容性

| 数据版本 | 兼容性 | 说明 |
|----------|--------|------|
| 相同版本 | 完全兼容 | - |
| 新 -> 旧 | 可能不兼容 | 新数据格式可能不被旧库识别 |
| 旧 -> 新 | 通常兼容 | 旧格式通常被支持 |

### 5.6.3 迁移指南

从标准 ICU 迁移到 OpenHarmony ICU：

1. **C++ 代码**:
   ```cpp
   // 添加初始化
   #include "ohos/init_data.h"
   SetHwIcuDirectory();
   ```

2. **Java 代码**:
   ```java
   // 修改 import
   // import com.ibm.icu.*; 
   import ohos.global.icu.*;
   ```

3. **构建配置**:
   ```gn
   // 修改依赖
   deps = [ "//third_party/icu/icu4c:shared_icuuc" ]
   ```
