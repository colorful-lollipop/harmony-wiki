# libphonenumber - 原始库简介

## 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | libphonenumber |
| **版本号** | 8.13.31 |
| **许可证** | Apache License 2.0 |
| **上游地址** | https://github.com/google/libphonenumber.git |
| **维护者** | Google |
| **发布频率** | 每两周一次（大约） |

---

## 功能描述

libphonenumber 是 Google 的通用电话号码处理库，提供 Java、C++ 和 JavaScript 三个版本。其主要功能包括：

### 核心功能

1. **电话号码解析**
   - 将任意格式的电话号码字符串解析为结构化的 `PhoneNumber` 对象
   - 自动识别国家/地区代码
   - 提取国家代码和国内号码

2. **号码验证**
   - `isPossibleNumber()` - 快速检查号码长度
   - `isValidNumber()` - 完整验证（长度和前缀）
   - 支持所有国家/地区的规则

3. **号码格式化**
   - 国际格式：`+86 138 0013 8000`
   - 国家格式：`0138 0013 8000`
   - E.164 格式：`+8613800138000`
   - 指定国家拨号格式

4. **号码类型识别**
   - 固定电话 (Fixed-line)
   - 移动电话 (Mobile)
   - 免费号码 (Toll-free)
   - 高费率号码 (Premium Rate)
   - 共享成本号码 (Shared Cost)
   - VoIP
   - 个人号码 (Personal Numbers)
   - 紧急号码 (Pager, Voicemail)

5. **号码匹配和比较**
   - `isNumberMatch()` - 判断两个号码是否可能相同
   - 提供置信度评分

6. **实时格式化** (`AsYouTypeFormatter`)
   - 用户输入时实时格式化
   - 自动添加空格、括号等格式

7. **文本中查找号码** (`findNumbers`)
   - 从任意文本中提取电话号码
   - 支持复杂文本场景

8. **地理编码** (`PhoneNumberOfflineGeocoder`)
   - 根据电话号码获取地理位置
   - 支持多语言描述

9. **运营商识别** (`PhoneNumberToCarrierMapper`)
   - 识别号码所属运营商
   - 注意：仅识别原始分配运营商，不包含携号转网

10. **时区识别** (`PhoneNumberToTimeZonesMapper`)
    - 根据电话号码获取时区信息

---

## 在 OpenHarmony 中的作用和定位

### 核心定位

libphonenumber 在 OpenHarmony 中作为**电信基础设施的基础组件**，主要服务于：

1. **SMS/MMS 通信**
   - 验证收件人电话号码格式
   - 格式化显示的电话号码
   - 根据号码获取地理位置

2. **系统服务集成**
   - 通过 NDK 向应用层暴露电话号码验证功能
   - 支持系统设置、联系人等模块的电话号码处理

3. **跨应用兼容性**
   - 提供统一的电话号码处理标准
   - 确保不同应用的电话号码格式一致

---

## 版本兼容性

### 上游版本 8.13.31

**重要特性**：
- 支持 200+ 个国家/地区的电话号码规则
- 包含最新的号码段分配信息
- 支持新兴国家的特殊规则

### OpenHarmony 版本 3.1

**OH 特性**：
- 完整兼容上游 API
- 新增运行时元数据更新能力（详见 [02_Patches.md](02_Patches.md)）
- 集成 OHOS 安全机制
- 支持 OHOS 构建系统（GN）

---

## 技术栈

### 编程语言
- **C++11** - 核心实现
- **Protocol Buffers** - 元数据序列化
- **ICU** - 国际化支持（正则表达式、Unicode）

### 依赖库
- **abseil-cpp** - Google C++ 常用库
- **protobuf** - Protocol Buffers 运行时
- **ICU (ICU4C)** - 国际化组件库

---

## 原始架构

### 核心类

| 类名 | 功能 |
|------|------|
| `PhoneNumberUtil` | 主入口类，提供解析、验证、格式化 API |
| `PhoneNumber` | 电话号码数据结构（protobuf 生成） |
| `PhoneNumberOfflineGeocoder` | 地理编码功能 |
| `PhoneNumberToCarrierMapper` | 运营商识别 |
| `AsYouTypeFormatter` | 实时格式化器 |
| `PhoneNumberMatcher` | 号码匹配器 |
| `ShortNumberInfo` | 短号码（紧急号码等）处理 |

### 元数据系统

电话号码规则存储在编译时的二进制数据中：

| 文件 | 内容 |
|------|------|
| `metadata.cc` | 完整电话号码元数据（15,000+ 行） |
| `lite_metadata.cc` | 精简版元数据（14,000+ 行） |
| `short_metadata.cc` | 短号码元数据（4,000+ 行） |
| `alternate_format.cc` | 备用格式元数据 |

---

## 使用场景

### 典型应用场景

1. **短信发送前验证**
   ```cpp
   PhoneNumberUtil util;
   PhoneNumber number;
   if (util.Parse("+8613800138000", "CN", &number)) {
     if (util.IsValidNumber(number)) {
       // 发送短信
     }
   }
   ```

2. **联系人电话号码格式化**
   ```cpp
   string formatted = util.Format(number, PhoneNumberUtil::INTERNATIONAL);
   // 输出: +86 138 0013 8000
   ```

3. **地理位置查询**
   ```cpp
   PhoneNumberOfflineGeocoder geocoder;
   string location = geocoder.GetDescriptionForNumber(number, Locale("zh"));
   // 输出: "北京市" 或 "广东"
   ```

4. **实时输入格式化**
   ```cpp
   AsYouTypeFormatter formatter = util.GetAsYouTypeFormatter("CN");
   cout << formatter.inputDigit('1');  // 输出: 1
   cout << formatter.inputDigit('3');  // 输出: 13
   cout << formatter.inputDigit('8');  // 输出: 138
   ```

---

## 与上游的差异

详见：[02_Patches.md](02_Patches.md) 和 [05_API_Differences.md](05_API_Differences.md)

**主要差异**：
1. 新增运行时元数据更新机制
2. 新增 OHOS 特定初始化代码
3. 集成安全库（`libsec_shared`）

---

## 参考资源

- **上游仓库**: https://github.com/google/libphonenumber
- **上游 Wiki**: https://github.com/google/libphonenumber/wiki
- **Javadoc**: https://javadoc.io/doc/com.googlecode.libphonenumber/libphonenumber/
- **Demo**: https://libphonenumber.appspot.com/ (Java 版本)

---

**最后更新**: 2026-02-08
