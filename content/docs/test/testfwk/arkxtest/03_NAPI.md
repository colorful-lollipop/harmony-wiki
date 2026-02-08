# N-API 接口文档

本文档详细描述 ArkXtest 各组件对外暴露的 N-API/ANI 接口，包括 API 清单、参数校验、错误码和调用链。

## UiTest N-API

### 模块注册

| 属性 | 值 |
|------|-----|
| 模块名 | `UiTest` |
| 注册文件 | `uitest/napi/uitest_napi.cpp` |
| 注册函数 | `napi_module_register` (L612) |
| 模块结构 | `napi_module` (L600-608) |

**代码证据**:
```cpp
// uitest/napi/uitest_napi.cpp:600-613
static napi_module module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,
    .nm_modname = "UiTest",
    .nm_priv = ((void *)0),
    .reserved = {0},
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&module);
}
```

### 导出函数

| JS 函数 | C++ 实现 | 静态 | 说明 |
|---------|----------|------|------|
| `scheduleEstablishConnection` | `ScheduleEstablishConnection` (L68) | Yes | 异步建立 IPC 连接 |
| `getUnCalledJsApis` | `GetUnCalledJsApis` (L556) | Yes | 获取未调用的 JS API 列表 |

### 导出类清单

| 类名 | 定义文件 | C++ 结构 | API 数量 |
|------|----------|----------|----------|
| `By` | `frontend_api_defines.h:229` | `FrontendClassDef` | 控件选择器（API 8，已废弃） |
| `On` | `frontend_api_defines.h:294` | `FrontendClassDef` | 控件选择器（API 9+） |
| `Driver` | `frontend_api_defines.h:325` | `FrontendClassDef` | UI 测试驱动 |
| `UiComponent` | `frontend_api_defines.h:395` | `FrontendClassDef` | 控件对象 |
| `UiWindow` | `frontend_api_defines.h:432` | `FrontendClassDef` | 窗口对象 |
| `On` | `frontend_api_defines.h:468` | `FrontendClassDef` | 事件观察器 |
| `PointerMatrix` | `frontend_api_defines.h:457` | `FrontendClassDef` | 指针矩阵 |

**证据**: `frontend_api_defines.h` 定义了所有前端 API 结构

### Driver 类 API

| JS 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|---------|------|--------|----------|------|
| `create()` | - | `Driver` | Async | 创建 Driver 实例 |
| `findComponent` | `On` | `UiComponent` | Async | 查找单个控件 |
| `findComponents` | `On` | `UiComponent[]` | Async | 查找多个控件 |
| `findWindow` | `WindowFilter` | `UiWindow` | Async | 查找窗口 |
| `click` | `x: number, y: number` | `void` | Async | 点击坐标 |
| `doubleClick` | `x: number, y: number` | `void` | Async | 双击坐标 |
| `longClick` | `x: number, y: number` | `void` | Async | 长按坐标 |
| `swipe` | `x1, y1, x2, y2, steps?` | `void` | Async | 滑动 |
| `fling` | `direction, x, y, speed` | `void` | Async | 快速滑动 |
| `drag` | `x1, y1, x2, y2` | `void` | Async | 拖拽 |
| `pressBack` | `delayMs?` | `void` | Async | 按返回键 |
| `pressHome` | `delayMs?` | `void` | Async | 按 Home 键 |
| `triggerKey` | `keyCode, delayMs?` | `void` | Async | 触发按键 |
| `screenCap` | `path, displayId?` | `void` | Async | 屏幕截图 |
| `screenCapture` | `path, rect?` | `void` | Async | 区域截图 |
| `inputText` | `point, text, mode?` | `void` | Async | 输入文本 |
| `waitForComponent` | `on, timeout` | `UiComponent` | Async | 等待控件出现 |
| `waitForIdle` | `waitMs, intervalMs` | `boolean` | Async | 等待空闲 |
| `getDisplaySize` | `displayId?` | `Point` | Async | 获取显示尺寸 |
| `getDisplayDensity` | `displayId?` | `Point` | Async | 获取显示密度 |
| `getDisplayRotation` | `displayId?` | `DisplayRotation` | Async | 获取显示旋转 |
| `createUIEventObserver` | - | `UIEventObserver` | Sync | 创建事件观察器 |
| `injectMultiPointerAction` | `PointerMatrix, swipeSpeed?` | `boolean` | Async | 多指针注入 |

**证据**: `frontend_api_defines.h:325-392` 定义了 Driver 类 API

### UiComponent 类 API

| JS 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|---------|------|--------|----------|------|
| `getText()` | - | `string` | Async | 获取文本 |
| `getId()` | - | `string` | Async | 获取 ID |
| `getType()` | - | `string` | Async | 获取类型 |
| `getDescription()` | - | `string` | Async | 获取描述 |
| `getHint()` | - | `string` | Async | 获取提示 |
| `getDisplayId()` | - | `number` | Async | 获取显示 ID |
| `getBounds()` | - | `Rect` | Async | 获取边界 |
| `getBoundsCenter()` | - | `Point` | Async | 获取中心点 |
| `isEnabled()` | - | `boolean` | Async | 是否启用 |
| `isFocused()` | - | `boolean` | Async | 是否聚焦 |
| `isSelected()` | - | `boolean` | Async | 是否选中 |
| `isClickable()` | - | `boolean` | Async | 是否可点击 |
| `isLongClickable()` | - | `boolean` | Async | 是否可长按 |
| `isScrollable()` | - | `boolean` | Async | 是否可滚动 |
| `isCheckable()` | - | `boolean` | Async | 是否可勾选 |
| `isChecked()` | - | `boolean` | Async | 是否已勾选 |
| `click()` | - | `void` | Async | 点击 |
| `longClick()` | - | `void` | Async | 长按 |
| `doubleClick()` | - | `void` | Async | 双击 |
| `inputText()` | `text, mode?` | `void` | Async | 输入文本 |
| `clearText()` | - | `void` | Async | 清除文本 |
| `scrollSearch()` | `on, forward?, speed?` | `UiComponent` | Async | 滚动搜索 |
| `dragTo()` | `target` | `void` | Async | 拖拽到 |
| `pinchOut()` | `scale` | `void` | Async | 捏合放大 |
| `pinchIn()` | `scale` | `void` | Async | 捏合缩小 |

**证据**: `frontend_api_defines.h:395-429` 定义了 Component 类 API

### UiWindow 类 API

| JS 方法 | 参数 | 返回值 | 同步/异步 | 说明 |
|---------|------|--------|----------|------|
| `getBundleName()` | - | `string` | Async | 获取包名 |
| `getTitle()` | - | `string` | Async | 获取标题 |
| `getWindowMode()` | - | `WindowMode` | Async | 获取窗口模式 |
| `getBounds()` | - | `Rect` | Async | 获取边界 |
| `getDisplayId()` | - | `number` | Async | 获取显示 ID |
| `isFocused()` | - | `boolean` | Async | 是否聚焦 |
| `isActive()` | - | `boolean` | Async | 是否激活 |
| `focus()` | - | `void` | Async | 聚焦 |
| `moveTo()` | `x, y` | `void` | Async | 移动 |
| `resize()` | `width, height, direction` | `void` | Async | 调整大小 |
| `split()` | - | `void` | Async | 分屏 |
| `maximize()` | - | `void` | Async | 最大化 |
| `resume()` | - | `void` | Async | 恢复 |
| `minimize()` | - | `void` | Async | 最小化 |
| `close()` | - | `void` | Async | 关闭 |

**证据**: `frontend_api_defines.h:432-454` 定义了 Window 类 API

### 枚举导出

| 枚举名 | 值 | 说明 |
|--------|-----|------|
| `MatchPattern` | EQUALS, CONTAINS, STARTS_WITH, ENDS_WITH, REG_EXP, REG_EXP_ICASE | 匹配模式 |
| `WindowMode` | FULLSCREEN, PRIMARY, SECONDARY, FLOATING | 窗口模式 |
| `ResizeDirection` | LEFT, RIGHT, UP, DOWN, LEFT_UP, LEFT_DOWN, RIGHT_UP, RIGHT_DOWN | 调整方向 |
| `DisplayRotation` | ROTATION_0, ROTATION_90, ROTATION_180, ROTATION_270 | 显示旋转 |
| `MouseButton` | MOUSE_BUTTON_LEFT, RIGHT, MIDDLE | 鼠标按键 |
| `UiDirection` | LEFT, RIGHT, UP, DOWN | UI 方向 |
| `WindowChangeType` | WINDOW_UNDEFINED, ADDED, REMOVED, BOUNDS_CHANGED | 窗口变化类型 |
| `ComponentEventType` | COMPONENT_UNDEFINED, CLICKED, LONG_CLICKED, SCROLL_START, SCROLL_END, TEXT_CHANGED | 组件事件类型 |

**证据**: `frontend_api_defines.h` 定义了所有枚举

---

## PerfTest N-API

### 模块注册

| 属性 | 值 |
|------|-----|
| 模块名 | `test.PerfTest` |
| 注册文件 | `perftest/napi/src/perftest_napi.cpp` |
| 注册函数 | `napi_module_register` (L503) |
| 模块结构 | `napi_module` (L495-502) |

### 导出函数

| JS 函数 | C++ 实现 | 静态 | 说明 |
|---------|----------|------|------|
| `scheduleEstablishConnection` | `ScheduleEstablishConnection` | Yes | 异步建立 IPC 连接 |
| `getConnectionStat` | `GetConnectionStat` | Yes | 获取连接状态 |

### PerfTest 类 API

| JS 方法 | C++ 实现 | 参数 | 返回值 | 说明 |
|---------|----------|------|--------|------|
| `create` | `GenericCallback` | `PerfTestStrategy` | `PerfTest` | 创建测试实例 |
| `run` | `GenericCallback` | - | `void` | 运行测试 |
| `getMeasureResult` | `GenericCallback` | `PerfMetric` | `PerfMeasureResult` | 获取测试结果 |
| `destroy` | `GenericCallback` | - | `void` | 销毁实例 |

**证据**: `frontend_api_defines.h:185-191` 定义了 PerfTest API

### PerfMetric 枚举

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `DURATION` | 0 | 执行时间 |
| `CPU_LOAD` | 1 | CPU 负载 |
| `CPU_USAGE` | 2 | CPU 使用率 |
| `MEMORY_RSS` | 3 | 常驻集大小 |
| `MEMORY_PSS` | 4 | 比例集大小 |
| `APP_START_RESPONSE_TIME` | 5 | 启动响应延迟 |
| `APP_START_COMPLETE_TIME` | 6 | 启动完成延迟 |
| `PAGE_SWITCH_COMPLETE_TIME` | 7 | 页面切换延迟 |
| `LIST_SWIPE_FPS` | 8 | 列表滑动帧率 |

**证据**: `data_collection.h:29-40` 定义了 PerfMetric 枚举

### 错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `NO_ERROR` | 成功 |
| 32400001 | `ERR_INITIALIZE_FAILED` | 初始化失败 |
| 32400002 | `ERR_INTERNAL` | 系统错误 |
| 32400003 | `ERR_INVALID_INPUT` | 无效输入参数 |
| 32400004 | `ERR_CALLBACK_FAILED` | 回调执行失败 |
| 32400005 | `ERR_DATA_COLLECTION_FAILED` | 数据采集失败 |
| 32400006 | `ERR_GET_RESULT_FAILED` | 获取结果失败 |
| 32400007 | `ERR_API_USAGE` | API 使用错误（不允许并发调用） |

**证据**: `frontend_api_defines.h:33-50` 定义了错误码

---

## TestServer IPC 接口

### IDL 定义

| 文件 | 说明 |
|------|------|
| `testserver/src/ITestServerInterface.idl` | IPC 接口定义 |
| `testserver/src/Types.idl` | 数据结构定义 |

### IPC 方法清单

| 方法 | 输入参数 | 输出参数 | 功能 |
|------|----------|----------|------|
| `CreateSession` | `SessionToken` | - | 创建会话 |
| `SetPasteData` | `String` | - | 设置剪贴板 |
| `ChangeWindowMode` | `int, unsigned int` | - | 窗口模式切换 |
| `TerminateWindow` | `int` | - | 终止窗口 |
| `MinimizeWindow` | `int` | - | 最小化窗口 |
| `PublishCommonEvent` | `CommonEventData` | `boolean` | 发布系统事件 |
| `FrequencyLock` | - | - | CPU 频率锁定 |
| `SpDaemonProcess` | `int, String` | - | SmartPerf 进程控制 |
| `CollectProcessMemory` | `int` | `ProcessMemoryInfo` | 采集进程内存 |
| `CollectProcessCpu` | `int, boolean` | `ProcessCpuInfo` | 采集进程 CPU |
| `GetValueFromDataShare` | `String, String` | `String` | DataShare 查询 |

**证据**: `ITestServerInterface.idl:19-33` 定义了 IPC 接口

### TestServer 错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `TEST_SERVER_OK` | 成功 |
| -1 | `TEST_SERVER_GET_INTERFACE_FAILED` | 获取 SA 接口失败 |
| 19000001 | `TEST_SERVER_ADD_DEATH_RECIPIENT_FAILED` | 添加死亡回调失败 |
| 19000002 | `TEST_SERVER_CREATE_PASTE_DATA_FAILED` | 创建剪贴板数据失败 |
| 19000003 | `TEST_SERVER_SET_PASTE_DATA_FAILED` | 设置剪贴板数据失败 |
| 19000004 | `TEST_SERVER_PUBLISH_EVENT_FAILED` | 发布事件失败 |
| 19000005 | `TEST_SERVER_SPDAEMON_PROCESS_FAILED` | SmartPerf 操作失败 |
| 19000006 | `TEST_SERVER_COLLECT_PROCESS_INFO_FAILED` | 采集进程信息失败 |
| 19000007 | `TEST_SERVER_OPERATE_WINDOW_FAILED` | 窗口操作失败 |
| 19000008 | `TEST_SERVER_DATASHARE_FAILED` | DataShare 操作失败 |

**证据**: `testserver/src/utils/test_server_error_code.h` 定义了错误码

---

## 参数校验规则

### UiTest 参数校验

| 参数类型 | 校验规则 | 位置 |
|----------|----------|------|
| `On` 选择器 | 非空，至少一个匹配条件 | `frontend_api_handler.cpp` |
| 坐标值 | 整数，在显示范围内 | `rect_algorithm.cpp` |
| 文本输入 | UTF-8 编码，长度限制 | `ui_input.cpp` |
| 超时时间 | 正整数 | `ui_driver.cpp` |

### PerfTest 参数校验

| 参数类型 | 校验规则 | 位置 |
|----------|----------|------|
| `PerfTestStrategy` | 非空，包含有效的 actionCode | `perf_test.cpp` |
| `PerfMetric` | 有效的枚举值 | `frontend_api_handler.cpp` |
| 迭代次数 | 1-1000 | `perf_test_strategy.cpp` |
| 超时时间 | 1-300000ms | `perf_test_strategy.cpp` |

---

## 调用链示例

### UiTest.findComponent 调用链

```
JS: Driver.findComponent(On.text('hello'))
    ↓
napi: uitest_napi.cpp:Driver.findComponent()
    ↓
IPC: ApiTransactor.SendRequest(GET_COMPONENT)
    ↓ CommonEvent IPC
Server: frontend_api_handler.cpp:HandleGetComponent()
    ↓
WidgetSelector: widget_selector.cpp:Select()
    ↓
Accessibility: GetRootFromWindows()
    ↓ 返回 WidgetInfo
Server → IPC → NAPI → JS: UiComponent 对象
```

### PerfTest 测试执行调用链

```
JS: perfTest.run()
    ↓
N-API: GenericCallback(OP_RUN)
    ↓
IPC: ApiCallerClient.Call()
    ↓
Server: FrontendApiHandler::HandleRun()
    ↓
PerfTest::Run()
    ↓
Loop iterations:
    - CallbackCodeNapi::ExecuteCallback(actionCode)
    - DataCollection::StartCollection()
    - DataCollection::StopCollection()
    ↓
Server → IPC → N-API → JS: void
```
