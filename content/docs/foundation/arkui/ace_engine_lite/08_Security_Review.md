# 安全风险评审

## 目的

本文档基于源代码分析 ace_engine_lite 的安全机制、攻击面、信任边界和潜在可利用点，并提供修复建议。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 安全相关代码和配置

---

## 安全机制概述

### 已实现的安全机制

| 安全机制 | 实现程度 | 位置 |
|---------|----------|------|
| 参数校验 | ⚠️ 部分 | 各模块入口 |
| 输入验证 | ⚠️ 部分 | 文件操作、数值范围 |
| 内存管理 | ✅ 良好 | JerryScript 引擎、bounds_checking |
| 错误处理 | ✅ 良好 | try-catch、错误码 |
| 日志记录 | ✅ 良好 | HILOG 系统集成 |
| 沙盒隔离 | ⚠️ 有限 | 应用级隔离（Ability 级别） |

---

## 攻击面分析

### 1. JavaScript 注入攻击

**攻击向量**：恶意 JS 代码注入到应用环境

**风险评估**：⚠️ **中等**

**攻击路径**：
```
恶意应用 → requireNative("malicious.module")
    → ModuleManager::RequireModule()
    → 执行任意模块初始化函数
```

**证据**：
- 模块加载：`frameworks/module_manager/module_manager.cpp:30-66`
- 产品模块支持：`frameworks/module_manager/module_manager.h:74-82`
- 私有模块支持：`frameworks/module_manager/module_manager.h:81-82`

**缓解措施**：
- ✅ 默认禁用 `FEATURE_PRODUCT_MODULE`
- ✅ 私有模块按 bundleName 隔离
- ⚠️ 需要运行时签名验证（未发现）

**修复建议**：
1. 实现模块签名验证
2. 禁用未授权的产品模块加载
3. 实现模块白名单机制

---

### 2. 数据绑定污染

**攻击向量**：通过修改 JS 对象属性影响 UI 渲染

**风险评估**：⚠️ **低-中等**

**攻击路径**：
```
恶意 JS:
  // 修改系统对象
  delete app.data.userId;
  // 覆盖关键属性
  app.data.sensitiveInfo = "malicious";
```

**证据**：
- Watcher 回调：`frameworks/src/core/base/js_fwk_common.h:238-269`
- 无访问控制检查：JSI 层未限制属性修改

**缓解措施**：
- ✅ Watcher 只监听指定属性
- ⚠️ 无法防止原型链污染

**修复建议**：
1. 实现属性访问控制
2. 使用 Object.freeze() 保护关键对象
3. 实现 Proxy 拦截属性访问

---

### 3. XSS（跨站脚本）攻击

**攻击向量**：通过 UI 组件注入恶意脚本

**风险评估**：⚠️ **中等**

**攻击路径**：
```
恶意输入:
  text: "`<script>`alert('XSS')</script>"
```

**证据**：
- 文本组件：`frameworks/src/core/components/text_component.cpp`
- 属性绑定：`frameworks/src/core/base/js_fwk_common.cpp`

**缓解措施**：
- ⚠️ 未发现 HTML 转义机制
- ⚠️ 未发现内容安全策略（CSP）

**修复建议**：
1. 实现 HTML 实体编码
2. 实现文本清理策略
3. 考虑使用安全的 DOM 操作 API

---

### 4. 输入验证绕过 / 路径遍历

**攻击向量**：通过特殊输入绕过参数校验，利用路径遍历访问非授权文件

**风险评估**：⚠️ **中等**

**攻击路径**：
```
恶意输入:
  router.replace({
    uri: "../../../etc/passwd",
    params: {}
  })

  // 或构造恶意文件路径
  image.src = "../../../data/secret.txt"
```

**代码证据**：

**路径构造（存在风险）**：
```cpp
// frameworks/src/core/router/js_page_state_machine.cpp:127-135
err = strcpy_s(jsPagePath_, len, jsPageSpecific.jsIndexFilePath);
err = strcpy_s(jsPagePath_, len, uri);
err = strcat_s(jsPagePath_, len, sourceFileSuffix);
```

**路径规范化（已保护）**：
```cpp
// frameworks/src/core/base/js_fwk_common.cpp:595-615
if (realpath(orgFullPath, fullPath) == nullptr) {
    HILOG_ERROR(HILOG_MODULE_ACE, "get real path failed");
    return -1;
}
```

**URI 提取**：
```cpp
// frameworks/src/core/router/js_page_state_machine.cpp:187-199
void StateMachine::BindUri(jerry_value_t object) {
    jerry_value_t uriKey = jerry_create_string((const jerry_char_t*)("uri"));
    jerry_value_t uriValue = jerry_get_property(object, uriKey);
    // ... 提取 URI 字符串
    uri_ = MallocStringOf(uriValue);  // Line 199: JS 输入转 C 字符串
}
```

**缓解措施**：
- ✅ 关键文件操作使用 `realpath()` 规范化路径
- ✅ 使用 `strcpy_s`/`strcat_s` 等安全函数
- ⚠️ 并非所有路径操作都经过规范化检查
- ⚠️ 未发现显式的路径遍历黑名单检查

**修复建议**：
1. 统一在文件操作前调用 `realpath()` 规范化路径
2. 验证路径是否在应用沙盒内
3. 添加 URI 白名单验证
4. 限制路径长度（当前有部分检查但不完整）

---

### 5. 权限检查缺失

**攻击向量**：未授权访问敏感功能或数据

**风险评估**：⚠️ **中等**

**攻击路径**：
```
恶意应用:
  // 尝试访问需要权限的功能
  featureAbility.subscribeMsg({secret: true})
  
  // 或访问受限 API
  const fs = requireNative("system.file");
  fs.readFile("/etc/passwd")
```

**证据**：
- 未发现权限检查：代码搜索未找到 permission 关键字
- 未发现访问令牌机制：Ability token 未验证使用权限
- 未发现 bundle name 验证：`frameworks/src/core/context/js_app_context.cpp:212-255`

**缓解措施**：
- ⚠️ Ability 框架提供 token 机制（`frameworks/src/core/context/ace_ability.cpp`）
- ⚠️ LiteOS-M 无复杂权限系统

**修复建议**：
1. 实现权限白名单验证
2. 添加 API 级别访问控制
3. 记录和审计敏感操作

---

### 6. 跨设备消息伪造

**攻击向量**：伪造设备间消息欺骗应用

**风险评估**：⚠️ **中等**

**攻击路径**：
```
恶意应用:
  // 模拟其他设备发送消息
  featureAbility.sendMsg({
    deviceId: "victim_device",
    bundleName: "victim_app",
    message: "malicious_data"
  })
```

**证据**：
- Feature Ability：`frameworks/src/core/modules/presets/feature_ability_module.cpp`
- AMS 依赖：`test/ace_test_config.gni:85-87`
- Token 验证：`frameworks/tools/qt/simulator/jsfwk/targets/simulator/mock/amsthread/ams_thread.cpp:42-333`

**缓解措施**：
- ✅ Token 机制（0xff 用于隐藏 JS Ability）
- ⚠️ 消息内容未验证来源
- ⚠️ 设备 ID 未验证

**修复建议**：
1. 实现消息签名验证
2. 验证设备 ID 合法性
3. 添加消息内容安全检查
4. 实现来源认证机制

---

### 7. 内存安全问题

**攻击向量**：缓冲区溢出、释放后使用、双重释放、内存泄漏

**风险评估**：⚠️ **低-中等**

**关键代码位置**：

**自定义内存分配器**：
```cpp
// interfaces/inner_api/builtin/base/ace_mem_base.h:56-73
void *ace_malloc(size_t size);    // 自定义 malloc
void *ace_calloc(size_t nmemb, size_t size);  // 自定义 calloc
void ace_free(void *ptr);         // 自定义 free
```

**JS 字符串转 C 字符串（需调用者释放）**：
```cpp
// frameworks/native_engine/jsi/jsi.cpp:633-667
char *JSI::ValueToString(JSIValue value) {
    jerry_size_t size = jerry_get_string_size(jVal);
    // 分配内存（size + 1）
    jerry_char_t *buffer = static_cast<jerry_char_t *>(
        ace_malloc(sizeof(jerry_char_t) * (size + 1)));
    if (buffer == nullptr) {
        return nullptr;
    }
    jerry_size_t length = jerry_string_to_char_buffer(jVal, buffer, size);
    buffer[length] = '\0';
    return reinterpret_cast<char *>(buffer);  // 调用者必须释放！
}
```

**内存分配点（多处）**：
```cpp
// frameworks/src/core/context/js_app_context.cpp:220-304
// 多处 ace_malloc 用于 ability path, bundle name, JS path, URI
char *abilityPath_ = static_cast<char *>(ace_malloc(len));
char *currentBundleName_ = static_cast<char *>(ace_malloc(len));
```

**JSON 解析内存操作**：
```cpp
// frameworks/src/core/modules/presets/cjson_parser.cpp:57
char *cacheBuffer_ = static_cast<char *>(ace_malloc(LOCALIZATION_SIZE));

// 多处 strcpy_s/strcat_s 操作缓存缓冲区
// Line 252, 259, 459-463, 553-562, 612
```

**缓解措施**：
- ✅ 使用 bounds_checking_function：`bundle.json:45`
- ✅ 使用安全字符串函数（`strcpy_s`, `strcat_s`, `memcpy_s`, `sprintf_s`）
- ✅ 内存管理封装：`frameworks/src/core/base/scope_js_value.h`
- ⚠️ 自定义分配器可能绕过某些保护
- ⚠️ 多处内存分配需要调用者正确释放

**修复建议**：
1. 确保所有内存分配使用安全函数
2. 启用栈保护（-fstack-protector-strong）
3. 启用地址空间布局随机化（ASLR）
4. 使用智能指针/RAII 模式管理内存生命周期
5. 对 `ValueToString` 等返回堆内存的函数添加显式注释

---

### 8. 时序攻击（竞态条件）

**攻击向量**：通过多线程/异步操作导致状态不一致

**风险评估**：⚠️ **低-中等**

**攻击路径**：
```
竞态条件场景:
  // 多个操作同时修改共享状态
  Task 1: 修改 app.data.counter = 1
  Task 2: 读取 app.data.counter  // 可能读到不一致值
```

**证据**：
- 单线程 JS 执行：JerryScript 主线程
- 异步任务：`frameworks/native_engine/async/js_async_work.cpp`
- 共享状态：AppStyleManager, JsAppContext

**缓解措施**：
- ✅ JS 单线程执行（避免 JS 竞态）
- ⚠️ 异步任务可能导致竞态

**修复建议**：
1. 对关键共享状态加锁
2. 实现原子操作
3. 避免在异步回调中修改共享状态

---

### 9. 信息泄露

**攻击向量**：通过错误消息或日志泄露敏感信息

**风险评估**：⚠️ **中等**

**攻击路径**：
```
信息泄露场景:
  // 错误消息包含内部路径
  throw new Error("Failed to open /storage/data/secret/file");
  
  // 日志输出敏感数据
  HILOG_INFO("User token: %s", userToken);
```

**证据**：
- DFX 模块：`frameworks/src/core/modules/dfx_module.cpp`
- 日志使用：`frameworks/src/core/modules/presets/console_log_impl.cpp`

**缓解措施**：
- ✅ 日志等级控制（INFO/WARN/ERROR）
- ⚠️ 可能泄露敏感路径到错误消息

**修复建议**：
1. 实现日志脱敏机制
2. 避免在错误消息中包含完整路径
3. 使用用户友好的通用错误消息

---

### 10. 资源管理漏洞 / JS 代码执行

**攻击向量**：路径遍历、未授权文件访问、资源耗尽、恶意 JS 代码执行

**风险评估**：⚠️ **中高**

**攻击路径**：
```
路径遍历:
  const image = requireNative("system.image");
  image.src = "../../../etc/passwd";

恶意 JS 执行:
  // 通过篡改 JS 文件或快照执行任意代码
  // 如果攻击者能修改应用的 JS 文件或 .bc 快照

资源耗尽:
  // 创建大量组件或定时器
  for (let i = 0; i < 100000; i++) {
    timer.setTimeout(() => {}, 0);
  }
```

**关键代码位置**：

**JS 代码执行入口（最高风险）**：
```cpp
// frameworks/src/core/context/js_app_context.cpp:72-130
jerry_value_t JsAppContext::Eval(char *fullPath, size_t fullPathLength, bool isAppEval) {
    // 1. 读取 JS 文件
    char *jsCode = EvaluateFile(isSnapshotMode, contentLength, fullPath, fullPathLength);
    
    // 2. 执行快照或解析 JS
    if (isSnapshotMode) {
        viewModel = jerry_exec_snapshot(snapshotContent, contentLength, 0, 1);
    } else {
        jerry_value_t retValue = jerry_parse(...);
        viewModel = jerry_run(retValue);
    }
    
    // 3. 渲染视图
    return Render(viewModel);
}
```

**文件读取与校验**：
```cpp
// frameworks/src/core/base/js_fwk_common.cpp:665-714
char *ReadFile(const char *fileFullPath, uint32_t &outLen, bool isSnapshot) {
    // 打开文件
    int32_t fd = open(fileFullPath, O_RDONLY);
    // 读取内容到缓冲区
    int32_t readLength = read(fd, fileContent, fileLength);
}
```

**资源路径处理**：
```cpp
// frameworks/src/core/context/js_app_context.cpp:338-398
bool JsAppContext::GetResourcePath(const char *uri, char *pathBuffer, uint32_t bufferSize) {
    // 检查 internal://app URI 前缀
    if (strncmp(uri, URI_PREFIX_DATA, URI_PREFIX_DATA_LENGTH) == 0) {
        // 构建数据路径
        sprintf_s(dataPath, dataPathSize + 1, "%s/%s", appDataRoot, currentBundleName_);
    }
}
```

**缓解措施**：
- ✅ 使用 `realpath()` 进行路径规范化（关键文件操作）
- ✅ Ability 沙盒（应用级隔离）
- ✅ 文件大小限制（FILE_CONTENT_LENGTH_MAX）
- ⚠️ JS 代码执行无额外的沙盒限制（依赖 JerryScript 引擎）
- ⚠️ 未发现显式的资源使用限制（定时器数量、组件数量）

**修复建议**：
1. 实现 JS 代码签名验证，防止篡改
2. 实现路径解析和规范化（统一使用 realpath）
3. 实现路径白名单，限制可访问的目录
4. 限制单个应用的资源使用量（定时器、组件数量）
5. 监控和限制内存使用

---

## 信任边界

### 应用边界

**隔离级别**：Ability 级（应用级）

**说明**：每个 Ability 运行在独立进程，有独立的内存空间

**证据**：`frameworks/src/core/context/ace_ability.cpp`

### JS 运行时边界

**沙盒机制**：JerryScript 引擎提供基本内存隔离

**说明**：每个应用使用独立的 JS 运行时实例

**证据**：`frameworks/src/core/context/js_app_context.cpp`

### 数据边界

**模块隔离**：JS 模块按需加载，不同应用可访问不同模块

**说明**：通过 ModuleManager 按需加载模块，应用无法访问其他应用的私有模块

**证据**：`frameworks/module_manager/module_manager.cpp`

---

## 可利用点总结

| 风险 | 严重程度 | 可利用可能性 | 状态 |
|------|----------|------------|------|
| JS 模块注入 | 中 | 中 | ⚠️ 部分缓解 |
| 数据绑定污染 | 低-中 | 低 | ⚠️ 需要加强 |
| XSS 攻击 | 中 | 中 | ⚠️ 需要修复 |
| 输入验证绕过 | 低-中 | 低 | ⚠️ 部分缓解 |
| 权限检查缺失 | 中 | 中 | ⚠️ 需要补充 |
| 跨设备消息伪造 | 中 | 中 | ⚠️ 部分缓解 |
| 内存安全问题 | 低 | 低 | ✅ 基本缓解 |
| 时序攻击 | 低-中 | 低 | ⚠️ 需要注意 |
| 信息泄露 | 中 | 低 | ⚠️ 需要注意 |
| 资源管理漏洞 | 中 | 中 | ⚠️ 需要加强 |

---

## 安全建议优先级

### 高优先级（必须修复）

1. **实现完整的输入验证机制**
   - 路径遍历检查
   - 参数范围验证
   - URI 格式验证

2. **实现模块签名和验证**
   - 验证模块来源
   - 防止加载未授权模块

3. **实现 HTML 转义机制**
   - 防止 XSS 攻击
   - 清理用户输入

### 中优先级（建议修复）

1. **加强消息验证**
   - 验证设备 ID
   - 签名消息内容
   - 来源认证

2. **实现日志脱敏**
   - 避免泄露敏感路径
   - 用户友好的错误消息

3. **实现资源限制**
   - 定时器数量限制
   - 组件数量限制
   - 内存使用监控

### 低优先级（长期改进）

1. **实现权限系统**
   - API 级别控制
   - 白名单验证

2. **实现更强的沙盒隔离**
   - 进程级隔离
   - 系统调用过滤

3. **实现安全审计机制**
   - 安全事件日志
   - 异常检测和告警

---

## 代码证据清单

### 关键安全文件

| 功能 | 文件 | 行号 | 风险等级 |
|------|------|------|----------|
| **JS 代码执行入口** | `frameworks/src/core/context/js_app_context.cpp` | 72-130 | 🔴 高 |
| **文件读取** | `frameworks/src/core/base/js_fwk_common.cpp` | 665-714 | 🔴 高 |
| **路径规范化** | `frameworks/src/core/base/js_fwk_common.cpp` | 595-615 | 🟡 中 |
| **路径构造（有风险）** | `frameworks/src/core/router/js_page_state_machine.cpp` | 127-135 | 🟡 中 |
| **URI 提取** | `frameworks/src/core/router/js_page_state_machine.cpp` | 187-199 | 🟡 中 |
| **模块加载** | `frameworks/module_manager/module_manager.cpp` | 30-66 | 🟡 中 |
| **字符串转 C 字符串** | `frameworks/native_engine/jsi/jsi.cpp` | 633-667 | 🟡 中 |
| **内存分配** | `frameworks/src/core/context/js_app_context.cpp` | 220-304 | 🟢 低 |
| **JSON 解析内存操作** | `frameworks/src/core/modules/presets/cjson_parser.cpp` | 57, 252, 259, 459-463, 553-562, 612 | 🟢 低 |
| **IPC 跨 Ability 通信** | `frameworks/src/core/modules/presets/feature_ability_module.cpp` | 120-493 | 🟡 中 |
| **自定义内存分配器** | `interfaces/inner_api/builtin/base/ace_mem_base.h` | 56-73 | 🟢 低 |
| **安全字符串函数** | 多处使用 | - | 🟢 低 |

### 安全函数使用统计

| 安全函数 | 使用位置 | 说明 |
|----------|----------|------|
| `strcpy_s` | `js_page_state_machine.cpp:127-135`, `cjson_parser.cpp:252, 459` | 安全字符串拷贝 |
| `strcat_s` | `js_page_state_machine.cpp:135`, `cjson_parser.cpp:259, 462` | 安全字符串拼接 |
| `sprintf_s` | `js_app_context.cpp:391`, `cjson_parser.cpp:76`, `mem_proc.cpp:57` | 安全格式化 |
| `memcpy_s` | `js_app_context.cpp:225`, `module_manager.cpp:126`, `feature_ability_module.cpp:324` | 安全内存拷贝 |
| `realpath` | `js_fwk_common.cpp:599` | 路径规范化 |
| `ace_malloc/ace_free` | 多处 | 自定义内存管理 |

### 信任边界检查点

| 边界 | 检查位置 | 检查机制 |
|------|----------|----------|
| JS → Native | `jsi.cpp:633-667` | JerryScript API 转换 |
| 文件访问 | `js_fwk_common.cpp:595-615` | realpath 规范化 |
| 模块加载 | `module_manager.cpp:30-66` | 模块名解析 + 白名单 |
| IPC 通信 | `feature_ability_module.cpp:170-490` | Token + 数据拷贝 |
| 内存分配 | `ace_mem_base.h:56-73` | 自定义分配器 |

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 概览
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [04_JS_API.md](04_JS_API.md) - JS API 参考
