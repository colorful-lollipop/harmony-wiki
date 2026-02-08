# 安全风险评审

> 攻击面、信任边界、可被利用点、修复建议

## 目的与适用范围

### 目的
本文档对 JSVM 进行安全风险评审，包括攻击面、信任边界、可被利用点和修复建议。

### 适用范围
- 需要了解安全风险的开发者
- 需要进行安全评审的开发者
- 需要进行安全加固的开发者

---

## 威胁模型

### 外部输入 → 敏感操作

JSVM 的主要威胁路径是从外部输入到敏感操作的执行：

```
外部输入（JavaScript 代码、JSON 等）
    ↓
JSVM-API 解析和验证
    ↓
V8 引擎执行
    ↓
敏感操作（文件 I/O、网络 I/O、系统调用）
```

**证据位置**:
- `interface/kits/jsvm.h` - JSVM-API
- `src/js_native_api_v8.cpp` - API 实现
- `src/platform/` - 平台层实现

---

## 攻击面清单

### 1. N-API / JSVM-API 攻击面

**攻击向量**: 通过 JSVM-API 传入恶意 JavaScript 代码。

**证据位置**:
- `interface/kits/jsvm.h:501-581` - `OH_JSVM_CompileScript()`, `OH_JSVM_RunScript()`
- `src/js_native_api_v8.cpp:1274-1549` - 编译与执行实现

**潜在风险**:
- 代码注入攻击
- 恶意 JavaScript 代码执行
- 内存破坏（V8 漏洞）

### 2. IPC 攻击面

**攻击向量**: 通过 IPC 传递恶意数据到 JSVM。

**证据位置**: `bundle.json:24-38` - 依赖组件（可能通过 IPC 交互）

**潜在风险**:
- 跨进程攻击
- 权限提升

### 3. 文件攻击面

**攻击向量**: 通过 JavaScript 代码访问文件系统。

**证据位置**:
- `src/platform/platform_ohos.cpp` - 文件 I/O 实现

**潜在风险**:
- 路径遍历攻击
- 任意文件读写

### 4. 网络攻击面

**攻击向量**: 通过 JavaScript 代码访问网络。

**证据位置**:
- `bundle.json:34` - nghttp2 依赖
- `src/platform/platform_ohos.cpp` - 网络 I/O 实现

**潜在风险**:
- SSRF 攻击（Server-Side Request Forgery）
- 网络钓鱼

### 5. Inspector 攻击面

**攻击向量**: 通过 Inspector WebSocket 接口访问和调试。

**证据位置**:
- `interface/kits/jsvm.h:1719-1747` - Inspector API
- `src/inspector/inspector_socket_server.cpp` - WebSocket 服务器

**潜在风险**:
- 未授权的调试访问
- 代码注入

---

## 信任边界

### 边界 1: 应用代码 ↔ JSVM-API

**信任级别**: 部分信任

**说明**: 应用代码通过 JSVM-API 与 JSVM 交互，JSVM 需要验证参数但信任部分业务逻辑。

**证据位置**: `interface/kits/jsvm.h` - JSVM-API 参数校验

### 边界 2: JSVM-Core ↔ V8 Engine

**信任级别**: 高信任

**说明**: JSVM-Core 完全信任 V8 Engine 的实现。

**证据位置**: `src/js_native_api_v8.cpp` - V8 调用

### 边界 3: JSVM ↔ Platform Layer

**信任级别**: 高信任

**说明**: JSVM 信任 Platform Layer 的实现。

**证据位置**: `src/platform/` - 平台抽象

### 边界 4: Inspector ↔ 外部

**信任级别**: 低信任

**说明**: Inspector 通过 WebSocket 与外部调试器通信，需要严格验证。

**证据位置**: `src/inspector/inspector_socket_server.cpp` - WebSocket 服务器

---

## 可被利用点

### 1. JavaScript 代码注入

**严重性**: 高

**证据**:
- 路径: `interface/kits/jsvm.h:501-507` - `OH_JSVM_CompileScript()`
- 路径: `interface/kits/jsvm.h:581` - `OH_JSVM_RunScript()`
- 实现: `src/js_native_api_v8.cpp:1274-1549`

**触发条件**: 应用接受用户输入并传递给 `OH_JSVM_CompileScript()` 或 `OH_JSVM_RunScript()`，未进行充分验证。

**攻击路径**:
```
用户输入 → 应用代码 → OH_JSVM_CompileScript() → V8 编译 → 执行恶意代码
```

**影响**:
- 任意代码执行
- 内存破坏
- 信息泄露

**修复建议**:
1. 严格验证 JavaScript 代码来源
2. 限制可执行的 API 和权限
3. 使用代码沙箱（TODO: 需确认是否支持）
4. 实施代码审计和静态分析

**证据位置**:
- `interface/kits/jsvm.h:501-507` - 编译 API
- `interface/kits/jsvm.h:581` - 执行 API

### 2. JSON 解析注入

**严重性**: 中

**证据**:
- 路径: `interface/kits/jsvm.h:1575` - `OH_JSVM_JsonParse()`
- 实现: `src/js_native_api_v8.cpp:1575-1591`

**触发条件**: 应用接受用户输入的 JSON 字符串并传递给 `OH_JSVM_JsonParse()`，未进行充分验证。

**攻击路径**:
```
用户输入（恶意 JSON）→ 应用代码 → OH_JSVM_JsonParse() → 解析恶意 JSON
```

**影响**:
- 拒绝服务攻击（构造超深嵌套的 JSON）
- 内存破坏（V8 JSON 解析器漏洞）

**修复建议**:
1. 限制 JSON 大小
2. 限制 JSON 嵌套深度
3. 使用安全的 JSON 解析器
4. 实施超时机制

**证据位置**:
- `interface/kits/jsvm.h:1575` - JSON 解析 API

### 3. 类型混淆攻击

**严重性**: 中

**证据**:
- 路径: `interface/kits/jsvm.h:1174-2500+` - 类型转换 API
- 实现: `src/js_native_api_v8.cpp` - 类型转换实现

**触发条件**: 应用不正确地使用 `OH_JSVM_GetValue...()` 系列 API，未检查返回状态。

**攻击路径**:
```
恶意 JavaScript 值 → 应用代码 → OH_JSVM_GetValueInt32()（未检查状态）→ 使用无效值
```

**影响**:
- 内存破坏
- 信息泄露
- 拒绝服务

**修复建议**:
1. 始终检查 JSVM API 返回状态
2. 验证返回的值类型
3. 使用静态分析工具检测未检查的状态码

**证据位置**:
- `interface/kits/jsvm.h:1174-2500+` - 类型转换 API
- `interface/kits/jsvm_types.h:277-334` - 状态码定义

### 4. 引用计数错误

**严重性**: 中

**证据**:
- 路径: `interface/kits/jsvm.h:823-904` - 引用管理 API
- 实现: `src/jsvm_reference.cpp`

**触发条件**: 应用不正确地管理 JSVM_Ref，导致引用计数错误。

**攻击路径**:
```
JSVM_Ref → 引用计数错误（泄漏或提前释放）→ 悬垂指针或内存泄漏
```

**影响**:
- 悬垂指针（Use-After-Free）
- 内存泄漏
- 拒绝服务

**修复建议**:
1. 使用 RAII 管理引用计数
2. 实施引用计数审计工具
3. 使用 Valgrind/ASan 检测内存错误

**证据位置**:
- `interface/kits/jsvm.h:823-904` - 引用管理 API
- `src/jsvm_reference.cpp` - 引用实现

### 5. Inspector 未授权访问

**严重性**: 高

**证据**:
- 路径: `interface/kits/jsvm.h:1719` - `OH_JSVM_OpenInspector()`
- 实现: `src/inspector/js_native_api_v8_inspector.cpp`
- WebSocket 服务器: `src/inspector/inspector_socket_server.cpp`

**触发条件**: 应用在生产环境中启用 Inspector，未限制访问来源。

**攻击路径**:
```
攻击者 → WebSocket 连接 → Inspector → 读取/修改 JavaScript 代码和状态
```

**影响**:
- 代码注入
- 信息泄露
- 调试会话劫持

**修复建议**:
1. 生产环境禁用 Inspector
2. 限制 Inspector 的监听地址（只监听 localhost）
3. 实施认证机制（TODO: 需确认是否支持）
4. 使用防火墙限制访问

**证据位置**:
- `interface/kits/jsvm.h:1719` - Inspector API
- `src/inspector/inspector_socket_server.cpp` - WebSocket 服务器

---

## 安全控制措施

### 输入校验

**位置**: JSVM-API 层

**措施**:
- 参数 NULL 检查
- 类型检查
- 范围检查

**证据位置**:
- `src/js_native_api_v8.cpp` - 各 API 的参数校验

### 权限控制

**位置**: 应用层

**措施**:
- 限制可访问的 API
- 实施沙箱（TODO: 需确认）
- 文件系统访问控制

**证据位置**: `src/platform/platform_ohos.cpp` - 平台层权限

### 内存安全

**位置**: V8 Engine

**措施**:
- V8 引擎的安全特性
- ASan/HWASan 支持

**证据位置**:
- `BUILD.gn:108-112` - ASAN/HWASAN 支持
- `jsvm.gni:34` - support_hwasan

---

## 检查范围与局限性

### 检查范围

本文档的安全性检查覆盖了以下方面：

✅ 代码审计（JSVM-API 层、Core 层）
✅ 威胁建模（外部输入 → 敏感操作）
✅ 攻击面分析（API、IPC、文件、网络、Inspector）
✅ 可被利用点识别（5 个关键风险）
✅ 修复建议

### 局限性

本文档存在以下局限性：

❌ **未覆盖 V8 Engine 内部安全性**: V8 引擎的安全性分析超出了本文档范围，建议参考 V8 官方安全公告。

❌ **未进行模糊测试（Fuzzing）**: 未对 JSVM 进行系统性的模糊测试，建议后续补充。

❌ **未进行形式化验证**: 未对 JSVM 的关键安全属性进行形式化验证。

❌ **未考虑侧信道攻击**: 未考虑基于时间、缓存的侧信道攻击。

❌ **依赖库安全性未深入分析**: 对第三方依赖（如 llhttp）的安全性分析不深入，建议单独评审。

---

## 相关链接

- [概览](./01_Overview.md) - 项目定位
- [对外 API 详细文档](./05_Public_API_Details.md) - API 安全注意事项
