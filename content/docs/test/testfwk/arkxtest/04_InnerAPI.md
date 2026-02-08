# 内部 API 文档

本文档描述 ArkXtest 各组件的内部模块接口、依赖方向、生命周期和稳定性标注。

## UiTest 核心模块

### 模块职责

| 模块 | 职责 | 关键类/文件 | 稳定性 |
|------|------|-------------|--------|
| **WidgetSelector** | 控件匹配选择 | `widget_selector.cpp/h` | 稳定 |
| **WidgetOperator** | 控件操作执行 | `widget_operator.cpp/h` | 稳定 |
| **UiDriver** | 测试驱动入口 | `ui_driver.cpp/h` | 稳定 |
| **UiModel** | UI 模型管理 | `ui_model.cpp/h` | 稳定 |
| **UiAction** | UI 动作注入 | `ui_action.cpp/h` | 稳定 |
| **WindowOperator** | 窗口操作 | `window_operator.cpp/h` | 稳定 |
| **FrontendApiHandler** | API 请求分发 | `frontend_api_handler.cpp` | 稳定 |
| **DumpHandler** | 布局信息导出 | `dump_handler.cpp` | 稳定 |

### WidgetSelector 模块

**职责**: 根据选择器条件匹配 UI 控件

**关键 API**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Select` | `WidgetQuery`, `ElementInfo[]` | `WidgetInfo[]` | 执行选择 |
| `MatchOne` | `WidgetQuery`, `ElementInfo` | `bool` | 匹配单个 |
| `MatchAll` | `WidgetQuery`, `ElementInfo[]` | `ElementInfo[]` | 匹配全部 |
| `AddCondition` | `WidgetAttr`, `string`, `MatchMode` | `WidgetQuery&` | 添加条件 |

**选择器支持属性**:

| 属性 | 类型 | 说明 |
|------|------|------|
| `id` | string | 控件 ID |
| `text` | string | 文本内容 |
| `type` | string | 控件类型 |
| `enabled` | bool | 是否启用 |
| `focused` | bool | 是否聚焦 |
| `selected` | bool | 是否选中 |
| `clickable` | bool | 是否可点击 |
| `scrollable` | bool | 是否可滚动 |
| `checkable` | bool | 是否可勾选 |
| `description` | string | 无障碍描述 |
| `hint` | string | 提示文本 |

**依赖方向**:

```
WidgetSelector
    ↓ (使用)
Accessibility ←── UiModel
    ↓ (查询)
WindowManager
```

**证据**: `uitest/core/widget_selector.cpp` 实现选择器逻辑

### WidgetOperator 模块

**职责**: 对匹配的控件执行操作

**关键 API**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Click` | `WidgetInfo`, `Point` | `ErrCode` | 点击 |
| `LongClick` | `WidgetInfo` | `ErrCode` | 长按 |
| `DoubleClick` | `WidgetInfo` | `ErrCode` | 双击 |
| `InputText` | `WidgetInfo`, `string` | `ErrCode` | 输入文本 |
| `Scroll` | `WidgetInfo`, `UiDirection` | `ErrCode` | 滚动 |
| `ClearText` | `WidgetInfo` | `ErrCode` | 清除文本 |

**依赖方向**:

```
WidgetOperator
    ↓ (调用)
UiAction ←── UiDriver
    ↓ (注入)
InputSystem
```

**证据**: `uitest/core/widget_operator.cpp` 实现操作逻辑

### FrontendApiHandler 模块

**职责**: 解析和分发前端 API 请求

**关键 API**:

| 方法 | 参数 | 返回值 | 对应 JS API |
|------|------|--------|-------------|
| `HandleFindComponent` | `ApiCallInfo` | `ApiReplyInfo` | Driver.findComponent |
| `HandleClick` | `ApiCallInfo` | `ApiReplyInfo` | Component.click |
| `HandleInputText` | `ApiCallInfo` | `ApiReplyInfo` | Component.inputText |
| `HandleSwipe` | `ApiCallInfo` | `ApiReplyInfo` | Driver.swipe |
| `HandleScreenCap` | `ApiCallInfo` | `ApiReplyInfo` | Driver.screenCap |

**API 类型枚举**:

| 枚举值 | 说明 |
|--------|------|
| `GET_COMPONENT` | 获取控件 |
| `GET_WINDOW` | 获取窗口 |
| `INJECT_ACTION` | 注入动作 |
| `DUMP_LAYOUT` | 导出布局 |
| `SCREEN_CAP` | 屏幕截图 |

**证据**: `uitest/core/frontend_api_handler.cpp` 实现 API 分发

---

## PerfTest 核心模块

### 模块职责

| 模块 | 职责 | 关键类/文件 | 稳定性 |
|------|------|-------------|--------|
| **PerfTest** | 测试编排主类 | `perf_test.cpp/h` | 稳定 |
| **PerfTestStrategy** | 测试策略配置 | `perf_test_strategy.cpp/h` | 稳定 |
| **FrontendApiHandler** | API 请求处理 | `frontend_api_handler.cpp` | 稳定 |
| **DataCollection** | 数据采集基类 | `data_collection.cpp/h` | 稳定 |
| **DurationCollection** | 执行时间采集 | `duration_collection.cpp` | 稳定 |
| **CpuCollection** | CPU 指标采集 | `cpu_collection.cpp` | 稳定 |
| **MemoryCollection** | 内存指标采集 | `memory_collection.cpp` | 稳定 |
| **AppStartTimeCollection** | 启动时间采集 | `app_start_time_collection.cpp` | 稳定 |
| **PageSwitchTimeCollection** | 页面切换采集 | `page_switch_time_collection.cpp` | 稳定 |
| **ListSwipeFpsCollection** | 帧率采集 | `list_swipe_fps_collection.cpp` | 稳定 |

### PerfTest 模块

**职责**: 管理测试执行生命周期

**关键 API**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `Create` | `PerfTestStrategy` | `PerfTest*` | 创建实例 |
| `Run` | - | `ErrCode` | 执行测试 |
| `GetMeasureResult` | `PerfMetric` | `PerfMeasureResult` | 获取结果 |
| `Destroy` | - | `void` | 销毁实例 |

**测试流程**:

```cpp
PerfTest::Run() {
    // 1. 验证策略配置
    // 2. 执行初始化代码 (resetCode)
    // 3. 循环执行测试迭代
    for (int i = 0; i < strategy.iterations; i++) {
        // 3.1 执行 actionCode
        // 3.2 启动数据采集
        // 3.3 停止采集并记录
    }
    // 4. 计算统计结果
}
```

**证据**: `perftest/core/src/perf_test.cpp` 实现测试编排

### DataCollection 模块

**职责**: 采集各类性能指标

**抽象基类方法**:

| 方法 | 说明 |
|------|------|
| `Init` | 初始化采集器 |
| `StartCollection` | 开始采集 |
| `StopCollection` | 停止采集 |
| `GetResult` | 获取采集数据 |

**采集器实现**:

| 采集器 | 数据来源 | 说明 |
|--------|----------|------|
| DurationCollection | `std::chrono` | 执行时间（毫秒） |
| CpuCollection | `/proc/stat` | CPU 负载和使用率 |
| MemoryCollection | `/proc/[pid]/status` | RSS/PSS 内存 |
| AppStartTimeCollection | HiSysEvent | 应用启动事件 |
| PageSwitchTimeCollection | HiSysEvent | 页面切换事件 |
| ListSwipeFpsCollection | HiTrace | 滑动帧率 |

**依赖方向**:

```
DataCollection
    ↓ (使用)
HiSysEvent ──→ 进程监控
/proc文件系统 ──→ CPU/内存
HiTrace ──→ 帧率追踪
```

**证据**: `perftest/collection/src/` 包含各采集器实现

---

## TestServer 核心模块

### 服务实现

| 模块 | 职责 | 关键文件 |
|------|------|----------|
| **TestServerService** | SA 服务主类 | `test_server_service.cpp` |
| **SessionManager** | 客户端会话管理 | `session_token.h` |
| **WindowOperator** | 窗口操作 | `test_server_service.cpp` |
| **ClipboardOperator** | 剪贴板操作 | `test_server_service.cpp` |
| **PerformanceCollector** | 性能数据采集 | `test_server_service.cpp` |

### TestServerService 模块

**职责**: 实现 SA 接口，处理客户端请求

**关键方法**:

| 方法 | 功能 | 权限检查 |
|------|------|----------|
| `CreateSession` | 创建会话，追踪客户端 | 无 |
| `DestroySession` | 销毁会话 | 无 |
| `SetPasteData` | 设置剪贴板 | `ARKXTEST_PASTEBOARD_ENABLE` |
| `ChangeWindowMode` | 切换窗口模式 | 无 |
| `TerminateWindow` | 终止窗口 | 无 |
| `MinimizeWindow` | 最小化窗口 | 无 |
| `PublishCommonEvent` | 发布系统事件 | `PUBLISH_SYSTEM_COMMON_EVENT` |
| `FrequencyLock` | CPU 频率锁定 | `MANAGE_SECURE_SETTINGS` |
| `CollectProcessMemory` | 采集内存 | 无 |
| `CollectProcessCpu` | 采集 CPU | 无 |
| `SpDaemonProcess` | SmartPerf 控制 | 无 |
| `GetValueFromDataShare` | DataShare 查询 | `ARKXTEST_KNUCKLE_ACTION_ENABLE` |

**生命周期**:

```cpp
void TestServerService::OnStart() {
    // 1. 检查权限
    if (!IsRootVersion() && !IsDeveloperMode()) {
        return; // 拒绝启动
    }
    // 2. 发布服务
    Publish(this);
}

void TestServerService::OnStop() {
    // 清理资源
}
```

**证据**: `testserver/src/service/test_server_service.cpp:87-106`

---

## IPC 通信层

### UiTest IPC

| 组件 | 文件 | 职责 |
|------|------|------|
| **ApiTransactor** | `ipc_transactor.cpp` | IPC 事务处理 |
| **ApiCallInfo** | `ipc_transactor.h` | 调用请求结构 |
| **ApiReplyInfo** | `ipc_transactor.h` | 响应结构 |

**ApiCallInfo 结构**:

| 字段 | 类型 | 说明 |
|------|------|------|
| `apiType` | `int` | API 类型枚举 |
| `params` | `string` | JSON 参数字符串 |
| `callerToken` | `string` | 调用者令牌 |

**ApiReplyInfo 结构**:

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | `int32_t` | 错误码 |
| `data` | `string` | JSON 返回数据 |
| `errorMsg` | `string` | 错误信息 |

**证据**: `uitest/connection/include/ipc_transactor.h`

### PerfTest IPC

| 组件 | 文件 | 职责 |
|------|------|------|
| **ApiCallerClient** | `api_caller_client.cpp` | 客户端调用 |
| **ApiCallerStub** | `api_caller_server.cpp` | 服务端响应 |
| **ApiCallInfo** | `common_utils.h` | 调用信息 |
| **ApiReplyInfo** | `common_utils.h` | 响应信息 |

**证据**: `perftest/connection/include/` 和 `src/`

---

## 稳定性标注

### 稳定接口（Stable）

以下接口对外稳定，不应有破坏性变更：

| 接口 | 位置 | 说明 |
|------|------|------|
| N-API 模块注册 | `uitest_napi.cpp` | 模块入口点 |
| JS 类导出 | `frontend_api_defines.h` | 所有 frontend class |
| 枚举导出 | `frontend_api_defines.h` | 所有枚举 |
| SA 接口 | `ITestServerInterface.idl` | IPC 方法 |

### 内部接口（Internal）

以下接口仅限框架内部使用：

| 接口 | 位置 | 使用者 |
|------|------|--------|
| `WidgetSelector::Select` | `widget_selector.cpp` | FrontendApiHandler |
| `DataCollection::StartCollection` | `data_collection.cpp` | PerfTest |
| `ApiTransactor::SendRequest` | `ipc_transactor.cpp` | N-API |
| `FrontendApiHandler::Handle*` | `frontend_api_handler.cpp` | IPC |

### 废弃接口（Deprecated）

| 接口 | 废弃版本 | 替代接口 |
|------|----------|----------|
| `By.*` | API 9 | `On.*` |

**证据**: `frontend_api_defines.h:229` 标注 `DEPRECATED`

---

## 依赖方向

### UiTest 依赖图

```
                    ┌─────────────────┐
                    │   uitest_napi  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌────────────┐  ┌────────────┐  ┌────────────┐
     │ipc_transactor│  │   Driver   │  │ Component  │
     └─────┬──────┘  └────────────┘  └────────────┘
           │                ▲               ▲
           ▼                │               │
     ┌────────────┐         │               │
     │test_server │◄────────┘               │
     │  client    │                          │
     └─────┬──────┘                          │
           │                                 │
     ┌─────▼─────┐                    ┌─────▼─────┐
     │ uitest    │                    │Accessibility│
     │ server    │                    └────────────┘
     └─────┬─────┘                           │
           │                                 │
     ┌─────▼─────┐                    ┌─────▼─────┐
     │ UiAction  │                    │WindowManager│
     └───────────┘                    └────────────┘
```

### PerfTest 依赖图

```
                    ┌─────────────────┐
                    │  perftest_napi  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌────────────┐  ┌────────────┐  ┌────────────┐
     │ApiCaller   │  │  PerfTest   │  │PerfTestStr │
     │  Client    │  │             │  │   ategy    │
     └─────┬──────┘  └─────┬──────┘  └────────────┘
           │                │
           ▼                ▼
     ┌────────────┐  ┌────────────┐
     │test_server │  │ Data       │
     │  client    │  │Collection  │
     └─────┬──────┘  └─────┬──────┘
           │                │
     ┌─────▼─────┐    ┌─────▼─────┐
     │ perftest   │    │HiSysEvent │
     │  server    │    │/proc      │
     └────────────┘    └───────────┘
```

**证据**: `BUILD.gn` 文件中的 `deps` 依赖关系
