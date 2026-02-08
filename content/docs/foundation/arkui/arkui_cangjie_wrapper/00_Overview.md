# 项目概览 (Overview)

## 项目定位

**arkui_cangjie_wrapper** 是 OpenHarmony 系统中 **ArkUI 开发框架的 Cangjie 语言封装层**，为使用 Cangjie 语言开发应用 UI 提供声明式 UI 能力。

### 核心定位

| 维度 | 描述 |
|------|------|
| **语言绑定** | Cangjie ↔ ArkUI 引擎的 FFI 桥接层 |
| **能力范围** | UI 组件、状态管理、动画、绘制、交互事件 |
| **适用设备** | Standard 设备（标准设备） |
| **版本状态** | Beta 特性 (v6.1) |

### 项目边界

```
┌─────────────────────────────────────────────────────────────┐
│                    arkui_cangjie_wrapper                    │
├─────────────────────────────────────────────────────────────┤
│  ✅ 职责范围                                                 │
│     - Cangjie 语法的 UI 组件声明与封装                       │
│     - 状态管理宏 (@State, @Prop, @Link 等)                  │
│     - UI 上下文 API (动画、路由、字体、测量)                 │
│     - 与 ace_engine 的 FFI 接口定义                         │
├─────────────────────────────────────────────────────────────┤
│  ❌ 非职责范围                                               │
│     - UI 渲染引擎核心实现 (属于 arkui_ace_engine)            │
│     - 系统能力管理 (属于 access_token)                     │
│     - 资源管理 (属于 global_cangjie_wrapper)                │
│     - 多媒体处理 (属于 multimedia_cangjie_wrapper)          │
└─────────────────────────────────────────────────────────────┘
```

### 代码证据

- **bundle.json**: `//foundation/arkui/arkui_cangjie_wrapper`
- **依赖声明**: 依赖 `ace_engine:cj_frontend_ohos` 作为前端引擎

## 核心能力

### 能力矩阵

| 能力类别 | 支持情况 | 说明 |
|----------|----------|------|
| **UI 组件** | ✅ 80+ 组件 | 文本、布局、绘制、媒体、导航等 |
| **状态管理** | ✅ 完整支持 | @State, @Prop, @Link, @Provide, @Consume 等 |
| **动画** | ✅ 支持 | animateTo, animator 等 |
| **页面路由** | ✅ 支持 | pushUrl, replaceUrl, back 等 |
| **自定义字体** | ✅ 支持 | registerFont 等 |
| **弹窗** | ✅ 支持 | promptAction 等 |
| **组件快照** | ❌ 未支持 | Component Screenshot |
| **主题换肤** | ❌ 未支持 | Theme Switching |
| **3D 渲染** | ❌ 未支持 | Component3D |

### 完整组件列表

详见 **[03_N-API.md](./03_N-API.md)**

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **系统类型** | OpenHarmony Standard |
| **API Level** | 22+ |
| **系统能力** | SystemCapability.ArkUI.ArkUI.Full |

### 资源占用

| 资源 | 大小 |
|------|------|
| **ROM** | 8930 KB |
| **RAM** | 8100 KB |

### 外部依赖

| 依赖仓库 | 用途 |
|----------|------|
| `arkui_ace_engine` | UI 后端引擎、Cangjie 前端 |
| `cangjie_ark_interop` | API 管理、FFI、标签系统 |
| `access_token` | 权限鉴权 |
| `global_cangjie_wrapper` | 资源管理 |
| `multimedia_cangjie_wrapper` | 图像处理 (PixelMap) |
| `arkweb_cangjie_wrapper` | WebView 控制 |
| `hiviewdfx_cangjie_wrapper` | 日志 (Hilog) |

## 关键概念

### Cangjie 声明式 UI

与 ArkTS 类似，Cangjie 使用 `@Component` 装饰器和 `build()` 方法声明 UI：

```cangjie
@Component
class EntryView {
    @State var message: String = "Hello, Cangjie ArkUI!"
    
    func build() {
        Column {
            Text(this.message)
                .fontSize(20)
                .fontColor(Color.Blue)
            
            Button("点击我")
                .onClick(() => {
                    this.message = "按钮被点击了！"
                })
        }
        .justifyContent(FlexAlign.Center)
    }
}
```

### 状态管理

- **@State**: 组件内状态，变化触发组件重建
- **@Prop**: 单向同步，父→子
- **@Link**: 双向绑定，父↔子
- **@Provide**: 提供给后代
- **@Consume**: 消费祖先提供
- **LocalStorage**: 组件本地存储
- **AppStorage**: 应用级全局存储

### 目录结构

详见 **[01_Directory_Structure.md](./01_Directory_Structure.md)**

### 架构设计

详见 **[02_Architecture.md](./02_Architecture.md)**

## 版本信息

| 版本 | 日期 | 变更 |
|------|------|------|
| 6.1 | 2025-02 | 当前版本 |

## 相关文档

- [README.md](../README.md)
- [API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/summary_cjnative_ohos.md)
- [开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/arkui-cj/cj-ui-development-overview.md)
