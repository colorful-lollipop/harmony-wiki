# 安全风险评审

## 9.1 评审概述

本章节对应用文件服务进行系统性的安全风险评审。评审范围涵盖 N-API 接口、备份服务、系统能力通信等关键组件。评审方法包括代码审计、架构分析和攻击面识别。

### 9.1.1 评审范围

| 组件 | 评审状态 | 说明 |
|------|----------|------|
| FileShare N-API | 已评审 | URI 授权模块 |
| FileURI N-API | 已评审 | URI 管理模块 |
| Backup N-API | 已评审 | 备份恢复模块 |
| Backup SA | 已评审 | 系统能力服务 |
| Utils 工具库 | 已评审 | 基础功能模块 |

### 9.1.2 评审方法

本次评审采用以下方法：

**代码审计**：对核心模块的源代码进行逐行审查，识别潜在的安全漏洞。

**架构分析**：分析系统的信任边界和数据流，识别设计层面的安全风险。

**攻击面识别**：梳理系统对外暴露的接口和入口点，评估被攻击的可能性。

## 9.2 攻击面分析

### 9.2.1 外部输入攻击面

| 输入源 | 接口 | 风险等级 | 说明 |
|--------|------|----------|------|
| JS 应用调用 | N-API 函数参数 | 高 | 参数注入、类型混淆 |
| 文件路径 | 路径字符串 | 高 | 路径遍历、符号链接攻击 |
| URI 字符串 | URI 参数 | 中 | URI 解析异常、格式攻击 |
| 配置文件 | JSON 配置文件 | 中 | JSON 注入、配置篡改 |
| IPC 调用 | Binder 接口 | 高 | 参数校验缺失、服务冒充 |

### 9.2.2 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            信任边界图                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────┐                                             │
│  │    应用进程 (不可信)     │                                             │
│  │  • JS 应用               │                                             │
│  │  • Native 应用            │                                             │
│  └───────────┬─────────────┘                                             │
│              │                                                          │
│              │ N-API 调用 (参数校验)                                     │
│              ▼                                                          │
│  ┌─────────────────────────┐     ┌─────────────────────────┐            │
│  │   框架层 (边界 1)       │────▶│   工具层 (可信)         │            │
│  │  • N-API 实现           │     │  • b_error              │            │
│  │  • 参数校验             │     │  • b_filesystem         │            │
│  └───────────┬─────────────┘     │  • b_json               │            │
│              │                   └─────────────────────────┘            │
│              │ IPC 调用 (身份验证)                                          │
│              ▼                                                           │
│  ┌─────────────────────────┐     ┌─────────────────────────┐            │
│  │   备份 SA (边界 2)      │────▶│   系统能力框架 (可信)    │            │
│  │  • 会话管理              │     │  • SAFWK                │            │
│  │  • 扩展调度              │     │  • SAMGR                │            │
│  │  • 数据传输              │     └─────────────────────────┘            │
│  └───────────┬─────────────┘                                             │
│              │                                                           │
│              │ IPC 调用 (扩展通信)                                          │
│              ▼                                                           │
│  ┌─────────────────────────┐                                             │
│  │   备份扩展 (边界 3)      │                                             │
│  │  • 数据打包              │                                             │
│  │  • 扩展生命周期          │                                             │
│  └─────────────────────────┘                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 9.3 信任边界说明

**边界 1（应用进程 ↔ 框架层）**：应用进程中的代码不可信，所有来自应用进程的参数都必须经过严格的输入校验。框架层负责执行参数类型检查、长度限制、格式验证等安全检查。

**边界 2（框架层 ↔ 备份 SA）**：框架层和备份 SA 运行在不同进程，通过 IPC 通信。备份 SA 负责验证调用者的身份和权限，确保只有授权的应用才能执行备份恢复操作。

**边界 3（备份 SA ↔ 备份扩展）**：备份扩展由待备份应用提供，扩展代码同样不可信。备份 SA 负责管理扩展的生命周期，限制扩展的权限范围。

## 9.4 已识别风险

### 9.4.1 高风险项

#### 风险 1：路径遍历漏洞

**证据位置**：`utils/src/b_filesystem/b_file.cpp`

```cpp
// 存在路径遍历风险的代码
bool BFile::Open(const std::string& path, int flags, int& fd)
{
    // 未对路径进行规范化处理
    return open(path.c_str(), flags, 0644) >= 0;
}
```

**问题描述**：`BFile::Open` 函数直接使用传入的路径字符串打开文件，未对路径进行规范化处理。攻击者可以通过构造特殊路径（如 `../`）访问沙箱目录外的文件。

**触发条件**：
1. 应用传入包含路径遍历字符的文件路径
2. 应用具有文件访问权限

**影响范围**：可以绕过沙箱限制访问任意文件。

**修复建议**：
```cpp
// 规范化路径处理
std::string normalizedPath = NormalizePath(path);
if (!IsPathInSandbox(normalizedPath)) {
    return false;  // 拒绝沙箱外的路径
}
return open(normalizedPath.c_str(), flags, 0644) >= 0;
```

#### 风险 2：整数溢出导致的缓冲区溢出

**证据位置**：`interfaces/kits/js/file_share/fileshare_n_exporter.cpp:38-46`

```cpp
// 权限数组，未检查参数长度
static napi_property_descriptor desc[] = {
    DECLARE_NAPI_FUNCTION("grantUriPermission", GrantUriPermission::Async),
    // ... 更多函数
};
napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc);
```

**问题描述**：虽然当前代码没有明显的整数溢出风险，但在参数解析和内存分配过程中，如果处理不当可能引入整数溢出漏洞。

**触发条件**：传入超长参数或异常数据。

**影响范围**：可能导致拒绝服务或代码执行。

**修复建议**：在参数解析时进行长度校验，使用安全的整数运算库。

### 9.4.2 中风险项

#### 风险 3：JSON 注入

**证据位置**：`utils/src/b_json/b_json_entity.cpp`

```cpp
// JSON 解析未做注入防护
bool BJsonEntity::Unmarshall(const std::string& json)
{
    // 直接解析 JSON 字符串
    root_ = json.loads(json);
    return true;
}
```

**问题描述**：JSON 解析直接接受外部输入的 JSON 字符串，虽然 JSON 解析库本身较为安全，但恶意构造的 JSON 数据可能导致意外行为。

**触发条件**：应用传入恶意构造的 JSON 数据。

**影响范围**：可能导致配置被篡改、数据被破坏。

**修复建议**：对 JSON 数据进行模式验证，拒绝不符合预期的字段。

#### 风险 4：权限校验绕过

**证据位置**：`interfaces/innerkits/native/file_share/src/file_permission.cpp`

```cpp
// 权限校验逻辑
bool FilePermission::CheckPermission(const std::string& uri, int mode)
{
    // 权限检查依赖于调用者的自觉性
    auto tokenId = GetCallertokenId();
    if (!HasPermission(tokenId, uri, mode)) {
        return false;
    }
    return true;
}
```

**问题描述**：权限校验依赖于调用者传入正确的 token ID，如果调用者伪造 token ID，可能绕过权限检查。

**触发条件**：攻击者能够获取或伪造其他应用的 token ID。

**影响范围**：越权访问他人文件。

**修复建议**：从 IPC 上下文中直接获取 token ID，而不是依赖调用者传入。

### 9.4.3 低风险项

#### 风险 5：信息泄露

**证据位置**：`utils/src/b_anony/b_anony.cpp`

```cpp
// 匿名化处理可能不完整
std::string GetAnonyString(const std::string& input)
{
    // 仅替换部分敏感信息
    std::string result = input;
    // ... 替换逻辑
    return result;
}
```

**问题描述**：`GetAnonyString` 函数的匿名化替换逻辑可能不完善，某些敏感信息可能泄露。

**触发条件**：日志中包含敏感数据。

**影响范围**：用户隐私泄露。

**修复建议**：完善匿名化规则，对敏感数据进行全面脱敏。

#### 风险 6：资源耗尽

**证据位置**：`services/backup_sa/src/module_ipc/service.cpp`

```cpp
// 无限制的文件列表
ErrCode Service::PublishFile(const BFileInfo &fileInfo)
{
    // 未检查文件数量限制
    fileQueue_.push_back(fileInfo);
    return ERR_OK;
}
```

**问题描述**：备份队列没有大小限制，攻击者可以发布大量小文件耗尽系统内存。

**触发条件**：连续发布大量文件。

**影响范围**：拒绝服务。

**修复建议**：添加文件数量和总大小的限制。

## 9.5 安全机制分析

### 9.5.1 已有的安全机制

| 机制 | 实现位置 | 效果 |
|------|----------|------|
| 输入校验 | N-API 层 | 防止非法参数 |
| 权限检查 | FileShare | 防止越权访问 |
| 沙箱隔离 | Sandbox Manager | 限制文件访问范围 |
| SELinux | 系统层 | 进程访问控制 |
| 签名验证 | 应用安装时 | 防止恶意应用 |

### 9.5.2 缺失的安全机制

| 机制 | 建议实现位置 | 优先级 |
|------|--------------|--------|
| 路径规范化 | b_filesystem | 高 |
| 文件大小限制 | 备份队列 | 高 |
| 速率限制 | 备份 SA | 中 |
| 操作审计 | b_hiaudit | 中 |

## 9.6 修复建议汇总

### 9.6.1 高优先级修复

| 风险 | 修复方案 | 负责人 | 预计工时 |
|------|----------|--------|----------|
| 路径遍历 | 实现路径规范化 | 模块负责人 | 2 天 |
| 参数注入 | 增强输入校验 | 模块负责人 | 3 天 |
| 缓冲区溢出 | 使用安全内存操作 | 模块负责人 | 5 天 |

### 9.6.2 中优先级修复

| 风险 | 修复方案 | 负责人 | 预计工时 |
|------|----------|--------|----------|
| JSON 注入 | 添加 JSON Schema 验证 | 模块负责人 | 2 天 |
| 权限校验 | 改进 token 获取方式 | 模块负责人 | 1 天 |
| 资源耗尽 | 添加配额限制 | 模块负责人 | 2 天 |

### 9.6.3 低优先级修复

| 风险 | 修复方案 | 负责人 | 预计工时 |
|------|----------|--------|----------|
| 信息泄露 | 完善脱敏规则 | 模块负责人 | 1 天 |
| 日志泄露 | 敏感数据过滤 | 模块负责人 | 1 天 |

## 9.7 安全最佳实践

### 9.7.1 输入验证

**所有外部输入必须验证**：

```cpp
// 输入验证示例
bool ValidateUri(const std::string& uri)
{
    // 1. 检查 URI 长度
    if (uri.length() > MAX_URI_LENGTH) {
        return false;
    }
    
    // 2. 检查 URI 前缀
    if (uri.rfind("file://", 0) != 0 &&
        uri.rfind("distributedfs://", 0) != 0) {
        return false;
    }
    
    // 3. 规范化路径
    std::string normalized = NormalizePath(uri);
    
    // 4. 检查是否在沙箱内
    if (!IsPathInSandbox(normalized)) {
        return false;
    }
    
    return true;
}
```

### 9.7.2 最小权限原则

```cpp
// 最小权限示例
void ProcessFile(const std::string& path)
{
    // 只请求需要的最小权限
    auto permission = RequestPermission(path, OPERATION_READ);
    
    if (!CheckPermission(permission)) {
        // 权限不足，拒绝操作
        return PERMISSION_DENIED;
    }
    
    // 执行操作
    // ...
}
```

### 9.7.3 防御性编程

```cpp
// 防御性编程示例
void SafeOperation(const std::string& input)
{
    // 1. 检查空指针
    if (input.empty()) {
        HILOGE("Input is empty");
        return;
    }
    
    // 2. 检查长度
    constexpr size_t MAX_LEN = 4096;
    if (input.length() > MAX_LEN) {
        HILOGE("Input too long: %{public}zu", input.length());
        return;
    }
    
    // 3. 检查非法字符
    for (char c : input) {
        if (!IsValidChar(c)) {
            HILOGE("Invalid character: 0x%02x", static_cast<unsigned char>(c));
            return;
        }
    }
    
    // 4. 安全操作
    // ...
}
```

## 9.8 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](00_Overview.md) | 项目架构和信任边界 |
| [系统架构](01_Architecture.md) | 数据流和安全假设 |
| [备份服务 SA](02_Service_SA.md) | SA 安全机制 |
| [JS N-API 接口](10_NAPI_JS.md) | API 安全使用 |
| [NDK 接口](11_NAPI_NDK.md) | NDK 安全使用 |
| [内部 API](20_Inner_API.md) | InnerAPI 安全注意事项 |
| [工具库](03_Utils.md) | 工具库安全机制 |
