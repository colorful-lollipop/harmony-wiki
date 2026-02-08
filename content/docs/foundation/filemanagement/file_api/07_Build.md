# File API 构建与产物

本文档详细描述 File API 的构建系统配置、GN 目标清单、编译产物和 Feature 开关，帮助开发者理解构建过程和产物分布。

## 1. 构建系统概述

### 1.1 构建工具

File API 使用 GN（Generate Ninja）作为构建系统，配合 Ninja 执行实际编译。GN 是一种现代构建系统，广泛用于 Chromium、OpenHarmony 等大型项目。

**主要文件**：

| 文件 | 位置 | 说明 |
|------|------|------|
| bundle.json | 根目录 | 组件配置文件，声明依赖和 targets |
| file_api.gni | 根目录 | GN 参数定义文件 |
| BUILD.gn | 各子目录 | 构建目标定义 |

### 1.2 构建命令

```bash
# 完整构建
hb build -f

# 仅构建 file_api
hb build -f --gn-args file_api=true

# 清理构建
hb build -f --clean

# 查看构建产物
ls out/xxx/packages/system/lib/module/
```

---

## 2. GN 目标清单

### 2.1 Framework Group Targets

**证据来源**：`bundle.json:60-70`

```json
"fwk_group": [
  "//foundation/filemanagement/file_api/interfaces/kits/js:ani_file_api",
  "//foundation/filemanagement/file_api/interfaces/kits/js:build_kits_js",
  "//foundation/filemanagement/file_api/interfaces/kits/ts/streamrw:streamrw_packages",
  "//foundation/filemanagement/file_api/interfaces/kits/ts/streamhash:streamhash_packages",
  "//foundation/filemanagement/file_api/interfaces/kits/cj:fs_ffi_packages",
  "//foundation/filemanagement/file_api/interfaces/kits/hyperaio:group_hyperaio"
]
```

| 目标路径 | 目标类型 | 主要源文件 | 产物 |
|----------|----------|------------|------|
| `//foundation/filemanagement/file_api/interfaces/kits/js:ani_file_api` | 静态库 | ani_helper/*.cpp | libani_file_api.a |
| `//foundation/filemanagement/file_api/interfaces/kits/js:build_kits_js` | 动态库 | mod_*/module.cpp, properties/*.cpp | libfile_api.z.so |
| `//foundation/filemanagement/file_api/interfaces/kits/ts/streamrw:streamrw_packages` | NPM 包 | ts/*.d.ts | @ohos/streamrw |
| `//foundation/filemanagement/file_api/interfaces/kits/ts/streamhash:streamhash_packages` | NPM 包 | ts/*.d.ts | @ohos/streamhash |
| `//foundation/filemanagement/file_api/interfaces/kits/cj:fs_ffi_packages` | CJ 包 | cj/src/*.cpp | @cj/fs_ffi |
| `//foundation/filemanagement/file_api/interfaces/kits/hyperaio:group_hyperaio` | 静态库 | hyperaio/src/*.cpp | libhyperaio.a |

### 2.2 Inner Kit Targets

**证据来源**：`bundle.json:72-186`

Inner Kit 是暴露给其他子系统使用的内部接口：

| 目标名称 | 头文件基础 | 头文件列表 | 产物类型 |
|----------|------------|------------|----------|
| `//foundation/filemanagement/file_api/interfaces/kits/native:remote_uri_native` | remote_uri | remote_uri.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/hyperaio:HyperAio` | hyperaio/include | hyperaio.h, hyperaio_trace.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/native:environment_native` | environment | environment_native.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/native:fileio_native` | fileio | fileio_native.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/rust:rust_file` | rust/include | rust_file.h | .ndk.json |
| `//foundation/filemanagement/file_api/utils/filemgmt_libfs:filemgmt_libfs` | filemgmt_libfs/include | filemgmt_libfs.h, fs_*.h | 静态库 |
| `//foundation/filemanagement/file_api/utils/filemgmt_libn:filemgmt_libn` | filemgmt_libn/include | n_*.h | 静态库 |
| `//foundation/filemanagement/file_api/utils/filemgmt_libhilog:filemgmt_libhilog` | filemgmt_libhilog | filemgmt_libhilog.h | 静态库 |
| `//foundation/filemanagement/file_api/interfaces/kits/c:environment` | environment | environment.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/c:fileio` | fileio | fileio.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/js:securitylabel` | mod_securitylabel | security_label.h | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/cj:cj_file_fs_ffi` | cj/src | 文件 | .ndk.json |
| `//foundation/filemanagement/file_api/interfaces/kits/cj:cj_statvfs_ffi` | cj/src | 文件 | .ndk.json |

### 2.3 Utility Library Targets

**证据来源**：`utils/*/BUILD.gn`

| 目标名称 | 产物类型 | 主要源文件 |
|----------|----------|------------|
| `//foundation/filemanagement/file_api/utils/common:filemgmt_common` | 头文件 | file_utils.h |
| `//foundation/filemanagement/file_api/utils/filemgmt_libfs:filemgmt_libfs` | 静态库 | fs_error.cpp |
| `//foundation/filemanagement/file_api/utils/filemgmt_libhilog:filemgmt_libhilog` | 静态库 | filemgmt_libhilog.cpp |
| `//foundation/filemanagement/file_api/utils/filemgmt_libn:filemgmt_libn` | 静态库 | n_*.cpp |

---

## 3. 编译产物

### 3.1 动态库产物

| 产物名称 | 路径 | 大小 | 说明 |
|----------|------|------|------|
| libfile_api.z.so | out/xxx/packages/system/lib/module/ | ~2MB | 主要 N-API 动态库 |
| libfile_api.z.nf.so | out/xxx/packages/system/lib/module/ | ~2MB | 无符号版本 |

### 3.2 静态库产物

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| libani_file_api.a | out/xxx/libs/ | ANI 接口静态库 |
| libhyperaio.a | out/xxx/libs/ | HyperAIO 静态库 |
| libfilemgmt_libfs.a | out/xxx/libs/ | 文件系统封装库 |
| libfilemgmt_libn.a | out/xxx/libs/ | N-API 抽象库 |
| libfilemgmt_libhilog.a | out/xxx/libs/ | 日志库 |

### 3.3 NDK 产物

NDK 产物位于 `out/xxx/prebuilt/ndk/` 目录：

| NDK 名称 | 头文件路径 | 库路径 |
|----------|------------|--------|
| libenvironment | prebuilt/ndk/**/environment/ | libenvironment.z.so |
| libfileio | prebuilt/ndk/**/fileio/ | libfileio.z.so |
| libremote_uri | prebuilt/ndk/**/remote_uri/ | libremote_uri.z.so |
| libhyperaio | prebuilt/ndk/**/hyperaio/ | libhyperaio.z.so |

### 3.4 NPM 包产物

| 包名称 | 路径 | 版本 |
|--------|------|------|
| @ohos/streamrw | interfaces/kits/ts/streamrw/ | 4.0 |
| @ohos/streamhash | interfaces/kits/ts/streamhash/ | 4.0 |

### 3.5 产物安装路径

| 产物类型 | 安装路径 |
|----------|----------|
| 动态库 | /system/lib/module/ |
| 静态库 | /system/libs/ |
| NDK 头文件 | /prebuilt/ndk/**/include/ |
| NDK 库 | /prebuilt/ndk/**/lib/ |

---

## 4. Feature 开关

### 4.1 可用 Feature

**证据来源**：`file_api.gni:22-25`

```gni
declare_args() {
    file_api_read_optimize = false
    file_api_feature_hyperaio = false
}
```

| Feature 名称 | 默认值 | 说明 | 依赖 |
|--------------|--------|------|------|
| file_api_read_optimize | false | 启用读取优化功能 | 无 |
| file_api_feature_hyperaio | false | 启用高性能异步 I/O | liburing |

### 4.2 Feature 详细说明

#### 4.2.1 file_api_read_optimize

启用读取优化功能，包括：

| 优化项 | 说明 |
|--------|------|
| 预读取优化 | 提前读取可能需要的数据 |
| 缓存优化 | 改进缓冲区管理 |
| I/O 调度优化 | 更好的 I/O 请求调度 |

**启用方式**：
```bash
hb build -f --gn-args file_api_read_optimize=true
```

#### 4.2.2 file_api_feature_hyperaio

启用 HyperAIO（高性能异步 I/O）功能，使用 Linux io_uring 接口：

| 特性 | 说明 |
|------|------|
| io_uring 支持 | 使用最新的异步 I/O 接口 |
| 零拷贝优化 | 减少内存拷贝次数 |
| 批量提交 | 支持批量提交 I/O 请求 |

**启用方式**：
```bash
hb build -f --gn-args file_api_feature_hyperaio=true
```

**系统要求**：
- Linux Kernel 5.1+
- liburing 库可用

---

## 5. 系统能力依赖

### 5.1 Syscap 声明

**证据来源**：`bundle.json:15-21`

```json
"syscap": [
  "SystemCapability.FileManagement.File.FileIO",
  "SystemCapability.FileManagement.File.FileIO.Lite",
  "SystemCapability.FileManagement.File.Environment",
  "SystemCapability.FileManagement.File.DistributedFile",
  "SystemCapability.FileManagement.File.Environment.FolderObtain"
]
```

| 系统能力 | 说明 | API 模块 |
|----------|------|----------|
| SystemCapability.FileManagement.File.FileIO | 完整文件 I/O | @ohos.fileio, @ohos.file.fs |
| SystemCapability.FileManagement.File.FileIO.Lite | 轻量级文件 I/O | @ohos.fileio (子集) |
| SystemCapability.FileManagement.File.Environment | 环境信息 | @ohos.file.environment |
| SystemCapability.FileManagement.File.DistributedFile | 分布式文件 | @ohos.file.fs |
| SystemCapability.FileManagement.File.Environment.FolderObtain | 文件夹获取 | @ohos.file.environment |

### 5.2 组件依赖

**证据来源**：`bundle.json:30-56`

| 依赖类型 | 组件名称 | 用途 |
|----------|----------|------|
| 框架 | ability_runtime | 应用框架运行时 |
| 框架 | bundle_framework | 包管理框架 |
| 权限 | access_token | 访问令牌管理 |
| 运行时 | runtime_core | 运行时核心 |
| 运行时 | napi | N-API 运行时 |
| 异步 I/O | libuv | 异步 I/O |
| 异步 I/O | liburing | 高性能 I/O（可选） |
| 日志 | hilog | 日志系统 |
| 安全 | openssl | 加密库 |
| 安全 | bounds_checking_function | 安全函数 |
| IPC | ipc | 进程间通信 |
| IPC | samgr | 服务管理 |
| 存储 | dfs_service | 分布式文件服务 |
| 存储 | app_file_service | 应用文件服务 |
| 账户 | os_account | 系统账户 |

---

## 6. 构建配置示例

### 6.1 完整构建配置

```bash
# 设置 GN 参数
file_api_read_optimize = false
file_api_feature_hyperaio = false

# 执行构建
hb build -f
```

### 6.2 带优化构建

```bash
# 设置 GN 参数
file_api_read_optimize = true
file_api_feature_hyperaio = true

# 执行构建
hb build -f --gn-args file_api_read_optimize=true file_api_feature_hyperaio=true
```

### 6.3 调试构建

```bash
# 启用调试符号
build_with_debug = true

# 执行构建
hb build -f --gn-args build_with_debug=true
```

---

## 7. 常见构建问题

### 7.1 构建失败排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 找不到头文件 | NDK 未正确安装 | 检查 ndk 安装路径 |
| 链接失败 | 依赖库缺失 | 检查组件依赖配置 |
| Feature 冲突 | 开关组合不支持 | 检查 Feature 兼容性 |
| 编译超时 | 源文件过多 | 使用分布式编译 |

### 7.2 构建产物验证

```bash
# 检查动态库
ls -la out/xxx/packages/system/lib/module/libfile_api.z.so

# 检查符号表
nm -D out/xxx/packages/system/lib/module/libfile_api.z.so | grep -i fileio

# 检查依赖
ldd out/xxx/packages/system/lib/module/libfile_api.z.so
```

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [03_CodeMap.md](03_CodeMap.md) | 目录结构与代码地图 |
| [04_Interface.md](04_Interface.md) | API 接口文档 |
| [08_Internals.md](08_Internals.md) | 内部实现细节 |

---

**最后更新**：2026-02-07

**版本**：1.0