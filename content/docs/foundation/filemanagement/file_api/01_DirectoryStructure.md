# 目录结构

## 顶层目录

```
foundation/filemanagement/file_api
├── figures/                    # 仓库图床
├── interfaces/                 # 对外接口实现
│   └── kits/                   # 各语言接口套件
├── utils/                      # 公共组件
├── wiki/                       # 工程 Wiki（本文档）
├── file_api.gni                # GN 全局配置
├── bundle.json                 # 组件配置
└── README.md                   # 项目说明
```

## 详细目录树

### interfaces/kits/ - 接口套件

```
interfaces/kits/
├── js/src/                     # N-API / ANI JavaScript 绑定
│   ├── mod_fs/                 # @ohos.file.fs 模块
│   │   ├── class_file/         # File 类实现
│   │   ├── class_stream/       # Stream 类实现
│   │   ├── class_stat/         # Stat 类实现
│   │   ├── class_watcher/      # Watcher 类实现
│   │   ├── class_tasksignal/   # TaskSignal 类实现
│   │   ├── class_randomaccessfile/  # RandomAccessFile 类
│   │   ├── class_readeriterator/    # ReaderIterator 类
│   │   ├── class_atomicfile/   # AtomicFile 类实现
│   │   ├── properties/         # 属性方法实现
│   │   │   ├── open.cpp        # open/openSync
│   │   │   ├── read.cpp        # read/readSync
│   │   │   ├── write.cpp       # write/writeSync
│   │   │   ├── copy.cpp        # copy/copySync
│   │   │   ├── move.cpp        # move/moveSync
│   │   │   └── ...
│   │   ├── module.cpp          # 模块注册
│   │   └── common_func.cpp     # 公共函数
│   │
│   ├── mod_fileio/             # @ohos.fileio 旧版模块
│   │   ├── class_dir/          # Dir 类
│   │   ├── class_stream/       # Stream 类
│   │   ├── class_stat/         # Stat 类
│   │   ├── class_dirent/       # Dirent 类
│   │   ├── class_file/         # File 类
│   │   ├── class_watcher/      # Watcher 类
│   │   ├── properties/         # 属性方法
│   │   └── module.cpp          # 模块注册
│   │
│   ├── mod_hash/               # @ohos.file.hash 模块
│   ├── mod_statvfs/            # @ohos.file.statvfs 模块
│   ├── mod_statfs/             # @ohos.file.statfs 模块
│   ├── mod_environment/        # @ohos.file.environment 模块
│   ├── mod_securitylabel/      # @ohos.file.securityLabel 模块
│   ├── mod_document/           # @ohos.file.document 模块
│   ├── mod_file/               # @system.file 模块
│   │
│   ├── common/                 # 公共代码
│   │   ├── napi/               # N-API 封装
│   │   │   ├── n_async/        # 异步工作实现
│   │   │   ├── n_class.cpp     # NClass 实现
│   │   │   ├── n_val.cpp       # NVal 实现
│   │   │   └── ...
│   │   ├── file_helper/        # 文件辅助
│   │   │   └── fd_guard.cpp    # FDGuard 实现
│   │   └── ani_helper/         # ANI 辅助
│   │
│   └── BUILD.gn                # JS 模块构建配置（1194行）
│
├── native/                     # Native C++ 接口
│   ├── task_signal/            # 任务信号（取消机制）
│   │   ├── task_signal.h       # TaskSignal 类定义
│   │   └── task_signal.cpp     # 实现
│   ├── remote_uri/             # 远程 URI 处理
│   ├── environment/            # 环境目录 Native 实现
│   └── fileio/                 # FileIO Native 实现
│
├── rust/                       # Rust FFI
│   ├── src/
│   │   ├── lib.rs              # Rust API 实现
│   │   └── ffi.rs              # FFI 绑定
│   └── include/rust_file.h     # C 头文件
│
├── cj/                         # Cangjie FFI
│   └── src/
│       ├── file_impl.h         # FileEntity 实现
│       ├── file_fs_impl.h      # FileFsImpl 实现
│       └── ...
│
├── c/                          # C NDK 接口
│   ├── fileio/
│   │   ├── fileio.h            # C API 头文件
│   │   └── fileio.cpp          # C API 实现
│   └── environment/
│
├── hyperaio/                   # HyperAIO 高性能异步 IO
│   ├── include/hyperaio.h      # 公共头文件
│   └── src/hyperaio.cpp        # io_uring 实现
│
└── ts/                         # TypeScript 类型定义和流实现
    ├── streamrw/               # Stream 读写
    └── streamhash/             # Stream 哈希
```

### utils/ - 工具库

```
utils/
├── filemgmt_libn/              # N-API 平台抽象层（LibN）
│   ├── include/
│   │   ├── filemgmt_libn.h     # 统一头文件
│   │   ├── n_napi.h            # N-API 封装
│   │   ├── n_val.h             # NVal 值封装
│   │   ├── n_class.h           # NClass 类封装
│   │   ├── n_func_arg.h        # 参数处理
│   │   ├── n_exporter.h        # 导出器基类
│   │   ├── n_error.h           # 错误处理
│   │   └── n_async/            # 异步框架
│   │       ├── n_async_work.h
│   │       ├── n_async_work_promise.h
│   │       ├── n_async_work_callback.h
│   │       └── n_ref.h
│   └── src/                    # 实现文件
│
├── filemgmt_libfs/             # 文件系统通用接口
│   ├── include/
│   │   ├── filemgmt_libfs.h    # 统一头文件
│   │   ├── fs_result.h         # 结果类型
│   │   ├── fs_error.h          # 错误类型
│   │   └── fs_array_buffer.h   # ArrayBuffer 处理
│   └── src/
│
├── filemgmt_libhilog/          # 日志组件
│   └── filemgmt_libhilog.h
│
└── common/                     # 其他公共代码
    └── include/file_utils.h    # 文件工具
```

## 模块职责

### LibN 框架 (utils/filemgmt_libn)

**职责**: 封装 N-API 底层接口，提供面向对象的 C++ API

**核心类**:
- `NVal`: N-API 值封装，类型转换
- `NClass`: JS 类定义和实例化
- `NExporter`: 模块导出器基类
- `NAsyncWorkPromise/Callback`: 异步工作封装

**证据**: `utils/filemgmt_libn/include/n_exporter.h:29`

### FileIO 模块 (interfaces/kits/js/src/mod_fileio)

**职责**: 旧版文件 IO API 实现

**导出类**:
- `PropNExporter`: 全局函数（access, mkdir, read 等）
- `FileNExporter`: File 类
- `DirNExporter`: Dir 类
- `StreamNExporter`: Stream 类
- `StatNExporter`: Stat 类
- `WatcherNExporter`: Watcher 类

**证据**: `interfaces/kits/js/src/mod_fileio/module.cpp:33-53`

### FS 模块 (interfaces/kits/js/src/mod_fs)

**职责**: 新版文件系统 API 实现（推荐）

**导出类**:
- `PropNExporter`: 全局函数
- `FileNExporter`: File 类（V9 增强版）
- `StreamNExporter`: Stream 类
- `StatNExporter`: Stat 类
- `WatcherNExporter`: Watcher 类（inotify 实现）
- `RandomAccessFileNExporter`: 随机访问文件
- `ReaderIteratorNExporter`: 行读取迭代器
- `TaskSignalNExporter`: 任务取消信号

**证据**: `interfaces/kits/js/src/mod_fs/module.cpp:54-88`

### HyperAIO 模块 (interfaces/kits/hyperaio)

**职责**: 基于 io_uring 的高性能异步 IO

**核心类**: `HyperAio`

**使用条件**:
- Feature Flag: `file_api_feature_hyperaio`
- 权限: `ohos.permission.ALLOW_IOURING`

**证据**: `interfaces/kits/hyperaio/include/hyperaio.h:84`

## 代码文件统计

| 目录 | 用途 | 估算代码量 |
|------|------|-----------|
| interfaces/kits/js/src/ | JS API 实现 | ~150 文件 |
| interfaces/kits/native/ | Native 接口 | ~10 文件 |
| utils/filemgmt_libn/ | LibN 框架 | ~20 文件 |
| utils/filemgmt_libfs/ | LibFS 工具 | ~5 文件 |

## 关键文件速查

| 文件 | 说明 | 行号 |
|------|------|------|
| `interfaces/kits/js/src/mod_fs/module.cpp` | FS 模块注册 | 108 |
| `interfaces/kits/js/src/mod_fileio/module.cpp` | FileIO 模块注册 | 57 |
| `interfaces/kits/js/BUILD.gn` | JS 模块构建配置 | 1194 |
| `file_api.gni` | 全局 GN 配置 | 26 |
| `utils/filemgmt_libn/include/n_exporter.h` | 导出器基类 | 47 |
