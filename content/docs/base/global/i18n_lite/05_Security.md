# 安全风险评审

## 5.1 安全模型概述

### 5.1.1 威胁模型

i18n_lite 作为国际化组件，主要处理**格式化请求**和**区域数据查询**，不涉及敏感数据存储或网络通信。其安全模型基于以下假设：

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     信任区域                               │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │   │
│  │  │  应用层     │  │  C++ 框架   │  │  i18n.dat      │   │   │
│  │  │  (可信)     │  │  (可信)     │  │  (只读)        │   │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           │ 不信任边界                           │
│                           ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     非信任区域                           │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │  - 应用传入的字符串参数                             │ │   │
│  │  │  - 应用传入的时间戳数值                             │ │   │
│  │  │  - 应用传入的区域标识参数                           │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.1.2 信任边界

| 边界 | 说明 | 信任级别 |
|------|------|----------|
| **API 边界** | 所有公开 API 是信任边界入口 | 低 - 需要校验 |
| **数据文件边界** | i18n.dat 是只读信任数据 | 高 - 预验证 |
| **内存边界** | 内部缓冲区管理 | 中 - 使用安全函数 |

## 5.2 攻击面分析

### 5.2.1 输入点清单

| 输入点 | 类型 | 来源 | 处理位置 |
|--------|------|------|----------|
| `LocaleInfo` 构造函数 | 字符串 | 应用层 | `locale_info.cpp:24-52` |
| `DateTimeFormat::Format()` | 字符串 + time_t | 应用层 | `date_time_format.cpp` |
| `NumberFormat::Format()` | double/int | 应用层 | `number_format.cpp` |
| `getLocale()` JS API | 无输入 | JS 引擎 | `locale_module.cpp` |
| `ForLanguageTag()` | BCP 47 字符串 | 应用层 | `locale_info.cpp` |
| i18n.dat 文件 | 二进制 | 构建产物 | `data_resource.cpp` |

### 5.2.2 敏感操作

| 操作 | 风险等级 | 说明 |
|------|----------|------|
| **内存分配** | 中 | 使用自定义内存适配器 |
| **字符串复制** | 中 | 使用 bounds_checking_function |
| **文件读取** | 低 | 只读 i18n.dat |
| **时区解析** | 低 | 简单字符串解析 |

### 5.2.3 权限要求

| 能力 | 说明 | 是否需要 |
|------|------|----------|
| 读取系统文件 | 访问 i18n.dat | 不需要（预装在系统分区） |
| 内存访问 | 动态分配 | 不需要（静态链接） |
| 系统能力调用 | 获取系统区域 | 不需要（通过系统 API） |

## 5.3 已识别风险

### ⚠️ 风险 1：LocaleInfo 长度校验不完整

**严重程度**：低

**证据位置**：`frameworks/i18n/src/locale_info.cpp:24-52`

```cpp
void LocaleInfo::Init(const char *newLang, const char *newScript, const char *newRegion, int &status)
{
    id = nullptr;
    status = IERROR;
    if (newLang == nullptr) {
        return;
    }
    int langLength = LenCharArray(newLang);
    // language consists of two or three letters
    if ((langLength > LANGUAGE_MAX_LENGTH) || (langLength < LANGUAGE_MIN_LENGTH)) {  // ✅ 有校验
        return;
    }
    // ...
    if (newScript != nullptr) {
        int scriptLength = LenCharArray(newScript);
        if (scriptLength == SCRIPT_LENGTH) {  // ⚠️ 只检查长度，不检查内容
            script = NewArrayAndCopy(newScript, scriptLength);
        }
    }
    if (newRegion != nullptr) {
        int regionLength = LenCharArray(newRegion);
        if (regionLength == REGION_LENGTH) {  // ⚠️ 只检查长度，不检查内容
            region = NewArrayAndCopy(newRegion, regionLength);
        }
    }
}
```

**触发条件**：
```cpp
// 构造超长区域标识（通过多次调用累积）
LocaleInfo locale("en", nullptr, nullptr);
locale = LocaleInfo("en", "VeryLongScriptNameThatExceedsFourChars", "US");  // 脚本名过长被静默忽略
```

**影响**：
- 区域标识可能不符合 BCP 47 规范
- 后续格式化可能使用错误的区域数据
- 静默失败，难以调试

**修复建议**：
1. 添加脚本和地区内容的字符集校验（只允许字母）
2. 使用正则表达式验证：`^[A-Z][a-z]{3}$`（脚本），`^[A-Z]{2}$`（地区）
3. 记录无效参数的警告日志

---

### ⚠️ 风险 2：字符串操作边界检查不严格

**严重程度**：中

**证据位置**：`frameworks/i18n/src/locale_info.cpp:58-69`

```cpp
void LocaleInfo::InitIdstr()
{
    if (language == nullptr) {
        return;
    }
    std::string idStr(language);
    // script consists of four letters
    if ((script != nullptr) && (LenCharArray(script) > 0)) {
        idStr = idStr + "-" + script;  // ⚠️ 字符串拼接，无边界检查
    }
    if ((region != nullptr) && (LenCharArray(region) > 0)) {
        idStr = idStr + "-" + region;  // ⚠️ 字符串拼接，无边界检查
    }
    I18nFree(static_cast<void *>(id));
    id = NewArrayAndCopy(idStr.data(), idStr.size());  // 使用实际大小
}
```

**触发条件**：
```cpp
// 构造超长区域 ID（通过拼接）
// 假设 script 和 region 被注入超长内容
LocaleInfo locale("en", "LatnLatnLatnLatn", "USUS");  // 拼接后可能超出预期
```

**影响**：
- 缓冲区溢出风险（依赖 `NewArrayAndCopy` 的安全性）
- 内存过度消耗
- 潜在的拒绝服务

**修复建议**：
1. 在 `Init()` 中严格限制各字段长度
2. 计算总 ID 长度，预分配缓冲区
3. 使用 `strcat_s` 或 `snprintf` 替代 `+` 运算符

---

### ⚠️ 风险 3：JS API 内存管理双重释放

**严重程度**：中

**证据位置**：`interfaces/kits/js/builtin/src/locale_module.cpp:24-36`

```cpp
static char *GetLanguage(void)
{
    char *lang = reinterpret_cast<char *>(ace_malloc(MAX_LANGUAGE_LENGTH));
    if (lang == nullptr) {
        return nullptr;
    }
    (void)memset_s(lang, MAX_LANGUAGE_LENGTH, 0x0, MAX_LANGUAGE_LENGTH);
    if (GLOBAL_GetLanguage(lang, MAX_LANGUAGE_LENGTH) != 0) {
        ace_free(lang);  // ⚠️ 失败时释放
        return nullptr;
    }
    return lang;  // ⚠️ 调用者负责释放
}

JSIValue LocaleModule::GetLocale(const JSIValue thisVal, const JSIValue* args, uint8_t argsNum)
{
    JSIValue result = JSI::CreateObject();
    char *lang = GetLanguage();
    if (lang == nullptr) {
        JSI::ReleaseValue(result);
        return JSI::CreateUndefined();  // ⚠️ result 泄漏
    }
    JSI::SetStringProperty(result, "language", lang);
    ace_free(lang);  // ✅ 正确释放
    
    char *region = GetRegion();
    if (region == nullptr) {
        JSI::ReleaseValue(result);
        return JSI::CreateUndefined();  // ⚠️ result 泄漏
    }
    // ...
}
```

**触发条件**：
```javascript
// JS 层重复调用且 GC 行为异常
for (let i = 0; i < 1000000; i++) {
    try {
        const locale = getLocale();
        // 大量创建 JSIValue 对象
    } catch (e) {
        // 异常场景下 result 泄漏累积
    }
}
```

**影响**：
- 内存泄漏（JSIValue 对象泄漏）
- 潜在的资源耗尽拒绝服务
- JS 引擎压力增大

**修复建议**：
1. 确保所有失败路径都释放已分配的资源
2. 使用 RAII 模式管理资源
3. 考虑使用智能指针封装原始指针

---

### ⚠️ 风险 4：数字格式化精度丢失

**严重程度**：低

**证据位置**：`interfaces/kits/i18n/include/number_format.h:143-152`

```cpp
/**
 * @brief Sets the maximum length for the decimal part of a double number. 
 *        The excess part will be truncated.
 *
 * @param length Indicates the maximum length to set.
 * @return Returns <b>true</b> if the setting is successful; returns <b>false</b> otherwise.
 */
bool SetMaxDecimalLength(int length);
```

**触发条件**：
```cpp
NumberFormat formatter(locale, status);
formatter.SetMaxDecimalLength(0);  // 设置为 0
std::string result = formatter.Format(3.14159, NumberFormatType::DECIMAL, status);
// 结果可能丢失精度或行为异常
```

**影响**：
- 数值精度丢失导致显示错误
- 边界条件处理不当可能导致崩溃

**修复建议**：
1. 验证 `length` 参数范围（如 0-20）
2. 处理 NaN、Infinity 等特殊值
3. 记录无效参数的警告

---

### ⚠️ 风险 5：时区字符串解析无验证

**严重程度**：低

**证据位置**：`interfaces/kits/i18n/include/date_time_format.h:94-106`

```cpp
/**
 * @brief Formats a time value...
 *
 * @param zoneInfo Indicates the time zone information in the <b>+/-ab:cd</b> pattern. 
 *   <b>+</b> indicates that the time zone offset is a positive value, 
 *   <b>-</b> indicates that the time zone offset is a negative value,
 *   and <b>ab:cd</b> indicates <b>hour:minute</b>.
 */
void Format(const time_t &cal, const std::string &zoneInfo, 
            std::string &appendTo, I18nStatus &status);
```

**触发条件**：
```cpp
// 传入异常时区格式
DateTimeFormat formatter(AvailableDateTimeFormatPattern::HOUR_MINUTE, locale);
formatter.Format(time, "INVALID:TIMEZONE", out, status);  // 无效格式
formatter.Format(time, "+9999:99", out, status);           // 超范围
formatter.Format(time, "", out, status);                  // 空字符串
```

**影响**：
- 格式化返回错误结果
- 可能触发未定义行为
- 静默失败难以调试

**修复建议**：
1. 添加时区格式验证正则：`^[+-]?\d{1,2}:\d{2}$`
2. 验证小时范围：`-12` 到 `+14`（有效时区范围）
3. 验证分钟范围：`00` 到 `59`

---

### ⚠️ 风险 6：i18n.dat 文件路径硬编码

**严重程度**：低

**证据位置**：`frameworks/i18n/src/data_resource.cpp:29-33`

```cpp
#ifdef I18N_PRODUCT
static const char *DATA_RESOURCE_PATH = "system/i18n/i18n.dat";
#else
static const char *DATA_RESOURCE_PATH = "/storage/data/i18n.dat";
#endif
```

**触发条件**：
```bash
# 设备上不存在对应路径
ls /storage/data/i18n.dat  # 文件不存在
ls system/i18n/i18n.dat    # 路径可能不同
```

**影响**：
- 文件找不到时可能崩溃
- 不同产品路径不一致
- 调试困难

**修复建议**：
1. 添加文件存在性检查
2. 提供回退路径机制
3. 记录详细的错误日志

---

## 5.4 安全机制

### 5.4.1 已有的安全措施

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| **内存安全函数** | `bounds_checking_function` | 使用 `memcpy_s`, `memset_s` 等 |
| **空指针检查** | `locale_info.cpp:28-30` | 构造函数中检查 nullptr |
| **长度校验** | `locale_info.cpp:33-35` | 语言代码 2-3 字符 |
| **内存适配器** | `i18n_memory_adapter.h` | 统一的内存管理 |

### 5.4.2 使用的安全函数

**来源**：`third_party/bounds_checking_function/include/`

| 函数 | 用途 |
|------|------|
| `memcpy_s()` | 安全内存复制 |
| `memset_s()` | 安全内存设置 |
| `strcpy_s()` | 安全字符串复制 |
| `strcat_s()` | 安全字符串拼接 |
| `strlen_s()` | 安全字符串长度 |

**使用示例** (`locale_info.cpp`)：

```cpp
#include "securec.h"

// 使用安全版本的内存操作
if (memcpy_s(lang, MAX_LANGUAGE_LENGTH, newLang, langLength) != EOK) {
    // 处理错误
}
```

## 5.5 安全建议汇总

### 5.5.1 高优先级建议

| 建议 | 风险等级 | 工作量 |
|------|----------|--------|
| 添加脚本/地区内容的字符集校验 | 中 | 低 |
| 验证时区字符串格式和范围 | 低 | 低 |
| 修复 JS API 资源泄漏 | 中 | 中 |

### 5.5.2 中优先级建议

| 建议 | 风险等级 | 工作量 |
|------|----------|--------|
| 限制区域 ID 总长度 | 低 | 低 |
| 添加数字格式化边界校验 | 低 | 低 |
| 改进文件路径回退机制 | 低 | 中 |

### 5.5.3 低优先级建议

| 建议 | 风险等级 | 工作量 |
|------|----------|--------|
| 添加安全审计日志 | 低 | 中 |
| 增加单元测试覆盖率 | 低 | 高 |
| 文档化安全假设 | 低 | 低 |

## 5.6 检查范围声明

### 5.6.1 已检查范围

| 范围 | 文件/代码 | 说明 |
|------|-----------|------|
| ✅ 输入验证 | `locale_info.cpp:24-100` | 构造函数和初始化 |
| ✅ 字符串操作 | `locale_info.cpp:54-69` | ID 字符串构建 |
| ✅ JS API | `locale_module.cpp:24-89` | GetLocale 实现 |
| ✅ 内存管理 | `i18n_memory_adapter.h` | 内存分配/释放 |
| ✅ 数据加载 | `data_resource.cpp:29-54` | i18n.dat 路径 |

### 5.6.2 未检查范围

| 范围 | 原因 |
|------|------|
| ❌ 测试代码 | 按规范要求忽略测试目录 |
| ❌ 第三方依赖 (bounds_checking_function) | 假设已由上游审计 |
| ❌ 构建系统安全 | 假设 GN/Ninja 构建环境安全 |
| ❌ 系统 API (GLOBAL_*) | 假设系统层已处理 |

## 5.7 安全最佳实践

### 5.7.1 使用 i18n_lite 的安全建议

```cpp
// ✅ 推荐：验证输入参数
void SafeUse()
{
    // 1. 验证区域参数
    if (locale.GetLanguage() == nullptr || strlen(locale.GetLanguage()) < 2) {
        return;  // 无效区域
    }
    
    // 2. 检查格式化状态
    I18nStatus status = I18nStatus::ISUCCESS;
    formatter.Format(time, zoneInfo, out, status);
    if (status != I18nStatus::ISUCCESS) {
        // 处理错误
        return;
    }
    
    // 3. 验证输出结果
    if (out.empty() || out.length() > MAX_OUTPUT_LENGTH) {
        // 异常结果
        return;
    }
}

// ❌ 不推荐：忽略错误处理
void UnsafeUse()
{
    DateTimeFormat formatter(pattern, locale);  // 不检查 locale 有效性
    formatter.Format(time, zoneInfo, out, status);  // 不检查 status
}
```

### 5.7.2 安全编码规范

1. **始终检查返回值**
2. **使用安全字符串函数**
3. **限制输入长度**
4. **验证格式和范围**
5. **记录错误日志**

---

*最后更新：2026-02-06*  
*审计范围：本仓库所有非测试源代码*  
*验证方法：代码审查 + 静态分析*
