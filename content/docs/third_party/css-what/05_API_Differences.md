# 05_API 差异

## 5.1 概述

### 结论：**无 API 差异**

css-what 在 OpenHarmony 中**未引入任何 OH 特有的 API**，也**未修改任何上游 API**。

| 差异类型     | 数量 | 说明            |
| ------------ | ---- | --------------- |
| **新增 API** | 0    | 无 OH 特有接口  |
| **变更 API** | 0    | 无上游 API 修改 |
| **废弃 API** | 0    | 无功能禁用      |
| **条件 API** | 0    | 无条件编译代码  |

---

## 5.2 API 清单

### 完整 API 列表

```typescript
// 导出接口
export { parse, stringify };

// parse: 解析 CSS 选择器
declare function parse(selector: string): Selector[][];

// stringify: 将 AST 还原为字符串
declare function stringify(selector: Selector[][]): string;
```

### 接口说明

| 函数        | 参数                     | 返回值         | 描述                  |
| ----------- | ------------------------ | -------------- | --------------------- |
| `parse`     | `selector: string`       | `Selector[][]` | 解析 CSS 选择器字符串 |
| `stringify` | `selector: Selector[][]` | `string`       | 将 AST 还原为字符串   |

---

## 5.3 无差异原因分析

### 5.3.1 功能通用性

```typescript
// CSS 选择器解析是标准 Web 功能
// W3C CSS Selectors Level 3/4
// 所有平台实现一致，无需适配
```

| 功能特性     | 是否通用    | 适配需求 |
| ------------ | ----------- | -------- |
| CSS 语法解析 | ✅ 完全通用 | 无       |
| AST 结构     | ✅ 完全通用 | 无       |
| 选择器匹配   | ✅ 完全通用 | 无       |

### 5.3.2 集成方式

```gn
# 直接使用上游源码，无任何修改
source = "src"
# 无条件编译
# 无 API 包装层
```

| 集成方式     | API 影响  |
| ------------ | --------- |
| 源码直接复制 | ✅ 无差异 |
| 无预处理器   | ✅ 无差异 |
| 无适配层     | ✅ 无差异 |

---

## 5.4 上游 API 兼容性

### API 稳定性

| 版本   | API 稳定性 | 备注             |
| ------ | ---------- | ---------------- |
| v7.0.0 | 稳定       | 当前 OH 使用版本 |
| v6.x   | 稳定       | 向前兼容         |
| v5.x   | 稳定       | 历史版本         |

### 升级注意事项

由于**无 API 差异**，升级时只需关注：

1. **功能兼容性**：新版本功能是否正常
2. **性能变化**：解析性能是否有提升或下降
3. **安全性**：是否修复了安全漏洞

```typescript
// API 使用示例（上游标准用法）
import { parse, stringify } from "css-what";

// 解析
const ast = parse("div.container > p");

// 还原
const selector = stringify(ast);

// 与上游完全一致，无需 OH 适配
```

---

## 5.5 未来可能的 API 需求

### 可能的 OH 特有需求

| 需求              | 可能性 | 说明                  |
| ----------------- | ------ | --------------------- |
| **OH 选择器扩展** | 低     | OH 如引入自定义选择器 |
| **性能优化 API**  | 中     | 添加缓存或批处理接口  |
| **诊断 API**      | 低     | 选择器调试/验证工具   |

### 建议

如有上述需求，建议：

1. **优先提交上游**：推动社区接受通用改进
2. **保持接口一致**：如需 OH 特有 API，保持与上游风格一致
3. **文档完善**：明确标注 OH 特有接口

---

## 5.6 使用建议

### 5.6.1 标准用法

```typescript
// 完全使用上游 API，无需特殊处理
import { parse, stringify } from "css-what";

// 解析选择器
const selector = '.container .item[data-type="card"]';
const ast = parse(selector);

// 还原选择器
const str = stringify(ast);
```

### 5.6.2 类型安全

```typescript
// 使用 TypeScript 类型定义
import type { Selector, AttributeToken, PseudoToken } from "css-what";

// 确保类型正确
const tokens: Selector[] = ast[0];
```

---

## 5.7 小结

| 评估项       | 结果 |
| ------------ | ---- |
| **API 差异** | 无   |
| **新增接口** | 0    |
| **修改接口** | 0    |
| **废弃接口** | 0    |
| **条件编译** | 无   |

### 关键结论

1. ✅ **完全兼容上游**：API 与上游完全一致
2. ✅ **零适配成本**：无需学习 OH 特有接口
3. ✅ **易于升级**：跟随上游版本无障碍

---

## 5.8 参考信息

### 相关文档

- [01_Overview.md](./01_Overview.md) - 库概述
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 在 OH 中的使用
- [06_Security.md](./06_Security.md) - 安全分析

### 上游文档

- [css-what API 文档](https://github.com/fb55/css-what#api)
- [CSS Selectors 规范](https://www.w3.org/TR/selectors-3/)

---

_文档版本：1.0_
_最后更新：2026-02-08_
