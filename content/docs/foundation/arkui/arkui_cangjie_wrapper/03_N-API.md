# N-API / Cangjie API 参考 (N-API)

## 概述

本文档列出 arkui_cangjie_wrapper 对外暴露的所有 Cangjie API，包括 UI 组件、状态管理、UI 上下文等模块。

### API 位置

- **Kit 入口**: `kit/ArkUI/index.cj`
- **模块实现**: `ohos/arkui/component/` 等

### API Level 标注

所有公开 API 均使用 `@APILevel` 宏标注：

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.ArkUI.ArkUI.Full"
]
```

---

## UI 组件 (UI Components)

### 基础组件 (Basic Components)

| 组件名 | 模块 | 关键属性 | 关键方法 | 备注 |
|--------|------|----------|----------|------|
| **Text** | `ohos.arkui.component.text` | `content`, `fontSize`, `fontColor` | - | 文本显示 |
| **Button** | `ohos.arkui.component.button` | `type`, `stateEffect` | `onClick` | 按钮 |
| **Image** | `ohos.arkui.component.image` | `src`, `objectFit`, `alt` | `onComplete`, `onError` | 图片 |
| **LoadingProgress** | `ohos.arkui.component.loading_progress` | `loadingProgress`, `color` | - | 加载指示器 |
| **Progress** | `ohos.arkui.component.progress` | `value`, `total`, `type` | - | 进度条 |
| **Slideros.arkui** | `oh.component.slider` | `value`, `min`, `max`, `step` | `onChange` | 滑动条 |
| **Rating** | `ohos.arkui.component.rating` | `rating`, `indicator` | `onChange` | 评分 |
| **Checkbox** | `ohos.arkui.component.checkbox` | `select`, `changeable` | `onChange` | 复选框 |
| **CheckboxGroup** | `ohos.arkui.component.checkbox_group` | `selectAll`, `changeable` | `onChange` | 复选框组 |
| **Radio** | `ohos.arkui.component.radio` | `checked`, `changeable` | `onChange` | 单选框 |
| **Switch** | `ohos.arkui.component.switch` | `isOn`, `changeable` | `onChange` | 开关 |
| **Toggle** | `ohos.arkui.component.toggle` | `isOn`, `toggleType` | `onChange` | 切换按钮 |

### 文本输入组件 (Text Input)

| 组件名 | 模块 | 关键属性 | 关键方法 | 备注 |
|--------|------|----------|----------|------|
| **TextInput** | `ohos.arkui.component.text_input` | `text`, `placeholder`, `type` | `onChange`, `onSubmit` | 单行输入 |
| **TextArea** | `ohos.arkui.component.text_area` | `text`, `placeholder` | `onChange` | 多行输入 |
| **Search** | `ohos.arkui.component.search` | `value`, `placeholder` | `onSubmit`, `onChange` | 搜索框 |

### 选择器组件 (Pickers)

| 组件名 | 模块 | 关键属性 | 关键方法 | 备注 |
|--------|------|----------|----------|------|
| **DatePicker** | `ohos.arkui.component.date_picker` | `start`, `end`, `selected` | `onChange` | 日期选择器 |
| **TextPicker** | `ohos.arkui.component.text_picker` | `range`, `selected` | `onChange` | 文本选择器 |
| **TimePicker** | `ohos.arkui.component.text_timer` | `hour`, `minute` | `onChange` | 时间显示 |

### 容器组件 (Containers)

| 组件名 | 模块 | 关键属性 | 关键方法 | 备注 |
|--------|------|----------|----------|------|
| **Column** | `ohos.arkui.component.column` | `space`, `justifyContent` | `alignItems` | 垂直布局 |
| **Row** | `ohos.arkui.component.row` | `space`, `justifyContent` | `alignItems` | 水平布局 |
| **Flex** | `ohos.arkui.component.flex` | `direction`, `wrap`, `justifyContent` | - | 弹性布局 |
| **Stack** | `ohos.arkui.component.stack` | `alignContent` | - | 层叠布局 |
| **Grid** | `ohos.arkui.component.grid` | `columns`, `rows`, `gap` | `onScrollIndex` | 网格布局 |
| **List** | `ohos.arkui.component.list` | `space`, `initialIndex` | `onScrollIndex`, `onReachStart`, `onReachEnd` | 列表 |
| **Scroll** | `ohos.arkui.component.scroll` | `scrollable`, `friction` | `scrollTo`, `scrollEdge` | 滚动容器 |
| **Swiper** | `ohos.arkui.component.swiper` | `autoplay`, `interval`, `indicator` | `showNext`, `showPrevious` | 轮播 |
| **Tabs** | `ohos.arkui.component.tab` | `barPosition`, `vertical` | `onChange` | 标签页 |
| **Navigation** | `ohos.arkui.component.navigation` | `title`, `menus` | `pushUrl`, `pop` | 导航容器 |

### 高级组件 (Advanced)

| 组件名 | 模块 | 关键属性 | 关键方法 | 备注 |
|--------|------|----------|----------|------|
| **Web** | `ohos.arkui.component.web` | `src`, `domStorageAccess` | `onPageStart`, `onPageEnd` | Web 视图 |
| **Video** | `ohos.arkui.component.video` | `src`, `autoplay`, `loop` | `start`, `pause`, `stop` | 视频播放 |
| **RichEditor** | `ohos.arkui.component.rich_editor` | `typoGraphy`, `caretColor` | `onReady`, `onSelect` | 富文本编辑 |
| **Canvas** | `ohos.arkui.component.canvas` | `width`, `height` | `clearRect`, `fillRect` | 自定义绘制 |
| **QRCode** | `ohos.arkui.component.qrcode` | `value`, `imageSize` | - | 二维码 |
| **Badge** | `ohos.arkui.component.badge` | `count`, `style` | - | 徽标 |
| **AlphabetIndexer** | `ohos.arkui.component.alphabet_indexer` | `arrayValue`, `selected` | `onSelect` | 字母索引 |
| **Gauge** | `ohos.arkui.component.gauge` | `value`, `min`, `max` | - | 仪表盘 |
| **Counter** | `ohos.arkui.component.counter` | `value`, `step` | `onInc`, `onDec` | 计数器 |
| **DataPanel** | `ohos.arkui.component.data_panel` | `dataValues`, `max` | - | 数据面板 |

### 弹窗组件 (Dialogs)

| 组件名 | 模块 | 关键属性 | 关键方法 | 备注 |
|--------|------|----------|----------|------|
| **AlertDialog** | `ohos.arkui.component.alert_dialog` | `title`, `message`, `primaryButton` | - | 警告弹窗 |
| **ActionSheet** | `ohos.arkui.component.action_sheet` | `title`, `message`, `sheets` | - | 操作菜单 |
| **CustomDialog** | `ohos.arkui.component.custom_dialog` | `controller` | `open`, `close` | 自定义弹窗 |

---

## 状态管理 (State Management)

### 状态装饰器

| 装饰器 | 类型 | 同步方向 | 作用域 | 说明 |
|--------|------|----------|--------|------|
| **@State** | 观察者 | - | 组件内 | 组件内状态，变化触发重建 |
| **@Prop** | 单向 | 父→子 | 父子组件 | 单向数据流 |
| **@Link** | 双向 | 父↔子 | 父子组件 | 双向绑定 |
| **@Provide** | 提供 | 祖先→后代 | 跨层级 | 提供给后代消费 |
| **@Consume** | 消费 | 后代←祖先 | 跨层级 | 消费祖先提供 |
| **@ObjectLink** | 链接 | - | 复杂对象 | 对象引用观察 |

### 存储类

| 类名 | 模块 | 说明 |
|------|------|------|
| **LocalStorage** | `ohos.arkui.state_management` | 组件本地存储 |
| **AppStorage** | `ohos.arkui.state_management` | 应用级全局存储 |
| **PersistentStorage** | `ohos.arkui.state_management` | 持久化存储 |
| **Environment** | `ohos.arkui.state_management` | 环境信息 |

### 可观察类型

| 类名 | 模块 | 说明 |
|------|------|------|
| **ObservedProperty** | `ohos.arkui.state_management` | 可观察属性 |
| **ObservedObject** | `ohos.arkui.state_management` | 可观察对象 |
| **ObservedArrayList** | `ohos.arkui.state_management` | 可观察数组 |
| **Observable** | `ohos.arkui.state_management` | 可观察基类 |

---

## UI 上下文 (UI Context)

### 路由 (Router)

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| **pushUrl** | `options: RouterOptions` | `Promise<boolean>` | 跳转到新页面 |
| **replaceUrl** | `options: RouterOptions` | `Promise<boolean>` | 替换当前页面 |
| **back** | `url?: string` | void | 返回上一页 |
| **getLength** | - | `number` | 获取页面栈长度 |
| **getState** | - | `RouterState` | 获取页面状态 |

### 弹窗 (PromptAction)

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| **showToast** | `options: ToastOptions` | `Promise<void>` | 显示Toast |
| **showDialog** | `options: DialogOptions` | `Promise<DialogResponse>` | 显示对话框 |
| **showActionMenu** | `options: ActionMenuOptions` | `Promise<ActionMenuResponse>` | 显示操作菜单 |

### 动画 (Animator)

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| **animateTo** | `options: AnimateOptions`, `event: () => void` | `Promise<void>` | 补间动画 |
| **createAnimator** | `options: AnimatorOptions` | `Animator` | 动画控制器 |

### 字体 (Font)

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| **registerFont** | `options: FontOptions` | `Promise<void>` | 注册自定义字体 |

### 测量 (Measure)

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| **measureText** | `options: MeasureOptions` | `TextMetrics` | 测量文本尺寸 |
| **measureTextSize** | `options: MeasureOptions` | `Size` | 测量文本尺寸（Size）|

---

## 基础类型 (Base Types)

### 长度类型

| 类型 | 模块 | 说明 |
|------|------|------|
| **Length** | `ohos.base` | 长度值抽象接口 |
| **LengthType** | `ohos.base` | 长度类型（VP, FP, PX, LPX）|
| **LengthUnit** | `ohos.base` | 长度单位枚举 |

### 颜色类型

| 类型 | 模块 | 说明 |
|------|------|------|
| **Color** | `ohos.base` | 颜色枚举 |
| **ResourceColor** | `ohos.base` | 资源颜色 |
| **EdgeEffect** | `ohos.base` | 边缘效果 |

### 公共类型

| 类型 | 模块 | 说明 |
|------|------|------|
| **Callback** | `ohos.base` | 回调函数类型 |
| **Resource** | `ohos.base` | 资源引用 |
| **Align** | `ohos.base` | 对齐方式 |
| **Direction** | `ohos.base` | 布局方向 |

---

## 动画曲线 (Curves)

| 曲线名 | 说明 |
|--------|------|
| **Linear** | 线性曲线 |
| **Ease** | 缓入缓出 |
| **EaseIn** | 缓入 |
| **EaseOut** | 缓出 |
| **EaseInOut** | 缓入缓出 |
| **FastOutSlowIn** | 快出慢入 |
| **Rhythm** | 节奏曲线 |
| **Smooth** | 平滑曲线 |
| **Spring** | 弹簧曲线 |

---

## 组件通用属性

### 通用属性

| 属性 | 类型 | 说明 |
|------|------|------|
| **width** | `Length` | 宽度 |
| **height** | `Length` | 高度 |
| **size** | `Size` | 尺寸 |
| **padding** | `Length`/`Padding` | 内边距 |
| **margin** | `Length`/`Margin` | 外边距 |
| **backgroundColor** | `ResourceColor` | 背景色 |
| **opacity** | `number` | 透明度 |
| **enabled** | `boolean` | 是否启用 |
| **visible** | `boolean` | 是否可见 |

### 通用事件

| 事件 | 参数 | 说明 |
|------|------|------|
| **onClick** | `() => void` | 点击事件 |
| **onTouch** | `(event: TouchEvent) => void` | 触摸事件 |
| **onHover** | `(isHover: boolean) => void` | 悬停事件 |
| **onFocus** | `() => void` | 获得焦点 |
| **onBlur** | `() => void` | 失去焦点 |

---

## API 变更历史

| 版本 | 变更内容 |
|------|----------|
| 22 | 初始 API Level，支持基础 UI 组件和状态管理 |

---

## 代码证据

| 模块 | 证据位置 |
|------|----------|
| Kit 导出 | `kit/ArkUI/index.cj:16-42` |
| 组件聚合 | `ohos/arkui/component/component.cj:18-103` |
| 路由 | `ohos/arkui/ui_context/cj_router.cj` |
| 弹窗 | `ohos/arkui/ui_context/cj_prompt_action.cj` |
| 状态管理 | `ohos/arkui/state_management/local_storage.cj` |

---

## 相关文档

- [00_Overview.md](./00_Overview.md) - 项目概览
- [01_Directory_Structure.md](./01_Directory_Structure.md) - 目录结构
- [02_Architecture.md](./02_Architecture.md) - 架构设计
