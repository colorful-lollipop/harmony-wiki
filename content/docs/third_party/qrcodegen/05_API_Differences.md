# 05_API 差异

> 本文档记录 qrcodegen 库在 OpenHarmony 适配版本与上游原始版本之间的 API 差异。

## 5.1 差异概览

| 差异类型 | 数量 | 影响程度 |
|----------|------|----------|
| 异常处理改为返回值 | 10+ 处 | **高** |
| 方法可见性变化 | 8 处 | **中** |
| 新增方法/字段 | 3 处 | **中** |
| 移除的常量 | 4 处 | **低** |
| 静态方法改为实例方法 | 3 处 | **中** |

## 5.2 异常处理差异

### 原始行为 vs OH 适配行为

| 函数/场景 | 上游版本 | OH 适配版本 |
|-----------|----------|-------------|
| **参数校验失败** | 抛出 `std::domain_error` | 设置 `flag = false`，返回 |
| **数据过长** | 抛出 `data_too_long` | 设置 `flag = false`，返回 |
| **位宽溢出** | 抛出 `std::domain_error` | 直接返回 |
| **内存分配失败** | N/A（上游假设分配成功） | 设置 `flag = false`，返回 |

### 错误码与状态

**OH 适配版本**：

```cpp
class QrCode {
public:
    // OH 适配：新增方法
    bool getFlag() const;  // 查询生成是否成功
private:
    bool flag_;  // 标记生成状态
};
```

**使用方式对比**：

```cpp
// 上游版本：使用异常
try {
    QrCode qr = QrCode::encodeText(text, Ecc::LOW);
    // 使用 qr
} catch (const data_too_long& e) {
    // 处理数据过长错误
} catch (const std::domain_error& e) {
    // 处理参数错误
}

// OH 适配版本：检查 flag
QrCode qr = QrCode::encodeText(text, Ecc::LOW);
if (!qr.getFlag()) {
    // 生成失败
    return;
}
// 正常使用 qr
```

## 5.3 方法可见性差异

### 公有方法改为私有

| 类 | 方法 | 上游可见性 | OH 可见性 | 原因 |
|----|------|------------|-----------|------|
| `QrCode` | `QrCode(int, Ecc, ...)` | public | private | 隐藏构造函数，强制使用工厂方法 |
| `QrCode` | `encodeSegments(...)` | public | private | 隐藏底层 API |
| `QrSegment` | `makeBytes(...)` | public | private | 强制使用 makeSegments |
| `QrCode` | `MIN_VERSION` | public static constexpr | private static constexpr | 内部常量 |
| `QrCode` | `MAX_VERSION` | public static constexpr | private static constexpr | 内部常量 |

### 影响分析

**对上游代码的影响**：

```cpp
// 上游版本：可以直接调用
QrCode qr(1, Ecc::LOW, data, 0);  // 直接构造
auto segs = QrSegment::makeBytes(data);  // 直接调用

// OH 适配版本：无法调用私有方法
QrCode::encodeText(text, Ecc::LOW);  // 只能使用此方法
```

**适配建议**：

如需在 OH 中使用底层 API，请通过 `encodeText()` 间接调用，或修改适配代码。

## 5.4 静态方法 vs 实例方法

### 发生变化的静态方法

| 方法 | 上游签名 | OH 适配签名 |
|------|----------|-------------|
| `reedSolomonComputeDivisor` | `static std::vector<uint8_t>(int)` | `std::vector<uint8_t>(int)` |
| `reedSolomonComputeRemainder` | `static std::vector<uint8_t>(...)` | `std::vector<uint8_t>(...)` |
| `reedSolomonMultiply` | `static uint8_t(uint8_t, uint8_t)` | `uint8_t(uint8_t, uint8_t)` |

### 原因

这些方法在 OH 适配版本中需要访问 `flag` 成员变量，因此改为实例方法。

## 5.5 新增 API

### getFlag()

**功能**：查询 QR 码生成是否成功

**声明位置**：`qrcodegen.hpp:379-384`

```cpp
#if defined(ACE_ENGINE_QRCODE_ABLE)
public: bool getFlag() const;
#endif
```

**返回值**：
- `true`：生成成功
- `false`：生成失败（参数错误、数据过长等）

**使用示例**：

```cpp
auto qrCode = qrcodegen::QrCode::encodeText(text, ecc);
if (!qrCode.getFlag()) {
    // 生成失败处理
    return;
}
```

### clearFunctionPatterns()

**功能**：清理函数模块数据，释放内存

**声明位置**：`qrcodegen.hpp:545-548`

```cpp
#if defined(ACE_ENGINE_QRCODE_ABLE)
private: void clearFunctionPatterns();
#endif
```

**内部使用**：由构造函数调用，清理不需要的临时数据。

### ERR_VERSION 常量

**功能**：标记生成失败的版本号

**声明位置**：`qrcodegen.hpp:557-559`

```cpp
#if defined(ACE_ENGINE_QRCODE_ABLE)
private: static constexpr int ERR_VERSION = 0;
#endif
```

## 5.6 移除的 API/常量

### PENALTY_* 常量

**移除原因**：自动掩码选择功能被禁用

| 常量 | 原始值 | 用途 |
|------|--------|------|
| `PENALTY_N1` | 3 | 相邻同色模块惩罚 |
| `PENALTY_N2` | 3 | 2x2 块同色惩罚 |
| `PENALTY_N3` | 40 | finder-like 模式惩罚 |
| `PENALTY_N4` | 10 | 深色/浅色平衡惩罚 |

### 相关移除的方法

- `getPenaltyScore()`
- `finderPenaltyCountPatterns()`
- `finderPenaltyTerminateAndCount()`
- `finderPenaltyAddHistory()`

## 5.7 QrSegment::Mode 差异

### 设计差异

**上游设计**：使用静态常量 + 指针

```cpp
class Mode {
private: int modeBits;
private: int numBitsCharCount[3];
};

// 静态常量（全局唯一）
static const Mode NUMERIC(0x1, 10, 12, 14);
static const Mode ALPHANUMERIC(0x2, 9, 11, 13);
// ...

// 使用指针访问
const Mode* mode_;
```

**OH 适配设计**：直接持有对象

```cpp
class Mode {
public: Mode(int mode, int cc0, int cc1, int cc2);
public: Mode() {}  // 默认构造
private: int modeBits;
private: int numBitsCharCount[3];
};

// 直接持有
Mode mode_;
```

### 访问方式差异

```cpp
// 上游版本
const QrSegment::Mode& mode = seg.getMode();
int bits = mode.getModeBits();  // 通过指针访问

// OH 适配版本
const QrSegment::Mode& mode = seg.getMode();
int bits = mode.getModeBits();  // 直接访问
```

## 5.8 错误处理策略对比

### 上游策略：快速失败（Fail-Fast）

```cpp
// 参数校验失败立即抛出异常
QrCode::QrCode(int ver, ...) {
    if (ver < MIN_VERSION || ver > MAX_VERSION)
        throw std::domain_error("Version value out of range");
    // ...
}
```

### OH 适配策略：优雅降级

```cpp
// 参数校验失败设置标志，继续执行
QrCode::QrCode(int ver, ...) : flag(true) {
    if (ver < MIN_VERSION || ver > MAX_VERSION) {
        flag = false;
        return;
    }
    // ... 后续检查继续执行
}
```

### 优缺点分析

| 策略 | 优点 | 缺点 |
|------|------|------|
| 异常 | 错误语义清晰，可携带信息 | 需要异常支持，可能影响性能 |
| flag | 适应无异常环境 | 错误信息有限，需要手动检查 |

## 5.9 兼容性矩阵

| 功能 | 上游 | OH 标准系统 | OH 轻量系统 |
|------|------|-------------|-------------|
| `encodeText()` | ✅ | ✅ | ✅ |
| `encodeBinary()` | ✅ | ❌ | ❌ |
| `makeNumeric()` | ✅ | ❌* | ❌* |
| `makeAlphanumeric()` | ✅ | ❌* | ❌* |
| `makeSegments()` | ✅ | ✅ | ✅ |
| `getFlag()` | ❌ | ✅ | ✅ |
| 自动掩码选择 | ✅ | ❌** | ❌** |

**说明**：
- `*`：OH 适配中这些方法仍然存在（`qrcodegen_static`），但 QRCode 组件使用 `makeSegments()` 间接调用
- `**`：OH 固定使用掩码 5，不进行自动选择

## 5.10 迁移建议

### 从上游迁移到 OH

1. **替换异常处理**：
   ```cpp
   // 之前
   try { qr = QrCode::encodeText(...); }
   catch (const data_too_long&) { /* 错误处理 */ }
   
   // 之后
   qr = QrCode::encodeText(...);
   if (!qr.getFlag()) { /* 错误处理 */ }
   ```

2. **替换底层 API 调用**：
   ```cpp
   // 之前
   auto segs = QrSegment::makeBytes(data);
   auto qr = QrCode::encodeSegments(segs, ...);
   
   // 之后（使用简化路径）
   auto qr = QrCode::encodeText(text, ...);
   ```

3. **移除版本号检查**：
   ```cpp
   // 之前
   if (qr.getVersion() > QrCode::MAX_VERSION) { ... }
   
   // 之后（MAX_VERSION 已私有化）
   if (qr.getVersion() > 40) { ... }  // 硬编码
   ```
