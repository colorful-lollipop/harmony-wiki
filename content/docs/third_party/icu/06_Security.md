# 06 - ICU 安全风险分析

## 6.1 概述

ICU 作为 OpenHarmony 的基础国际化库，其安全性直接影响系统安全。本文档分析 ICU 在 OpenHarmony 中的安全风险及应对策略。

---

## 6.2 已知 CVE 分析

### 6.2.1 ICU 74.2 相关 CVE

根据 NVD (National Vulnerability Database) 查询，ICU 74.2 版本相对较新，主要安全修复已包含。

#### 历史 CVE (已在 74.2 中修复)

| CVE ID | 版本 | 类型 | 严重性 | 描述 | 状态 |
|--------|------|------|--------|------|------|
| CVE-2023-XXXX | < 73.x | 缓冲区溢出 | High | 字符串处理边界检查 | 已修复 |
| CVE-2022-XXXX | < 72.x | 整数溢出 | Medium | 数字格式化整数溢出 | 已修复 |

**TODO(需确认)**: 需要查询最新 CVE 数据库确认 74.2 版本的具体 CVE 情况。

### 6.2.2 查询方式

```bash
# 查询 ICU 相关 CVE
# 访问 https://nvd.nist.gov/ 搜索 "ICU"
# 或使用命令行工具
```

---

## 6.3 潜在安全风险

### 6.3.1 输入处理风险

| 功能模块 | 风险点 | 潜在问题 |
|----------|--------|----------|
| 编码转换 (ucnv) | 非法输入序列 | 可能触发断言或异常 |
| 正则表达式 (uregex) | 复杂模式 | 可能导致 ReDoS |
| 日期解析 (udat) | 非法格式 | 可能产生未定义行为 |
| 数字解析 (unum) | 超长数字 | 可能导致溢出 |

#### 编码转换安全建议

```cpp
// 安全做法: 检查错误码
UErrorCode status = U_ZERO_ERROR;
UConverter* conv = ucnv_open("utf-8", &status);

if (U_FAILURE(status)) {
    // 处理错误
    return;
}

// 转换时检查状态
ucnv_toUnicode(conv, ..., &status);
if (U_FAILURE(status)) {
    // 处理转换错误
}

ucnv_close(conv);
```

#### 正则表达式安全建议

```cpp
// 风险: 复杂正则可能导致 ReDoS
// 如: (a+)+b 匹配 aaaaaaaaaaaaaaaaaaaaaaaaaaaaa!

// 缓解措施:
// 1. 限制正则模式复杂度
// 2. 设置匹配超时
// 3. 验证用户输入的正则表达式
```

### 6.3.2 数据文件安全

| 风险点 | 描述 | 影响 |
|--------|------|------|
| 数据篡改 | ICU 数据文件被修改 | 可能导致崩溃或错误结果 |
| 数据注入 | 加载恶意数据文件 | 可能触发解析漏洞 |
| 路径遍历 | 通过配置加载任意路径数据 | 信息泄露 |

#### OpenHarmony 防护措施

1. **数据文件保护**:
   - 数据文件存放在 `/system` 分区 (只读)
   - 需要 root 权限才能修改

2. **路径限制**:
   - 数据路径在编译时确定
   - 运行时通过 `SetHwIcuDirectory()` 设置
   - 不支持任意路径加载

3. **数据校验**:
   - ICU 数据文件有校验和
   - 加载时进行完整性检查

### 6.3.3 内存安全风险

| 模块 | 风险 | 缓解措施 |
|------|------|----------|
| 字符串操作 | 缓冲区溢出 | 使用 ICU 提供的安全 API |
| 格式化 | 格式化字符串漏洞 | 使用 ICU 格式化 API，避免 printf |
| 正则表达式 | 栈溢出 | 限制递归深度 |

#### 安全编码示例

```cpp
// 不安全: 直接使用指针
UChar buffer[100];
uint32_t len = GetInputLength();
// 如果 len > 100，会发生溢出
memcpy(buffer, input, len);

// 安全: 使用 ICU 的边界检查 API
UChar buffer[100];
int32_t len = u_strncpy(buffer, input, 100);
```

---

## 6.4 OH Patch 引入的攻击面

### 6.4.1 农历功能安全分析

**文件**: `icu4c/source/ohos/lunar_calendar.cpp`

#### 潜在风险

| 风险点 | 描述 | 风险等级 |
|--------|------|----------|
| 数组越界 | 年份范围检查 | 低 |
| 整数溢出 | 天数计算 | 低 |

#### 代码审查

```cpp
// 安全检查 1: 年份范围验证
bool LunarCalendar::VerifyDate(int32_t year, int32_t month, int32_t day) {
    if ((year < VALID_START_YEAR) || (year > VALID_END_YEAR)) {
        return false;  // 拒绝非法年份
    }
    // ...
}

// 安全检查 2: 数组访问保护
int32_t LunarCalendar::GetLeapMonthInYear(int32_t year) {
    if ((year < VALID_START_YEAR) || (year > VALID_END_YEAR)) {
        return -1;  // 非法返回 -1
    }
    return lunarDateInfo[year - START_YEAR] & 0xf;  // 安全的数组访问
}
```

**结论**: 农历功能实现中有完善的边界检查，风险较低。

### 6.4.2 初始化功能安全分析

**文件**: `icu4c/source/ohos/init_data.cpp`

#### 潜在风险

| 风险点 | 描述 | 风险等级 |
|--------|------|----------|
| 竞争条件 | 多线程初始化 | 低 (已使用 mutex) |
| 路径注入 | 自定义路径 | 低 (ArkUI-X 专用) |

#### 代码审查

```cpp
// 线程安全保护
void SetHwIcuDirectory() {
    std::lock_guard<std::mutex> lock(dataMutex);  // 防止竞争条件
    if (status != 0) { return; }  // 确保只初始化一次
    u_setDataDirectory(g_hwDirectory);
    status = 1;
}
```

**结论**: 初始化功能有线程安全保护，风险较低。

---

## 6.5 安全升级策略

### 6.5.1 上游版本跟踪

| 跟踪项 | 频率 | 负责人 |
|--------|------|--------|
| CVE 公告 | 每周 | 安全团队 |
| 版本发布 | 每月 | ICU 维护者 |
| 安全补丁 | 实时 | ICU 维护者 |

### 6.5.2 升级检查清单

- [ ] 检查新版本的 CVE 修复列表
- [ ] 评估对 OH 特有代码的影响
- [ ] 运行安全测试套件
- [ ] 验证农历功能正确性
- [ ] 检查 NDK 符号兼容性
- [ ] 更新安全文档

### 6.5.3 应急响应

**发现高危 CVE 时的处理流程**:

1. **评估影响** (1 天内)
   - 确认 CVE 是否影响当前版本
   - 评估对 OH 系统的影响程度

2. **制定方案** (2 天内)
   - 确定升级或打补丁方案
   - 评估回退风险

3. **实施修复** (根据严重程度)
   - 高危: 3 天内
   - 中危: 1 周内
   - 低危: 下次常规升级

4. **验证发布**
   - 安全测试通过
   - 功能测试通过
   - 发布安全公告

---

## 6.6 安全测试建议

### 6.6.1 模糊测试 (Fuzzing)

| 目标模块 | 测试输入 | 工具 |
|----------|----------|------|
| ucnv_convert | 随机字节序列 | libFuzzer |
| udat_parse | 随机字符串 | AFL |
| uregex_matches | 随机正则模式 | libFuzzer |

### 6.6.2 静态分析

| 工具 | 用途 | 检查项 |
|------|------|--------|
| Clang Static Analyzer | 静态分析 | 内存泄漏、空指针 |
| Coverity | 深度分析 | 复杂缺陷模式 |
| CodeQL | 安全查询 | 已知漏洞模式 |

### 6.6.3 安全测试用例

```cpp
// 测试用例 1: 非法编码序列
TEST_F(ICUSecurityTest, InvalidEncoding) {
    UErrorCode status = U_ZERO_ERROR;
    UConverter* conv = ucnv_open("utf-8", &status);
    
    // 输入非法 UTF-8 序列
    const char* invalid = "\xff\xfe";
    UChar output[10];
    int32_t len = ucnv_toUnicode(conv, output, 10, &invalid, 2, NULL, &status);
    
    // 应返回错误，不崩溃
    EXPECT_TRUE(U_FAILURE(status));
    ucnv_close(conv);
}

// 测试用例 2: 超长输入
TEST_F(ICUSecurityTest, VeryLongInput) {
    UErrorCode status = U_ZERO_ERROR;
    
    // 生成超长字符串
    std::string longString(1000000, 'a');
    
    UChar* str = new UChar[1000000];
    // 不应溢出或崩溃
    int32_t len = u_strFromUTF8(str, 1000000, NULL, 
                                 longString.c_str(), -1, &status);
    
    delete[] str;
}
```

---

## 6.7 安全最佳实践

### 6.7.1 应用开发者指南

1. **输入验证**:
   - 始终验证用户输入
   - 限制输入长度
   - 检查字符编码

2. **错误处理**:
   - 检查所有 ICU API 的 `UErrorCode`
   - 不要忽略错误返回值
   - 优雅处理异常情况

3. **资源管理**:
   - 使用 RAII 模式管理 ICU 对象
   - 确保及时释放资源

```cpp
// 好的做法: 使用智能指针管理 ICU 对象
class ICUConverter {
    UConverter* conv;
public:
    ICUConverter(const char* name) {
        UErrorCode status = U_ZERO_ERROR;
        conv = ucnv_open(name, &status);
        if (U_FAILURE(status)) throw std::runtime_error("Failed");
    }
    ~ICUConverter() { ucnv_close(conv); }
    // 禁用拷贝
    ICUConverter(const ICUConverter&) = delete;
};
```

### 6.7.2 系统集成安全

1. **权限控制**:
   - ICU 数据文件只允许系统写入
   - NDK API 遵循最小权限原则

2. **沙箱隔离**:
   - 应用无法直接修改 ICU 数据
   - 通过框架层 API 访问

3. **更新机制**:
   - 安全更新通过系统 OTA
   - 紧急修复可单独推送

---

## 6.8 总结

| 安全维度 | 风险等级 | 缓解措施 |
|----------|----------|----------|
| 已知 CVE | 低 | 使用最新版本 (74.2) |
| 输入处理 | 中 | 严格输入验证 |
| 数据文件 | 低 | 系统分区保护 |
| OH 特有代码 | 低 | 代码审查 + 测试 |
| 升级安全 | 中 | 建立升级流程 |

**TODO(需确认)**:
- [ ] 查询最新的 CVE 数据库
- [ ] 补充具体的模糊测试用例
- [ ] 确认安全响应流程
