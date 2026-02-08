# 02_适配分析

> **核心文档**：本文档详细记录 qrcodegen 库在 OpenHarmony 中的所有适配点。

## 概述

本库**没有独立的 `.patch` 文件**，所有 OpenHarmony 适配都通过源代码中的 **`ACE_ENGINE_QRCODE_ABLE`** 条件编译宏实现。适配代码与上游代码共存于 `cpp/qrcodegen.hpp` 和 `cpp/qrcodegen.cpp` 文件中。

## 适配机制

### 条件编译使用统计

| 文件 | `#if defined(ACE_ENGINE_QRCODE_ABLE)` | `#if !defined(ACE_ENGINE_QRCODE_ABLE)` |
|------|----------------------------------------|------------------------------------------|
| `qrcodegen.hpp` | 22 处 | 14 处 |
| `qrcodegen.cpp` | 56 处 | 30 处 |
| **合计** | **78 处** | **44 处** |

## 适配分类详解

### 适配类别一：异常处理改为返回值处理

**适配目的**：避免 C++ 异常抛出，适应嵌入式/轻量级环境的异常处理限制。

#### 修改位置：`qrcodegen.cpp` 多处

**原始问题**：上游代码广泛使用 `std::domain_error`、`std::invalid_argument`、`std::length_error` 等异常。

**修改内容**：使用 `flag` 成员变量记录错误状态，通过 `getFlag()` 方法查询。

```cpp
// 原始代码（抛出异常）
void QrCode::applyMask(int msk) {
    if (msk < 0 || msk > 7)
        throw std::domain_error("Mask value out of range");
    // ...
}

// OH 适配（设置标志位）
void QrCode::applyMask(int msk) {
#if defined(ACE_ENGINE_QRCODE_ABLE)
    if (msk < -1 || msk > 7)
        return;  // 直接返回，由 flag 记录错误
#endif
    // ...
}
```

**OH 价值**：使库可在禁用异常处理的编译环境下正常工作。

**回归风险**：中。升级上游时需检查新版本是否引入新的异常使用。

---

### 适配类别二：构造函数适配

**适配目的**：适应不同的错误处理策略。

#### QrCode 构造函数

**修改位置**：`qrcodegen.cpp:400-483`

**原始行为**：参数校验失败时抛出 `std::domain_error` 异常。

**OH 适配**：
- 新增 `flag` 成员变量
- 校验失败设置 `flag = false` 并直接返回
- 后续操作前检查 `flag` 状态

```cpp
// OH 适配代码
QrCode::QrCode(int ver, Ecc ecl, const vector<uint8_t> &dataCodewords, int msk) :
#if defined(ACE_ENGINE_QRCODE_ABLE)
        version(ver), errorCorrectionLevel(ecl), flag(true) {
    if (ver < MIN_VERSION || ver > MAX_VERSION) {
        flag = false;
        return;
    }
    if (msk < -1 || msk > 7) {
        flag = false;
        return;
    }
#else
        version(ver), errorCorrectionLevel(ecl) {
    if (ver < MIN_VERSION || ver > MAX_VERSION)
        throw std::domain_error("Version value out of range");
    if (msk < -1 || msk > 7)
        throw std::domain_error("Mask value out of range");
#endif
    // ...
}
```

---

### 适配类别三：API 可见性调整

**适配目的**：简化 AceEngine 的使用，隐藏不推荐的底层 API。

#### 修改概览

| 方法/常量 | 原可见性 | OH 可见性 | 原因 |
|-----------|----------|-----------|------|
| `QrSegment::makeBytes()` | public | private | 强制使用 `makeSegments()` 自动选择编码模式 |
| `QrCode::encodeSegments()` | public | private | 隐藏底层 API，使用 `encodeText()` |
| `QrCode::QrCode()` | public | private | 隐藏构造函数，使用工厂方法 |
| `QrCode::MIN_VERSION` | public | private | 内部常量 |
| `QrCode::MAX_VERSION` | public | private | 内部常量 |

**修改位置**：`qrcodegen.hpp`

```cpp
#if defined(ACE_ENGINE_QRCODE_ABLE)
// 私有方法
private: static QrSegment makeBytes(const std::vector<std::uint8_t> &data);
private: static QrCode encodeSegments(const std::vector<QrSegment> &segs, Ecc ecl,
        int minVersion=1, int maxVersion=40, int mask=-1, bool boostEcl=true);
#else
// 公有方法
public: static QrSegment makeBytes(const std::vector<std::uint8_t> &data);
public: static QrCode encodeSegments(const std::vector<QrSegment> &segs, Ecc ecl, ...);
#endif
```

**OH 价值**：简化 QRCode 组件的使用，防止误用底层 API。

**回归风险**：低。可见性变化不影响功能，仅影响 API 封装。

---

### 适配类别四：Reed-Solomon 纠错方法调整

**适配目的**：统一错误处理方式。

#### 修改内容

以下静态方法改为实例方法：

| 方法 | 原签名 | OH 签名 |
|------|--------|---------|
| `reedSolomonComputeDivisor` | `static std::vector<uint8_t>(int)` | `std::vector<uint8_t>(int)` |
| `reedSolomonComputeRemainder` | `static std::vector<uint8_t>(...)` | `std::vector<uint8_t>(...)` |
| `reedSolomonMultiply` | `static uint8_t(uint8_t, uint8_t)` | `uint8_t(uint8_t, uint8_t)` |

**修改位置**：`qrcodegen.hpp:456-529`

```cpp
#if defined(ACE_ENGINE_QRCODE_ABLE)
private: std::vector<std::uint8_t> reedSolomonComputeDivisor(int degree);
#else
private: static std::vector<std::uint8_t> reedSolomonComputeDivisor(int degree) const;
#endif
```

---

### 适配类别五：自动掩码选择禁用

**适配目的**：简化生成流程，提高确定性。

#### 修改内容

**原始行为**：当 `mask = -1` 时，自动计算所有 8 种掩码的惩罚分数，选择最优掩码。

**OH 适配**：固定使用掩码 5。

```cpp
// qrcodegen.cpp:284-291
QrCode QrCode::encodeText(const char *text, Ecc ecl) {
    vector<QrSegment> segs = QrSegment::makeSegments(text);
#if defined(ACE_ENGINE_QRCODE_ABLE)
    return encodeSegments(segs, ecl, MIN_VERSION, MAX_VERSION, 5);  // 固定掩码 5
#else
    return encodeSegments(segs, ecl);  // 自动选择最优掩码
#endif
}
```

**影响**：
- 移除了 `getPenaltyScore()` 相关代码
- 移除了 `finderPenalty*()` 相关辅助方法
- 移除了 `PENALTY_N1` 等评分常量

**OH 价值**：减少计算量，提高生成速度。

**回归风险**：中。升级上游时需确认上游的自动掩码算法是否有改进可以借鉴。

---

### 适配类别六：QrSegment::Mode 类调整

**适配目的**：适应不同的内存布局策略。

#### 修改内容

**原始行为**：`Mode` 使用静态常量对象，指针访问。

**OH 适配**：`Mode` 作为普通对象，直接持有数据。

```cpp
// 原始设计（指针访问）
class Mode final {
private: int modeBits;
private: int numBitsCharCount[3];
private: const Mode *mode;  // 指针指向静态常量
};

// OH 适配（直接持有）
class Mode final {
private: int modeBits;
private: int numBitsCharCount[3];
private: Mode mode;  // 直接持有 Mode 对象
};
```

**修改位置**：`qrcodegen.hpp:48-100`, `qrcodegen.cpp:44-68`

```cpp
#if defined(ACE_ENGINE_QRCODE_ABLE)
public: Mode(int mode, int cc0, int cc1, int cc2);
public: Mode() {}  // 默认构造函数
#else
private: Mode(int mode, int cc0, int cc1, int cc2);
#endif
```

---

### 适配类别七：新增成员变量

**适配目的**：支持新的错误处理机制。

#### flag 成员变量

```cpp
// qrcodegen.hpp:355-359
#if defined(ACE_ENGINE_QRCODE_ABLE)
private: bool flag;
#endif

// qrcodegen.cpp:403
QrCode::QrCode(...) :
#if defined(ACE_ENGINE_QRCODE_ABLE)
        version(ver), errorCorrectionLevel(ecl), flag(true) {
#else
        version(ver), errorCorrectionLevel(ecl) {
#endif
```

#### ERR_VERSION 常量

```cpp
// qrcodegen.hpp:557-559
#if defined(ACE_ENGINE_QRCODE_ABLE)
private: static constexpr int ERR_VERSION = 0;
#endif
```

---

### 适配类别八：辅助函数调整

#### clearFunctionPatterns() 函数

**新增目的**：清理函数模块数据，释放内存。

```cpp
// qrcodegen.cpp:1009-1015
#if defined(ACE_ENGINE_QRCODE_ABLE)
void QrCode::clearFunctionPatterns() {
    isFunction.clear();
    isFunction.shrink_to_fit();
}
#endif
```

#### getFlag() 方法

**新增目的**：查询 QR 码生成是否成功。

```cpp
// qrcodegen.cpp:485-489
#if defined(ACE_ENGINE_QRCODE_ABLE)
bool QrCode::getFlag() const {
    return flag;
}
#endif
```

---

## 适配清单表

| 序号 | 适配类别 | 修改文件 | 修改位置 | 修改目的 | 回归风险 |
|------|----------|----------|----------|----------|----------|
| 1 | 异常处理改返回值 | cpp/*.cpp | 78 处 | 适应无异常环境 | 中 |
| 2 | 构造函数适配 | cpp/qrcodegen.cpp:400-483 | 1 处 | 错误处理策略 | 中 |
| 3 | API 可见性调整 | cpp/qrcodegen.hpp | 10+ 处 | 简化使用 | 低 |
| 4 | RS 方法改为实例方法 | cpp/qrcodegen.hpp/cpp | 3 处 | 统一错误处理 | 低 |
| 5 | 禁用自动掩码选择 | cpp/qrcodegen.cpp:287 | 1 处 | 简化流程 | 中 |
| 6 | Mode 类调整 | cpp/qrcodegen.hpp/cpp | 5 处 | 内存布局优化 | 中 |
| 7 | 新增 flag 变量 | cpp/qrcodegen.hpp/cpp | 2 处 | 错误状态跟踪 | 低 |
| 8 | 辅助函数调整 | cpp/qrcodegen.cpp | 2 处 | 内存管理 | 低 |

## 升级上游版本建议

### 检查清单

- [ ] `ACE_ENGINE_QRCODE_ABLE` 条件编译块是否完整迁移
- [ ] `flag` 变量的设置位置是否需要更新
- [ ] 新的异常使用是否需要适配
- [ ] API 签名变化是否需要调整可见性
- [ ] 上游是否引入新的优化（如新的掩码算法）

### 适配流程

1. 备份当前 `cpp/` 目录
2. 对比上游新版本与当前版本的差异
3. 逐个迁移 `ACE_ENGINE_QRCODE_ABLE` 条件块
4. 测试 QRCode 组件功能正常
5. 运行单元测试

## 已知限制

1. **不支持 Kanji 模式**：OH 适配版本移除了 `makeNumeric()` 和 `makeAlphanumeric()` 的直接调用入口
2. **固定掩码**：使用掩码 5，而非自动选择的最佳掩码
3. **无异常信息**：生成失败时无法获取具体错误原因，仅知成功/失败
