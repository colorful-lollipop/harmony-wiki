# 目录结构与模块职责 (Directory Structure)

## 顶层目录

```
arkui_cangjie_wrapper/
├── figures/                    # README 架构图
├── kit/                       # Cangjie ArkUI Kit 接口（对外暴露）
│   └── ArkUI/                 # Kit 模块入口
├── ohos/                      # Cangjie ArkUI 框架实现（内部）
│   ├── animator/              # 动画接口
│   ├── arkui/                 # UI 组件与框架
│   │   ├── component/         # 80+ UI 组件
│   │   ├── component_snapshot/# 组件快照（未支持）
│   │   ├── component_utils/   # 组件工具类
│   │   ├── shape/             # 图形绘制组件
│   │   ├── state_macro_manage/# 状态管理宏定义
│   │   ├── state_management/  # 状态管理框架
│   │   └── ui_context/        # UI 上下文库
│   ├── base/                  # 基础类型定义
│   ├── curves/                # 动画曲线
│   ├── font/                  # 自定义字体管理
│   ├── measure/               # 文本测量计算
│   └── prompt_action/         # 提示对话框
└── test/                      # 测试用例目录（不计入文档范围）
```

## kit/ArkUI - Kit 接口层

**职责**: 对外暴露的 Cangjie API 入口，统一导出所有框架能力

### 目录结构

```
kit/ArkUI/
├── BUILD.gn                   # Kit 构建配置
└── index.cj                   # Kit 导出入口
```

### 代码证据

| 文件 | 行号 | 说明 |
|------|------|------|
| `kit/ArkUI/index.cj` | 16-42 | package 定义 + public import 导出 |

### 导出内容

```cangjie
package kit.ArkUI

public import ohos.display.*
public import ohos.display.Rect as DisplayRect
public import ohos.display.Orientation as DisplayOrientation
public import ohos.window.*
public import ohos.window.Rect as WindowRect
public import ohos.window.Orientation as WindowOrientation

public import ohos.base.*
public import ohos.arkui.component.*
public import ohos.arkui.state_management.*

public import ohos.curves.*

public import ohos.arkui.component_utils.*

public import ohos.arkui.shape.*

public import ohos.arkui.ui_context.*
```

## ohos/arkui/component - UI 组件模块

**职责**: 提供 80+ UI 组件的定义与封装

### 子模块列表

| 分类 | 组件数量 | 示例 |
|------|----------|------|
| **基础组件** | 10+ | Button, Text, Image, Column, Row |
| **布局组件** | 15+ | Flex, Stack, Grid, List, Scroll |
| **导航组件** | 5+ | Navigation, Tabs, Stepper |
| **弹窗组件** | 10+ | Dialog, AlertDialog, ActionSheet |
| **选择组件** | 10+ | Slider, Switch, Checkbox, Radio |
| **绘制组件** | 5+ | Canvas, Circle, Rect, Path |
| **高级组件** | 25+ | Web, Video, RichEditor, Calendar |

### 典型组件结构

```
component/{name}/
├── {name}.cj           # 组件定义
├── {name}_attr.cj     # 组件属性
└── {name}_method.cj   # 组件方法（可选）
```

### 示例: Button 组件

```
component/button/
├── button.cj
└── button_attr.cj
```

### 代码证据

| 文件 | 行号 | 说明 |
|------|------|------|
| `ohos/arkui/component/BUILD.gn` | 16-107 | 80+ 组件 target 定义 |
| `ohos/arkui/component/component.cj` | 18-103 | 组件导出聚合 |

## ohos/arkui/state_management - 状态管理

**职责**: 提供完整的状态管理能力

### 子模块列表

| 文件 | 职责 |
|------|------|
| `app_storage.cj` | 应用级全局状态存储 |
| `environment.cj` | 环境信息（设备类型、主题等）|
| `local_storage.cj` | 组件本地状态存储 |
| `local_storage_inter_op.cj` | 与 JS 的 LocalStorage 互操作 |
| `observable.cj` | 可观察对象基类 |
| `observed_array_list.cj` | 可观察数组列表 |
| `observed_property.cj` | 可观察属性实现 |
| `persistent_storage.cj` | 持久化状态存储 |
| `subscriber_manager.cj` | 订阅者管理 |
| `view_stack_processor.cj` | 视图栈处理 |

### 状态类型

| 类型 | 作用域 | 说明 |
|------|--------|------|
| **@State** | 组件内 | 组件内状态，变化触发组件重建 |
| **@Prop** | 父子 | 单向数据流，父→子 |
| **@Link** | 父子 | 双向绑定 |
| **@Provide** | 祖先→后代 | 跨层级提供状态 |
| **@Consume** | 后代 | 消费祖先提供 |
| **LocalStorage** | 组件实例 | 组件级存储 |
| **AppStorage** | 应用级 | 全局存储 |

## ohos/arkui/ui_context - UI 上下文

**职责**: 提供 UI 相关的全局能力

### 子模块列表

| 文件 | 职责 |
|------|------|
| `cj_animator.cj` | 动画控制器 |
| `cj_font.cj` | 自定义字体注册 |
| `cj_measure.cj` | 文本测量计算 |
| `cj_prompt_action.cj` | 弹窗提示（46KB，复杂模块）|
| `cj_router.cj` | 页面路由 |
| `cj_ui_context.cj` | UI 上下文主接口 |

## ohos/base - 基础类型

**职责**: 提供基础类型定义

### 子模块列表

| 文件 | 职责 |
|------|------|
| `callback_type.cj` | 回调类型定义 |
| `cj_concurrency.cj` | 并发相关类型 |
| `collection.cj` | 集合类型 |
| `color.cj` | 颜色类型（Color, ResourceColor 等）|
| `common_types.cj` | 公共类型定义 |
| `length.cj` | 长度类型（Length, LengthUnit 等）|
| `main_context.cj` | 主上下文 |
| `resource.cj` | 资源类型 |
| `reuse_params.cj` | 复用参数 |

### 关键类型

```cangjie
// 长度类型示例
public interface Length {
    prop value: Float64
    prop unitType: LengthUnit
}
```

## ohos/curves - 动画曲线

**职责**: 提供预定义的动画曲线

### 代码证据

| 文件 | 行号 | 说明 |
|------|------|------|
| `kit/ArkUI/index.cj` | 33 | `public import ohos.curves.*` |

## 模块依赖关系

```
kit.ArkUI
├── ohos.arkui.component (80+ 组件)
├── ohos.arkui.component_utils (工具类)
├── ohos.arkui.shape (形状组件)
├── ohos.arkui.state_management (状态管理)
├── ohos.arkui.ui_context (UI 上下文)
├── ohos.base (基础类型)
├── ohos.curves (动画曲线)
└── window (external: 窗口管理)

ohos.arkui.component
├── 80+ individual component modules
└── ace_engine:cj_frontend_ohos (external)

ohos.base
├── cangjie_ark_interop (external: FFI, labels, exception)
└── hiviewdfx_cangjie_wrapper (external: Hilog)
```

## 稳定性标注

| 模块 | 稳定性 | 依据 |
|------|--------|------|
| kit/ArkUI | Stable | 对外暴露的 Kit 接口 |
| ohos.arkui.component | Stable | 完整测试覆盖 |
| ohos.arkui.state_management | Stable | 核心功能 |
| ohos.base | Stable | 基础类型 |
| ohos/arkui/state_macro_manage | Internal | 宏实现，可能变化 |

## 相关文档

- [00_Overview.md](./00_Overview.md) - 项目概览
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [03_N-API.md](./03_N-API.md) - 组件 API 清单
