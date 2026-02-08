# 06 - 安全风险分析

本文档分析 weex-loader 的安全风险，包括已知 CVE、潜在攻击面和升级建议。

---

## 6.1 已知 CVE 分析

### 6.1.1 搜索 CVE 记录

通过对 Apache weex-loader v0.7.12 及依赖库的 CVE 搜索：

| CVE ID | 影响版本 | 严重程度 | 描述 | 在 OH 版本中的状态 |
|--------|----------|----------|------|------------------|
| 暂无 | - | - | - | - |

**说明**: 截至 2026-02-07，未发现 Apache weex-loader v0.7.12 的直接 CVE 记录。

### 6.1.2 依赖库 CVE

weex-loader 依赖以下 npm 包，需要关注其安全状况：

| 依赖包 | 版本 | 用途 | CVE 状态 |
|--------|------|------|----------|
| `@babel/cli` | 7.20.7 | Babel CLI 工具 | 需关注 |
| `@babel/core` | 7.20.12 | Babel 核心 | 需关注 |
| `uglify-js` | 3.17.4 | 代码压缩 | 需关注 |

**建议**: 定期使用 `npm audit` 检查依赖包的安全状况。

---

## 6.2 潜在攻击面分析

### 6.2.1 代码注入风险

**风险描述**: weex-loader 处理用户提供的 HML/CSS/JS 代码，存在代码注入风险。

**风险点**:

1. **eval 使用** (src/json.js):

```javascript
// src/json.js (line 64)
try {
  source = JSON.stringify(eval('(' + source + ')'))
} catch (e) {
  ...
}
```

**风险**: 在 Card 设备处理中使用 `eval()` 执行用户输入的代码。

**缓解措施**:
- 仅在 Card 设备 (`process.env.DEVICE_LEVEL === 'card'`) 使用
- 输入经过正则表达式预处理
- 异常捕获处理

**建议**: 
- TODO(需确认) - 评估是否可以替换 `eval()` 为更安全的解析方式
- 考虑使用 `JSON.parse` 或受限的解析器

### 6.2.2 路径遍历风险

**风险描述**: 处理文件路径时可能存在路径遍历漏洞。

**风险点**:

1. **模块路径解析** (src/util.js):

```javascript
// src/util.js - checkModuleIsVaild
const json5Path = path.join(process.env.projectPath, 
  '../../../../', 'oh-package.json5');
```

**风险**: 使用 `../` 向上遍历目录。

**缓解措施**:
- 路径经过 `path.join` 规范化
- 文件存在性检查 (`fs.existsSync`)

2. **资源路径处理** (src/loader.js):

```javascript
// src/loader.js
const filePath = path.join(path.dirname(resourcePath), src);
```

**风险**: `src` 属性可能包含恶意路径。

**缓解措施**:
- 路径检查 (`src.match(/^(\/|\/)/)`)
- 文件存在性验证

**建议**:
- 对所有用户输入的路径进行严格验证
- 使用路径规范化库

### 6.2.3 正则表达式 DoS 风险

**风险描述**: 复杂的正则表达式可能导致 ReDoS (Regular Expression Denial of Service)。

**风险点**:

```javascript
// src/util.js
const libReg = /^lib(.+)\.so$/
const REG_SYSTEM = /@(system|ohos)\.(\S+)/g;
```

**评估**: 当前正则表达式较为简单，风险较低。

### 6.2.4 依赖库风险

**风险描述**: 依赖的第三方库可能存在安全漏洞。

**关键依赖**:

1. **weex-scripter**: 处理 JavaScript 代码
2. **weex-styler**: 处理 CSS 样式
3. **parse5**: HTML 解析

**建议**:
- 定期更新依赖库到最新版本
- 使用 `npm audit` 扫描依赖漏洞

---

## 6.3 OH Patch 引入的新攻击面

### 6.3.1 环境变量依赖

**风险描述**: 依赖环境变量控制编译行为，可能被恶意利用。

**影响的环境变量**:
- `DEVICE_LEVEL`
- `abilityType`
- `projectPath`
- `aceManifestPath`

**风险评估**: **低**

**原因**:
- 这些变量由构建系统设置
- 用户无法直接控制

**建议**:
- 验证环境变量的合法性
- 避免直接使用用户输入设置环境变量

### 6.3.2 模块导入逻辑

**风险描述**: OH 特有的模块导入逻辑可能引入安全风险。

**代码分析** (src/util.js):

```javascript
// 模块导入解析
export function parseRequireModule(source, resourcePath) {
  // 替换 require('@system.xxx') → requireModule('@system.xxx')
  // 替换 require('@ohos.xxx') → requireNapi('xxx')
  // 替换 require('libxxx.so') → requireNapi('xxx', true)
}
```

**风险评估**: **低**

**原因**:
- 仅做字符串替换，不执行代码
- 最终执行由运行时控制

### 6.3.3 Lite 设备模块校验

**风险描述**: Lite 设备的模块依赖校验可能绕过。

**代码分析** (src/util.js):

```javascript
export function checkModuleIsVaild(requireStatementExec, resourcePath) {
  // 检查模块是否在 oh-package.json5 的 dependencies 中
  const json5Path = path.join(process.env.projectPath, 
    '../../../../', 'oh-package.json5');
  // ...
}
```

**风险评估**: **中**

**原因**:
- 路径遍历到项目外的 `oh-package.json5`
- 依赖文件内容的正确性

**建议**:
- 限制路径遍历范围
- 验证 json5 文件内容的完整性

---

## 6.4 安全升级策略

### 6.4.1 依赖库升级

**建议频率**: 每季度检查一次

**升级步骤**:

```bash
# 1. 检查依赖漏洞
cd third_party/weex-loader
npm audit

# 2. 更新依赖
npm update

# 3. 验证构建
python build_weex_loader_library.py ...

# 4. 运行测试
npm test

# 5. 验证 ace_js2bundle
# 在 ace_js2bundle 中测试编译功能
```

### 6.4.2 上游版本升级

**安全风险考虑**:

1. **安全补丁**: 上游版本可能包含安全修复
2. **新漏洞**: 新版本可能引入新漏洞
3. **回归风险**: 升级可能导致 OH 特有功能失效

**升级前安全检查**:

- [ ] 检查上游版本的 CVE 记录
- [ ] 检查依赖库的 CVE 记录
- [ ] 审查代码变更的安全影响
- [ ] 进行安全测试

### 6.4.3 代码审查清单

对 weex-loader 的修改进行安全审查：

- [ ] **代码注入**: 是否使用 `eval()` 或类似功能
- [ ] **路径处理**: 是否对用户输入的路径进行验证
- [ ] **正则表达式**: 是否存在复杂的正则表达式
- [ ] **依赖引入**: 新依赖是否经过安全评估
- [ ] **环境变量**: 是否验证环境变量的合法性

---

## 6.5 安全建议

### 6.5.1 短期建议 (1-3 个月)

1. **替换 eval()**:
   - 将 `src/json.js` 中的 `eval()` 替换为更安全的解析方式
   - 建议使用 `JSON.parse` 或沙箱解析器

2. **路径验证**:
   - 对所有用户输入的路径进行严格验证
   - 使用路径规范化库

3. **依赖审计**:
   - 运行 `npm audit` 检查依赖漏洞
   - 更新有漏洞的依赖

### 6.5.2 中期建议 (3-6 个月)

1. **安全测试**:
   - 添加针对代码注入的测试用例
   - 添加针对路径遍历的测试用例

2. **代码审查**:
   - 建立代码审查流程
   - 安全相关修改需要双人审查

3. **监控告警**:
   - 订阅上游项目的安全公告
   - 订阅依赖库的安全公告

### 6.5.3 长期建议 (6-12 个月)

1. **自动化安全扫描**:
   - 集成 SAST (静态应用安全测试) 工具
   - 集成依赖漏洞扫描

2. **安全文档**:
   - 建立安全开发规范
   - 定期进行安全培训

---

## 6.6 应急响应

### 6.6.1 发现安全漏洞时的处理流程

1. **评估影响**:
   - 确定漏洞的严重程度
   - 评估影响范围

2. **修复漏洞**:
   - 开发安全补丁
   - 进行安全测试

3. **发布更新**:
   - 发布安全公告
   - 更新文档

4. **跟踪验证**:
   - 验证修复效果
   - 监控是否引入新问题

### 6.6.2 联系方式

- **维护者**: sunbingxin@huawei.com
- **OpenHarmony 安全团队**: security@openharmony.io
- **Apache Weex**: security@weex.apache.org

---

## 6.7 总结

| 风险类型 | 风险等级 | 状态 |
|----------|----------|------|
| 已知 CVE | 低 | 无已知 CVE |
| 代码注入 (eval) | 中 | 需要修复 |
| 路径遍历 | 低 | 已缓解 |
| 正则表达式 DoS | 低 | 风险可控 |
| 依赖库漏洞 | 中 | 需定期审计 |
| 环境变量依赖 | 低 | 风险可控 |
| 模块导入逻辑 | 低 | 风险可控 |
| Lite 模块校验 | 中 | 需要改进 |

**总体安全状况**: **良好**，但需要注意 `eval()` 的使用和依赖库的安全更新。

---

**文档结束** | [返回 README](./README.md)
