# 02 - Patch 详细分析

本文档详细分析 decimal.js 在 OpenHarmony 中的 Patch 情况。

---

## 2.1 Patch 清单概览

### 搜索结果

在库根目录执行 Patch 搜索：

```bash
find . -name "*.patch" -o -name "patches" -type d
```

**结果**: **无 Patch 文件**

### Patch 统计

| 统计项 | 数量 | 说明 |
|--------|------|------|
| **Patch 文件总数** | 0 | 无 .patch 文件 |
| **Patch 目录** | 无 | 无 patches/ 目录 |
| **当前生效 Patch** | 0 | - |
| **历史 Patch** | 有（已删除） | git 历史显示曾有 patches 目录 |

---

## 2.2 Git 历史分析

### Patch 相关提交

从 git log 发现以下与 Patch 相关的历史：

```
commit d8c33bf - del patches
commit 0cba843 - !11 删除临时添加的patches文件 Merge pull request !11
commit d1985fd - fix decimal ABCcode and install path
commit 1453296 - !10 Decimal增加math前缀修改 Merge pull request !10
commit 570ca42 - !12 适配arkts接口进行修改 Merge pull request !12
```

### 历史 Patch 说明

根据 git 历史，decimal.js 曾经：

1. **有临时 patches 目录**
   - 在早期适配阶段添加了临时 patches
   - 用于适配 ArkTS 接口
   - 后续已删除

2. **主要修改内容**（从历史提交推断）
   - `fix negated to negate`: 函数名修正
   - `Decimal增加math前缀修改`: 模块路径调整
   - `fix decimal ABCcode and install path`: 安装路径和字节码导出修正

**当前状态**: 所有修改已通过其他方式解决（BUILD.gn 配置、decimal.cpp 适配层），不再需要 Patch。

---

## 2.3 无 Patch 的原因分析

### 为什么不需要 Patch？

#### 1. 纯 JavaScript 库

| 特性 | decimal.js | 说明 |
|------|------------|------|
| 语言 | 纯 JavaScript | 无 C/C++ 原生代码 |
| ECMAScript 版本 | ES3 | 兼容性极好 |
| 平台依赖 | 无 | 不依赖特定平台 API |
| 外部依赖 | 零 | 自包含库 |

**结论**: 无需平台特定适配代码。

#### 2. 功能适配在构建层完成

OH 需要的适配通过以下方式完成，无需修改源码：

| 适配需求 | 解决方案 | 文件 |
|----------|----------|------|
| 编译为 ArkTS 字节码 | es2abc 编译器 | BUILD.gn |
| 模块注册 | NAPI 适配层 | decimal.cpp |
| 导出字节码 | 二进制对象嵌入 | BUILD.gn + decimal.cpp |

#### 3. 不修改 API

OH 直接使用上游 API，无以下变更：

- ❌ 无新增 API
- ❌ 无 API 行为变更
- ❌ 无 API 废弃

---

## 2.4 源码对比分析

### 与上游源码的差异

对比 OH 版本与上游 v10.5.0：

#### 文件差异

| 文件 | OH 特有 | 上游也有 | 说明 |
|------|---------|----------|------|
| `decimal.mjs` | ✓ | ✓ | 内容一致 |
| `decimal.js` | ✓ | ✓ | 内容一致 |
| `decimal.d.ts` | ✓ | ✓ | 内容一致 |
| `decimal.cpp` | ✓ | ✗ | OH 特有：NAPI 适配层 |
| `BUILD.gn` | ✓ | ✗ | OH 特有：构建配置 |
| `README_zh.md` | ✓ | ✗ | OH 特有：中文文档 |
| `bundle.json` | ✓ | ✗ | OH 特有：组件配置 |
| `OAT.xml` | ✓ | ✗ | OH 特有：开源合规 |

#### 源码文件内容对比

**decimal.mjs 文件头对比**:

```javascript
// OH 版本（third_party/decimal.js/decimal.mjs）
/*!
 *  decimal.js v10.5.0
 *  An arbitrary-precision Decimal type for JavaScript.
 *  https://github.com/MikeMcl/decimal.js
 *  Copyright (c) 2022 Michael Mclaughlin <M8ch88l@gmail.com>
 *  MIT Licence
 */

// 注意：OH 版本在文件中添加了 BusinessError 错误类定义
class BusinessError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'BusinessError';
    this.code = code;
  }
}
const RANGE_ERROR_CODE = 10200001;
const TYPE_ERROR_CODE = 401;
const PRECISION_LIMIT_EXCEEDED_ERROR_CODE = 10200060;
const CRYPTO_UNAVAILABLE_ERROR_CODE = 10200061;
```

**发现**: OH 版本在 `decimal.mjs` 中添加了 `BusinessError` 错误类和错误码定义，这是上游版本没有的。

**这是 OH 特有的修改！**

---

## 2.5 OH 特有修改详细分析

### 修改位置

文件: `decimal.mjs`（第 9-19 行）

```javascript
class BusinessError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'BusinessError';
    this.code = code;
  }
}
const RANGE_ERROR_CODE = 10200001;
const TYPE_ERROR_CODE = 401;
const PRECISION_LIMIT_EXCEEDED_ERROR_CODE = 10200060;
const CRYPTO_UNAVAILABLE_ERROR_CODE = 10200061;
```

### 修改目的

**原始问题**: ArkTS/ETS 框架需要统一的错误处理机制，使用错误码（error code）而非仅错误消息。

**修改内容**: 

1. **新增 BusinessError 类**
   - 继承自 JavaScript Error
   - 添加 `code` 属性，用于 ArkTS 错误码映射

2. **定义 OH 错误码**
   - `RANGE_ERROR_CODE = 10200001`: 范围错误
   - `TYPE_ERROR_CODE = 401`: 类型错误
   - `PRECISION_LIMIT_EXCEEDED_ERROR_CODE = 10200060`: 精度超限
   - `CRYPTO_UNAVAILABLE_ERROR_CODE = 10200061`: 加密不可用

### 修改在代码中的使用

在 decimal.mjs 中，这些错误码用于以下场景：

```javascript
// 范围错误示例（截取）
if (sd < 1 || sd > MAX_DIGITS) {
  throw new BusinessError(
    '[DecimalError] Invalid argument: ' + sd,
    RANGE_ERROR_CODE
  );
}

// 类型错误示例
if (!isFinite(x)) {
  throw new BusinessError(
    '[DecimalError] Invalid argument: ' + x,
    TYPE_ERROR_CODE
  );
}
```

### Patch 形式说明

**这不是传统的 .patch 文件修改，而是直接修改源码。**

原因：
1. 修改位置固定（文件开头）
2. 修改量少（约 15 行）
3. 便于版本升级时重新应用

---

## 2.6 升级建议

### 上游版本升级流程

由于无 .patch 文件，但有源码修改，升级步骤：

```bash
# 1. 备份当前版本
cp decimal.mjs decimal.mjs.backup
cp decimal.js decimal.js.backup

# 2. 替换为新版本（从上游下载）
curl -o decimal.mjs https://raw.githubusercontent.com/MikeMcl/decimal.js/master/decimal.mjs
curl -o decimal.js https://raw.githubusercontent.com/MikeMcl/decimal.js/master/decimal.js

# 3. 重新应用 OH 特有修改（在文件开头添加 BusinessError）
# 编辑 decimal.mjs，在第 7 行后添加错误类定义

# 4. 验证 es2abc 编译
./build.sh --product-name rk3568 --target third_party/decimal.js

# 5. 运行测试
```

### 升级注意事项

| 检查项 | 说明 | 优先级 |
|--------|------|--------|
| BusinessError 重新添加 | 确保错误类和错误码在新版本中 | 高 |
| API 兼容性 | 检查上游是否有 API 变更 | 中 |
| es2abc 编译 | 确保新版本能通过 ArkTS 编译 | 高 |
| NAPI 加载 | 验证模块能正常加载 | 高 |

### 低风险点

- ✅ 纯 JavaScript，无原生代码兼容性风险
- ✅ API 稳定，版本间破坏性变更少
- ✅ 修改位置固定，易于重新应用

### 需关注

- ⚠️ 错误码映射是否与 ArkTS 框架保持同步
- ⚠️ 新版本中错误处理逻辑是否有变化

---

## 2.7 与其他库 Patch 策略对比

| 库 | Patch 数量 | 策略 | decimal.js 优势 |
|----|-----------|------|-----------------|
| curl | 大量 | 多 Patch 文件 | 无 Patch，维护简单 |
| openssl | 大量 | 多 Patch 文件 | 无 Patch，维护简单 |
| decimal.js | 0（但有源码修改） | 直接修改 | 版本升级容易 |

---

## 2.8 总结

### Patch 状态总结

| 项目 | 状态 |
|------|------|
| **.patch 文件** | 无 |
| **源码修改** | 有（BusinessError 错误类） |
| **修改方式** | 直接修改 decimal.mjs |
| **修改大小** | 约 15 行 |
| **修改位置** | 文件开头（易于重新应用） |

### 关键结论

1. **无传统 Patch 文件**: 不需要维护 .patch 文件，简化版本管理
2. **有源码修改**: 添加了 ArkTS 错误码支持（BusinessError）
3. **升级友好**: 修改位置固定，版本升级时易于重新应用
4. **低风险**: 纯 JavaScript，无平台依赖

### 维护建议

1. **版本升级**: 直接替换源码后重新添加错误类定义
2. **错误码同步**: 与 ArkTS 框架团队保持错误码映射同步
3. **测试重点**: es2abc 编译、模块加载、错误抛出

---

*下一章: [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配*
