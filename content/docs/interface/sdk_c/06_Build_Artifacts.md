# 编译产物与安装路径

> **目的**: 理解编译产物类型、输出路径和运行时加载关系  
> **适用范围**: 发布工程师、需要了解 SDK 结构的开发者  
> **生成时间**: 2025-02-06

---

## 1. 编译产物类型

### 1.1 产物分类

| 产物类型 | 扩展名 | 说明 | 示例 |
|----------|--------|------|------|
| **共享库** | `.so` | NDK 动态链接库 | `libhilog.so`, `libace_napi.z.so` |
| **静态库** | `.a` | 静态链接库（较少） | - |
| **头文件** | `.h` | C/C++ 头文件 | `log.h`, `native_window.h` |
| **符号定义** | `.ndk.json` | API 符号列表 | `libhilog.ndk.json` |

### 1.2 库命名规范

根据 `docs/capi_naming.md:77-81`：

```
lib<name>.so        # 标准 NDK 库
lib<name>.z.so      # 带版本号的 NDK 库
liboh<name>.so      # OpenHarmony 自研库（加 oh 前缀避免冲突）
```

**命名示例**:
- `libhilog.so` - HiLog 日志库
- `libace_napi.z.so` - NAPI 运行时库
- `libohcamera.so` - 相机库（oh 前缀）
- `libohaudio.so` - 音频库（oh 前缀）

---

## 2. 编译输出目录结构

### 2.1 SDK 输出目录

根据 `docs/howto_add.md:24-43`：

```
out/sdk-native/
├── os-specific/                    # 平台相关文件
│   ├── darwin/                     #   macOS 平台
│   ├── linux/                      #   Linux 平台
│   └── windows/                    #   Windows 平台
│       ├── llvm/                   #     交叉编译工具链
│       └── build-tools/            #     构建工具（cmake、ninja）
│
└── os-irrelevant/                  # 平台无关文件
    └── sysroot/                    #   系统根目录
        └── usr/
            ├── include/            #     C API 头文件
            │   ├── hilog/          #       HiLog 头文件
            │   ├── napi/           #       NAPI 头文件
            │   ├── window/         #       窗口管理头文件
            │   └── ...             #       其他模块头文件
            │
            └── lib/                #     NDK 库文件
                ├── libhilog.so
                ├── libace_napi.z.so
                └── ...
```

### 2.2 头文件安装路径

```
sysroot/usr/include/
├── hilog/                          # HiLog 日志
│   └── log.h
├── napi/                           # NAPI
│   ├── native_api.h
│   └── common.h
├── window/                         # 窗口管理
│   └── native_window.h
├── buffer/                         # 缓冲区
│   └── native_buffer.h
├── drawing/                        # 2D 绘制
│   └── drawing_canvas.h
└── ...                             # 其他模块
```

### 2.3 库文件安装路径

```
sysroot/usr/lib/
├── libhilog.so                     # 日志库
├── libace_napi.z.so                # NAPI 运行时
├── libnative_window.so             # 原生窗口
├── libnative_buffer.so             # 原生缓冲区
├── libpixelmap.so                  # 图像 PixelMap
├── librawfile.so                   # 原始资源文件
└── ...                             # 其他库
```

---

## 3. 运行时加载关系

### 3.1 设备端库路径

根据 `docs/user_guide.md:64`：

```
/system/lib/ndk/          # 32 位 NDK 库
/system/lib64/ndk/        # 64 位 NDK 库
```

### 3.2 加载时序

```
应用启动
    │
    ▼
┌─────────────────────────────────────┐
│ 1. 加载器加载应用可执行文件          │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 2. 解析 ELF 依赖（DT_NEEDED）        │
│    例如: libhilog.so, libace_napi.z.so│
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 3. 从 /system/{lib|lib64}/ndk/      │
│    加载依赖的 NDK 库                 │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 4. 符号解析与重定位                  │
│    根据 .ndk.json 定义的符号列表     │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 5. 执行应用代码                      │
└─────────────────────────────────────┘
```

### 3.3 典型依赖链

```
应用
    ├── libace_napi.z.so          (NAPI 运行时)
    │       └── libc.so           (C 库)
    ├── libhilog.so               (日志)
    │       └── libc.so
    ├── libnative_window.so       (窗口)
    │       ├── libace_napi.z.so
    │       └── libc.so
    └── ...
```

---

## 4. SDK 包结构

### 4.1 分发包目录

```
native-[platform]-x64-[version].zip
├── build/                          # CMake 配置
│   └── cmake/                      #   toolchain 文件
├── build-tools/                    # 构建工具
│   ├── cmake/                      #   CMake 可执行文件
│   └── ninja/                      #   Ninja 可执行文件
├── docs/                           # 文档
├── llvm/                           # LLVM 工具链
│   ├── bin/                        #   clang, clang++, llvm-ar 等
│   ├── lib/                        #   LLVM 库
│   └── lib64/clang/                #   编译器头文件
├── sysroot/                        # 系统根目录
│   └── usr/
│       ├── include/                #   头文件
│       └── lib/                    #   库文件
├── nativeapi_syscap_config.json    # SysCap 配置
├── NOTICE.txt                      # 许可证声明
└── oh-uni-package.json             # 包元数据
```

### 4.2 SysCap 配置文件

`nativeapi_syscap_config.json`:
```json
{
    "SystemCapability.ArkUI.ArkUI.Napi": [
        "napi/native_api.h",
        "napi/common.h"
    ],
    "SystemCapability.HiviewDFX.Hilog": [
        "hilog/log.h"
    ],
    ...
}
```

---

## 5. 关键产物速查表

### 5.1 核心库产物

| 模块 | 库文件名 | 头文件路径 |
|------|----------|------------|
| **NAPI** | `libace_napi.z.so` | `napi/native_api.h` |
| **HiLog** | `libhilog.so` | `hilog/log.h` |
| **窗口** | `libnative_window.so` | `window/native_window.h` |
| **缓冲区** | `libnative_buffer.so` | `buffer/native_buffer.h` |
| **绘制** | `libnative_drawing.so` | `drawing/drawing_canvas.h` |
| **图像** | `libpixelmap.so` | `multimedia/image_framework/image/pixelmap_native.h` |
| **音频** | `libohaudio.so` | `multimedia/audio_framework/ohaudio.h` |
| **相机** | `libohcamera.so` | `multimedia/camera_framework/camera.h` |
| **密钥** | `libhuks_ndk.so` | `security/huks/native_huks_api.h` |

### 5.2 第三方库产物

| 库名 | 文件名 | 用途 |
|------|--------|------|
| **musl libc** | `libc.so` | C 标准库 |
| **zlib** | `libz.so` | 压缩 |
| **libuv** | `libuv.so` | 异步 IO |
| **OpenSSL** | `libssl.so`, `libcrypto.so` | TLS/加密 |
| **ICU** | `libicuuc.so`, `libicui18n.so` | 国际化 |

---

## 6. 代码证据

| 结论 | 证据文件 | 关键内容 |
|------|----------|----------|
| 库路径 | `docs/user_guide.md:64` | `/system/{lib｜lib64}/ndk/` |
| 输出目录 | `docs/howto_add.md:24-26` | `out/sdk-native/` |
| 命名规范 | `docs/capi_naming.md:77-81` | `libxxx.so` 格式 |

---

## 7. 相关跳转

- **上一章**: [GN 构建](./05_GN_Build.md)
- **下一章**: [安全分析](./07_Security_Analysis.md)
- **构建指南**: `docs/howto_add.md`
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**编译产物文档 - 基于代码生成**
