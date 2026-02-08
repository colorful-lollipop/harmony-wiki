# 常见问题 (FAQ)

## 目的

本文档汇总 ace_engine_lite 开发和调试过程中的常见问题、解决方案和调试路径。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 构建和运行时相关问题

---

## 构建相关问题

### Q1: 编译错误 "undefined reference to XXX"

**症状**：
```
error: undefined reference to 'JSI::CreateObject'
```

**原因**：缺少 JSI 头文件引用

**解决方案**：
1. 检查 `include_dirs` 是否正确设置
2. 确保所有依赖的 `public_deps` 正确声明
3. 清理编译缓存：`gn clean`

**相关配置**：`ace_lite.gni:74-142`

---

### Q2: 链接错误 "cannot find -ljerryscript"

**症状**：
```
error: cannot find -ljerryscript
```

**原因**：JerryScript 静态库未编译或路径错误

**解决方案**：
1. 检查 JerryScript 是否正确编译
2. 验证 `jerry_engine` 目标是否正确构建
3. 检查 `jerry-core` 依赖路径

**相关文件**：
- `//third_party/jerryscript/jerry-engine`（liteos_m）
- `//third_party/jerryscript/jerry-core`（liteos_a/linux）

---

### Q3: 特性开关导致编译失败

**症状**：启用了某个 FEATURE 后编译错误

**原因**：依赖的子系统未启用或功能不完整

**解决方案**：
1. 检查 `bundle.json` 中依赖的部件是否启用
2. 查看 `ace_lite.gni` 中的条件编译
3. 确保对应的 feature 宏已定义

**相关文件**：
- `bundle.json:24-47`（deps 列表）
- `ace_lite.gni:24-72`（部件依赖）

---

## 运行时问题

### Q4: requireNative 返回 undefined

**症状**：
```javascript
const module = requireNative("system.app");
console.log(module); // undefined
```

**原因**：模块未注册或名称错误

**解决方案**：
1. 检查模块名称是否正确（区分大小写）
2. 检查 `OHOS_MODULES` 数组是否包含该模块
3. 检查对应的 feature flag 是否启用
4. 查看 `ModuleManager::RequireModule` 的错误日志

**相关代码**：
- `frameworks/module_manager/ohos_module_config.h:97-158`（OHOS_MODULES 定义）
- `frameworks/module_manager/module_manager.cpp:30-66`（RequireModule 实现）

---

### Q5: 渲染无显示

**症状**：应用运行但界面为空白

**原因**：
1. `render()` 函数未调用或返回错误
2. JS 代码语法错误导致解析失败
3. 组件未正确添加到视图树
4. 样式未正确应用

**调试步骤**：
1. 检查 HILOG 输出是否有错误信息
2. 使用 `console.log()` 调试 JS 代码执行流程
3. 在 `RenderComponent()` 中添加断点或日志
4. 检查组件是否正确返回 UIView*

**相关代码**：
- `frameworks/src/core/context/js_app_context.cpp:177-194`（Eval 和 Render）
- `frameworks/src/core/components/component.cpp`（RenderComponent）

---

### Q6: 路由不工作

**症状**：
```javascript
router.replace({uri: 'About'});
// 页面未切换
```

**原因**：
1. 页面路径错误
2. params 格式错误
3. Ability 状态异常
4. JS Ability 未正确初始化

**调试步骤**：
1. 检查目标页面文件是否存在
2. 验证 JS 代码中是否定义了 `ROUTER_PAGE` 对象
3. 检查 `StateMachine::Init()` 是否成功
4. 查看 `JsPageStateMachine::Init()` 的错误日志

**相关代码**：
- `frameworks/src/core/modules/router_module.cpp:32-54`（Replace API）
- `frameworks/src/core/router/js_page_state_machine.cpp:159-177`（StateMachine::Init）

---

### Q7: 样式不生效

**症状**：内联样式或 class 选择器未应用

**原因**：
1. 样式表未正确初始化
2. 样式优先级问题
3. 选择器语法错误
4. 条件仲裁器问题

**调试步骤**：
1. 检查 `.css` 文件语法和格式
2. 在 `AppStyleManager::InitStyleSheet()` 中添加日志
3. 使用浏览器开发工具检查样式解析
4. 验证组件 ID 和 class 名称

**相关代码**：
- `frameworks/src/core/stylemgr/app_style_manager.cpp:91-106`（InitStyleSheet）
- `frameworks/src/core/stylemgr/app_style_sheet.cpp:44-71`（InitSheet）

---

### Q8: 内存泄漏

**症状**：应用运行一段时间后崩溃或性能下降

**原因**：
1. JSIValue 未正确释放（引用计数泄漏）
2. Component 未正确清理资源
3. 事件监听器未注销
4. 定时器未清除

**调试步骤**：
1. 使用内存分析工具（如 JerryScript 内存快照）
2. 在构造函数和析构函数中添加日志
3. 检查 `JSI::ReleaseValue()` 调用是否完整
4. 使用 `bounds_checking_function` 检测内存越界

**相关代码**：
- `frameworks/src/core/components/component.cpp`（析构函数）
- `frameworks/src/core/base/scope_js_value.h:31-61`（ScopeJSValue）
- `frameworks/src/core/context/js_timer_list.cpp`（定时器管理）

---

## 开发工具问题

### Q9: Qt 模拟器无法启动

**症状**：编译成功但模拟器可执行文件无法运行

**原因**：
1. 缺少 Mock 服务依赖
2. Qt 版本不兼容
3. 环境变量未正确设置
4. 资源文件路径错误

**解决方案**：
1. 检查 Qt 安装：`qt5` 或更高版本
2. 查看模拟器日志：`frameworks/tools/qt/simulator/`
3. 确保所有 mock 服务正确编译
4. 设置正确的环境变量（如 `QT_QPA_PLATFORM`）

**相关文件**：
- `frameworks/tools/qt/simulator/`（模拟器源码）
- `frameworks/targets/simulator/BUILD.gn`（模拟器构建配置）

---

### Q10: Snapshot 模式问题

**症状**：应用启动后立即崩溃或行为异常

**原因**：
1. `.bc` 字节码文件损坏或过期
2. 运行时模式切换失败
3. JS 引擎初始化顺序问题

**解决方案**：
1. 重新构建应用生成正确的 `.bc` 文件
2. 删除 `js_snapshot_enable` 文件强制 JS 模式
3. 清理编译产物重新构建
4. 检查 `JsAppEnvironment::InitJsFramework()` 初始化顺序

**相关代码**：
- `frameworks/src/core/context/js_app_context.cpp:132-164`（文件读取和模式选择）
- `frameworks/src/core/context/js_app_environment.cpp:84`（JS 框架初始化）

---

## 配置问题

### Q11: Feature flag 未生效

**症状**：启用了某个 FEATURE 但编译时未包含相应代码

**原因**：
1. GN 配置文件中的条件判断错误
2. Feature 名称拼写错误
3. 宏定义位置错误
4. 依赖未启用导致功能不可用

**解决方案**：
1. 检查 `ace_lite.gni` 中的 feature 定义
2. 检查对应 `.gni` 或 `BUILD.gn` 中的 `if` 条件
3. 使用 `gn args --current` 查看所有有效变量
4. 确保依赖的部件已启用（通过 bundle.json）

**相关文件**：
- `ace_lite.gni`（全局 feature 定义）
- `frameworks/targets/liteos_a/acelite_config.h`（平台特定配置）

---

## 调试技巧

### 使用 HILOG 日志

```javascript
// 在 JS 代码中输出调试信息
const util = requireNative("system.util");
util.log("Debug: current state = " + state);
```

**证据**：`frameworks/src/core/modules/presets/console_module.cpp`

### 使用 JerryScript 调试器

```cpp
// 通过 JSI::CreateErrorWithCode() 创建带错误码的异常
JSI::CreateErrorWithCode(JSI_ERR_CODE_OPERATION_FAILED, "Operation failed");
```

**证据**：`interfaces/inner_api/builtin/jsi/jsi.h`

### 性能分析

**启用 Profiler**：
```gn
# 在 BUILD.gn 中启用
ohos_build_type == "debug"
```

**相关代码**：`frameworks/src/core/modules/presets/profiler_module.cpp`

---

## 性能优化建议

### 减少重新渲染

1. 合并连续的 DOM 操作
2. 使用文档片段批量更新
3. 避免频繁的样式重新计算

### 减少内存分配

1. 重用 JSIValue 对象
2. 使用对象池管理组件
3. 及时释放临时对象

### 异步优化

1. 将耗时操作放入异步任务
2. 使用 `JsAsyncWork::DispatchAsyncWork()`
3. 避免阻塞 JS 主线程

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 概览
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [04_JS_API.md](04_JS_API.md) - JS API 参考
- [06_GN_Targets.md](06_GN_Targets.md) - GN 构建目标
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物
- [08_Security_Review.md](08_Security_Review.md) - 安全风险评审
