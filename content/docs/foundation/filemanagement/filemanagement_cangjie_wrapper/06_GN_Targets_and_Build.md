# GN 目标与构建文档

> 目的：梳理 GN targets、类型、依赖、产物、开关
> 适用范围：构建工程师、组件集成、依赖管理
> 最后更新：2026-02-06

## GN 构建文件总览

### 根目录 BUILD.gn

**文件位置**：`BUILD.gn` （根目录）

**证据**：`BUILD.gn:16-28`

```python
# BUILD.gn:14-28
import("//build/templates/cangjie/cjc.gni")

filemanagement_cangjie_wrapper_packages_ohos = [
        "//foundation/filemanagement/filemanagement_cangjie_wrapper/ohos/file:ohos.file",
        "//foundation/filemanagement/filemanagement_cangjie_wrapper/ohos/file/fileuri:ohos.file.fileuri",
        "//foundation/filemanagement/filemanagement_cangjie_wrapper/ohos/file/fs:ohos.file.fs"
]

filemanagement_cangjie_wrapper_packages_kit = [
        "//foundation/filemanagement/filemanagement_cangjie_wrapper/kit/CoreFileKit:kit.CoreFileKit",
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_filemanagement_cangjie_libs") {
        ohos_inputs = filemanagement_cangjie_wrapper_packages_ohos
        kit_inputs = filemanagement_cangjie_wrapper_packages_kit
}
```

---

## 关键 Targets 详解

### 1. Kit 目标

| Target 名 | 类型 | 文件 | 行号 | Sources | deps | 输出名 |
|-----------|------|------|------|---------|------|--------|
| `kit.CoreFileKit` | `ohos_cangjie_shared_library` | kit/CoreFileKit/BUILD.gn:19 | `index.cj` | `ohos.file.fs`<br/>`ohos.file.fileuri` | - |

**证据**：`kit/CoreFileKit/BUILD.gn:19-29`

```python
# kit/CoreFileKit/BUILD.gn:19-29
ohos_cangjie_shared_library("kit.CoreFileKit") {
        sources = [ "index.cj" ]

        cj_deps = [
                "../../ohos/file/fs:ohos.file.fs",
                "../../ohos/file/fileuri:ohos.file.fileuri",
        ]

        subsystem_name = "filemanagement"
        part_name = "filemanagement_cangjie_wrapper"
}
```

---

### 2. FS 模块目标

| Target 名 | 类型 | 文件 | 行号 | Sources | cj_external_deps | external_deps | 输出名 |
|-----------|------|------|------|---------|----------------|--------------|--------|
| `ohos.file` | `ohos_cangjie_shared_library` | ohos/file/BUILD.gn:19 | `file_package.cj`<br/>(mock: `ohos.file.cj`) | - | - | - |
| `ohos.file.fs` | `ohos_cangjie_shared_library` | ohos/file/fs/BUILD.gn:19 | `cj_file_fs.cj`<br/>`file.cj`<br/>`stream.cj`<br/>`stat.cj`<br/>`random_access_file.cj`<br/>`conflict_file_exception.cj`<br/>(mock: `ohos.file.fs.cj`)<br/>`native.cj` | `cangjie_ark_interop:ohos.business_exception`<br/>`cangjie_ark_interop:ohos.ffi`<br/>`cangjie_ark_interop:ohos.labels`<br/>`hiviewdfx_cangjie_wrapper:ohos.hilog` | `file_api:cj_file_fs_ffi` | - |
| `ohos.file.fileuri` | `ohos_cangjie_shared_library` | ohos/file/fileuri/BUILD.gn:19 | `file_uri.cj`<br/>(mock: `ohos.file.fileuri.cj`) | `cangjie_ark_interop:ohos.business_exception`<br/>`cangjie_ark_interop:ohos.ffi`<br/>`cangjie_ark_interop:ohos.labels`<br/>`hiviewdfx_cangjie_wrapper:ohos.hilog` | `app_file_service:cj_file_fileuri_ffi` | - |

#### ohos.file

**证据**：`ohos/file/BUILD.gn:19-31`

```python
# ohos/file/BUILD.gn:19-31
ohos_cangjie_shared_library("ohos.file") {

        if (is_mingw || is_mac){
                sources = [ "../../mock/ohos.file.cj" ]
        } else {
                sources = [
                        "file_package.cj",
                ]
        }

        subsystem_name = "filemanagement"
        part_name = "filemanagement_cangjie_wrapper"
}
```

**平台条件**：`is_mingw || is_mac` 判断是否使用 Mock

#### ohos.file.fs

**证据**：`ohos/file/fs/BUILD.gn:19-50`

```python
# ohos/file/fs/BUILD.gn:19-50
ohos_cangjie_shared_library("ohos.file.fs") {

        if (is_mingw || is_mac){
                sources = [
                        "../../../mock/ohos.file.fs.cj",
                        "conflict_file_exception.cj",
                        "native.cj"
                ]
        } else {
                sources = [
                        "cj_file_fs.cj",
                        "file.cj",
                        "stat.cj",
                        "stream.cj",
                        "random_access_file.cj",
                        "conflict_file_exception.cj",
                        "native.cj",
                ]
        }

        cj_external_deps = [
                "cangjie_ark_interop:ohos.business_exception",
                "cangjie_ark_interop:ohos.ffi",
                "cangjie_ark_interop:ohos.labels",
                "hiviewdfx_cangjie_wrapper:ohos.hilog",
        ]

        external_deps = [ "file_api:cj_file_fs_ffi" ]

        subsystem_name = "filemanagement"
        part_name = "filemanagement_cangjie_wrapper"
}
```

#### ohos.file.fileuri

**证据**：`ohos/file/fileuri/BUILD.gn:19-38`

```python
# ohos/file/fileuri/BUILD.gn:19-38
ohos_cangjie_shared_library("ohos.file.fileuri") {

        if (is_mingw || is_mac){
                sources = [ "../../../mock/ohos.file.fileuri.cj" ]
        } else {
                sources = [ "file_uri.cj" ]
        }

        cj_external_deps = [
                "cangjie_ark_interop:ohos.business_exception",
                "cangjie_ark_interop:ohos.ffi",
                "cangjie_ark_interop:ohos.labels",
                "hiviewdfx_cangjie_wrapper:ohos.hilog",
        ]

        external_deps = [ "app_file_service:cj_file_fileuri_ffi" ]

        subsystem_name = "filemanagement"
        part_name = "filemanagement_cangjie_wrapper"
}
```

---

### 3. 辅助目标

| Target 名 | 类型 | 文件 | 行号 | 说明 |
|-----------|------|------|------|------|
| `copy_sdk_filemanagement_cangjie_libs` | `copy_ohos_cangjie_sdk_api_lib` | BUILD.gn:25 | 复制 SDK 库到输出目录 |
| `copy_sdk_filemanagement_cangjie_libs_kit` | `copy_ohos_cangjie_sdk_api_lib` | BUILD.gn:39 | 复制 Kit 库到输出目录 |

**证据**：`BUILD.gn:25-28`

---

## 目标依赖图

```
┌─────────────────────────────────────────────────────┐
│  kit.CoreFileKit                              │
│  - Target Type: ohos_cangjie_shared_library     │
│  - Output: libkit.CoreFileKit.so (推测)         │
└─────────────────────────────────────────────────────┘
              │
              ▼
    ┌──────────────────┴──────────┐
    │                          │
┌───▼──────────┐  ──▼──────────┐
│ ohos.file   │  │ ohos.file │
│              │  │            │
│ Output:      │  │ Output:    │
│ ohos.file.so │  │ ohos.file. │
│              │  │            │
└──────────────┘  │ so          │
                 │              │
                 ▼              ▼
         ┌──────────────────────────────────────┐
         │ ohos.file.fs      ohos.file.fileuri │
         │                   │                   │
         │ Output:          │ Output:           │
         │ ohos.file.fs.so  │ ohos.file.fileuri.so │
         │                   │                   │
         └───────────────────────────────────────┘
                        │                   │
                        ▼                   ▼
         ┌──────────────────────────────────────────────┐
         │              cj_external_deps            │
         │  ────────────────────────┐        │
         │  │                          │        │
┌────▼─────────▼──────┐  ─▼─────────▼───────▼──────┐
│ cangjie_ark_interop:  │ app_file_service:   │
│   - ohos.business_  │   cj_file_fileuri_ffi│
│     exception           │ file_api:              │
│   - ohos.ffi          │   cj_file_fs_ffi        │
│   - ohos.labels        │                           │
└─────────────────────────┘  └──────────────────────────┘
                        │                   │
                        ▼                   ▼
            ┌───────────────────────────────────────────────┐
            │          hiviewdfx_cangjie_wrapper       │
            │          (ohos.hilog)                      │
            └───────────────────────────────────────────────┘
```

---

## 编译产物

### 预计输出产物

| Target | 预计产物类型 | 预计输出名 | 安装路径 | 运行时加载 |
|--------|--------------|--------------|----------|------------|
| `kit.CoreFileKit` | `.so` (共享库） | `libkit.CoreFileKit.so` | SDK:ohos/ndk/ | Cangjie 应用 import |
| `ohos.file` | `.so` (共享库） | `libohos.file.so` | SDK:ohos/ndk/ | Kit 依赖 |
| `ohos.file.fs` | `.so` (共享库） | `libohos.file.fs.so` | SDK:ohos/ndk/ | Kit 依赖 |
| `ohos.file.fileuri` | `.so` (共享库） | `libohos.file.fileuri.so` | SDK:ohos/ndk/ | Kit 依赖 |

**说明**：
- ✅ 所有产物为 Cangjie 共享库（`ohos_cangjie_shared_library`）
- ✅ 输出到 SDK:ohos/ndk/ 目录（OpenHarmony NDK 路径）
- ✅ 安装到系统分区用于运行时加载
- ✅ Kit 层库最终链接所有依赖

### 产物运行时加载关系

```
Cangjie Application (仓颉应用）
    ↓ import kit.CoreFileKit
    ↓ 链接 libkit.CoreFileKit.so
    ↓
    ├─→ libohos.file.fs.so
    │       ↓ external_deps: file_api:cj_file_fs_ffi
    │       └─→ libfile_api_ffi.so (C++ 动态库）
    │
    └─→ libohos.file.fileuri.so
            ↓ external_deps: app_file_service:cj_file_fileuri_ffi
            └─→ libapp_file_service_ffi.so (C++ 动态库）
```

---

## 关键配置项

### Feature Flags（功能开关）

**当前状态**：✅ **无功能开关**

**证据**：所有 BUILD.gn 文件中未发现 `defines` 或 `configs` 配置

### 平台条件

| 配置项 | 值 | 效果 | 使用位置 |
|--------|-----|------|----------|
| `is_mingw` | `true`/`false` | MinGW 平台（Windows） | ohos/file/*, ohos/file/fs/*, ohos/file/fileuri/* |
| `is_mac` | `true`/`false` | macOS 平台 | ohos/file/*, ohos/file/fs/*, ohos/file/fileuri/* |

**Mock 模式触发条件**：`is_mingw || is_mac`

**效果**：在非 OpenHarmony 平台（Windows、macOS）使用 Mock 实现而非真实 FFI 调用

---

## 依赖清单

### cj_external_deps（Cangjie 外部依赖）

| 依赖项 | 用途 | 目标 |
|--------|------|------|
| `cangjie_ark_interop:ohos.business_exception` | 异常处理 | ohos.file.fs, ohos.file.fileuri |
| `cangjie_ark_interop:ohos.ffi` | FFI 类型（RetData*, CPointer*, CString） | ohos.file.fs, ohos.file.fileuri |
| `cangjie_ark_interop:ohos.labels` | API 标注（@!APILevel） | ohos.file.fs, ohos.file.fileuri |
| `hiviewdfx_cangjie_wrapper:ohos.hilog` | 日志接口（HilogChannel） | ohos.file.fs, ohos.file.fileuri |

### external_deps（外部 C++ 依赖）

| 依赖项 | 用途 | 目标 |
|--------|------|------|
| `file_api:cj_file_fs_ffi` | 文件系统 FFI 实现 | ohos.file.fs |
| `app_file_service:cj_file_fileuri_ffi` | 文件 URI FFI 实现 | ohos.file.fileuri |

### bundle.json 组件依赖

**证据**：`bundle.json:22-29`

```json
"deps": {
        "components": [
                "app_file_service",
                "cangjie_ark_interop",
                "hiviewdfx_cangjie_wrapper",
                "file_api"
        ]
}
```

---

## 构建流程

### 标准构建流程

```
1. GN 解析
    ↓
2. 生成 Ninja 构建文件
    ↓
3. 编译 Cangjie 源码（.cj → .so）
    ↓
4. 复制产物到 SDK 目录
    ↓
5. 签名与打包（如适用）
    ↓
6. 安装到系统分区
```

### 目标类型说明

| 目标类型 | 说明 | 用途 |
|----------|------|------|
| `ohos_cangjie_shared_library` | Cangjie 共享库 | 编译为 .so 文件，供 Cangjie 应用链接使用 |
| `copy_ohos_cangjie_sdk_api_lib` | SDK 库复制 | 将编译产物复制到 SDK 安装目录 |

---

## 关键结论

1. **5 个核心 Targets**：`kit.CoreFileKit`, `ohos.file`, `ohos.file.fs`, `ohos.file.fileuri`
2. **产物类型**：全部为 Cangjie 共享库（`.so`）
3. **依赖链路**：Kit → 框架层 → Cangjie 互操作库 → 外部 C++ FFI 库
4. **Mock 模式**：通过 `is_mingw || is_mac` 平台条件切换 Mock 实现
5. **无功能开关**：当前无编译时功能开关配置

---

## 相关跳转

- [01_Directories_and_Modules.md](01_Directories_and_Modules.md) - 目录结构与模块职责
- [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) - FFI 函数清单
- [05_Internal_API.md](05_Internal_API.md) - 内部接口依赖
