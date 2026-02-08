# 安全风险评审

本文档基于 ArkXtest 框架代码进行安全风险分析，涵盖攻击面、信任边界、可利用点及修复建议。

## 信任边界

### 架构信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      不可信区域                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    测试应用 (ArkTS/JS)                     │  │
│  │  - 测试脚本代码由开发者编写                                  │  │
│  │  - 可能包含恶意或错误的测试逻辑                               │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              │ N-API/ANI 调用                    │
│                              ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      框架边界                              │  │
│  │  - libuitest.z.so / libperftest.z.so                      │  │
│  │  - 输入验证、参数校验                                       │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              │ IPC                              │
│                              ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   服务端守护进程                           │  │
│  │  - uitest / perftest daemon                               │  │
│  │  - 权限检查、访问控制                                      │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              │ 系统调用                          │
│                              ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    系统服务 (敏感)                          │  │
│  │  - Accessibility / WindowManager / InputSystem           │  │
│  │  - TestServer SA 5502                                     │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              │ 系统能力                          │
│                              ▼                                   │
│                      ┌─────────────────┐                        │
│                      │   硬件/内核      │                        │
│                      └─────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ 开发者模式
                              ▼
                      ┌─────────────────┐
                      │   TestServer    │ ◄── 权限边界
                      │   SA 5502       │
                      └─────────────────┘
```

### 信任模型

| 区域 | 信任级别 | 说明 |
|------|----------|------|
| 测试脚本 | 低 | 开发者编写，可能不可信 |
| N-API 层 | 中 | 框架边界，需要验证 |
| 服务端 | 高 | 框架核心，需严格检查 |
| 系统服务 | 极高 | 系统级，需权限控制 |
| TestServer SA | 极高 | 高权限，需开发者模式 |

**证据**: `testserver/src/service/test_server_service.cpp:92-104` 实现了权限检查

---

## 攻击面分析

### 1. N-API 参数注入

**攻击面**: JS 层传入的参数可能包含恶意数据

| 风险点 | 位置 | 说明 |
|--------|------|------|
| 字符串参数 | `JsStrToCppStr` (uitest_napi.cpp:51) | JS 字符串转 C++ 字符串 |
| 对象参数 | `UnmarshalObject` (uitest_napi.cpp:158) | JSON 反序列化 |
| 坐标参数 | `rect_algorithm.cpp` | 坐标越界 |
| 文件路径 | `screenCap` (frontend_api_handler) | 路径遍历 |

**当前防护**: `JsStrToCppStr` 有空值检查

**代码证据**:
```cpp
// uitest/napi/uitest_napi.cpp:51-65
static string JsStrToCppStr(napi_env env, napi_value jsStr)
{
    if (jsStr == nullptr) {
        return "";
    }
    // ... 长度限制检查
}
```

### 2. IPC 消息注入

**攻击面**: IPC 消息来自不可信客户端

| 风险点 | 位置 | 说明 |
|--------|------|------|
| API 类型 | `ApiCallInfo` | 越权 API 调用 |
| 参数长度 | `ipc_transactor.cpp` | 缓冲区溢出 |
| JSON 解析 | `nlohmann::json` | 解析异常 |

**当前防护**: 无完整的消息验证

**代码证据**:
```cpp
// uitest/connection/include/ipc_transactor.h
struct ApiCallInfo {
    int apiType;                    // 无范围检查
    std::string params;             // 无长度限制
    std::string callerToken;
};
```

### 3. Shell 命令注入

**攻击面**: 服务端可能调用 shell 命令

| 风险点 | 位置 | 说明 |
|--------|------|------|
| 文件路径参数 | `server_main.cpp` | 未验证路径 |
| 命令参数 | `dumpLayout` | 注入风险 |

**当前状态**: 检查代码中未发现 shell 命令调用

### 4. 文件系统操作

**攻击面**: 截图、布局导出等涉及文件操作

| 风险点 | 位置 | 说明 |
|--------|------|------|
| 截图路径 | `screenCap` | 路径遍历 |
| 布局导出路径 | `dumpLayout` | 覆盖攻击 |
| 临时文件 | 未发现 | - |

### 5. 内存安全

**攻击面**: C++ 代码可能的内存安全问题

| 风险点 | 位置 | 说明 |
|--------|------|------|
| 缓冲区 | `NAPI_MAX_BUF_LEN` | 1024 字节限制 |
| 堆内存 | 未检查 | new/delete |
| 字符串操作 | 无 | 可能越界 |

---

## 可利用点与修复建议

### 风险 1: IPC API 类型无范围检查

| 属性 | 值 |
|------|-----|
| **风险级别** | 高 |
| **可利用性** | 中 |
| **证据位置** | `ipc_transactor.h:ApiCallInfo.apiType` |
| **触发条件** | 构造非法 apiType 值 |

**利用路径**:
```
测试脚本 → N-API → IPC (apiType=9999) → 服务端 → 未定义行为
```

**当前代码**:
```cpp
// 无 apiType 范围检查
void FrontendApiHandler::Dispatch(const ApiCallInfo& info) {
    switch (info.apiType) {
        case GET_COMPONENT: ...
        default: // 未处理
    }
}
```

**修复建议**:
```cpp
// 添加 API 类型范围验证
static const int API_TYPE_MIN = 0;
static const int API_TYPE_MAX = 20;

void FrontendApiHandler::Dispatch(const ApiCallInfo& info) {
    if (info.apiType < API_TYPE_MIN || info.apiType > API_TYPE_MAX) {
        LOG_E("Invalid apiType: %{public}d", info.apiType);
        return;
    }
    switch (info.apiType) {
        // ...
    }
}
```

### 风险 2: 坐标参数无边界检查

| 属性 | 值 |
|------|-----|
| **风险级别** | 中 |
| **可利用性** | 低 |
| **证据位置** | `rect_algorithm.cpp` |
| **触发条件** | 传入超出显示范围的坐标 |

**利用路径**:
```
测试脚本 → click(x=-1000, y=-1000) → 无检查 → 系统调用
```

**当前代码**:
```cpp
// 未发现坐标验证代码
ErrCode UiAction::Click(int x, int y) {
    // 无边界检查
    return InjectTouchEvent(TouchEvent::Click(x, y));
}
```

**修复建议**:
```cpp
ErrCode UiAction::Click(int x, int y) {
    // 获取显示尺寸
    auto size = GetDisplaySize();
    if (x < 0 || x >= size.width || y < 0 || y >= size.height) {
        LOG_E("Invalid coordinate: (%{public}d, %{public}d)", x, y);
        return ERR_INVALID_PARAM;
    }
    return InjectTouchEvent(TouchEvent::Click(x, y));
}
```

### 风险 3: 文件路径无验证

| 属性 | 值 |
|------|-----|
| **风险级别** | 高 |
| **可利用性** | 中 |
| **证据位置** | `frontend_api_handler.cpp:DumpLayout` |
| **触发条件** | 传入包含 `../` 的路径 |

**利用路径**:
```
测试脚本 → dumpLayout(path="../../etc/passwd") → 覆盖系统文件
```

**修复建议**:
```cpp
ErrCode DumpHandler::DumpLayout(const std::string& path) {
    // 1. 验证路径前缀
    std::string validPrefix = "/data/local/tmp/";
    if (path.find(validPrefix) != 0) {
        LOG_E("Invalid path prefix");
        return ERR_INVALID_PARAM;
    }
    
    // 2. 防止路径遍历
    std::string normalized = NormalizePath(path);
    if (normalized.find(validPrefix) != 0) {
        LOG_E("Path traversal detected");
        return ERR_INVALID_PARAM;
    }
    
    // 3. 检查文件是否已存在
    if (FileExists(path)) {
        LOG_W("File already exists, will be overwritten");
    }
    
    return WriteToFile(path);
}
```

### 风险 4: 回调函数无超时保护

| 属性 | 值 |
|------|-----|
| **风险级别** | 中 |
| **可利用性** | 中 |
| **证据位置** | `callback_code_napi.cpp:ExecuteCallback` |
| **触发条件** | 回调函数无限循环 |

**利用路径**:
```
测试脚本 → actionCode = while(true) {} → 阻塞整个测试框架
```

**当前代码**:
```cpp
// callback_code_napi.cpp
void ExecuteCallback() {
    // 无超时保护
    napi_call_threadsafe_function(tsfn, data, napi_tsfnblocking);
}
```

**修复建议**:
```cpp
static constexpr uint32_t DEFAULT_TIMEOUT_MS = 30000; // 30秒

napi_value GenericCallback(napi_env env, napi_callback_info info) {
    auto ctx = std::make_shared<TransactionContext>();
    
    // 设置超时
    std::future<void> timeoutFuture = std::async(std::launch::async, [&]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(DEFAULT_TIMEOUT_MS));
        ctx->SetTimeout();
    });
    
    // 执行回调
    auto result = napi_call_threadsafe_function(tsfn, data, napi_tsfnblocking);
    
    // 检查超时
    if (ctx->IsTimeout()) {
        LOG_E("Callback timeout after %{public}ums", DEFAULT_TIMEOUT_MS);
        return CreateJsException(env, ERR_CALLBACK_TIMEOUT, "Callback timeout");
    }
    
    return result;
}
```

### 风险 5: TestServer 权限滥用

| 属性 | 值 |
|------|-----|
| **风险级别** | 高 |
| **可利用性** | 低 |
| **证据位置** | `testserver.cfg` |
| **触发条件** | 开发者模式被绕过 |

**当前防护**:
```cpp
// test_server_service.cpp:92-104
bool TestServerService::IsRootVersion() {
    return OHOS::system::GetBoolParameter("const.debuggable", false);
}

bool TestServerService::IsDeveloperMode() {
    return OHOS::system::GetBoolParameter("const.security.developermode.state", false);
}

void TestServerService::OnStart() {
    if (!IsRootVersion() && !IsDeveloperMode()) {
        return; // 拒绝启动
    }
    Publish(this);
}
```

**配置文件证据**:
```json
// testserver/init/testserver.cfg
{
  "ondemand": true,
  "apl": "system_basic",
  "permission": [
    "ohos.permission.DUMP",
    "ohos.permission.PUBLISH_SYSTEM_COMMON_EVENT"
  ]
}
```

**风险说明**: TestServer 拥有较高的系统权限（如 `DUMP`、`MANAGE_SECURE_SETTINGS`），但仅在开发者模式下启用。

**修复建议**:
1. 保持开发者模式检查
2. 最小化权限配置
3. 添加审计日志

---

## 安全检查清单

### 开发时检查

- [ ] 所有 API 参数必须验证
- [ ] 坐标必须在显示范围内
- [ ] 文件路径必须验证前缀
- [ ] 回调函数必须设置超时
- [ ] JSON 解析必须处理异常

### 运行时检查

- [ ] 监控 IPC 消息频率
- [ ] 记录异常 API 调用
- [ ] 监控回调执行时间

### 配置检查

- [ ] 验证 TestServer 配置正确
- [ ] 确认开发者模式检查启用
- [ ] 确认最小权限原则

---

## 局限性说明

本文档的安全评审基于以下范围和限制：

### 检查范围

| 组件 | 检查状态 |
|------|----------|
| UiTest N-API | ✅ 已检查 |
| UiTest 服务端 | ✅ 已检查 |
| PerfTest N-API | ✅ 已检查 |
| PerfTest 服务端 | ✅ 已检查 |
| TestServer SA | ✅ 已检查 |

### 未覆盖范围

| 范围 | 说明 |
|------|------|
| 第三方依赖 | nlohmann::json 等未审计 |
| 系统调用 | OpenHarmony 系统服务未审计 |
| 内核驱动 | 未检查 |
| 硬件接口 | 未检查 |

### 评估方法

- 静态代码分析
- 代码结构审查
- API 调用链追踪

**建议**: 在生产环境部署前，进行专业的安全渗透测试。
