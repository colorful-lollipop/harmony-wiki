# 编译产物

## 概述

Previewer 项目编译后产出多种产物，包括可执行文件和动态库。

## 可执行文件

### Previewer (Rich)

| 属性 | 值 |
|------|------|
| **输出名** | `Previewer` (Windows: `Previewer.exe`) |
| **源文件** | `RichPreviewer.cpp` |
| **目标类型** | ohos_executable |
| **平台** | Windows/macOS/Linux |

### Simulator (Lite)

| 属性 | 值 |
|------|------|
| **输出名** | `Simulator` (Windows: `Simulator.exe`) |
| **源文件** | `ThinPreviewer.cpp` |
| **目标类型** | ohos_executable |
| **平台** | Windows/macOS/Linux |

---

## 动态库

### libide_util

| 属性 | 值 |
|------|------|
| **目标名** | `ide_util` |
| **类型** | ohos_shared_library |
| **Windows** | `ide_util.dll` |
| **macOS** | `libide_util.dylib` |
| **Linux** | `libide_util.so` |

**头文件**:
- `util/KeyboardHelper.h`
- `util/ClipboardHelper.h`

### ide_extension

| 属性 | 值 |
|------|------|
| **目标名** | `ide_extension` |
| **类型** | ohos_shared_library |
| **Windows** | `ide_extension.dll` |
| **macOS** | `libide_extension.dylib` |
| **Linux** | `libide_extension.so` |

**头文件**:
- `jsapp/rich/external/EventRunner.h`
- `jsapp/rich/external/EventHandler.h`
- `jsapp/rich/external/StageContext.h`
- `jsapp/rich/external/JsMockUtil.h`

---

## 第三方库依赖

### 静态链接

| 库 | 用途 |
|------|------|
| `libwebsockets` | WebSocket 通信 |
| `libsec_static` | 安全函数 (bounds_checking_function) |
| `turbojpeg` | JPEG 编码 |
| `cjson` | JSON 解析 |
| `ace_lite` | ACELite 引擎 |

### 平台特定

| 平台 | 库 |
|------|------|
| **Windows** | `psapi`, `ws2_32`, `shlwapi`, `dbghelp` |
| **macOS** | `Cocoa.framework`, `Carbon.framework` |
| **Linux** | `X11`, `zlib` |

---

## 安装路径

根据 OpenHarmony SDK 结构，产物预计安装在：

```
${OUT_DIR}/
├── previewer/
│   ├── common/
│   │   └── bin/
│   │       ├── Previewer.exe
│   │       ├── Simulator
│   │       ├── libide_util.dll/dylib/so
│   │       └── libide_extension.dll/dylib/so
│   └── liteWearable/
│       └── config/
│           ├── SourceHanSansSC-Regular.otf
│           ├── font.bin
│           └── line_cj.brk
```

---

## 相关文档

- 构建系统: [05_Build_System.md](./05_Build_System.md)
- 安全评审: [07_Security_Review.md](./07_Security_Review.md)
