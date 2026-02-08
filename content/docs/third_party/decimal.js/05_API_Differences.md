# 05 - API 差异

本文档说明 decimal.js 在 OpenHarmony 中与上游版本的 API 差异。

---

## 5.1 差异概览

### 总体评估

| 维度 | 状态 | 说明 |
|------|------|------|
| **新增 API** | 无 | OH 未添加新 API |
| **删除 API** | 无 | OH 未删除上游 API |
| **行为变更** | 有 | 错误处理机制变更 |
| **参数变更** | 无 | API 参数保持一致 |

### 核心结论

**decimal.js 在 OH 中的 API 与上游基本一致**，唯一的差异是**错误处理机制**。

---

## 5.2 错误处理差异

### 上游版本（v10.5.0）

**错误类型**: 使用标准 JavaScript `Error` 对象

```javascript
// 上游代码示例（标准 Error）
if (sd < 1 || sd > MAX_DIGITS) {
  throw new Error('[DecimalError] Invalid argument: ' + sd);
}
```

**错误信息**:
- 仅包含错误消息字符串
- 无错误码

### OpenHarmony 版本

**错误类型**: 使用自定义 `BusinessError` 类

```javascript
// OH 特有：定义 BusinessError
class BusinessError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'BusinessError';
    this.code = code;
  }
}

// OH 错误码定义
const RANGE_ERROR_CODE = 10200001;
const TYPE_ERROR_CODE = 401;
const PRECISION_LIMIT_EXCEEDED_ERROR_CODE = 10200060;
const CRYPTO_UNAVAILABLE_ERROR_CODE = 10200061;

// OH 代码示例（BusinessError）
if (sd < 1 || sd > MAX_DIGITS) {
  throw new BusinessError(
    '[DecimalError] Invalid argument: ' + sd,
    RANGE_ERROR_CODE
  );
}
```

**错误信息**:
- 包含错误消息字符串
- 包含错误码（code 属性）

### 差异对比表

| 特性 | 上游 | OH | 影响 |
|------|------|-----|------|
| 错误类 | `Error` | `BusinessError` | 可访问 `error.code` |
| 错误码 | 无 | 有 | ArkTS 框架可识别错误类型 |
| 错误名 | `Error` | `BusinessError` | `error.name` 不同 |

### 错误码映射

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 10200001 | `RANGE_ERROR_CODE` | 参数超出有效范围 |
| 401 | `TYPE_ERROR_CODE` | 参数类型错误 |
| 10200060 | `PRECISION_LIMIT_EXCEEDED_ERROR_CODE` | 精度限制超出 |
| 10200061 | `CRYPTO_UNAVAILABLE_ERROR_CODE` | 加密功能不可用 |

### 应用层错误处理

在 ArkTS 应用中捕获错误：

```typescript
import { Decimal } from '@kit.ArkTS';

try {
  // 设置无效的精度
  Decimal.set({ precision: 9999999999 });
} catch (error) {
  // OH 版本：可以访问 error.code
  console.log(`错误名: ${error.name}`);      // 'BusinessError'
  console.log(`错误消息: ${error.message}`); // 错误描述
  console.log(`错误码: ${error.code}`);      // 10200001
  
  // 根据错误码处理
  if (error.code === 10200001) {
    console.log('范围错误：参数超出有效范围');
  }
}
```

---

## 5.3 API 一致性声明

### 完全一致的 API

以下 API 在 OH 中与上游完全一致：

#### 构造函数
- `new Decimal(value)`

#### 实例方法
- `abs()`, `acos()`, `acosh()`, `add()`, `asin()`, `asinh()`
- `atan()`, `atan2()`, `atanh()`, `cbrt()`, `ceil()`
- `clamp()`, `cos()`, `cosh()`, `div()`, `divToInt()`
- `dp()`, `eq()`, `floor()`, `gt()`, `gte()`
- `hypot()`, `isFinite()`, `isInt()`, `isNaN()`, `isNeg()`
- `isPos()`, `isZero()`, `lt()`, `lte()`, `ln()`
- `log()`, `log10()`, `log2()`, `minus()`, `mod()`
- `mul()`, `neg()`, `plus()`, `pow()`, `round()`
- `sd()`, `sin()`, `sinh()`, `sqrt()`, `tan()`
- `tanh()`, `toBinary()`, `toDP()`, `toExponential()`
- `toFixed()`, `toFraction()`, `toHex()`, `toJSON()`
- `toNearest()`, `toNumber()`, `toOctal()`, `toDP()`
- `toPrecision()`, `toSignificantDigits()`, `toString()`
- `trunc()`, `valueOf()`

#### 静态方法
- `Decimal.abs()`, `Decimal.acos()`, `Decimal.acosh()`
- `Decimal.add()`, `Decimal.asin()`, `Decimal.asinh()`
- `Decimal.atan()`, `Decimal.atan2()`, `Decimal.atanh()`
- `Decimal.cbrt()`, `Decimal.ceil()`, `Decimal.clamp()`
- `Decimal.cos()`, `Decimal.cosh()`, `Decimal.div()`
- `Decimal.exp()`, `Decimal.floor()`, `Decimal.hypot()`
- `Decimal.ln()`, `Decimal.log()`, `Decimal.log10()`, `Decimal.log2()`
- `Decimal.max()`, `Decimal.min()`, `Decimal.mod()`
- `Decimal.mul()`, `Decimal.neg()`, `Decimal.pow()`
- `Decimal.random()`, `Decimal.round()`, `Decimal.sign()`
- `Decimal.sin()`, `Decimal.sinh()`, `Decimal.sqrt()`
- `Decimal.sum()`, `Decimal.tan()`, `Decimal.tanh()`
- `Decimal.trunc()`

#### 配置方法
- `Decimal.clone()`
- `Decimal.config()` / `Decimal.set()`

#### 属性
- `Decimal.precision`
- `Decimal.rounding`
- `Decimal.modulo`
- `Decimal.toExpNeg`
- `Decimal.toExpPos`
- `Decimal.minE`
- `Decimal.maxE`
- `Decimal.crypto`
- `Decimal.ROUND_UP`, `Decimal.ROUND_DOWN`, ...

### 常量定义

所有舍入模式常量完全一致：

| 常量 | 值 | 说明 |
|------|-----|------|
| `Decimal.ROUND_UP` | 0 | 远离零舍入 |
| `Decimal.ROUND_DOWN` | 1 | 向零舍入 |
| `Decimal.ROUND_CEIL` | 2 | 向 +Infinity 舍入 |
| `Decimal.ROUND_FLOOR` | 3 | 向 -Infinity 舍入 |
| `Decimal.ROUND_HALF_UP` | 4 | 四舍五入 |
| `Decimal.ROUND_HALF_DOWN` | 5 | 五舍六入 |
| `Decimal.ROUND_HALF_EVEN` | 6 | 银行家舍入法 |
| `Decimal.ROUND_HALF_CEIL` | 7 | 向 +Infinity 的半舍入 |
| `Decimal.ROUND_HALF_FLOOR` | 8 | 向 -Infinity 的半舍入 |
| `Decimal.EUCLID` | 9 | 欧几里得除法 |

---

## 5.4 迁移指南

### 从 Node.js 迁移到 ArkTS

如果你已有使用 decimal.js 的 Node.js 代码：

#### 步骤 1: 修改导入语句

```typescript
// Node.js 版本
const Decimal = require('decimal.js');
// 或
import Decimal from 'decimal.js';

// ArkTS 版本
import { Decimal } from '@kit.ArkTS';
```

#### 步骤 2: 调整错误处理（可选）

```typescript
// Node.js 版本
try {
  // ...
} catch (error) {
  console.log(error.message);
}

// ArkTS 版本（可以利用错误码）
try {
  // ...
} catch (error) {
  console.log(error.message);
  if (error.code) {
    console.log(`错误码: ${error.code}`);
  }
}
```

#### 步骤 3: 验证功能

大部分代码无需修改即可运行。重点验证：
- 精度设置是否正确
- 特殊值处理（NaN, Infinity）
- 三角函数结果

---

## 5.5 功能完整性声明

### 与上游功能对比

| 功能类别 | 上游 | OH | 状态 |
|----------|------|-----|------|
| 基础运算 | ✅ | ✅ | 完整 |
| 三角函数 | ✅ | ✅ | 完整 |
| 对数/指数 | ✅ | ✅ | 完整 |
| 多进制转换 | ✅ | ✅ | 完整 |
| 链式调用 | ✅ | ✅ | 完整 |
| 精度配置 | ✅ | ✅ | 完整 |
| 克隆/多实例 | ✅ | ✅ | 完整 |
| 错误处理 | ✅ | ✅+ | 增强（添加错误码） |

**结论**: OH 版本功能完整，仅增强错误处理。

---

## 5.6 总结

### API 差异总结

| 差异类型 | 数量 | 详情 |
|----------|------|------|
| 新增 API | 0 | 无 |
| 删除 API | 0 | 无 |
| 行为变更 | 1 | 错误处理（Error → BusinessError） |
| **总差异** | **1** | **极小** |

### 兼容性评估

- **向上兼容**: 100%（OH 代码可直接在 Node.js 运行，错误码会被忽略）
- **向下兼容**: 99%（Node.js 代码在 OH 运行，仅需修改导入语句）

### 使用建议

1. **直接使用**: 绝大多数情况下，直接使用上游文档的 API
2. **错误处理**: 如需精细错误处理，可利用 `error.code`
3. **参考文档**: 上游文档 https://mikemcl.github.io/decimal.js/ 完全适用

---

*下一章: [06_Security.md](./06_Security.md) - 安全风险分析*
