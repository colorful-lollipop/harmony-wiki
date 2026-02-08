# File API 目录结构与代码地图

本文档提供 File API 的完整目录结构和代码导航地图，帮助开发者快速定位关键代码文件，理解各模块的组织方式。

## 1. 目录结构总览

### 1.1 顶层目录

**证据来源**：`ls -la /Volumes/lexar/code/d/work/oh/foundation/filemanagement/file_api/`

```
foundation/filemanagement/file_api/
├── figures/                     # 架构图和文档资源
├── interfaces/                   # API 接口层（核心代码）
│   └── kits/                   # Kits 接口暴露
├── utils/                      # 公共组件库
├── wiki/                       # 文档目录
└── _work/                      # 工作目录（评估、计划、笔记）
```

**目录职责说明**：

| 目录 | 职责 | 主要内容 |
|------|------|----------|
| figures | 文档资源 | 架构图、流程图 |
| interfaces/kits | 对外接口 | N-API、C API、Native API 实现 |
| utils | 公共组件 | LibN、libfs、libhilog |
| wiki | 文档 | Wiki 文档 |
| _work | 工作目录 | 评估、计划、笔记 |

### 1.2 接口目录结构

**证据来源**：`ls -laR /Volumes/lexar/code/d/work/oh/foundation/filemanagement/file_api/interfaces/kits/`

```
interfaces/kits/
├── c/                          # C 语言接口
│   ├── common/                 # 公共定义
│   │   └── error_code.h       # 错误码定义
│   ├── environment/            # 环境接口
│   │   ├── BUILD.gn
│   │   ├── environment.c
│   │   ├── environment.h
│   │   └── libenvironment.ndk.json
│   └── fileio/                 # 文件 I/O 接口
│       ├── BUILD.gn
│       ├── fileio.c
│       ├── fileio.h
│       └── libfileio.ndk.json
│
├── cj/                         # Cangjie FFI
│   ├── BUILD.gn
│   └── src/                    # FFI 实现（61 个文件）
│       ├── copy.cpp/h
│       ├── file_ffi.cpp/h
│       ├── file_fs_ffi.cpp/h
│       ├── file_fs_impl.cpp/h
│       ├── stat_ffi.cpp/h
│       ├── stat_impl.cpp/h
│       ├── stream_ffi.cpp/h
│       ├── stream_impl.cpp/h
│       ├── watcher_impl.cpp/h
│       ├── translistener.cpp/h
│       └── ... (更多文件)
│
├── hyperaio/                   # 高性能异步 I/O
│   ├── BUILD.gn
│   ├── include/
│   │   ├── hyperaio.h        # 主头文件
│   │   ├── hyperaio_trace.h  # 跟踪支持
│   │   └── libhilog.h        # 日志接口
│   └── src/
│       ├── hyperaio.cpp       # 核心实现
│       └── hyperaio_trace.cpp
│
├── js/                        # JavaScript N-API（核心）
│   ├── BUILD.gn
│   └── src/
│       ├── common/             # 公共工具
│       ├── mod_document/      # 文档模块
│       ├── mod_environment/   # 环境模块
│       ├── mod_file/         # 沙箱文件模块
│       ├── mod_fileio/       # 文件 I/O 模块
│       ├── mod_fs/           # 文件系统模块
│       ├── mod_hash/         # 哈希模块
│       ├── mod_securitylabel/# 安全标签模块
│       ├── mod_statfs/       # 文件系统状态模块
│       └── mod_statvfs/       # 统计信息模块
│
├── native/                     # Native 接口
│   ├── remote_uri/            # 远程 URI
│   ├── environment_native/   # 原生环境
│   ├── fileio_native/        # 原生文件 I/O
│   └── rust_file/            # Rust 接口
│
└── rust/                       # Rust 接口
    └── include/
        └── rust_file.h
```

### 1.3 工具目录结构

**证据来源**：`ls -laR /Volumes/lexar/code/d/work/oh/foundation/filemanagement/file_api/utils/`

```
utils/
├── common/                     # 通用工具
│   └── include/
│       └── file_utils.h       # 文件工具函数
│
├── filemgmt_libfs/            # 文件系统封装库
│   ├── BUILD.gn
│   ├── include/
│   │   ├── filemgmt_libfs.h  # 库主头文件
│   │   ├── fs_array_buffer.h
│   │   ├── fs_error.h       # 错误定义
│   │   └── fs_result.h       # 结果封装
│   └── src/
│       └── fs_error.cpp
│
├── filemgmt_libhilog/         # 日志组件
│   ├── BUILD.gn
│   └── filemgmt_libhilog.h
│
└── filemgmt_libn/             # N-API 抽象层（核心）
    ├── BUILD.gn
    ├── include/
    │   ├── filemgmt_libn.h   # 库主头文件
    │   ├── n_async/          # 异步工作封装
    │   │   ├── n_async_context.h
    │   │   ├── n_async_work.h
    │   │   ├── n_async_work_callback.h
    │   │   ├── n_async_work_promise.h
    │   │   └── n_ref.h
    │   ├── n_class.h         # 类封装
    │   ├── n_error.h         # 错误处理
    │   ├── n_exporter.h      # 导出器
    │   ├── n_func_arg.h      # 函数参数
    │   ├── n_napi.h         # N-API 辅助
    │   └── n_val.h          # 值转换
    └── src/
        ├── n_async/          # 异步实现
        │   ├── n_async_work_callback.cpp
        │   ├── n_async_work_promise.cpp
        │   └── n_ref.cpp
        ├── n_class.cpp
        ├── n_error.cpp
        ├── n_func_arg.cpp
        └── n_val.cpp
```

---

## 2. 核心文件定位

### 2.1 模块入口文件

每个 N-API 模块都有一个入口文件（module.cpp 或 *_napi.cpp），负责模块注册：

| 模块 | 入口文件 | 行数 |
|------|----------|------|
| @ohos.fileio | `interfaces/kits/js/src/mod_fileio/module.cpp` | ~200 行 |
| @ohos.file.fs | `interfaces/kits/js/src/mod_fs/module.cpp` | ~300 行 |
| @ohos.file.hash | `interfaces/kits/js/src/mod_hash/module.cpp` | ~150 行 |
| @ohos.file.statfs | `interfaces/kits/js/src/mod_statfs/statfs_napi.cpp` | ~150 行 |
| @ohos.file.statvfs | `interfaces/kits/js/src/mod_statvfs/statvfs_napi.cpp` | ~150 行 |
| @ohos.file.environment | `interfaces/kits/js/src/mod_environment/environment_napi.cpp` | ~150 行 |
| @ohos.file.securityLabel | `interfaces/kits/js/src/mod_securitylabel/securitylabel_napi.cpp` | ~150 行 |
| @ohos.file.document | `interfaces/kits/js/src/mod_document/document_napi.cpp` | ~150 行 |
| @ohos.file.file | `interfaces/kits/js/src/mod_file/module.cpp` | ~150 行 |

### 2.2 模块注册点示例

**证据来源**：`interfaces/kits/js/src/mod_fileio/module.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fileio/module.cpp
// 功能：模块注册入口

// N-API 模块定义
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = InitModule,
    .nm_modname = "fileio",
    .nm_priv = nullptr,
    .reserved = { 0 }
};

// 模块初始化函数
napi_value InitModule(napi_env env, napi_value exports) {
    // 注册类
    napi_value FileStat = RegisterStatClass(env, exports);
    napi_value Dir = RegisterDirClass(env, exports);
    napi_value Stream = RegisterStreamClass(env, exports);
    napi_value Watcher = RegisterWatcherClass(env, exports);
    
    // 注册函数
    napi_property_descriptor descriptors[] = {
        DECLARE_NAPI_FUNCTION("open", Open),
        DECLARE_NAPI_FUNCTION("close", Close),
        DECLARE_NAPI_FUNCTION("read", Read),
        // ... 更多函数
    };
    
    napi_define_properties(env, exports, 
        sizeof(descriptors) / sizeof(descriptors[0]), descriptors);
    
    return exports;
}

// 模块注册调用
NAPI_MODULE(fileio, InitModule)
```

### 2.3 类定义文件

| 类名 | 头文件 | 实现文件 | 模块 |
|------|--------|----------|------|
| File | `file_entity.h` | `file_n_exporter.cpp` | @ohos.fileio |
| Dir | `dir_entity.h` | `dir_n_exporter.cpp` | @ohos.fileio |
| Stream | `stream_entity.h` | `stream_n_exporter.cpp` | @ohos.fileio |
| Stat | `stat_entity.h` | `stat_n_exporter.cpp` | @ohos.fileio |
| Watcher | `watcher_entity.h` | `watcher_n_exporter.cpp` | @ohos.fileio |
| FS_File | `file_entity.h` | `fs_file.cpp` | @ohos.file.fs |
| FS_Stream | `stream_entity.h` | `fs_stream.cpp` | @ohos.file.fs |
| FS_Stat | `stat_entity.h` | `fs_stat.cpp` | @ohos.file.fs |
| FS_Watcher | `watcher_entity.h` | `fs_watcher.cpp` | @ohos.file.fs |

### 2.4 属性操作文件

| 功能 | 文件路径 | 模块 |
|------|----------|------|
| open | `mod_fs/properties/open.cpp` | @ohos.file.fs |
| close | `mod_fs/properties/close.cpp` | @ohos.file.fs |
| read | `mod_fs/properties/read.cpp` | @ohos.file.fs |
| write | `mod_fs/properties/write.cpp` | @ohos.file.fs |
| stat | `mod_fs/properties/stat.cpp` | @ohos.file.fs |
| mkdir | `mod_fs/properties/mkdir.cpp` | @ohos.file.fs |
| rmdir | `mod_fs/properties/rmdir.cpp` | @ohos.file.fs |
| unlink | `mod_fs/properties/unlink.cpp` | @ohos.file.fs |
| rename | `mod_fs/properties/rename.cpp` | @ohos.file.fs |
| copy | `mod_fs/properties/copy.cpp` | @ohos.file.fs |
| move | `mod_fs/properties/move.cpp` | @ohos.file.fs |
| truncate | `mod_fs/properties/truncate.cpp` | @ohos.file.fs |
| access | `mod_fs/properties/access.cpp` | @ohos.file.fs |
| chmod | `mod_fs/properties/chmod.cpp` | @ohos.file.fs |
| chown | `mod_fs/properties/chown.cpp` | @ohos.file.fs |
| symlink | `mod_fs/properties/symlink.cpp` | @ohos.file.fs |
| link | `mod_fs/properties/link.cpp` | @ohos.file.fs |
| utimes | `mod_fs/properties/utimes.cpp` | @ohos.file.fs |
| lstat | `mod_fs/properties/lstat.cpp` | @ohos.file.fs |
| lseek | `mod_fs/properties/lseek.cpp` | @ohos.file.fs |
| fsync | `mod_fs/properties/fsync.cpp` | @ohos.file.fs |
| fdatasync | `mod_fs/properties/fdatasync.cpp` | @ohos.file.fs |
| open_dir | `mod_fs/properties/open_dir.cpp` | @ohos.file.fs |
| read_dir | `mod_fs/properties/read_dir.cpp` | @ohos.file.fs |
| list_file | `mod_fs/properties/listfile.cpp` | @ohos.file.fs |
| watcher | `mod_fs/properties/watcher.cpp` | @ohos.file.fs |

---

## 3. 代码导航地图

### 3.1 功能到文件映射

| 功能需求 | 关键文件 | 说明 |
|----------|----------|------|
| **模块注册** | `module.cpp` | 各模块的入口文件 |
| **类定义** | `*_entity.h` + `*_n_exporter.cpp` | JS 对象的 C++ 映射 |
| **函数实现** | `properties/*.cpp` | 具体 API 的实现 |
| **工具函数** | `common/*.cpp` | 公共工具代码 |
| **错误处理** | `common/uni_error.cpp` | 统一错误处理 |
| **参数解析** | `common/napi/n_val.cpp` | JS 参数转换 |
| **异步工作** | `common/napi/n_async/*.cpp` | 异步操作管理 |

### 3.2 关键代码位置速查

#### 3.2.1 文件打开

| 操作 | 文件 | 行号范围 |
|------|------|----------|
| N-API 入口 | `mod_fs/properties/open.cpp` | ~100 行 |
| 参数解析 | `common/napi/n_val.cpp` | ~50 行 |
| URI 解析 | `common_func.cpp` | ~80 行 |
| 权限检查 | `mod_fs/properties/access.cpp` | ~60 行 |
| 系统调用 | `mod_fs/properties/open_core.cpp` | ~40 行 |

#### 3.2.2 文件读写

| 操作 | 文件 | 行号范围 |
|------|------|----------|
| 读入口 | `mod_fs/properties/read.cpp` | ~120 行 |
| 写入口 | `mod_fs/properties/write.cpp` | ~120 行 |
| 缓冲区处理 | `common/file_filter.h` | ~50 行 |
| 错误处理 | `common/uni_error.cpp` | ~80 行 |

#### 3.2.3 目录操作

| 操作 | 文件 | 行号范围 |
|------|------|----------|
| 创建目录 | `mod_fs/properties/mkdir_core.cpp` | ~60 行 |
| 删除目录 | `mod_fs/properties/rmdir_core.cpp` | ~40 行 |
| 读取目录 | `mod_fs/properties/read_dir.cpp` | ~80 行 |
| 列出文件 | `mod_fs/properties/listfile_core.cpp` | ~100 行 |

#### 3.2.4 文件属性

| 操作 | 文件 | 行号范围 |
|------|------|----------|
| 获取属性 | `mod_fs/properties/stat.cpp` | ~100 行 |
| 修改权限 | `mod_fs/properties/chmod.cpp` | ~60 行 |
| 修改所有者 | `mod_fs/properties/chown.cpp` | ~60 行 |
| 修改时间 | `mod_fs/properties/utimes.cpp` | ~50 行 |

### 3.3 常用代码片段

#### 3.3.1 N-API 函数模板

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp
// 功能：N-API 异步函数模板

napi_value OpenAsync(napi_env env, napi_callback_info info) {
    // 1. 参数解析
    size_t argc = 3;
    napi_value argv[3];
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
    
    // 2. 获取字符串参数
    std::string path;
    if (!GetStringParam(env, path, argv[0])) {
        NAPI_ASSERT(env, false, "Failed to get path");
        return nullptr;
    }
    
    // 3. 获取数值参数
    int32_t flags;
    napi_get_value_int32(env, argv[1], &flags);
    
    // 4. 创建异步上下文
    OpenContext* ctx = new OpenContext();
    ctx->path = path;
    ctx->flags = flags;
    
    // 5. 创建异步工作
    napi_value resourceName;
    napi_create_string_utf8(env, "OpenFile", NAPI_AUTO_LENGTH, &resourceName);
    
    napi_create_async_work(env, nullptr, resourceName,
        [](napi_env env, void* data) {
            // 执行回调：打开文件
            OpenContext* ctx = static_cast<OpenContext*>(data);
            ctx->fd = open(ctx->path.c_str(), ctx->flags);
            ctx->error = (ctx->fd < 0) ? errno : 0;
        },
        [](napi_env env, napi_status status, void* data) {
            // 完成回调
            OpenContext* ctx = static_cast<OpenContext*>(data);
            if (ctx->error != 0) {
                napi_throw_errno(env, ctx->error);
            } else {
                napi_create_int32(env, ctx->fd, &ctx->result);
            }
            delete ctx;
        },
        ctx, &ctx->work);
    
    // 6. 队列异步工作
    napi_queue_async_work(env, ctx->work);
    
    return nullptr;
}
```

#### 3.3.2 类注册模板

**证据来源**：`interfaces/kits/js/src/mod_fs/class_file/file_n_exporter.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/class_file/file_n_exporter.cpp
// 功能：JS 类注册模板

napi_value RegisterFileClass(napi_env env, napi_value exports) {
    // 1. 定义构造函数
    napi_value constructor;
    napi_define_class(env, "File", NAPI_AUTO_LENGTH,
        [](napi_env env, napi_callback_info info) -> napi_value {
            // 构造函数实现
            size_t argc = 1;
            napi_value argv[1];
            napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);
            
            FileEntity* entity = new FileEntity();
            napi_wrap(env, argv[0], entity, 
                [](napi_env env, void* data) {
                    delete static_cast<FileEntity*>(data);
                }, nullptr, nullptr);
            
            return argv[0];
        },
        &constructor);
    
    // 2. 注册属性
    napi_property_descriptor properties[] = {
        DECLARE_NAPI_GETTER("fd", GetFd, constructor),
        DECLARE_NAPI_GETTER("name", GetName, constructor),
        // ... 更多属性
    };
    
    napi_define_properties(env, constructor,
        sizeof(properties) / sizeof(properties[0]), properties);
    
    // 3. 导出类
    napi_set_named_property(env, exports, "File", constructor);
    
    return exports;
}
```

---

## 4. 构建配置

### 4.1 GN 构建文件

| 构建文件 | 目标 | 说明 |
|----------|------|------|
| `bundle.json` | 组件配置 | 声明组件、依赖、features |
| `file_api.gni` | GN 参数 | 定义 Feature 开关 |
| `interfaces/kits/js/BUILD.gn` | JS 模块 | 构建 N-API 模块 |
| `interfaces/kits/c/*/BUILD.gn` | C 模块 | 构建 C API |
| `interfaces/kits/native/*/BUILD.gn` | Native 模块 | 构建 Native API |
| `utils/*/BUILD.gn` | 工具库 | 构建公共组件 |

### 4.2 构建目标清单

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

| 目标名称 | 类型 | 产物 |
|----------|------|------|
| ani_file_api | 静态库 | libani_file_api.a |
| build_kits_js | 动态库 | libfile_api.z.so |
| streamrw_packages | NPM 包 | @ohos.streamrw |
| streamhash_packages | NPM 包 | @ohos.streamhash |
| fs_ffi_packages | CJ 包 | @cj/fs_ffi |
| group_hyperaio | 静态库 | libhyperaio.a |

---

## 5. 测试目录

### 5.1 测试代码位置

以下测试目录按规范应被忽略：

| 目录 | 说明 |
|------|------|
| `interfaces/kits/test/` | 接口测试 |
| `test/` | 单元测试 |
| `unittest/` | 单元测试 |
| `fuzztest/` | 模糊测试 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [02_Architecture.md](02_Architecture.md) | 架构设计 |
| [04_Interface.md](04_Interface.md) | API 接口文档 |

---

**最后更新**：2026-02-07

**版本**：1.0
