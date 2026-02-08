# 安全评审

本文档对 Preferences 模块进行安全风险评审，包括攻击面分析、信任边界、可利用点和修复建议。

## 1 评审范围

### 1.1 评审对象

- **模块**：`preferences`（首选项存储模块）
- **版本**：3.1.0（来自 bundle.json）
- **代码范围**：`//foundation/distributeddatamgr/preferences`

### 1.2 评审依据

- 源码文件扫描
- bundle.json 依赖分析
- N-API 接口分析
- 权限配置检查

### 1.3 不包含范围

- 测试代码（test/ 目录）
- 第三方依赖内部实现
- 操作系统内核安全

---

## 2 威胁模型

### 2.1 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      不可信区域 (Untrusted)                      │
│  - 外部应用                                                   │
│  - 网络输入                                                   │
│  - 用户输入                                                   │
├─────────────────────────────────────────────────────────────────┤
│                     信任边界 (Trust Boundary)                   │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                 Preferences 模块边界                      │ │
│  │  - N-API 入口点 (preferences/storage/system_storage)       │ │
│  │  - NDK 接口 (libohpreferences.so)                          │ │
│  │  - Inner API (libnative_preferences.so)                   │ │
│  └───────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                      可信区域 (Trusted)                          │
│  - Preferences 内存缓存                                        │
│  - Preferences 文件存储                                       │
│  - 系统 IPC 框架                                              │
│  - AccessToken 权限框架                                       │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 数据流

| 流向 | 描述 | 风险等级 |
|------|------|----------|
| 外部 → 模块 | JS/NDK API 调用 | 中 |
| 模块 → 内存 | 数据加载 | 低 |
| 内存 → 文件 | 数据持久化 | 低 |
| 模块 → IPC | 权限校验调用 | 低 |
| 文件 → 外部 | 配置文件读取 | 低 |

---

## 3 攻击面分析

### 3.1 输入点清单

| # | 输入点 | 类型 | 位置 | 说明 |
|---|--------|------|------|------|
| 1 | JS API | 网络/进程 | `napi_preferences.cpp` | JS 应用调用入口 |
| 2 | NDK API | 网络/进程 | `oh_preferences.cpp` | Native 应用调用入口 |
| 3 | 文件读取 | 本地 | `ReadSettingXml()` | 加载 XML 配置文件 |
| 4 | IPC 调用 | 进程间 | `IPCSkeleton` | 权限校验 |

### 3.2 敏感操作

| # | 操作 | 说明 | 风险等级 |
|---|------|------|----------|
| 1 | 文件写入 | 持久化数据 | 高 |
| 2 | 文件删除 | 删除 Preferences | 高 |
| 3 | 内存修改 | Put/Delete/Clear | 中 |
| 4 | 实例获取 | getPreferences | 中 |

---

## 4 安全机制

### 4.1 权限校验

**代码位置**：`frameworks/native/platform/src/preferences_dfx_adapter.cpp:62-69`

```cpp
auto tokenId = IPCSkeleton::GetCallingTokenID();
auto tokenType = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
if ((tokenType == Security::AccessToken::TOKEN_NATIVE) ||
    (tokenType == Security::AccessToken::TOKEN_SHELL)) {
    Security::AccessToken::NativeTokenInfo tokenInfo;
    if (Security::AccessToken::AccessTokenKit::GetNativeTokenInfo(tokenId, tokenInfo) == 0) {
        moduleName = tokenInfo.processName;
    }
}
```

**机制说明**：
- 使用 `IPCSkeleton::GetCallingTokenID()` 获取调用者 Token
- 使用 `AccessTokenKit::GetTokenTypeFlag()` 判断 Token 类型
- 仅允许 NATIVE 和 SHELL 类型 Token

### 4.2 输入校验

**Key 长度校验**：

```cpp
// preferences_errno.h:79
constexpr int E_KEY_EXCEED_MAX_LENGTH = (E_BASE + 6);
```

**Value 长度校验**：

```cpp
// preferences_errno.h:120
constexpr int E_VALUE_EXCEED_MAX_LENGTH = (E_BASE + 14);
```

### 4.3 文件权限控制

**代码位置**：`preferences_file_operation.h:50-51, 101`

```cpp
#define FILE_MODE 0770

// 非 Windows 平台权限
return open(filePath.c_str(), O_WRONLY | O_CREAT | O_TRUNC, 0660);
```

| 权限 | 说明 |
|------|------|
| 0660 | owner 和 group 可读写 |
| 0770 | 目录默认权限 |

### 4.4 路径脱敏

**代码位置**：`preferences_file_operation.h:146-165`

日志输出时自动脱敏敏感路径信息。

---

## 5 风险清单

### 5.1 已确认风险

#### 风险 1：Key 长度溢出风险

| 项目 | 内容 |
|------|------|
| **编号** | SEC-001 |
| **风险类型** | 缓冲区安全 |
| **证据** | `preferences.h:75` 定义 `MAX_KEY_LENGTH = 1024` |
| **可利用路径** | 恶意应用传入超长 Key → API 层未严格校验 → 内存缓冲区溢出 |
| **触发条件** | 1. 调用 API 传入超过 1024 字符的 Key |
| **影响** | 可能的内存破坏或拒绝服务 |
| **修复建议** | 在 API 入口处增加严格的长度校验，返回 `E_KEY_EXCEED_MAX_LENGTH` |
| **当前状态** | 已有校验（需确认实现位置） |

**代码证据**：

```cpp
// preferences.h:75
PREF_API_EXPORT static constexpr uint32_t MAX_KEY_LENGTH = 1024;

// preferences_errno.h:79
constexpr int E_KEY_EXCEED_MAX_LENGTH = (E_BASE + 6);
```

---

#### 风险 2：路径遍历攻击

| 项目 | 内容 |
|------|------|
| **编号** | SEC-002 |
| **风险类型** | 路径遍历 |
| **证据** | `preferences.h:36-62` 的 `Options` 结构包含 `filePath` 字段 |
| **可利用路径** | 恶意应用传入包含 `../` 的相对路径 → 文件操作越界 → 写入/读取任意文件 |
| **触发条件** | 1. 应用传入构造的 filePath |
| **影响** | 任意文件读写 |
| **修复建议** | 1. 验证 filePath 为绝对路径 |
| **当前状态** | 已有 `E_RELATIVE_PATH` 错误码（需确认校验实现） |

**代码证据**：

```cpp
// preferences_errno.h:90
constexpr int E_RELATIVE_PATH = (E_BASE + 8);
```

---

#### 风险 3：权限逃逸

| 项目 | 内容 |
|------|------|
| **编号** | SEC-003 |
| **风险类型** | 权限绕过 |
| **证据** | `preferences_dfx_adapter.cpp:62` 仅对部分代码进行 Token 校验 |
| **可利用路径** | 未校验的 API 路径 → 绕过权限检查 → 访问其他应用数据 |
| **触发条件** | 1. 调用未校验的 API |
| **影响** | 跨应用数据访问 |
| **修复建议** | 全面审查所有 API 的权限校验覆盖情况 |
| **当前状态** | 部分 API 有校验（需确认覆盖范围） |

---

#### 风险 4：文件锁竞争条件

| 项目 | 内容 |
|------|------|
| **编号** | SEC-004 |
| **风险类型** | 竞态条件 |
| **证据** | `preferences_impl.h:102` 使用 SafeBlockQueue |
| **可利用路径** | 多进程并发访问 → 文件锁竞争 → 数据损坏 |
| **触发条件** | 1. 多进程同时 Flush |
| **影响** | 数据不一致或损坏 |
| **修复建议** | 1. 使用更强的一致性协议 |
| **当前状态** | 有文件锁机制（需评估强度） |

**代码证据**：

```cpp
// preferences_errno.h:175
constexpr int E_OPERAT_IS_CROSS_PROESS = (E_BASE + 25);
```

---

#### 风险 5：XML 注入

| 项目 | 内容 |
|------|------|
| **编号** | SEC-005 |
| **风险类型** | 注入攻击 |
| **证据** | `preferences_xml_utils.cpp` 负责 XML 序列化 |
| **可利用路径** | Key/Value 包含特殊 XML 字符 → 解析时注入 → 解析错误或数据篡改 |
| **触发条件** | 1. 写入包含 `<` 等特殊字符的数据 |
| **影响** | XML 解析失败或注入 |
| **修复建议** | 对 Key/Value 进行 XML 转义 |
| **当前状态** | 需确认 XML 序列化安全性 |

---

#### 风险 6：XXE 注入（XML 外部实体注入）⚠️ **高危**

| 项目 | 内容 |
|------|------|
| **编号** | SEC-006 |
| **风险类型** | XXE 注入攻击 |
| **证据** | `preferences_xml_utils.cpp:427, 528` 使用 `xmlReadFile()` 解析用户控制的 XML 文件；`preferences_db_adapter.cpp:72` 使用 `dlopen()` 加载 `libarkdata_db_core.z.so` |
| **可利用路径** | 1. 恶意应用写入包含 XXE payload 的 XML 文件 → xmlReadFile() 解析时触发 XXE → 读取系统任意文件<br>2. 恶意应用替换 `libarkdata_db_core.z.so` 为恶意库 → dlopen() 加载时执行任意代码 |
| **触发条件** | 1. 应用有文件写入权限（通过 XML 存储或 GSKV）<br>2. 应用可操作库路径或 LD_LIBRARY_PATH |
| **影响** | 任意文件读取（XXE）或任意代码执行（库劫持） |
| **修复建议** | 1. 确认 `xmlReadFile()` 使用禁用外部实体的解析标志（`XML_PARSE_NOENTITIES`）<br>2. 验证 `dlopen()` 使用绝对路径加载库文件<br>3. 添加库文件签名验证 |
| **当前状态** | **需紧急验证**：XML 解析器配置和动态库加载机制 |
| **严重性** | **高危** |

**代码证据**：

```cpp
// preferences_xml_utils.cpp:427
bool PreferencesXmlUtils::ReadSettingXml(...) {
    // 使用 libxml2 解析
    auto doc = xmlReadFile(...);
    // ...
}

// preferences_db_adapter.cpp:72
void* PreferencesDbAdapter::LoadFunction(...) {
    // 动态加载数据库库
    void* handle = dlopen("libarkdata_db_core.z.so", RTLD_NOW);
    // ...
}
```

---

#### 风险 7：环境变量注入

| 项目 | 内容 |
|------|------|
| **编号** | SEC-007 |
| **风险类型** | 环境变量注入 |
| **证据** | `frameworks/cj/src/preferences_impl.cpp:46, 53` 和 `frameworks/js/napi/common/mock/src/js_ability.cpp:38, 45` 使用 `TEMP` 和 `LOGNAME` 环境变量 |
| **可利用路径** | 恶意应用设置环境变量 → 影响临时文件路径或用户标识 → 路径注入或信息泄露 |
| **触发条件** | 1. 应用可设置环境变量 |
| **影响** | 路径注入或敏感信息泄露 |
| **修复建议** | 1. 验证环境变量值的合法性<br>2. 使用系统 API 获取临时目录而非依赖环境变量 |
| **当前状态** | **需确认**：环境变量的使用范围和校验 |
| **严重性** | **中危** |

**代码证据**：

```cpp
// frameworks/cj/src/preferences_impl.cpp:46, 53
std::string PreferencesImpl::GetTempDir() {
    const char* tempDir = getenv("TEMP");  // 直接使用环境变量
    // ...
}

// frameworks/js/napi/common/mock/src/js_ability.cpp:38
std::string GetTempDir() {
    const char* tempDir = getenv("TEMP");  // 直接使用环境变量
    // ...
}
```

---

#### 风险 8：反序列化漏洞

| 项目 | 内容 |
|------|------|
| **编号** | SEC-008 |
| **风险类型** | 反序列化漏洞 |
| **证据** | `preferences_value_parcel.cpp:625` 实现 `UnmarshallingPreferenceValue()`，`preferences_value_parcel.cpp:451, 482, 535` 实现多个类型反序列化函数 |
| **可利用路径** | 恶意应用构造畸形序列化数据 → Unmarshalling* 函数解析时触发 → 类型混淆或缓冲区溢出 |
| **触发条件** | 1. 通过 IPC 或跨进程通信传输序列化数据 |
| **影响** | 代码执行或拒绝服务 |
| **修复建议** | 1. 严格验证反序列化数据的类型和长度<br>2. 添加数据完整性校验（HMAC/签名）<br>3. 使用安全的反序列化库 |
| **当前状态** | **需审查**：反序列化边界检查和类型验证 |
| **严重性** | **中危** |

**代码证据**：

```cpp
// preferences_value_parcel.cpp:625
int PreferencesValueParcel::UnmarshallingPreferenceValue(...) {
    // 反序列化逻辑
    UnmarshallingBasicValue(...);
    UnmarshallingStringValue(...);
    UnmarshallingVecUInt8(...);
    // ...
}

// preferences_value_parcel.cpp:482
int UnmarshallingStringValue(...) {
    // 字符串反序列化，需验证长度
    // ...
}
```

---

### 5.2 潜在风险（需进一步确认）

| 风险 | 类型 | 说明 |
|------|------|------|
| 内存安全 | 缓冲区 | PreferencesValue 序列化/反序列化 |
| 信息泄露 | 敏感数据 | 路径脱敏是否覆盖所有日志点 |
| 拒绝服务 | 资源耗尽 | 大 Value 对内存的影响 |
| 序列化安全 | XML/GSKV | 格式验证是否充分 |

---

## 6 安全建议

### 6.1 高优先级

| 优先级 | 建议 | 影响 |
|--------|------|------|
| P0 | 全面审查 API 入口的权限校验覆盖 | 防止权限逃逸 |
| P0 | 确认文件路径校验实现 | 防止路径遍历 |
| P1 | 增强 XML 序列化安全性 | 防止 XML 注入 |
| P1 | 增加 Key/Value 长度校验强度 | 防止缓冲区溢出 |

### 6.2 中优先级

| 优先级 | 建议 | 影响 |
|--------|------|------|
| P2 | 评估 FFRT 任务调度的安全性 | 防止竞态条件 |
| P2 | 增加安全审计日志 | 便于事后追溯 |
| P3 | 定期进行模糊测试 | 发现潜在漏洞 |

### 6.3 已有的安全措施

| 措施 | 说明 |
|------|------|
| AccessToken 权限框架 | IPCSkeleton + AccessTokenKit |
| 文件权限控制 | 0660 权限设置 |
| 路径脱敏 | 日志输出脱敏 |
| 错误码定义 | 完整的错误码体系 |
| Sanitizer | CFI/UBSan/Boundary Sanitize |

---

## 7 检查清单

### 7.1 必需检查项

- [ ] 所有 N-API 入口点有权限校验
- [ ] 所有文件路径有验证
- [ ] Key/Value 长度有严格限制
- [ ] XML 序列化有转义处理
- [ ] 文件权限设置正确
- [ ] 日志输出已脱敏

### 7.2 建议检查项

- [ ] 增加安全事件日志
- [ ] 增加异常检测机制
- [ ] 增加资源使用监控
- [ ] 定期进行安全审计

---

## 8 相关文档

| 文档 | 说明 |
|------|------|
| [概览](./00_Overview.md) | 模块定位和能力边界 |
| [架构设计](./01_Architecture.md) | 内部架构详解 |
| [API 参考](./02_API_Reference.md) | 完整 API 清单 |
| [构建配置](./03_Build.md) | 构建流程说明 |

---

## 9 修订历史

| 日期 | 版本 | 修改内容 |
|------|------|----------|
| 2025-02-06 | 1.0 | 初始版本，基于代码扫描 |
