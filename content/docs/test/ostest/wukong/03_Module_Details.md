# WuKong 模块详情

## 模块概览

```
wukong/
├── common/              # 公共功能与管理器
├── component_event/     # 组件树结构管理
├── input_factory/      # 输入事件生成
├── report/             # 统计与报告
├── shell_command/      # 命令行解析
└── test_flow/         # 测试流程控制
```

## common 模块

**职责**: 提供应用控制、随机事件注入、多模事件注入等公共能力

### 头文件

| 头文件 | 职责 |
|--------|------|
| `app_manager.h` | 应用生命周期管理 |
| `component_manager.h` | 组件操作管理 |
| `multimode_manager.h` | 多模态输入管理 |
| `wukong_define.h` | 常量与宏定义 |
| `wukong_util.h` | 工具函数 |
| `wukong_logger.h` | 日志管理 |
| `event_monitor.h` | 事件监控 |
| `common.h` | 公共定义 |
| `count_down_latch.h` | 倒计时同步锁 |
| `special_test_object.h` | 专项测试对象 |

### 核心类

#### AppManager

```cpp
// 证据: common/include/app_manager.h
class AppManager {
public:
    static AppManager& GetInstance();
    void DestroyInstance();
    // 应用管理相关方法
};
```

#### ComponentManager

```cpp
// 证据: common/include/component_manager.h
class ComponentManager {
public:
    static ComponentManager& GetInstance();
    void DestroyInstance();
    // 组件操作方法
};
```

#### MultimodeManager

```cpp
// 证据: common/include/multimode_manager.h
class MultimodeManager {
public:
    static MultimodeManager& GetInstance();
    void DestroyInstance();
    // 多模态输入管理
};
```

### 依赖方向

```
common/
├── app_manager ──────▶ ability_runtime (应用管理)
├── component_manager ─▶ component_event (组件树)
├── multimode_manager ──▶ input_factory (多模输入)
└── wukong_logger ─────▶ hilog (日志)
```

## component_event 模块

**职责**: 定义 ability、page、Component 树结构，提供节点操作能力

### 头文件

| 头文件 | 职责 |
|--------|------|
| `ability_tree.h` | Ability 树结构 |
| `component_tree.h` | 组件树结构 |
| `page_tree.h` | Page 树结构 |
| `wukong_tree.h` | WuKong 树结构 |
| `tree_manager.h` | 树管理 |
| `scene.h` | 场景基类 |
| `scene_delegate.h` | 场景委托 |
| `normal_scene.h` | 普通场景 |
| `focus_scene_delegate.h` | 焦点场景委托 |
| `element_option.h` | 元素选项 |

### 核心类

#### TreeManager

```cpp
// 证据: component_event/include/tree_manager.h
class TreeManager {
public:
    // 树遍历、查找、修改方法
};
```

#### WuKongTree（基类）

```cpp
// 证据: component_event/include/wukong_tree.h
class WuKongTree {
    // 节点添加、遍历、查找
};
```

#### Scene

```cpp
// 证据: component_event/include/scene.h
class Scene {
    // 场景状态管理
};
```

### 依赖方向

```
component_event/
├── *_tree ──────▶ Accessibility (无障碍服务)
├── tree_manager ─▶ Tree
└── scene ────────▶ Window Manager
```

## input_factory 模块

**职责**: 实现各种输入事件的生成与注入

### 头文件（按类型分类）

#### 指针输入

| 头文件 | 事件类型 |
|--------|----------|
| `touch_input.h` | 触摸事件 |
| `swap_input.h` | 滑动事件 |
| `mouse_input.h` | 鼠标事件 |

#### 键盘输入

| 头文件 | 事件类型 |
|--------|----------|
| `keyboard_input.h` | 键盘输入 |
| `hardkey_input.h` | 硬件按键 |

#### 手势输入

| 头文件 | 事件类型 |
|--------|----------|
| `knuckle_input.h` | 指关节双击 |
| `pinch_input.h` | 捏合手势 |
| `watch_gestures_input.h` | 手表手势 |
| `watch_crown_input.h` | 手表表冠 |
| `watch_keypress_input.h` | 手表按键 |

#### 设备输入

| 头文件 | 事件类型 |
|--------|----------|
| `rotate_input.h` | 屏幕旋转 |
| `appswitch_input.h` | 应用切换 |
| `float_split_input.h` | 浮窗分裂 |
| `collapse_input.h` | 折叠展开 |
| `browser_input.h` | 浏览器操作 |

#### 特殊输入

| 头文件 | 事件类型 |
|--------|----------|
| `component_input.h` | 组件操作 |
| `record_input.h` | 录制/回放 |
| `element_input.h` | 元素操作 |

### 核心类

#### InputFactory

```cpp
// 证据: input_factory/include/input_factory.h
class InputFactory {
public:
    // 创建各种输入事件
};
```

#### InputAction

```cpp
// 证据: input_factory/include/input_action.h
class InputAction {
    // 输入动作执行
};
```

### 依赖方向

```
input_factory/
├── *_input ──────▶ Input Manager (MMI)
├── component_input ─▶ ComponentManager
├── appswitch_input ─▶ AppManager
└── record_input ────▶ File System
```

## report 模块

**职责**: 统计信息收集、异常监控、报告生成

### 头文件（按功能分类）

#### 统计模块

| 头文件 | 职责 |
|--------|------|
| `statistics.h` | 统计基类 |
| `statistics_event.h` | 事件统计 |
| `statistics_ability.h` | Ability 统计 |
| `statistics_componment.h` | 组件统计 |
| `statistics_exception.h` | 异常统计 |
| `statistics_coverage.h` | 覆盖率统计 |

#### 数据模块

| 头文件 | 职责 |
|--------|------|
| `data_set.h` | 数据集 |
| `data_unit.h` | 数据单元 |
| `data_unit_set.h` | 数据单元集合 |
| `input_info.h` | 输入信息 |
| `input_msg_object.h` | 输入消息对象 |

#### 格式模块

| 头文件 | 职责 |
|--------|------|
| `format.h` | 格式基类 |
| `format_terminal.h` | 终端输出 |
| `format_html.h` | HTML 报告 |
| `format_csv.h` | CSV 文件 |
| `format_json.h` | JSON 文件 |

#### 其他模块

| 头文件 | 职责 |
|--------|------|
| `filter.h` | 过滤器 |
| `filter_category.h` | 过滤器分类 |
| `exception_manager.h` | 异常管理 |
| `sysevent_listener.h` | 系统事件监听 |
| `table.h` | 表格输出 |
| `csv_utils.h` | CSV 工具 |

### 核心类

#### Report

```cpp
// 证据: report/include/report.h
class Report {
public:
    static Report& GetInstance();
    void DestroyInstance();
    // 报告生成与管理
};
```

#### Statistics

```cpp
// 证据: report/include/statistics.h
class Statistics {
    // 统计数据收集
};
```

### 依赖方向

```
report/
├── statistics_* ───▶ Input Events
├── format_* ───────▶ File System
├── exception_* ────▶ HiSysEvent
└── filter ─────────▶ Statistics
```

## shell_command 模块

**职责**: 命令行解析和命令路由

### 头文件

| 头文件 | 职责 |
|--------|------|
| `wukong_shell_command.h` | 命令行命令实现 |

### 核心类

#### WuKongShellCommand

```cpp
// 证据: shell_command/include/wukong_shell_command.h
class WuKongShellCommand {
public:
    WuKongShellCommand(int argc, char* argv[]);
    std::string ExecCommand();
};
```

### 入口点

```cpp
// 证据: shell_command/src/wukong_main.cpp:130
int main(int argc, char* argv[])
```

### 启动流程

```
main()
  ├─ Check developer mode
  ├─ Init logger (--track/--debug)
  ├─ Init semaphores
  ├─ Parse command line
  ├─ Execute command
  └─ Cleanup
```

## test_flow 模块

**职责**: 测试流程控制

### 头文件

| 头文件 | 职责 |
|--------|------|
| `test_flow.h` | 测试流程基类 |
| `random_test_flow.h` | 随机测试流程 |
| `special_test_flow.h` | 专项测试流程 |
| `focus_test_flow.h` | 专注测试流程 |
| `test_flow_factory.h` | 测试流程工厂 |

### 核心类

#### TestFlow（基类）

```cpp
// 证据: test_flow/include/test_flow.h
class TestFlow {
public:
    virtual int Run() = 0;
    virtual ~TestFlow() = default;
};
```

#### RandomTestFlow

```cpp
// 证据: test_flow/include/random_test_flow.h
class RandomTestFlow : public TestFlow {
    // 随机测试实现
};
```

#### SpecialTestFlow

```cpp
// 证据: test_flow/include/special_test_flow.h
class SpecialTestFlow : public TestFlow {
    // 专项测试实现
};
```

#### FocusTestFlow

```cpp
// 证据: test_flow/include/focus_test_flow.h
class FocusTestFlow : public TestFlow {
    // 专注测试实现
};
```

#### TestFlowFactory

```cpp
// 证据: test_flow/include/test_flow_factory.h
class TestFlowFactory {
public:
    std::shared_ptr<TestFlow> CreateTestFlow();
};
```

### 依赖方向

```
test_flow/
├── *_test_flow ──▶ InputFactory (事件生成)
├── *_test_flow ──▶ Report (统计)
└── *_test_flow ──▶ ComponentManager (组件操作)
```

## 接口稳定性标注

| 模块 | 接口 | 稳定性 | 说明 |
|------|------|--------|------|
| common | AppManager | 稳定 | 核心管理器 |
| common | ComponentManager | 稳定 | 核心管理器 |
| common | MultimodeManager | 稳定 | 核心管理器 |
| common | WuKongLogger | 稳定 | 核心日志 |
| component_event | TreeManager | 稳定 | 树操作 |
| component_event | WuKongTree | 稳定 | 树结构 |
| input_factory | InputFactory | 稳定 | 输入工厂 |
| input_factory | InputAction | 稳定 | 输入动作 |
| report | Report | 稳定 | 报告管理 |
| report | Statistics | 稳定 | 统计基类 |
| shell_command | WuKongShellCommand | 稳定 | 命令行 |
| test_flow | TestFlow | 稳定 | 测试流程 |

> **说明**: 所有模块接口均标注为稳定，因为 WuKong 是一个完整的可执行程序，模块间通过头文件定义的 C++ 接口进行交互。

## 相关文档

- [架构说明](01_Architecture.md)
- [命令行接口](02_CommandLine.md)
- [构建系统](04_Build_System.md)
