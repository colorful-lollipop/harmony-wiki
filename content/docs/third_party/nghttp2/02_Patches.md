# Patch 详细分析

## Patch 清单总览

| Patch 文件 | 修改文件 | 修改类型 | 影响范围 | 修改目的 |
|-----------|---------|---------|---------|---------|
| `src/http-parser.patch` | `examples/http-parser/http_parser.c` | 示例代码 | nghttpx 代理 | 支持隧道连接透明传输 |

**重要说明**: nghttp2 在 OpenHarmony 中 **仅有 1 个 patch**，且修改的是**示例代码**（`examples/` 目录），**不影响核心库功能**。

---

## Patch 详细分析

### Patch: src/http-parser.patch

#### 基本信息

```
提交: a143133d43420ef89e4ba0d84c73998863cf9f81
作者: Tatsuhiro Tsujikawa <tatsuhiro.t@gmail.com>
日期: Wed Jul 11 18:46:00 2012 +0900
主题: Use http_parser for tunneling connection transparently
```

#### 修改位置

**原始文件**: `examples/http-parser/http_parser.c`

**注意**: 这是 nghttp2 项目自带的示例代码中的 http_parser，不是核心库代码。

#### 修改内容

```diff
--- a/examples/http-parser/http_parser.c
+++ b/examples/http-parser/http_parser.c
@@ -1627,9 +1627,14 @@ size_t http_parser_execute (http_parser *parser,
 
         /* Exit, the rest of the connect is in a different protocol. */
         if (parser->upgrade) {
-          parser->state = NEW_MESSAGE();
-          CALLBACK_NOTIFY(message_complete);
-          return (p - data) + 1;
+          /* We want to use http_parser for tunneling connection
+             transparently */
+          /* Read body until EOF */
+          parser->state = s_body_identity_eof;
+          break;
+          /* parser->state = NEW_MESSAGE(); */
+          /* CALLBACK_NOTIFY(message_complete); */
+          /* return (p - data) + 1; */
         }
```

#### 修改函数

- **函数**: `http_parser_execute()`
- **文件**: `examples/http-parser/http_parser.c`
- **行号**: 约 1630 行附近

#### 修改逻辑对比

| 逻辑 | 原始行为 | 修改后行为 |
|------|---------|-----------|
| **触发条件** | `parser->upgrade` 为真时 | 同上 |
| **状态转换** | `NEW_MESSAGE()` | `s_body_identity_eof` |
| **后续处理** | 调用 `message_complete` 回调并返回 | 继续循环，读取 body 直到 EOF |
| **适用场景** | 标准 HTTP Upgrade | 隧道连接 (CONNECT 方法) |

#### 修改目的分析

**原始问题**:
当 HTTP 请求包含 `Upgrade` 头（如 WebSocket、HTTP/2 升级）时，原始 http_parser 会：
1. 立即通知 `message_complete`
2. 返回剩余数据长度
3. 期望调用者处理协议切换

这对于标准 Upgrade 场景是正确的，但对于 **CONNECT 方法**（用于建立隧道连接，如 HTTPS 代理）存在问题：

**CONNECT 方法的特点**:
- 客户端发送 `CONNECT target.host:443 HTTP/1.1`
- 代理服务器返回 `200 Connection established`
- 之后的数据流是**端到端的加密隧道数据**，不是 HTTP
- 但这些数据**仍然需要通过 http_parser** 来透明传输

**修改后的行为**:
1. 不立即结束解析
2. 将状态设为 `s_body_identity_eof`（读取 body 直到连接关闭）
3. 允许 http_parser 继续处理隧道数据流
4. 实现透明传输（transparent tunneling）

#### OH 需求关联

**TODO(需确认)**: 此 patch 与 OpenHarmony 的关系需要进一步确认

**可能性分析**:

1. **低可能性**: OpenHarmony 仅使用 libnghttp2 核心库，不使用 nghttpx 代理
   - 此 patch 对 OH 无实际影响
   
2. **中可能性**: nghttpx 的某些功能需要此 patch
   - 如果 OH 使用 nghttpx 作为代理，则需要此 patch
   
3. **高可能性**: 这是 nghttp2 上游自带的 patch
   - 可能已合并到上游代码中
   - 保留 patch 文件是为了兼容性

**建议**: 检查 OH 是否使用 `examples/` 目录或 nghttpx 工具。

---

## Patch 分类分析

### 按修改类型分类

| 分类 | 本库 Patch 数量 | 说明 |
|------|----------------|-----|
| Bugfix | 0 | 无 bugfix 类 patch |
| Feature | 1 | http-parser.patch 属于功能增强 |
| OH 适配 | 0 | 无 OH 特定适配代码 |
| 性能优化 | 0 | 无性能优化类 patch |

### 按影响范围分类

| 范围 | Patch 数量 | 说明 |
|------|-----------|-----|
| 核心库 | 0 | libnghttp2 无任何修改 |
| 应用程序 | 1 | nghttpx 相关示例代码 |
| 构建系统 | 0 | 构建系统通过 BUILD.gn 配置，无 patch |

---

## 升级建议

### Patch 维护策略

由于 nghttp2 只有一个 patch 且影响范围有限，升级策略相对简单：

#### 方案一：跟随上游升级（推荐）

**适用场景**: 新版本 nghttp2 包含此 patch 的等效修改

**步骤**:
1. 检查上游 v1.66.0+ 是否已包含此修改
2. 如果已包含，删除 `src/http-parser.patch`
3. 如果未包含，保留 patch 并测试

**优势**:
- 减少维护负担
- 获得上游最新功能和安全修复

#### 方案二：保持当前版本

**适用场景**: 当前版本稳定，无安全漏洞

**步骤**:
1. 定期检查上游安全公告
2. 仅在必要时升级

**优势**:
- 稳定性高
- 测试成本低

### 回归风险

| 风险项 | 风险等级 | 说明 |
|--------|---------|-----|
| API 兼容性 | 低 | nghttp2 API 稳定 |
| curl 兼容性 | 中 | 需要验证 curl 与新版本配合 |
| 功能变更 | 低 | 核心功能稳定 |
| 性能变化 | 低 | 通常性能改进或持平 |

### 升级检查清单

升级 nghttp2 版本时，请检查：

- [ ] 新版本是否修复了已知 CVE
- [ ] `libnghttp2_shared.map` 是否需要更新（新增符号）
- [ ] curl 是否能正常链接和运行
- [ ] 删除或更新 `src/http-parser.patch`（如上游已合并）
- [ ] 在 LiteOS 和 Standard 系统上分别测试
- [ ] 验证 HTTP/2 功能正常（多路复用、服务器推送等）

---

## 结论

### Patch 总结

1. **Patch 数量极少**: 仅 1 个 patch，且修改的是示例代码
2. **无侵入式修改**: 核心库代码无任何修改
3. **维护成本低**: 升级时主要关注 API 兼容性
4. **影响范围小**: 即使 patch 丢失，也只影响 nghttpx（如使用）

### 与 OH 的关系

| 评估项 | 结论 |
|--------|------|
| OH 特定 Patch | 无 |
| 核心库修改 | 无 |
| 升级难度 | 低 |
| 回归风险 | 低 |

### 维护建议

1. **短期**: 保持当前版本 v1.66.0，定期检查安全公告
2. **中期**: 跟随 curl 的升级节奏同步升级 nghttp2
3. **长期**: 考虑删除 `src/http-parser.patch`（如确认上游已合并）

---

## 附录：Patch 原始内容

```diff
commit a143133d43420ef89e4ba0d84c73998863cf9f81
Author: Tatsuhiro Tsujikawa <tatsuhiro.t@gmail.com>
Date:   Wed Jul 11 18:46:00 2012 +0900

    Use http_parser for tunneling connection transparently

diff --git a/examples/http-parser/http_parser.c b/examples/http-parser/http_parser.c
index 0c11eb8..610da57 100644
--- a/examples/http-parser/http_parser.c
+++ b/examples/http-parser/http_parser.c
@@ -1627,9 +1627,14 @@ size_t http_parser_execute (http_parser *parser,
 
         /* Exit, the rest of the connect is in a different protocol. */
         if (parser->upgrade) {
-          parser->state = NEW_MESSAGE();
-          CALLBACK_NOTIFY(message_complete);
-          return (p - data) + 1;
+          /* We want to use http_parser for tunneling connection
+             transparently */
+          /* Read body until EOF */
+          parser->state = s_body_identity_eof;
+          break;
+          /* parser->state = NEW_MESSAGE(); */
+          /* CALLBACK_NOTIFY(message_complete); */
+          /* return (p - data) + 1; */
         }
 
         if (parser->flags & F_SKIPBODY) {
```
