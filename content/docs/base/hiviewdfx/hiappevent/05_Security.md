# 安全风险评审

## 5.1 威胁模型概述

HiAppEvent 作为 OpenHarmony 系统中的应用事件服务组件，承担着应用行为数据采集、存储和分发的重要职责。理解组件的安全边界和潜在风险对于保障系统整体安全至关重要。本章基于代码审计结果，对 HiAppEvent 进行全面的安全风险评估，识别潜在的攻击面和可被利用点，并提供相应的修复建议。

### 5.1.1 组件信任边界

**信任边界划分**：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              信任边界                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐          ┌─────────────────┐          ┌─────────────┐  │
│  │  可信区域（内核） │          │  半可信区域（系统）│          │ 不可信区域 │ │
│  ├─────────────────┤          ├─────────────────┤          ├─────────────┤  │
│  │  - Linux Kernel │          │  - HiAppEvent    │          │  - 应用进程  │ │
│  │  - HAL          │          │  - 系统服务      │          │  - 第三方    │ │
│  │  - 驱动         │          │  - 系统库        │          │  - 网络      │ │
│  └─────────────────┘          └─────────────────┘          └─────────────┘  │
│                                                                              │
│        ▲                              ▲                           ▲          │
│        │                              │                           │          │
│        │                              │                           │          │
│  ┌────┴────┐                   ┌─────┴─────┐              ┌────┴────┐    │
│  │ 系统调用 │                   │ IPC 调用  │              │  API 调用│    │
│  └─────────┘                   └───────────┘              └─────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**区域说明**：

**可信区域（内核态）**：包括 Linux 内核、硬件抽象层、系统驱动等。这一区域具有最高权限，可以直接访问硬件和修改系统内存。HiAppEvent 与这一区域的交互主要通过系统调用（syscall）实现。

**半可信区域（系统态）**：包括 HiAppEvent 本身、系统服务、系统库等。这一区域运行在用户态，但具有较高的系统权限，可以访问系统资源和管理数据。HiAppEvent 的核心逻辑运行在这一区域。

**不可信区域（应用态）**：包括应用进程、第三方代码、网络数据等。这一区域是潜在的攻击来源，所有来自这一区域的输入都需要进行严格的校验。

### 5.1.2 数据流分析

**事件数据流**：

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  应用进程     │ ───▶ │  N-API 接口   │ ───▶ │  事件校验    │ ───▶ │  事件存储    │
│              │      │              │      │              │      │              │
│  - JS 代码   │      │  - 参数解析   │      │  - 白名单    │      │  - 文件      │
│  - Native    │      │  - 类型转换   │      │  - 长度检查  │      │  - 数据库    │
│  - 攻击面    │      │  - 攻击面    │      │  - 攻击面    │      │              │
└──────────────┘      └──────────────┘      └──────────────┘      └──────────────┘
```

## 5.2 攻击面识别

### 5.2.1 N-API 接口攻击面

**攻击面说明**：N-API 接口是应用层与 HiAppEvent 交互的主要通道，也是最主要的外部数据输入点。所有通过 N-API 传入的数据都可能成为攻击向量。

**入口函数**：`frameworks/js/napi/src/napi_hiappevent_js.cpp`

**相关函数**：

| 函数名 | 行号 | 攻击面描述 |
|-------|------|-----------|
| Write() | 33 | 接收事件名称、类型、参数对象 |
| Configure() | 71 | 接收配置对象 |
| InitNapiClass() | 120 | 初始化常量类 |

### 5.2.2 NDK 接口攻击面

**攻击面说明**：NDK 接口供 C/C++ 原生应用直接调用，是高性能场景的入口。由于接口直接暴露给应用层，存在被恶意应用利用的风险。

**入口文件**：`frameworks/native/ndk/hiappevent_ndk.c`

### 5.2.3 配置攻击面

**攻击面说明**：配置文件和运行时配置参数可以被恶意修改或注入恶意配置。

**相关文件**：

| 配置文件 | 路径 | 风险等级 |
|---------|------|---------|
| hiappevent.cfg | /data/service/el2/100/hiappevent/ | 中 |
| 配置参数 | 系统属性 | 中 |

### 5.2.4 存储攻击面

**攻击面说明**：本地存储的事件数据可能被篡改或窃取。

| 存储类型 | 路径 | 保护措施 |
|---------|------|---------|
| 事件日志 | /data/log/hiappevent/ | 权限控制 |
| 缓存数据 | /data/storage/hiappevent/ | 权限控制 |
| 数据库 | - | SQLite 加密 |

## 5.3 安全风险分析

基于代码审计结果，本节列出 HiAppEvent 组件中发现的安全风险点。每条风险均包含：证据（代码路径和符号）、触发条件、潜在影响和修复建议。

### 5.3.1 风险一：事件名称校验不严格

**风险等级**：中

**证据位置**：`frameworks/js/napi/src/napi_hiappevent_js.cpp:33-69`

**代码描述**：Write 函数在解析事件名称时，未对事件名称的格式进行严格校验。虽然存在长度检查（最大 128 字符），但未校验特殊字符和路径遍历字符。

**触发条件**：
```
应用调用 write() 接口时传入包含特殊字符的事件名称
```

**潜在影响**：
- 如果事件名称被用于构造文件路径，可能导致路径遍历攻击
- 包含控制字符的事件名称可能导致日志解析错误
- 可能用于构造注入攻击

**修复建议**：
```cpp
// 建议在事件校验模块中添加事件名称格式校验
bool ValidateEventName(const char* name) {
    // 检查空指针
    if (name == nullptr) {
        return false;
    }
    
    // 检查长度
    size_t len = strlen(name);
    if (len == 0 || len > 128) {
        return false;
    }
    
    // 检查非法字符
    const char* invalidChars = "\n\r\t/\\..;$'\"`";
    for (size_t i = 0; i < strlen(invalidChars); i++) {
        if (strchr(name, invalidChars[i]) != nullptr) {
            return false;
        }
    }
    
    return true;
}
```

### 5.3.2 风险二：JSON 解析潜在拒绝服务

**风险等级**：中

**证据位置**：`frameworks/native/libhiappevent/utility/event_json_util.cpp`

**代码描述**：JSON 解析器在处理超大规模或嵌套过深的 JSON 对象时，可能导致栈溢出或内存耗尽。

**触发条件**：
```
应用调用 write() 接口时传入超大或深度嵌套的 keyValues 对象
```

**潜在影响**：
- 解析器崩溃导致应用闪退
- 内存耗尽导致系统不稳定
- 拒绝服务攻击

**修复建议**：
```cpp
// 建议添加 JSON 解析深度和大小限制
class EventJsonUtil {
public:
    static constexpr size_t MAX_JSON_SIZE = 64 * 1024;  // 64KB
    static constexpr size_t MAX_DEPTH = 16;
    
    static bool ParseWithLimit(const char* jsonStr, AppEventPack& eventPack) {
        // 检查 JSON 大小
        size_t len = strlen(jsonStr);
        if (len > MAX_JSON_SIZE) {
            HILOG_ERROR(LOG_CORE, "JSON size exceeds limit: %zu", len);
            return false;
        }
        
        // 使用流式解析器处理超大 JSON
        JSONCPP_STREAM_解析器 parser;
        // 设置深度限制
        parser.SetMaxDepth(MAX_DEPTH);
        
        return parser.Parse(jsonStr) && ValidateParsed(eventPack);
    }
};
```

### 5.3.3 风险三：Watcher 回调缺少认证

**风险等级**：低

**证据位置**：`frameworks/native/libhiappevent/observer/app_event_watcher.cpp`

**代码描述**：事件观察者的回调函数在触发时，未验证回调注册者的身份。恶意应用可能注册 Watcher 监听其他应用的事件。

**触发条件**：
```
恶意应用调用 OH_HiAppEvent_AddWatcher() 注册事件观察者
```

**潜在影响**：
- 敏感事件数据被窃取
- 用户隐私泄露
- 违反应用隔离原则

**修复建议**：
```cpp
// 建议在添加 Watcher 时进行权限校验
int OH_HiAppEvent_AddWatcher(HiAppEvent_Watcher* watcher) {
    // 获取调用者 UID
    uid_t callerUid = IPCSkeleton::GetCallingUid();
    
    // 检查是否具有观察其他应用事件的权限
    if (!HasPermission(callerUid, "hiappevent.WATCH_ALL_EVENTS")) {
        // 仅允许观察本应用产生的事件
        watcher->SetOwningUid(callerUid);
    }
    
    return OBSERVER_MGR.AddWatcher(watcher);
}
```

### 5.3.4 风险四：配置文件注入风险

**风险等级**：低

**证据位置**：`frameworks/native/libhiappevent/dfr/event_config_mgr.cpp`

**代码描述**：配置文件解析器在加载配置文件时，未进行路径规范化和权限校验，可能导致配置注入攻击。

**触发条件**：
```
攻击者能够修改应用的配置文件目录
```

**潜在影响**：
- 恶意配置被加载
- 打点功能被禁用
- 安全监控失效

**修复建议**：
```cpp
bool EventConfigMgr::LoadConfig(const std::string& configPath) {
    // 路径规范化
    std::string normalizedPath = NormalizePath(configPath);
    
    // 验证路径在允许的目录范围内
    std::string configDir = GetConfigBaseDir();
    if (!normalizedPath.starts_with(configDir)) {
        HILOG_ERROR(LOG_CORE, "Config path out of allowed range");
        return false;
    }
    
    // 验证文件权限
    if (!VerifyFileOwner(normalizedPath)) {
        HILOG_ERROR(LOG_CORE, "Config file permission check failed");
        return false;
    }
    
    // 验证签名（如果签名机制可用）
    if (!VerifyConfigSignature(normalizedPath)) {
        HILOG_ERROR(LOG_CORE, "Config signature verification failed");
        return false;
    }
    
    return LoadAndParseConfig(normalizedPath);
}
```

### 5.3.5 风险五：本地文件敏感信息泄露

**风险等级**：低

**证据位置**：`frameworks/native/libhiappevent/cache/app_event_dao.cpp`

**代码描述**：事件日志文件使用明文存储，包含敏感参数值（如用户 ID、认证令牌等）的日志可能被未授权访问。

**触发条件**：
```
事件参数中包含敏感信息，且日志文件被未授权读取
```

**潜在影响**：
- 用户隐私泄露
- 敏感凭证泄露
- 身份信息暴露

**修复建议**：
```cpp
// 建议对敏感参数值进行脱敏处理
bool AppEventDao::SaveEvent(const AppEventPack& event) {
    AppEventPack sanitizedEvent = event;
    
    // 敏感参数名列表
    static const std::vector<std::string> SENSITIVE_PARAMS = {
        "password", "token", "secret", "credential",
        "user_id", "id_card", "phone", "email"
    };
    
    // 对敏感参数值进行脱敏
    for (const auto& paramName : SENSITIVE_PARAMS) {
        if (sanitizedEvent.HasParam(paramName)) {
            std::string value = sanitizedEvent.GetParamValue(paramName);
            sanitizedEvent.SetParamValue(paramName, MaskValue(value));
        }
    }
    
    // 加密存储（可选）
    std::string encryptedData = EncryptEventData(sanitizedEvent);
    
    return WriteToStorage(encryptedData);
}
```

### 5.3.6 风险六：内存安全问题

**风险等级**：低

**证据位置**：`frameworks/native/libhiappevent/hiappevent_c.cpp`

**代码描述**：C 接口中使用裸指针和手动内存管理，存在潜在的内存泄漏和悬空指针风险。

**触发条件**：
```
应用错误使用 C 接口（如未正确释放 ParamList）
```

**潜在影响**：
- 内存泄漏
- 悬空指针访问
- 进程崩溃

**修复建议**：
```cpp
// 建议使用智能指针封装内存管理
class ParamListImpl {
public:
    using Ptr = std::unique_ptr<ParamListImpl, ParamListDeleter>;
    
    static Ptr Create() {
        return Ptr(new ParamListImpl());
    }
    
    // 禁止拷贝
    ParamListImpl(const ParamListImpl&) = delete;
    ParamListImpl& operator=(const ParamListImpl&) = delete;
    
private:
    ParamListImpl() = default;
    
    std::vector<ParamNode> params_;
};

struct ParamListDeleter {
    void operator()(ParamListImpl* p) {
        if (p) {
            // 清理所有参数节点
            delete p;
        }
    }
};
```

## 5.4 安全设计建议

### 5.4.1 输入校验增强

**建议一**：建立统一的参数校验框架，在 N-API 和 NDK 入口处统一进行参数校验。

**建议二**：支持白名单模式，仅允许预定义的事件名称和白名单参数。

**建议三**：对参数值进行长度、类型、格式的全面校验。

### 5.4.3 访问控制增强

**建议一**：实现基于 UID/GID 的访问控制，确保应用只能访问自己的事件数据。

**建议二**：为 Watcher 功能添加权限声明和运行时校验。

**建议三**：对配置文件添加签名验证机制。

### 5.4.4 数据保护增强

**建议一**：对包含敏感信息的事件参数提供自动脱敏选项。

**建议二**：支持事件数据的加密存储。

**建议三**：实现存储配额监控，防止存储耗尽攻击。

### 5.4.5 内存安全增强

**建议一**：将 C 接口迁移为 C++ 风格，使用 RAII 和智能指针。

**建议二**：引入 AddressSanitizer 和 MemorySanitizer 进行内存安全检测。

**建议三**：定期进行代码审计，关注内存安全问题。

## 5.5 安全检查清单

以下清单可用于 HiAppEvent 组件的安全评估：

| 检查项 | 检查方法 | 预期结果 |
|-------|---------|---------|
| 事件名称校验 | 代码审计 | 存在完整的长度、格式校验 |
| 参数值校验 | 代码审计 | 存在长度、类型、范围校验 |
| 配置文件安全 | 代码审计+配置检查 | 配置文件有权限保护 |
| Watcher 权限 | 代码审计 | Watcher 添加有权限校验 |
| 敏感数据脱敏 | 代码审计 | 敏感参数有脱敏机制 |
| 内存安全 | 静态分析+动态测试 | 无内存泄漏和悬空指针 |
| 拒绝服务防护 | 压力测试 | 能够抵御超大输入攻击 |

## 5.6 安全相关代码路径汇总

| 安全机制 | 代码文件 | 函数/变量 |
|---------|---------|----------|
| 参数校验 | hiappevent_verify.cpp | VerifyAppEvent() |
| 配置加载 | event_config_mgr.cpp | LoadConfig() |
| 事件存储 | app_event_dao.cpp | SaveEvent() |
| 观察者管理 | app_event_observer_mgr.cpp | AddWatcher() |
| 内存管理 | app_event_util.cpp | - |

## 5.7 后续安全工作

HiAppEvent 组件的安全加固是一个持续的过程。建议后续关注以下方向：

- 引入形式化验证，对关键安全逻辑进行数学证明
- 建立安全测试框架，包括模糊测试（Fuzz Testing）
- 定期进行第三方安全审计
- 完善安全事件的响应和修复流程
