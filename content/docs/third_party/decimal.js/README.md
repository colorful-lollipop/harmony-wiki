# decimal.js - OpenHarmony Wiki

**decimal.js** 在 OpenHarmony 中的集成与适配文档。

---

## 快速概览

| 属性 | 信息 |
|------|------|
| **原始库** | decimal.js - JavaScript 任意精度 Decimal 类型库 |
| **上游版本** | v10.5.0 |
| **上游地址** | https://github.com/MikeMcl/decimal.js.git |
| **许可证** | MIT |
| **OH 组件名** | @ohos/decimal.js |
| **OH 子系统** | thirdparty |
| **Patch 数量** | **0**（无 Patch） |
| **主要适配** | BUILD.gn + decimal.cpp（NAPI 适配层） |

---

## 核心特点

### 1. 无代码修改

decimal.js 是 OpenHarmony 中**少有的无需任何 Patch**的第三方库。原因：

- 纯 JavaScript（ECMAScript 3），无平台依赖
- 功能自包含，无需系统特定适配
- 通过 es2abc 编译器直接转为 ArkTS 字节码

### 2. 构建系统适配

OH 特有的工作集中在：

- **BUILD.gn**: GN 构建脚本，使用 es2abc 转换字节码
- **decimal.cpp**: C++ 适配层，注册 NAPI 模块，导出字节码
- **输出**: `libdecimal.z.so` 共享库

### 3. 基础系统组件

- 在 rich、wearable、tv 三种产品形态中**默认包含**
- 通过 `@kit.ArkTS` 暴露给应用层
- 提供 ArkTS 应用高精度浮点运算能力

---

## 文档导航

### 核心文档

| 文档 | 内容 | 阅读建议 |
|------|------|----------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 中的作用 | 首次阅读 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） | 了解升级影响 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 详解、构建流程 | 构建开发者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、示例 | 应用开发者 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异（本库无差异） | API 参考 |
| [06_Security.md](./06_Security.md) | CVE 风险、安全建议 | 安全评估 |

### 工作文档

- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估报告
- [_work/NOTES.md](./_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](./_work/PLAN.md) - 任务进度

---

## 快速使用

### ArkTS 应用中使用

```typescript
import { Decimal } from '@kit.ArkTS';

// 创建高精度数值
let a = new Decimal(0.1);
let b = new Decimal(0.2);
let sum = a.add(b);
console.log(sum.toString()); // '0.3'（精确值）

// 对比普通 JavaScript
console.log(0.1 + 0.2); // 0.30000000000000004（精度丢失）
```

### 系统模块依赖

在 BUILD.gn 中添加：
```gn
deps = [ "//third_party/decimal.js:decimal" ]
```

---

## 关键信息

### 版本状态

- **上游最新**: v10.5.0 (2022-12-04)
- **OH 代码**: v10.5.0（已同步）
- **OH Bundle**: 10.4.3（版本号未更新，不影响功能）

### 维护建议

1. **升级风险**: 低（无 Patch，直接替换源码即可）
2. **测试重点**: es2abc 编译、NAPI 加载、ArkTS 调用
3. **版本号同步**: 建议更新 bundle.json 中的版本号

---

## 相关链接

- [上游仓库](https://github.com/MikeMcl/decimal.js)
- [上游文档](https://mikemcl.github.io/decimal.js/)
- [OpenHarmony 子系统 README_zh.md](../README_zh.md)

---

*最后更新: 2025-02-08*
