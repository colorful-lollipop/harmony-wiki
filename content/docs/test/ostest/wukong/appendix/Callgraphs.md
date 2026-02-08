# WuKong 关键调用链

## 入口调用链

### main() → 命令执行

```
shell_command/src/wukong_main.cpp:130
    │
    ├─ [1] 开发者模式检查
    │   └─ OHOS::system::GetBoolParameter("const.security.developermode.state")
    │
    ├─ [2] 初始化日志
    │   └─ WuKongLogger::GetInstance()->SetLevel()/Start()
    │
    ├─ [3] 初始化信号量
    │   ├─ NamedSemaphore::Open()/Create()
    │   └─ NamedSemaphore::Wait()/Post()
    │
    ├─ [4] 解析命令行
    │   └─ WuKongShellCommand::WuKongShellCommand(argc, argv)
    │
    ├─ [5] 执行命令
    │   └─ WuKongShellCommand::ExecCommand()
    │       │
    │       ├─> SpecialTestFlow::Run()
    │       ├─> RandomTestFlow::Run()
    │       └─> FocusTestFlow::Run()
    │
    └─ [6] 资源清理
        └─ FreeSingtion() → 各 Manager::DestroyInstance()
```

**证据**: `shell_command/src/wukong_main.cpp:130-174`

---

## 命令执行调用链

### WuKongShellCommand::ExecCommand()

```
wukong_shell_command.cpp
    │
    ├─ [1] 参数验证
    │   └─ TestFlowFactory::CreateTestFlow()
    │
    └─ [2] 流程创建
        └─ std::shared_ptr<TestFlow> → TestFlow::Run()
```

---

## 测试流程调用链

### RandomTestFlow::Run()

```
random_test_flow.cpp
    │
    ├─ [1] 输入工厂创建
    │   └─ InputFactory::CreateInputAction()
    │
    ├─ [2] 随机事件生成
    │   ├─ Random::Seed(s)
    │   └─ Random::Next(weight)
    │
    ├─ [3] 事件注入循环
    │   │
    │   ├─> InputFactory::Create*Input()  // touch/swap/keyboard...
    │   │   │
    │   │   ├─> TouchInput::Inject()
    │   │   ├─> SwapInput::Inject()
    │   │   ├─> KeyboardInput::Inject()
    │   │   └─> ... (其他输入类型)
    │   │
    │   ├─> Report::UpdateStatistics()
    │   │   └─> Statistics::AddEvent()
    │   │
    │   └─> Thread::Sleep(interval)
    │
    └─ [4] 结果报告
        └─ Report::GenerateReport()
```

**证据**: `test_flow/src/random_test_flow.cpp`, `input_factory/src/input_factory.cpp`

---

### SpecialTestFlow::Run()

```
special_test_flow.cpp
    │
    ├─ [1] 解析专项类型
    │   └─ SpecialTestObject::Parse()
    │
    ├─ [2] 执行专项测试
    │   │
    │   ├─> SleepWakeTest::Run()
    │   │   └─> MultimodeManager::Suspend/Resume()
    │   │
    │   ├─> SwapTest::Run()
    │   │   └─> SwapInput::Inject(start, end)
    │   │
    │   ├─> TouchTest::Run()
    │   │   └─> TouchInput::Inject(x, y)
    │   │
    │   └─> ComponentTest::Run()
    │       └─> ComponentInput::Traverse()
    │
    └─ [3] 报告生成
        └─ Report::GenerateReport()
```

**证据**: `test_flow/src/special_test_flow.cpp`, `common/include/special_test_object.h`

---

### FocusTestFlow::Run()

```
focus_test_flow.cpp
    │
    ├─ [1] 获取焦点类型
    │   └─ FocusType::Parse()
    │
    ├─ [2] 筛选目标组件
    │   └─ ComponentManager::FindByType()
    │
    ├─ [3] 深度测试
    │   │
    │   ├─> ComponentManager::GetChildNodes()
    │   ├─> ComponentInput::FocusInject()
    │   └─> InputFactory::CreateInputAction()
    │
    └─ [4] 统计报告
        └─ Report::GenerateReport()
```

**证据**: `test_flow/src/focus_test_flow.cpp`

---

## 组件树操作调用链

### 组件遍历

```
component_event/src/component_tree.cpp
    │
    ├─ [1] 构建树
    │   └─ ComponentTree::Build()
    │       └─ Accessibility::GetRootNodes()
    │
    ├─ [2] 遍历节点
    │   └─ ComponentTree::Traverse()
    │       └─ WuKongTree::VisitNode()
    │
    └─ [3] 查找节点
        └─ ComponentTree::FindById()
            └─ WuKongTree::Search()
```

**证据**: `component_event/include/component_tree.h`, `component_event/include/wukong_tree.h`

---

### Ability 树操作

```
component_event/src/ability_tree.cpp
    │
    ├─ [1] 获取 Ability 列表
    │   └─ AbilityManager::GetTopAbility()
    │
    ├─ [2] 构建 Ability 树
    │   └─ AbilityTree::Build()
    │
    └─ [3] 页面导航
        └─ AbilityManager::Navigate()
```

**证据**: `component_event/include/ability_tree.h`

---

## 报告生成调用链

### 统计收集

```
report/src/statistics.cpp
    │
    ├─ [1] 事件统计
    │   └─ StatisticsEvent::Collect()
    │
    ├─ [2] 组件统计
    │   └─ StatisticsComponent::Collect()
    │
    ├─ [3] 异常统计
    │   └─ StatisticsException::Collect()
    │       └─ ExceptionManager::GetEvents()
    │
    └─ [4] 覆盖率统计
        └─ StatisticsCoverage::Calculate()
```

---

### 格式输出

```
report/src/report.cpp
    │
    ├─ [1] 格式选择
    │   └─ FormatFactory::Create()
    │
    ├─ [2] 终端输出
    │   └─ FormatTerminal::Output()
    │
    ├─ [3] HTML 报告
    │   └─ FormatHtml::Generate()
    │       └─ libpng::SaveScreenshot()
    │
    ├─ [4] CSV 文件
    │   └─ FormatCsv::Save()
    │
    └─ [5] JSON 报告
        └─ FormatJson::Save()
```

**证据**: `report/include/report.h`, `report/include/format.h`

---

## 异常监控调用链

```
report/src/exception_manager.cpp
    │
    ├─ [1] 注册监听
    │   └─ SysEventListener::Register()
    │
    ├─ [2] 监听系统事件
    │   └─ HiSysEvent::Subscribe()
    │
    └─ [3] 异常处理
        └─ ExceptionManager::Handle()
            └─ HiSysEventWrite()
```

**证据**: `report/include/exception_manager.h`, `report/include/sysevent_listener.h`

---

## 输入事件调用链

### 触摸事件

```
input_factory/src/touch_input.cpp
    │
    ├─ [1] 创建事件
    │   └─ TouchEvent::Create()
    │
    ├─ [2] 设置坐标
    │   └─ TouchEvent::SetPosition(x, y)
    │
    └─ [3] 注入系统
        └─ InputManager::InjectTouchEvent()
```

---

### 键盘事件

```
input_factory/src/keyboard_input.cpp
    │
    ├─ [1] 生成键值
    │   └─ KeyboardEvent::Create()
    │
    ├─ [2] 设置键码
    │   └─ KeyboardEvent::SetKeyCode()
    │
    └─ [3] 注入系统
        └─ InputManager::InjectKeyEvent()
```

---

## 关键依赖调用

### 系统组件

| 调用 | 目标组件 | 说明 |
|------|----------|------|
| `AbilityManager::StartAbility()` | ability_runtime | 拉起应用 |
| `InputManager::InjectEvent()` | input | 注入输入事件 |
| `Accessibility::GetRootNodes()` | accessibility | 获取无障碍节点 |
| `WindowManager::GetDisplayInfo()` | window_manager | 获取显示信息 |
| `HiSysEventWrite()` | hisysevent | 写入系统事件 |
| `Hilog::Info()` | hilog | 输出日志 |
| `BundleManager::GetBundleList()` | bundle_framework | 获取应用列表 |

**证据**: `BUILD.gn:100-129` 依赖配置

---

## 调用图总结

```
                    ┌─────────────────┐
                    │  wukong_main    │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
     │  ShellCmd   │ │  TestFlow   │ │   Report    │
     └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
            │               │               │
            │     ┌────────┼────────┐      │
            │     ▼        ▼        ▼      │
            │  ┌─────────────────────┐    │
            │  │   InputFactory      │◄───┘
            │  │   (事件注入核心)    │
            │  └──────────┬──────────┘
            │             │
     ┌──────┼─────────────┼─────────────┐
     ▼      ▼             ▼             ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│Ability │ │ Input  │ │ Window │ │HiSys   │
│Manager │ │ Manager│ │ Manager│ │ Event  │
└────────┘ └────────┘ └────────┘ └────────┘
```
