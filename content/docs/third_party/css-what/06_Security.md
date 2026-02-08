# 06\_安全风险分析

## 6.1 安全概述

### 库的安全评级

| 评估项       | 评级      | 说明                 |
| ------------ | --------- | -------------------- |
| **已知 CVE** | ✅ 无     | 未发现公开 CVE       |
| **代码审计** | ✅ 良好   | 社区活跃，持续维护   |
| **依赖安全** | ✅ 安全   | 无外部依赖           |
| **输入验证** | ⚠️ 需注意 | 解析器本身的安全使用 |

### 安全特点

| 安全特性          | 状态 | 说明               |
| ----------------- | ---- | ------------------ |
| **无外部依赖**    | ✅   | 零供应链风险       |
| **纯 TypeScript** | ✅   | 无原生代码漏洞     |
| **活跃维护**      | ✅   | 社区活跃，及时修复 |
| **小代码库**      | ✅   | 代码量小，易于审计 |

---

## 6.2 CVE 分析

### 已知漏洞

**结论：未发现 css-what 的已知 CVE**

```bash
# 搜索结果
$ npm audit css-what
# found 0 vulnerabilities
```

### CVE 历史

| CVE ID | 严重程度 | 修复版本 | 状态       |
| ------ | -------- | -------- | ---------- |
| 无     | -        | -        | 无已知漏洞 |

### 漏洞扫描结果

```bash
# npm audit 结果
found 0 vulnerabilities
```

---

## 6.3 安全风险点

### 6.3.1 输入验证风险

#### 风险描述

虽然 css-what 解析器本身无已知漏洞，但**错误的使用方式**可能导致安全问题：

| 风险场景     | 描述                 | 风险等级 |
| ------------ | -------------------- | -------- |
| **DoS 攻击** | 恶意构造的超长选择器 | ⚠️ 中    |
| **ReDoS**    | 正则表达式拒绝服务   | ⚠️ 中    |
| **内存溢出** | 深度嵌套的选择器     | ⚠️ 低    |

#### 风险代码示例

```typescript
// ❌ 风险：未限制选择器长度
function parseUserSelector(selector: string) {
    return parse(selector); // 无长度限制
}

// ✅ 安全：限制选择器长度
const MAX_SELECTOR_LENGTH = 10000;

function parseUserSelector(selector: string) {
    if (selector.length > MAX_SELECTOR_LENGTH) {
        throw new Error("Selector too long");
    }
    return parse(selector);
}
```

### 6.3.2 正则表达式 DoS (ReDoS)

#### 风险分析

css-what 使用正则表达式解析选择器，理论上存在 ReDoS 风险：

```typescript
// 潜在风险的正则表达式模式
// 如：aaaaaaaaaaaaaaaaaaaaaaaaaa!
// 需要验证上游是否有 ReDoS 防护
```

#### 缓解措施

| 措施             | 实施                   | 效果    |
| ---------------- | ---------------------- | ------- |
| **输入长度限制** | 在调用前限制选择器长度 | ✅ 有效 |
| **超时控制**     | 解析操作设置超时       | ✅ 有效 |
| **正则优化**     | 上游持续优化正则表达式 | ✅ 有效 |

### 6.3.3 内存溢出风险

#### 风险场景

```typescript
// ❌ 风险：深度嵌套的选择器
const nested = "div ".repeat(10000) + "span";

// 可能导致内存压力
const ast = parse(nested);
```

#### 缓解措施

```typescript
// ✅ 安全：限制选择器复杂度
const MAX_SELECTOR_DEPTH = 20;
const MAX_SELECTOR_LENGTH = 10000;

function safeParse(selector: string) {
    if (selector.length > MAX_SELECTOR_LENGTH) {
        throw new Error("Selector exceeds maximum length");
    }

    const depth = countNestingDepth(selector);
    if (depth > MAX_SELECTOR_DEPTH) {
        throw new Error("Selector exceeds maximum depth");
    }

    return parse(selector);
}
```

---

## 6.4 OH 特有安全考量

### 6.4.1 集成风险

由于**无 Patch**，集成过程中无额外安全风险：

| 风险项               | 评估      | 说明               |
| -------------------- | --------- | ------------------ |
| **Patch 引入的风险** | ✅ 无     | 无 Patch，无此风险 |
| **依赖引入的风险**   | ✅ 无     | 无外部依赖         |
| **构建过程风险**     | ✅ 低     | 标准 GN 构建       |
| **运行时风险**       | ⚠️ 需注意 | 正常使用           |

### 6.4.2 运行时安全

```typescript
// 在 jsframework 中的安全使用建议
class CssSelectorParser {
    private static readonly MAX_LENGTH = 10000;

    static parse(selector: string) {
        // 1. 输入验证
        if (!selector || selector.length > this.MAX_LENGTH) {
            throw new Error("Invalid selector");
        }

        // 2. 解析
        try {
            return parse(selector);
        } catch (e) {
            // 3. 错误处理
            console.error("Selector parsing failed:", e);
            return null;
        }
    }
}
```

---

## 6.5 安全最佳实践

### 6.5.1 使用建议

```typescript
// ✅ 推荐：安全的使用模式

// 1. 输入验证
function validateSelector(selector: string): boolean {
    // 长度限制
    if (selector.length > 10000) {
        return false;
    }

    // 格式检查
    if (!/^[\w\s\-_#.:[\]=()>+~*/]*$/.test(selector)) {
        return false;
    }

    return true;
}

// 2. 安全解析
function safeParse(selector: string) {
    if (!validateSelector(selector)) {
        throw new Error("Invalid selector");
    }

    try {
        return parse(selector);
    } catch (e) {
        // 静默处理或记录日志
        return null;
    }
}
```

### 6.5.2 监控建议

```typescript
// 3. 监控和日志
interface ParseMetrics {
    successCount: number;
    failCount: number;
    avgParseTime: number;
    maxParseTime: number;
}

function parseWithMetrics(selector: string) {
    const start = Date.now();

    try {
        const result = parse(selector);

        // 记录成功指标
        metrics.successCount++;
        metrics.avgParseTime =
            (metrics.avgParseTime + (Date.now() - start)) / 2;

        return result;
    } catch (e) {
        // 记录失败指标
        metrics.failCount++;

        // 告警（如果失败率过高）
        if (metrics.failCount / metrics.successCount > 0.1) {
            alert("High selector parse failure rate");
        }

        return null;
    }
}
```

---

## 6.6 安全更新策略

### 6.6.1 更新建议

| 场景             | 操作       | 优先级 |
| ---------------- | ---------- | ------ |
| **上游安全更新** | 尽快升级   | 🔴 高  |
| **上游功能更新** | 评估后升级 | 🟡 中  |
| **常规维护**     | 定期升级   | 🟢 低  |

### 6.6.2 更新流程

```bash
# 1. 检查上游版本
npm view css-what version

# 2. 检查变更日志
npm view css-what changelog

# 3. 检查安全公告
npm audit

# 4. 测试验证
npm test

# 5. 构建验证
hb build css-what
```

---

## 6.7 安全建议总结

### 短期建议

1. ✅ **当前版本安全**：无需立即行动
2. ✅ **持续监控**：关注上游安全公告
3. ✅ **输入验证**：在使用处添加验证逻辑

### 中期建议

1. 🟡 **依赖扫描**：集成到 CI/CD 流程
2. 🟡 **性能监控**：监控 ReDoS 风险
3. 🟡 **更新策略**：制定定期更新计划

### 长期建议

1. 🟢 **代码审计**：周期性的安全审计
2. 🟢 **安全测试**：集成安全测试用例
3. 🟢 **应急响应**：准备安全事件响应流程

---

## 6.8 相关资源

### 安全工具

| 工具                     | 用途         |
| ------------------------ | ------------ |
| `npm audit`              | 依赖漏洞扫描 |
| `snyk`                   | 依赖安全分析 |
| `GitHub Security Alerts` | 安全漏洞通知 |

### 上游安全资源

- [css-what 安全策略](https://github.com/fb55/css-what/security)
- [npm 安全最佳实践](https://docs.npmjs.com/security-best-practices)

---

## 6.9 小结

### 安全评估

| 评估项       | 状态      | 备注         |
| ------------ | --------- | ------------ |
| **已知 CVE** | ✅ 无     | 当前版本安全 |
| **依赖风险** | ✅ 无     | 无外部依赖   |
| **代码质量** | ✅ 良好   | 社区活跃维护 |
| **使用风险** | ⚠️ 需注意 | 需正确使用   |

### 关键建议

1. ✅ **库本身安全**：无需担忧库本身的安全漏洞
2. ⚠️ **正确使用**：关注输入验证和使用方式
3. 🟢 **持续监控**：关注上游安全更新

---

## 参考信息

### 相关文档

- [01_Overview.md](./01_Overview.md) - 库概述
- [02_Patches.md](./02_Patches.md) - Patch 分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 在 OH 中的使用

### 安全资源

- [npm 安全文档](https://docs.npmjs.com/security)
- [OWASP 输入验证](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [ReDoS 防护指南](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS)

---

_文档版本：1.0_
_最后更新：2026-02-08_
