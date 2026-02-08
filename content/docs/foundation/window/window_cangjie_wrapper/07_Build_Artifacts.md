# 编译产物

> 产物清单、安装路径、运行时加载关系

---

## 目的

本文档说明 `window_cangjie_wrapper` 的编译产物，包括库文件、安装位置和运行时加载关系。

---

## 产物清单

### 共享库文件

| 产物名称 | 文件名 | 大小（估算） | 证据 |
|----------|----------|--------------|--------|
| **ohos.window 库** | `libohos.window.so` | ~200KB（435KB ROM 占用） | `BUILD.gn:16`, `bundle.json:17` |
| **ohos.display 库** | `libohos.display.so` | ~150KB | `BUILD.gn:16`, `bundle.json:17` |
| **SDK 合并** | 复制到 SDK libs 目录 | - | `BUILD.gn:21-23` |

### 头文件与接口

| 类型 | 说明 | 证据 |
|------|--------|--------|
| **仓颉源文件** | `.cj` 仓颉源码，包含类型定义和 API 实现 | `ohos/` 目录 |
| **仓颉 FFI 接口** | `foreign` 块定义，对应 Native C/C++ 函数 | 所有 `.cj` 文件 |
| **仓颉注解** | `@!APILevel` 定义 API Level 和 Syscap | 所有公共 API |

---

## 安装路径

### 系统库路径

| 文件 | 安装路径 | 用途 |
|------|----------|--------|
| `libohos.window.so` | `/system/lib64/libohos.window.so` | 系统级库，所有应用共享 |
| `libohos.display.so` | `/system/lib64/libohos.display.so` | 系统级库，所有应用共享 |
| `libwindow_manager.so` | `/system/lib64/libwindow_manager.so` | 窗口管理子系统 Native 库 |

**证据**：OpenHarmony 标准设备系统库布局

### SDK 路径

| 用途 | 路径 | 证据 |
|------|----------|--------|
| **开发 SDK** | `/usr/lib64/ohos/sdk/libs/` | 开发工具链 |
| **设备镜像** | `/system/lib64/` | 运行设备 |

---

## 运行时加载关系

### 加载流程图

```mermaid
graph TB
    subgraph仓颉应用[Cangjie App]
        A1[App 代码]
    end

    subgraph仓颉运行时[Cangjie Runtime]
        R1[libohos.window.so]
        R2[libohos.display.so]
    end

    subgraph Native服务层[Native Services]
        N1[libwindow_manager.so]
        N2[libability_runtime.so]
        N3[libhilog.so]
    end

    A1 --> R1
    A1 --> R2
    R1 --> N1
    R2 --> N1
    R1 --> N2
    R1 --> N3
```

### 依赖加载顺序

| 加载顺序 | 库 | 原因 | 证据 |
|----------|------|--------|--------|
| 1 | `libhilog.so` | 日志基础设施，最先加载 | `cj_window_log.cj:24` |
| 2 | `libability_runtime.so` | 能力运行时，提供 BaseContext | `window.cj:20` |
| 3 | `libwindow_manager.so` | 窗口管理 Native 实现，提供 FFI 函数 | `window.cj:32-146` |
| 4 | `libohos.window.so` | 仓颉窗口封装 | `window.cj:269-894` |
| 5 | `libohos.display.so` | 仓颉显示封装 | `display.cj:485-726` |

---

## 运行时依赖详情

### Native 层依赖

#### window_manager 子系统

**功能**：提供窗口管理和显示设备管理的 FFI 实现

**关键库**：
- `libwindow_manager.so` - 主窗口管理服务
- `libcj_window_ffi.so` - 仓颉 FFI 接口实现
- `libcj_display_ffi.so` - 显示 FFI 接口实现

**证据**：`ohos/window/BUILD.gn:39`（`window_manager:cj_window_ffi`）、`ohos/display/BUILD.gn:33`（`window_manager:cj_display_ffi`）

#### ability_runtime 子系统

**功能**：提供能力运行时，管理 Ability 生命周期

**关键接口**：
- `GetContext()` - 获取 Ability 上下文（StageContext）
- `VerifyAccessToken()` - 权限验证

**证据**：`window.cj:192-196`（`FFIGetContext`）、`window.cj:187`（权限检查）

---

## 运行时资源

### 内存占用

| 资源类型 | 占用量 | 说明 |
|----------|----------|--------|
| **ROM** | 435KB | 静态库大小（bundle.json:17） |
| **RAM** | 401KB | 运行时内存占用（bundle.json:18） |

### CPU 架构

| 平台 | 架构 | 说明 |
|--------|--------|--------|
| OpenHarmony 标准设备 | ARM64 | 默认架构（未明确指定） |

---

## 动态加载机制

### FFI 调用机制

**特点**：
1. 仓颉代码通过 `foreign` 块声明 FFI 函数
2. 运行时动态解析 Native 符号
3. 调用约定：所有 FFI 函数以 `FfiOHOS` 前缀

**示例**：
```cangjie
foreign {
    func FfiOHOSCreateWindow(name: CString, ...): RetDataI64
    func FfiOHOSWindowShowWindow(id: Int64): Int32
}
```

**证据**：`window.cj:32-146`、`display.cj:27-93`

### 资源生命周期

#### RemoteDataLite 机制

**职责**：统一管理 Native 资源的生命周期

**生命周期**：
1. 创建实例时，调用 `super(id)` 初始化 `myDataId`
2. 销毁实例时，调用 `~init()` 执行 `releaseFFIData(myDataId)`
3. Native 层自动跟踪和释放资源

**示例**：
```cangjie
public class Window <: RemoteDataLite {
    init(id: Int64) {
        super(id)  // 初始化 myDataId
    }

    ~init() {
        releaseFFIData(myDataId)  // 释放 Native 资源
    }
}
```

**证据**：`window.cj:290-297`、`display.cj:683-690`

---

## 版本兼容性

### API Level 兼容性

| API Level | 功能 | 状态 | 证据 |
|----------|--------|--------|--------|
| 22+ | 所有当前功能 | ✅ 支持 | 所有 API `@!APILevel[since: "22"]` |
| < 22 | 不支持 | ❌ 不兼容 | bundle.json 指定 API Level 22 |

### Syscap 兼容性

| Syscap | 功能范围 | 证据 |
|--------|----------|--------|
| `SystemCapability.WindowManager.WindowManager.Core` | 核心窗口管理能力 | 所有核心 API |
| `SystemCapability.Window.SessionManager` | 会话管理能力 | 部分高级 API（shiftAppWindowFocus, minimize） |

**证据**：所有 API `@!APILevel[syscap: "..."]` 注解

---

## 构建调试

### 编译命令

```bash
# 标准编译
./build.sh --product-name {product} --ccache

# 清理编译产物
./build.sh --product-name {product} --clean
```

### 输出目录

| 构建类型 | 输出目录 | 路径 |
|----------|----------|--------|
| **仓颉源码** | `out/{product}/gen/` | 中间生成文件 |
| **共享库** | `out/{product}/libs/` | `.so` 文件 |
| **SDK** | `out/{product}/libs/sdk/` | 复制的 SDK 库 |

---

## 关键结论

| 结论 | 证据 |
|------|--------|
| 生成两个共享库：`libohos.window.so` 和 `libohos.display.so` | BUILD.gn targets |
| 库文件安装到系统 `/system/lib64/`，供所有应用动态加载 | OpenHarmony 标准 |
| 依赖 Native 层的 `window_manager` 和 `ability_runtime` 子系统 | BUILD.gn external_deps |
| 通过 RemoteDataLite 自动管理 Native 资源生命周期，避免内存泄露 | 继承模式 |
| 所有 FFI 调用采用统一的命名规范（`FfiOHOS*`） | foreign 块定义 |

---

**生成时间**: 2025-02-06
