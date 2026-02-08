# 项目概览

> window_cangjie_wrapper：OpenHarmony 窗口管理子系统的仓颉语言封装层

---

## 目的

本文档提供 `window_cangjie_wrapper` 项目的高层次概览，帮助新人快速理解项目定位、边界条件和核心能力。

---

## 适用范围

- ✅ 覆盖 `ohos.window` 和 `ohos.display` 两个仓颉模块
- ✅ 说明核心能力、运行环境、关键概念
- ✅ 列出与 ArkTS 版本的功能差异
- ❌ 不包含测试用例实现细节
- ❌ 不包含 Native 层（window_manager 子系统）实现细节

---

## 项目定位

### 子系统与组件

| 维度 | 值 |
|--------|-----|
| **子系统** | window（窗口管理） |
| **组件** | window_cangjie_wrapper |
| **版本** | 6.1 |
| **许可** | Apache License 2.0 |
| **API Level** | 22+ |
| **适用设备** | standard（标准设备） |

### 资源占用

根据 `bundle.json:21-18`：

| 资源 | 占用量 |
|--------|---------|
| **ROM** | 435KB |
| **RAM** | 401KB |

---

## 核心能力

### 窗口管理（ohos.window）

窗口管理模块提供窗口生命周期管理和属性设置能力：

| 能力类别 | 具体功能 |
|----------|----------|
| **窗口创建与查找** | `createWindow()`, `findWindow()`, `getLastWindow()` |
| **窗口显示控制** | `showWindow()`, `destroyWindow()`, `moveWindowTo()`, `resize()` |
| **窗口属性设置** | 背景色、亮度、焦点、保持亮屏、隐私模式、触摸响应等 |
| **窗口布局控制** | 全屏、沉浸式、宽高比、方向设置 |
| **系统栏管理** | 状态栏/导航栏颜色、图标、动画 |
| **窗口生命周期** | 最小化、最大化、恢复、焦点转移 |
| **回调机制** | 窗口事件、尺寸变化、避让区域、键盘高度等 |

**证据**：`ohos/window/window.cj` (895 行) - `foreign` 块定义了约 45 个 FFI 函数

### 显示设备管理（ohos.display）

显示设备管理模块提供显示信息查询和监听能力：

| 能力类别 | 具体功能 |
|----------|----------|
| **Display 查询** | `getDefaultDisplaySync()`, `getAllDisplays()` |
| **Display 属性** | ID、名称、状态、刷新率、方向、分辨率、DPI 等 |
| **折叠屏管理** | `isFoldable()`, `getFoldStatus()`, `getFoldDisplayMode()`, `getCurrentFoldCreaseRegion()` |
| **显示事件监听** | 显示设备增删、折叠状态变化、折叠角度变化等 |

**证据**：`ohos/display/display.cj` (726 行) - `foreign` 块定义了约 40 个 FFI 函数

---

## 运行环境

### 系统要求

| 要求项 | 描述 |
|--------|--------|
| **操作系统** | OpenHarmony API Level 22+ |
| **设备类型** | standard（标准设备） |
| **依赖子系统** | window（窗口管理）、ability_runtime（能力运行时） |

### 权限要求

| 权限 | 使用场景 | 证据 |
|--------|----------|--------|
| `ohos.permission.SYSTEM_FLOAT_WINDOW` | 创建 TYPE_FLOAT 窗口 | `window.cj:187` - `createWindow()` 注解 |
| `ohos.permission.PRIVACY_WINDOW` | 设置隐私窗口模式 | `window.cj:527` - `setWindowPrivacyMode()` 注解 |

### Syscap 要求

| Syscap | 说明 |
|--------|--------|
| `SystemCapability.WindowManager.WindowManager.Core` | 核心窗口管理能力 |
| `SystemCapability.Window.SessionManager` | 会话管理能力（部分 API） |

**证据**：所有 API 均使用 `@!APILevel[syscap: "..."]` 注解

---

## 关键概念

### FFI (Foreign Function Interface)

通过仓颉的 `foreign` 块调用 Native 层 C/C++ 函数：

```cangjie
foreign {
    func FfiOHOSCreateWindow(name: CString, windowType: UInt32, ...): RetDataI64
}
```

**特点**：
- 直接调用 `window_manager:cj_window_ffi` 和 `cj_display_ffi` 中的实现
- 使用 `RetDataI64`, `RetStruct` 等结构体返回数据
- 通过 `unsafe` 块进行指针操作

**证据**：`ohos/window/window.cj:32-146`、`ohos/display/display.cj:27-93`

### RemoteDataLite

继承自 `ohos.ffi.RemoteDataLite` 的基类，用于管理远程数据的生命周期：

```cangjie
public class Window <: RemoteDataLite {
    init(id: Int64) {
        super(id)
    }

    ~init() {
        releaseFFIData(myDataId)
    }
}
```

**证据**：`window.cj:269-297`、`display.cj:485-690`

### 回调机制

使用 `HashMap` 和 `Mutex` 管理回调注册：

```cangjie
let callbackMaps = HashMap<String, ArrayList<(CallbackObject, Int64)>>()

synchronized(REGISTER_MUTEX) {
    // 注册/注销回调
}
```

**证据**：`window.cj:270-286`、`display.cj:97-108`

---

## 与 ArkTS 版本对比

### 未提供的功能

根据 `README_zh.md:57-62`，以下 ArkTS 能力暂未提供：

| 能力 | ArkTS API | 仓颉 API 状态 |
|------|-----------|--------------|
| **画中画窗口** | `js-apis-pipWindow.md` | ❌ 未实现 |
| **闪控球窗口** | `js-apis-floatingBall.md` | ❌ 未实现 |
| **屏幕截图** | `js-apis-screenshot.md` | ❌ 未实现 |

**证据**：`README_zh.md:57-62` 明确说明暂未提供

### 功能对等

| 功能类别 | ArkTS | 仓颉 | 对等性 |
|----------|--------|--------|--------|
| 窗口创建/销毁 | ✅ | ✅ | 完全对等 |
| 窗口属性设置 | ✅ | ✅ | 完全对等 |
| Display 查询 | ✅ | ✅ | 完全对等 |
| 回调机制 | ✅ | ✅ | 完全对等 |

---

## 相关链接

### 内部文档
- [01_Project_Positioning.md](./01_Project_Positioning.md) - 详细项目定位
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](./03_Architecture.md) - 架构说明
- [04_External_API.md](./04_External_API.md) - **对外 API 清单**
- [05_Internal_API.md](./05_Internal_API.md) - 内部 API
- [06_GN_Targets.md](./06_GN_Targets.md) - GN targets
- [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 编译产物
- [08_Security_Review.md](./08_Security_Review.md) - 安全风险评审
- [09_FAQ.md](./09_FAQ.md) - 常见问题

### 外部参考
- [仓颉窗口 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/arkui-cj/cj-apis-window.md)
- [仓颉屏幕属性 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/arkui-cj/cj-apis-display.md)
- [窗口开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/windowmanager/application-window-stage.md)
- [屏幕属性开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/displaymanager/screenProperty-guideline.md)

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 本项目是 OpenHarmony 窗口管理子系统的仓颉语言封装层 | `bundle.json:13-14` |
| 提供 window 和 display 两个模块，覆盖窗口管理和显示设备管理 | `ohos/` 目录结构 |
| 通过 FFI 接口调用 window_manager 子系统的 Native 实现 | `window.cj:32-146`, `display.cj:27-93` |
| API Level 22+，仅支持 standard 设备 | `bundle.json:16`, `bundle.json:17` |
| 与 ArkTS 版本相比，缺少画中画、闪控球、屏幕截图能力 | `README_zh.md:57-62` |

---

**生成时间**: 2025-02-06
