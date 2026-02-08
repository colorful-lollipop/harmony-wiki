# 常见构建/运行/调试问题

> 基于代码证据的常见问题、定位方法和解决方案

---

## 目的

本文档提供 `window_cangjie_wrapper` 开发过程中的常见问题解决方案，帮助开发者快速定位和解决问题。

---

## 构建相关问题

### 1. 编译错误："未找到 foreign 函数"

**问题现象**：
```
Error: Undefined reference to 'FfiOHOSCreateWindow'
```

**原因**：
- Native 层（`window_manager:cj_window_ffi`）未正确编译
- FFI 符号未导出
- 外部依赖配置错误

**证据**：`ohos/window/BUILD.gn:39`（`external_deps = ["window_manager:cj_window_ffi"]`）

**定位方法**：
1. 检查 Native 层（`window_manager` 子系统）编译是否成功
2. 查看 `window_manager` 子系统的 FFI 符号导出配置
3. 确认 `external_deps` 配置正确

**解决方案**：
```bash
# 1. 编译 window_manager 子系统
./build.sh --product-name {product} --build-target window_manager

# 2. 检查 FFI 符号是否导出
nm -D out/{product}/lib64/libwindow_manager.so | grep "FfiOHOSCreateWindow"

# 3. 清理重新编译
./build.sh --product-name {product} --clean
./build.sh --product-name {product} --build-target window_manager --build-target window_cangjie_wrapper
```

---

### 2. 链接错误："未定义的符号"

**问题现象**：
```
Error: undefined reference to 'RemoteDataLite'
```

**原因**：
- `arkui_cangjie_wrapper` 模块未正确编译
- 仓颉 SDK 未正确安装

**证据**：`ohos/window/BUILD.gn:28`（`cj_external_deps = ["arkui_cangjie_wrapper:ohos.base"]`）

**定位方法**：
1. 检查 `arkui_cangjie_wrapper` 编译产物是否存在
2. 查看 SDK libs 目录是否有 `libohos.base.so`

**解决方案**：
```bash
# 1. 编译仓颉基础依赖
./build.sh --product-name {product} --build-target arkui_cangjie_wrapper

# 2. 确认编译产物
ls -l out/{product}/libs/sdk/ | grep "libohos.base"

# 3. 清理重新编译
./build.sh --product-name {product} --clean
```

---

### 3. BUILD.gn 语法错误

**问题现象**：
```
Error: Unexpected token in build file
```

**原因**：
- GN 文件语法错误
- 括号不匹配
- 列表格式错误

**证据**：所有 BUILD.gn 文件

**定位方法**：
1. 使用 GN 语法检查工具
2. 参考官方 BUILD.gn 文档
3. 检查括号和逗号匹配

**解决方案**：
```gn
# 确保 target 名称使用双引号
ohos_cangjie_shared_library("ohos.window") {
    sources = [ ... ]
}

# 确保 deps 使用正确的语法
cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability",
    # 不是:
    # ability_cangjie_wrapper:ohos.app.ability
]
```

---

## 运行时相关问题

### 1. 错误码 1300006：窗口上下文异常

**问题现象**：
```
BusinessException: 1300006 - This window context is abnormal.
```

**原因**：
- 使用了非 UIAbilityContext
- 传入的上下文无法获取有效的 StageContext

**证据**：`window.cj:224-236`

```cangjie
// FFiGetContext 返回空指针
let stageContext = unsafe {
    FFIGetContext(config.ctx.getID())
}
if (stageContext.isNull()) {
    throw BusinessException(1300006, "only UIAbilityContext is supported")
}
```

**定位方法**：
1. 检查调用链，确定是 `createWindow()` 还是 `getLastWindow()`
2. 确认传入的上下文类型
3. 添加日志调试：`WINDOW_LIB_LOG.info("[Window] Context type: ${config.ctx}")`

**解决方案**：
```cangjie
// 确保使用 UIAbilityContext
import ohos.app.ability.UIAbilityContext

// 不要使用以下上下文类型：
// - ServiceExtensionContext
// - ExtensionContext
// - FormExtensionContext
```

---

### 2. 错误码 1300002：窗口状态异常

**问题现象**：
```
BusinessException: 1300002 - This window state is abnormal.
```

**原因**：
- 尝试操作已销毁的窗口
- 窗口 ID 无效
- Native 层窗口已被销毁

**证据**：`cj_window_utils.cj:26`

**定位方法**：
1. 检查窗口生命周期
2. 确认窗口是否已调用 `destroyWindow()`
3. 检查 `Window.INSTANCE_MAP` 是否还包含该窗口

**解决方案**：
```cangjie
// 方法 1：使用 try-catch 捕获错误
try {
    window.showWindow()
} catch (e: BusinessException) {
    if (e.code == 1300002) {
        // 窗口已销毁，需要重新获取
        let newWindow = getLastWindow(ctx)
        // 继续操作
    }
}

// 方法 2：检查窗口状态
if (window.isWindowShowing()) {
    window.showWindow()
}
```

---

### 3. 错误码 1400003：显示管理服务异常

**问题现象**：
```
BusinessException: 1400003 - This display manager service works abnormally.
```

**原因**：
- Native 层的窗口管理服务崩溃
- FFI 调用返回服务异常错误码

**证据**：`display.cj:133-136`、`display.cj:158-161`、`display.cj:273-276`

**定位方法**：
1. 检查 Native 服务是否运行
2. 查看系统日志确认服务状态
3. 检查是否存在资源冲突

**解决方案**：
```bash
# 1. 检查 window_manager 服务状态
ps -ef | grep window_manager

# 2. 重启窗口管理服务
hilog -r window_manager

# 3. 检查内存占用
free -h

# 4. 等待服务恢复，重试操作
```

---

### 4. 权限错误 201/202

**问题现象**：
```
BusinessException: 201 - Permission verification failed.
BusinessException: 202 - Application which is not a system application uses system API.
```

**原因**：
- 未申请必要权限（如 `SYSTEM_FLOAT_WINDOW`）
- 非 system 应用尝试使用系统 API
- 应用签名问题

**证据**：`window.cj:187`、`cj_window_utils.cj:35-36`

**定位方法**：
1. 检查 `module.json5` 中的 `requestPermissions` 配置
2. 确认应用类型（system_app vs normal_app）
3. 验证权限申请

**解决方案**：
```json
// 在 module.json5 中添加权限请求
{
  "module": {
    "name": "entry",
    "type": "entry",
    "requestPermissions": [
      {
        "name": "ohos.permission.SYSTEM_FLOAT_WINDOW",
        "reason": "$string:float_window_reason"
      }
    ]
  }
}
```

---

## 调试技巧

### 1. 启用详细日志

**方法**：配置 HiLog 日志级别

**证据**：`cj_window_log.cj:24`、`cj_display_log.cj:23`

**位置**：
- `ohos/window/cj_window_log.cj:24` - `let WINDOW_LIB_LOG = HilogChannel(0, 0xD004200, "CJ-Window-Manager")`
- `ohos/display/cj_display_log.cj:23` - `let DISPLAY_LOG = HilogChannel(0, 0xD004201, "CJ-Display")`

**配置方法**：
```bash
# 设置全局日志级别为 DEBUG
hilog -b Q -v D 0xD004200

# 只查看窗口相关日志
hilog -v Q -t CJ-Window-Manager

# 只查看显示相关日志
hilog -v Q -t CJ-Display

# 保存日志到文件
hilog -v Q -f /data/log/window_cangjie.log
```

---

### 2. FFI 调用追踪

**方法**：添加日志记录 FFI 调用

**位置**：所有 FFI 调用点（`window.cj:32-146`、`display.cj:27-93`）

**示例**：
```cangjie
public func createWindow(config: Configuration): Window {
    unsafe {
        let stageContext = unsafe {
            FFIGetContext(config.ctx.getID())
        }
        WINDOW_LIB_LOG.info("[Window] createWindow: name=${config.name}, type=${config.windowType}")

        var ret: ?RetDataI64 = None
        try (cname = LibC.mallocCString(config.name).asResource()) {
            ret = FfiOHOSCreateWindow(cname.value, config.windowType.getValue(), stageContext, ...)
        }
        WINDOW_LIB_LOG.info("[Window] createWindow FFI result: code=${ret.getOrThrow().code}, id=${ret.getOrThrow().data}")
        checkRet(ret.getOrThrow().code, "[Window] createWindow: ")
    }
}
```

---

### 3. 回调调试

**方法**：记录回调注册和触发

**位置**：`window.cj:777-801`、`display.cj:295-301`

**示例**：
```cangjie
func onKeyboardHeightChange(callbackType: String, callback: Callback1Argument<UInt32>): Unit {
    synchronized(REGISTER_MUTEX) {
        var value = callbackMaps.entryView(callbackType)
        WINDOW_LIB_LOG.info("[Window] onKeyboardHeightChange: callbackType=${callbackType}, callbacks=${value.value.getOrThrow().size}")

        let wrapper = {
            data: CPointer<Unit> =>
            let val = CPointer<UInt32>(data).read()
            WINDOW_LIB_LOG.info("[Window] keyboardHeight callback triggered: height=${val}")
            callback.invoke(None, val)
        }
        // ...
    }
}
```

---

### 4. 内存泄露检测

**方法**：使用 Native 内存分析工具

**证据**：`window.cj:290-297`、`display.cj:683-690`

**步骤**：
```bash
# 1. 使用 AddressSanitizer 编译
./build.sh --product-name {product} --ccache --args="--sanitize=address"

# 2. 运行应用并执行正常操作
./out/{product}/bin/window_test

# 3. 分析内存泄露报告
asan_report

# 4. 查找泄露源点
# 根据报告中的调用栈定位代码位置
```

---

## 性能问题

### 1. 回调注册慢

**问题现象**：大量回调注册时性能下降

**原因**：
- HashMap 扩容频繁
- Mutex 竞争导致性能下降

**证据**：`window.cj:270-286`（`callbackMaps`）

**定位方法**：
1. 使用性能分析工具（如 Profiler）
2. 检查回调注册频率
3. 优化 HashMap 初始容量

**解决方案**：
```cangjie
// 预初始化 HashMap 容量
let callbackMaps = HashMap<String, ArrayList<(CallbackObject, Int64)>>(
    [
        ("windowSizeChange", ArrayList<(CallbackObject, Int64)>(capacity: 16)),  // 预设容量
        ("avoidAreaChange", ArrayList<(CallbackObject, Int64)>(capacity: 8)),
        // ...
    ]
)
```

---

### 2. FFI 调用开销

**问题现象**：频繁调用 FFI 函数导致性能下降

**原因**：
- 仓颉与 Native 层的跨语言调用开销
- 频繁的属性设置（如每帧更新）

**证据**：所有 FFI 调用点（`window.cj:32-146`）

**定位方法**：
1. 使用性能分析工具
2. 检查 FFI 调用热点
3. 优化调用频率（合并设置、批量操作）

**解决方案**：
```cangjie
// 批量设置属性，减少 FFI 调用
// 不推荐：频繁调用单个 setter
setWindowBrightness(0.5)
setWindowBackgroundColor("#FFFFFF")
setWindowFocusable(true)

// 推荐：使用批量设置方法（如果 Native 层支持）
// 或在应用层做节流和批处理
```

---

## 架构相关问题

### 1. WindowStage vs Window 混淆

**问题现象**：不清楚何时使用 Window 还是 WindowStage

**原因**：
- 两个类都提供窗口相关功能
- 文档说明不够清晰

**证据**：`window.cj`（Window 类）、`window_stage.cj`（WindowStage 类）

**区别**：
| 特性 | Window | WindowStage |
|------|--------|--------------|
| 用途 | 窗口实例管理 | 窗口阶段管理，管理主窗口和子窗口 |
| 关键 API | `findWindow()`, `createWindow()` | `getMainWindow()`, `createSubWindow()`, `loadContent()` |
| 上下文 | BaseContext（通用） | 必须来自 UIAbilityContext |
| 生命周期 | 独立管理 | 与窗口生命周期绑定 |

**使用建议**：
- 管理单个窗口时使用 `Window`
- 管理主窗口和子窗口时使用 `WindowStage`
- 需要加载页面内容时使用 `WindowStage.loadContent()`

---

### 2. Display 属性缓存

**问题现象**：频繁访问 Display 属性导致性能问题

**原因**：
- 每次访问都触发 FFI 调用
- Native 层可能未缓存

**证据**：`display.cj:496-681`

**定位方法**：
1. 分析哪些 Display 属性被频繁访问
2. 考虑在应用层缓存
3. 使用属性变化回调代替轮询

**解决方案**：
```cangjie
// 不推荐：频繁轮询
while (true) {
    let width = display.width  // 每次 FFI 调用
    sleep(16)  // 轮询间隔
}

// 推荐：使用属性变化回调（如果 Native 层支持）
// 或者在应用层做智能缓存
class DisplayCache {
    let cachedWidth: UInt32
    let cachedHeight: UInt32

    public func getWidth(): UInt32 {
        if (needsRefresh) {
            cachedWidth = display.width
        }
        return cachedWidth
    }
}
```

---

## 关键结论

| 问题类别 | 问题数量 | 解决方案数量 |
|----------|----------|--------------|
| 构建问题 | 3 | 3 |
| 运行时问题 | 4 | 4 |
| 调试技巧 | 4 | 4 |
| 性能问题 | 2 | 2 |
| 架构问题 | 2 | 2 |

**证据**：基于代码分析（BUILD.gn、错误处理、日志模块等）

---

## 相关文档

- [04_External_API.md](./04_External_API.md) - API 错误码详细说明
- [08_Security_Review.md](./08_Security_Review.md) - 安全问题和修复建议

---

**生成时间**: 2025-02-06
