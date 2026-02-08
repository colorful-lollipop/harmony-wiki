# 04 - OpenHarmony 中的依赖关系与使用

本文档详细说明 decimal.js 在 OpenHarmony 中的依赖关系、使用场景和具体示例。

---

## 4.1 依赖关系概览

### 依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                     decimal.js 依赖关系                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    OpenHarmony 系统                       │  │
│  │                                                           │  │
│  │   ┌─────────────┐                                        │  │
│  │   │ ArkTS 应用   │                                        │  │
│  │   └──────┬──────┘                                        │  │
│  │          │ import { Decimal } from '@kit.ArkTS'          │  │
│  │          ▼                                                │  │
│  │   ┌─────────────┐                                        │  │
│  │   │ @kit.ArkTS  │                                        │  │
│  │   └──────┬──────┘                                        │  │
│  │          │                                                │  │
│  │          ▼                                                │  │
│  │   ┌──────────────────┐                                   │  │
│  │   │ libdecimal.z.so  │  ◄─── decimal.js 输出              │  │
│  │   │ arkts.math.Decimal│                                   │  │
│  │   └────────┬─────────┘                                   │  │
│  │            │ deps: napi:ace_napi                          │  │
│  │            ▼                                              │  │
│  │   ┌──────────────────┐                                   │  │
│  │   │ NAPI 框架         │                                   │  │
│  │   └──────────────────┘                                   │  │
│  │                                                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 直接依赖者

**搜索结果**: 未发现其他系统模块直接依赖 `//third_party/decimal.js:decimal`

```bash
# 在 oh/ 目录下搜索 BUILD.gn 中包含 third_party/decimal.js 的文件
# 结果: 无匹配
```

**分析**: decimal.js 主要面向**应用层**提供 API，而非系统模块间的依赖。

### 间接依赖（产品配置）

在 `productdefine/common/inherit/` 中发现以下配置：

| 产品形态 | 配置文件 | 配置内容 |
|----------|----------|----------|
| **标准版 (rich)** | `rich.json` | `"component": "decimal.js"` |
| **穿戴版 (wearable)** | `wearable.json` | `"component": "decimal.js"` |
| **电视版 (tv)** | `tv.json` | `"component": "decimal.js"` |

**结论**: decimal.js 是**基础系统组件**，在所有主要产品形态中默认包含。

---

## 4.2 依赖链分析

### 向上依赖（decimal.js 依赖谁）

```
decimal.js
    │
    ├── deps: napi:ace_napi (编译/运行时依赖)
    │       └── NAPI 框架，用于模块注册和字节码导出
    │
    └── build: es2abc (编译时依赖)
            └── ArkCompiler 前端，将 JS 转为 ABC 字节码
```

### 向下依赖（谁依赖 decimal.js）

```
decimal.js
    │
    ├── @kit.ArkTS (系统框架)
    │       └── 导出 Decimal API 给应用
    │
    └── ArkTS 应用 (最终用户)
            └── import { Decimal } from '@kit.ArkTS'
```

### 依赖类型说明

| 依赖 | 类型 | 说明 |
|------|------|------|
| `napi:ace_napi` | 编译+运行时 | NAPI 框架，必需 |
| `es2abc` | 编译时 | ArkTS 编译器，必需 |
| `@kit.ArkTS` | 运行时 | 系统框架，自动提供 |

---

## 4.3 使用场景

### 场景 1: 金融计算

**需求**: 精确计算货币金额，避免浮点误差

```typescript
import { Decimal } from '@kit.ArkTS';

// 商品价格计算
let price = new Decimal('19.99');
let quantity = new Decimal(3);
let taxRate = new Decimal('0.08');

// 小计
let subtotal = price.times(quantity);  // 59.97

// 税额（精确到分）
let tax = subtotal.times(taxRate);     // 4.7976
let taxRounded = tax.toDecimalPlaces(2, Decimal.ROUND_HALF_UP);  // 4.80

// 总计
let total = subtotal.plus(taxRounded); // 64.77

console.log(`总价: ${total.toString()}`);  // '64.77'
```

**对比普通 JavaScript**:
```typescript
// 错误示例
let wrongTotal = 19.99 * 3 * 1.08;
console.log(wrongTotal);  // 64.7676，可能有精度问题
```

### 场景 2: 科学计算

**需求**: 高精度三角函数和数学运算

```typescript
import { Decimal } from '@kit.ArkTS';

// 设置高精度
Decimal.set({ precision: 50 });

// 计算圆周率相关
let radius = new Decimal(5);
let area = Decimal.pi.times(radius.pow(2));
console.log(area.toString());  // 精确到 50 位的圆周率计算

// 三角函数
let angle = new Decimal(30);  // 度
let radians = angle.times(Decimal.pi).div(180);
let sinValue = Decimal.sin(radians);
console.log(sinValue.toString());  // 0.5（精确值）
```

### 场景 3: 统计分析

**需求**: 大数据集统计，避免累积误差

```typescript
import { Decimal } from '@kit.ArkTS';

// 数据求和（避免累积误差）
let data = [
  '0.1', '0.2', '0.3', '0.4', '0.5',
  '0.6', '0.7', '0.8', '0.9', '1.0'
];

// 使用 Decimal.sum（内部使用 Kahan 求和算法）
let sum = Decimal.sum(...data);
console.log(sum.toString());  // '5.5'

// 对比普通 JavaScript
let wrongSum = data.reduce((a, b) => a + parseFloat(b), 0);
console.log(wrongSum);  // 可能有微小误差
```

### 场景 4: 配置精度

**需求**: 动态调整计算精度

```typescript
import { Decimal } from '@kit.ArkTS';

// 保存当前配置
let originalConfig = {
  precision: Decimal.precision,
  rounding: Decimal.rounding
};

// 设置金融计算精度（2 位小数）
Decimal.set({ 
  precision: 20,  // 计算精度
  rounding: Decimal.ROUND_HALF_UP  // 四舍五入
});

// 执行计算...
let result = new Decimal('10').div(3);
console.log(result.toDecimalPlaces(2).toString());  // '3.33'

// 恢复原始配置
Decimal.set(originalConfig);
```

---

## 4.4 快速使用示例

### 完整示例代码

```typescript
// Index.ets
import { Decimal } from '@kit.ArkTS';

@Entry
@Component
struct DecimalDemo {
  @State result: string = '';

  build() {
    Column({ space: 20 }) {
      Text('Decimal.js 高精度计算示例')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Button('测试基础运算')
        .onClick(() => {
          this.testBasicOperations();
        })

      Button('测试精度问题')
        .onClick(() => {
          this.testPrecisionIssue();
        })

      Button('测试三角函数')
        .onClick(() => {
          this.testTrigonometry();
        })

      Text(this.result)
        .fontSize(14)
        .padding(20)
    }
    .width('100%')
    .padding(20)
  }

  testBasicOperations() {
    let a = new Decimal(0.1);
    let b = new Decimal(0.2);
    let sum = a.add(b);

    this.result = `基础运算:
0.1 + 0.2 = ${sum.toString()}
普通 JS: ${0.1 + 0.2}`;
  }

  testPrecisionIssue() {
    // 金融计算示例
    let price = new Decimal('99.99');
    let discount = new Decimal('0.15');
    let discountedPrice = price.times(new Decimal(1).minus(discount));

    this.result = `金融计算:
原价: ${price.toString()}
折扣: ${discount.times(100)}%
折后价: ${discountedPrice.toDecimalPlaces(2)}`;
  }

  testTrigonometry() {
    // 高精度三角函数
    Decimal.set({ precision: 30 });
    let angle = new Decimal(45);
    let radians = angle.times(Decimal.pi).div(180);
    let sinValue = Decimal.sin(radians);
    let cosValue = Decimal.cos(radians);

    this.result = `三角函数 (45度):
sin: ${sinValue.toString()}
cos: ${cosValue.toString()}
sin² + cos² = ${sinValue.pow(2).plus(cosValue.pow(2))}`;
  }
}
```

### 模块依赖方式

#### 方式 1: 系统模块使用（BUILD.gn）

如果你的系统模块需要依赖 decimal.js：

```gn
# 在你的 BUILD.gn 中
ohos_shared_library("my_module") {
  # ...
  deps = [
    "//third_party/decimal.js:decimal",
  ]
}
```

**注意**: 目前主要用于应用层，系统模块直接依赖较少见。

#### 方式 2: ArkTS 应用使用（ETS）

应用层直接使用：

```typescript
import { Decimal } from '@kit.ArkTS';

// 使用 Decimal...
```

---

## 4.5 性能考虑

### 性能特点

| 操作 | 性能 | 说明 |
|------|------|------|
| 创建 Decimal | 较慢 | 需要解析字符串或数字 |
| 基础运算 | 中等 | 比原生 Number 慢 10-100 倍 |
| 三角函数 | 较慢 | 需要泰勒级数展开 |
| 比较操作 | 快 | 直接比较内部表示 |

### 优化建议

1. **批量计算时复用实例**
   ```typescript
   // 好的做法：创建一次，多次使用
   let base = new Decimal('100');
   for (let i = 0; i < 1000; i++) {
     let result = base.plus(i);
   }
   ```

2. **设置合理精度**
   ```typescript
   // 不需要过高精度时，降低精度以提高性能
   Decimal.set({ precision: 20 });  // 默认 20，金融计算足够
   ```

3. **避免频繁类型转换**
   ```typescript
   // 避免：Decimal → string → Decimal
   let x = new Decimal('10');
   let y = new Decimal(x.toString());  // 不必要的转换
   
   // 好的做法：直接复制
   let z = new Decimal(x);  // 或 x.clone()
   ```

---

## 4.6 常见问题

### Q1: 为什么不用原生的 BigInt？

**A**: BigInt 只支持整数，不支持小数。decimal.js 支持任意精度的小数运算。

### Q2: Decimal 和 JavaScript Number 如何互转？

**A**:
```typescript
// Number → Decimal
let d = new Decimal(0.1);

// Decimal → Number
let n = d.toNumber();

// Decimal → String
let s = d.toString();
```

### Q3: 链式调用是否影响性能？

**A**: 链式调用每次都会创建新实例，性能略低但代码可读性好。性能敏感场景可拆分：

```typescript
// 链式调用（可读性好）
let result = x.plus(y).times(z).div(w);

// 拆分（略快，但代码更长）
let temp1 = x.plus(y);
let temp2 = temp1.times(z);
let result = temp2.div(w);
```

### Q4: 是否支持复杂的数学表达式解析？

**A**: decimal.js 本身不支持表达式解析。如需解析字符串表达式，可使用 math.js 库（内部使用 decimal.js）。

---

## 4.7 依赖关系 Mermaid 图

```mermaid
graph TB
    subgraph "OpenHarmony 系统"
        A[ArkTS 应用] -->|import| B[@kit.ArkTS]
        B -->|加载| C[libdecimal.z.so]
        C -->|注册| D[NAPI 框架]
        C -->|包含| E[decimal.abc 字节码]
        C -->|包含| F[decimal.mjs 源码]
    end
    
    subgraph "产品配置"
        G[rich.json]
        H[wearable.json]
        I[tv.json]
    end
    
    G -->|包含组件| C
    H -->|包含组件| C
    I -->|包含组件| C
    
    subgraph "构建依赖"
        J[BUILD.gn]
        K[es2abc 编译器]
        L[decimal.cpp]
    end
    
    J -->|调用| K
    K -->|生成| E
    J -->|编译| L
    L -->|链接| C
```

---

## 4.8 总结

### 依赖关系总结

| 依赖方向 | 目标 | 类型 |
|----------|------|------|
| decimal.js → | napi:ace_napi | 运行时依赖 |
| decimal.js → | es2abc | 编译时依赖 |
| rich/wearable/tv → | decimal.js | 产品依赖 |
| @kit.ArkTS → | decimal.js | 框架导出 |

### 使用方式总结

| 用户类型 | 使用方式 |
|----------|----------|
| ArkTS 应用 | `import { Decimal } from '@kit.ArkTS'` |
| 系统模块 | `deps = ["//third_party/decimal.js:decimal"]` |

### 定位

decimal.js 是 OpenHarmony 的**基础数学库**，为所有应用提供高精度浮点运算能力，解决 JavaScript 原生 Number 的精度问题。

---

*下一章: [05_API_Differences.md](./05_API_Differences.md) - API 差异*
