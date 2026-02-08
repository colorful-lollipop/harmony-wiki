# 编译产物

> 本文档描述 multimedia_cangjie_wrapper 的编译产物、输出路径与运行时加载关系

---

## 目的与适用范围

**目的**：帮助开发者了解构建输出，进行产物分析和部署

**适用范围**：构建工程师、系统集成工程师

---

## 产物类型

本仓库编译产生以下类型的产物：

| 产物类型 | 扩展名 | 说明 |
|----------|--------|------|
| 仓颉共享库 | `.so` | FFI 层和 Kit 层的仓颉共享库 |
| SDK API 库 | 目录 | 复制到 SDK 的接口定义 |

---

## 主要编译产物

### FFI 层共享库

| 目标名 | 产物名 | 类型 |
|--------|--------|------|
| `ohos.multimedia` | `libohos.multimedia.so` | 共享库 |
| `ohos.multimedia.camera` | `libohos.multimedia.camera.so` | 共享库 |
| `ohos.multimedia.image` | `libohos.multimedia.image.so` | 共享库 |
| `ohos.multimedia.media` | `libohos.multimedia.media.so` | 共享库 |
| `ohos.file.photo_access_helper` | `libohos.file.photo_access_helper.so` | 共享库 |

### Kit 层共享库

| 目标名 | 产物名 | 类型 |
|--------|--------|------|
| `kit.CameraKit` | `libkit.CameraKit.so` | 共享库 |
| `kit.ImageKit` | `libkit.ImageKit.so` | 共享库 |
| `kit.MediaKit` | `libkit.MediaKit.so` | 共享库 |
| `kit.MediaLibraryKit` | `libkit.MediaLibraryKit.so` | 共享库 |

---

## 产物输出路径

### 构建输出目录

```
out/{产品名}/{芯片平台}/obj/
└── foundation/multimedia/multimedia_cangjie_wrapper/
    ├── ohos/multimedia/
    │   └── ohos.multimedia/
    │       └── libohos.multimedia.so
    ├── ohos/multimedia/camera/
    │   └── ohos.multimedia.camera/
    │       └── libohos.multimedia.camera.so
    ├── ohos/multimedia/image/
    │   └── ohos.multimedia.image/
    │       └── libohos.multimedia.image.so
    ├── ohos/multimedia/media/
    │   └── ohos.multimedia.media/
    │       └── libohos.multimedia.media.so
    ├── ohos/file/photo_access_helper/
    │   └── ohos.file.photo_access_helper/
    │       └── libohos.file.photo_access_helper.so
    └── kit/
        ├── CameraKit/
        │   └── kit.CameraKit/
        │       └── libkit.CameraKit.so
        ├── ImageKit/
        │   └── kit.ImageKit/
        │       └── libkit.ImageKit.so
        ├── MediaKit/
        │   └── kit.MediaKit/
        │       └── libkit.MediaKit.so
        └── MediaLibraryKit/
            └── kit.MediaLibraryKit/
                └── libkit.MediaLibraryKit.so
```

### SDK 输出目录

```
out/{产品名}/{芯片平台}/cjlibs/
└── api/
    └── ohos/
        └── multimedia/
            ├── ohos.multimedia/
            │   └── index.cj
            ├── ohos.multimedia.camera/
            │   └── index.cj
            ├── ohos.multimedia.image/
            │   └── index.cj
            ├── ohos.multimedia.media/
            │   └── index.cj
            └── file/
                └── photo_access_helper/
                    └── index.cj
```

### Kit SDK 输出目录

```
out/{产品名}/{芯片平台}/cjlibs/
└── api/
    └── kit/
        ├── CameraKit/
        │   └── index.cj
        ├── ImageKit/
        │   └── index.cj
        ├── MediaKit/
        │   └── index.cj
        └── MediaLibraryKit/
            └── index.cj
```

---

## 运行时加载关系

### 库依赖链

```
应用
  ↓ 加载
libkit.CameraKit.so
  ↓ 依赖
libohos.multimedia.camera.so
  ↓ 依赖（FFI）
camera_framework (libcamera_framework.z.so)
  ↓ IPC
Camera Service (系统服务)
```

### 依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│                        应用层                                 │
│              仓颉应用 import kit.CameraKit                    │
└─────────────────────────────────────────────────────────────┘
                              ↓ 运行时加载
┌─────────────────────────────────────────────────────────────┐
│                        Kit 层                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │libkit.Came- │ │libkit.Image-│ │libkit.Media-│            │
│  │raKit.so     │ │Kit.so       │ │LibraryKit.so│            │
│  └──────┬──────┘ └──────┬──────┘ └─────────────┘            │
└─────────┼───────────────┼───────────────────────────────────┘
          │               │
          ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│                        FFI 层                                 │
│  ┌────────────────┐ ┌────────────────┐                      │
│  │libohos.multim- │ │libohos.multim- │                      │
│  │edia.camera.so  │ │edia.image.so   │                      │
│  └────────┬───────┘ └────────┬───────┘                      │
└───────────┼──────────────────┼───────────────────────────────┘
            │                  │
            ▼                  ▼
┌─────────────────────────────────────────────────────────────┐
│                      底层 C++ 框架                            │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐    │
│  │camera_frame-   │ │image_frame-    │ │media_library   │    │
│  │work.so         │ │work.so         │ │.so             │    │
│  └────────────────┘ └────────────────┘ └────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 产物安装路径

### 系统镜像路径

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `libohos.multimedia*.so` | `/system/lib64/` 或 `/system/lib/` | 仓颉 FFI 库 |
| `libkit.*Kit.so` | `/system/lib64/` 或 `/system/lib/` | Kit 库 |

### SDK 路径

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `cjlibs/api/ohos/**/*` | `{SDK_ROOT}/cjlibs/api/ohos/` | ohos 命名空间 API |
| `cjlibs/api/kit/**/*` | `{SDK_ROOT}/cjlibs/api/kit/` | kit 命名空间 API |

---

## 产物大小估算

### 源码大小（基于代码统计）

| 模块 | 源文件数 | 估算代码行数 |
|------|---------|-------------|
| kit/ | 4 | ~80 |
| ohos/multimedia/camera/ | 10 | ~2,500 |
| ohos/multimedia/image/ | 9 | ~2,000 |
| ohos/multimedia/media/ | 3 | ~300 |
| ohos/file/photo_access_helper/ | 8 | ~1,800 |
| mock/ | 5 | ~100 |
| **总计** | **39** | **~6,780** |

### 二进制大小（参考 bundle.json）

| 资源类型 | 占用量 |
|----------|--------|
| ROM | 1300 KB |
| RAM | 1212 KB |

---

## 关键结论

1. **分层产物**：Kit 层产物与 FFI 层产物分离
2. **共享库**：所有产物为 `.so` 共享库
3. **SDK 复制**：通过 `copy_ohos_cangjie_sdk_api_lib` 复制到 SDK
4. **运行时依赖**：应用 → Kit → FFI → 底层框架 → 系统服务

---

## 相关跳转

- [GN Targets](05_GN_Targets.md)
- [架构说明](01_Architecture.md)
