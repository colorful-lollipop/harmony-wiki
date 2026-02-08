# 附录 A: 关键调用链

## @ohos.file.fs 调用链

### fs.open() - Promise 模式

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant FS as mod_fs module.cpp
    participant Prop as PropNExporter
    participant Open as open_core.cpp
    participant URI as RemoteUri
    participant Native as Native API
    participant UV as libuv
    participant Kernel as Kernel
    
    JS->>FS: fs.open(path, mode)
    FS->>FS: Export() 初始化
    FS->>FS: InitOpenMode()
    FS->>Prop: PropNExporter 导出
    
    Prop->>Prop: ExportAsync()
    Prop->>Open: Open::Async()
    
    Open->>Open: GetOpenMode()
    Open->>Open: ParseFileLocation()
    
    alt 远程 URI
        Open->>URI: IsRemoteUri()
        URI->>URI: GetCallingTokenID()
        URI->>URI: VerifyAccessToken()
        URI->>Native: DataShareHelper::Creator()
        Native-->>URI: fd
    else 本地路径
        Open->>Open: GetRealPath()
        Open->>Open: CheckBundleName()
    end
    
    Open->>UV: uv_fs_open()
    UV->>Kernel: ::open()
    Kernel-->>UV: fd
    UV-->>Open: callback
    
    Open->>Open: new FileEntity(fd)
    Open->>Prop: NClass::InstantiateClass()
    Prop-->>FS: Promise resolve
    FS-->>JS: File object
```

**关键文件路径**:
1. `interfaces/kits/js/src/mod_fs/module.cpp:54-88` - 模块注册
2. `interfaces/kits/js/src/mod_fs/properties/prop_n_exporter.cpp` - 属性导出
3. `interfaces/kits/js/src/mod_fs/properties/open.cpp` - open 实现
4. `interfaces/kits/native/remote_uri/remote_uri.cpp` - URI 处理
5. `interfaces/kits/js/src/mod_fs/class_file/file_entity.h` - File 实体

### fs.read() - Promise 模式

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant FileNExp as FileNExporter
    participant Entity as FileEntity
    participant UV as libuv
    participant Kernel as Kernel
    
    JS->>FileNExp: file.read(buffer, options)
    FileNExp->>FileNExp: Read()
    
    FileNExp->>Entity: NClass::GetEntityOf()
    Entity-->>FileNExp: fd
    
    FileNExp->>FileNExp: ValidateArgs()
    
    FileNExp->>UV: NAsyncWorkPromise::Schedule()
    UV->>Kernel: uv_fs_read() -> ::read/pread()
    Kernel-->>UV: bytesRead
    
    UV-->>FileNExp: PromiseOnComplete
    FileNExp->>FileNExp: CreateResultObject()
    FileNExp-->>JS: { bytesRead, buffer, offset }
```

**关键文件路径**:
1. `interfaces/kits/js/src/mod_fs/class_file/file_n_exporter.cpp` - File 类方法
2. `interfaces/kits/js/src/mod_fs/class_file/file_entity.h` - 实体定义
3. `utils/filemgmt_libn/src/n_async/n_async_work_promise.cpp` - Promise 实现

### fs.copy() - 带进度和取消

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant Prop as PropNExporter
    participant Copy as CopyCore
    participant Progress as TransListener
    participant Signal as TaskSignal
    participant UV as libuv
    participant Kernel as Kernel
    
    JS->>Prop: fs.copy(src, dst, options)
    Prop->>Copy: Copy::Async()
    
    Copy->>Copy: GetRealPath(src)
    Copy->>Copy: GetRealPath(dst)
    Copy->>Copy: ValidatePath()
    
    Copy->>Progress: CreateTransListener()
    Progress->>Progress: StartNotifyThread()
    
    alt 取消信号存在
        Copy->>Signal: CheckCancelIfNeed()
        Signal-->>Copy: canceled
        Copy-->>JS: ECANCELLED
    end
    
    loop 分块复制
        Copy->>Signal: IsCanceled()
        Copy->>UV: uv_fs_sendfile()
        UV->>Kernel: ::sendfile()
        Kernel-->>UV: bytesCopied
        UV-->>Copy: callback
        
        Copy->>Progress: OnProgress(bytesCopied/total)
        Progress->>JS: notifyCallback()
    end
    
    Copy->>Progress: StopNotifyThread()
    Copy-->>JS: Promise resolve
```

**关键文件路径**:
1. `interfaces/kits/js/src/mod_fs/properties/copy_core.cpp` - 复制核心
2. `interfaces/kits/js/src/mod_fs/properties/copy_listener/trans_listener.cpp` - 进度监听
3. `interfaces/kits/native/task_signal/task_signal.cpp` - 取消信号

## @ohos.fileio 调用链

### fileio.open() - Callback 模式

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant Module as mod_fileio module.cpp
    participant Prop as PropNExporter
    participant Open as properties/open.cpp
    participant UV as libuv
    participant Kernel as Kernel
    
    JS->>Module: fileio.open(path, mode, callback)
    Module->>Module: Export()
    Module->>Prop: PropNExporter 导出
    
    Prop->>Open: Open::Async()
    Open->>Open: GetPath()
    Open->>Open: GetMode()
    
    Open->>UV: NAsyncWorkCallback::Schedule()
    UV->>Kernel: uv_fs_open() -> ::open()
    Kernel-->>UV: fd
    
    UV-->>Open: CallbackComplete
    Open->>Open: new FileEntity(fd)
    Open->>Prop: NClass::InstantiateClass()
    Prop->>JS: callback(err, File)
```

**关键文件路径**:
1. `interfaces/kits/js/src/mod_fileio/module.cpp:33-53`
2. `interfaces/kits/js/src/mod_fileio/properties/prop_n_exporter.cpp`
3. `interfaces/kits/js/src/mod_fileio/properties/open.cpp`

## LibN 框架调用链

### NAsyncWorkPromise::Schedule()

```mermaid
sequenceDiagram
    participant Caller as 调用者
    participant NAWP as NAsyncWorkPromise
    participant NAPI as N-API
    participant UV as libuv
    participant Worker as Worker Thread
    participant Main as Main Thread
    
    Caller->>NAWP: Schedule(name, cbExec, cbComplete)
    
    NAWP->>NAPI: napi_create_promise()
    NAPI-->>NAWP: deferred, promise
    
    NAWP->>NAPI: napi_create_async_work()
    NAPI-->>NAWP: async_work
    
    NAWP->>UV: napi_queue_async_work()
    UV->>Worker: 调度到线程池
    
    Worker->>Worker: PromiseOnExec()
    Worker->>Caller: cbExec()
    Caller-->>Worker: UniError
    
    UV->>Main: PromiseOnComplete()
    Main->>Caller: cbComplete()
    Caller-->>Main: NVal
    
    alt 成功
        Main->>NAPI: napi_resolve_deferred()
    else 失败
        Main->>NAPI: napi_reject_deferred()
    end
    
    Main->>NAPI: napi_delete_async_work()
    
    NAPI-->>Caller: promise
```

**关键文件路径**:
1. `utils/filemgmt_libn/include/n_async/n_async_work_promise.h`
2. `utils/filemgmt_libn/src/n_async/n_async_work_promise.cpp`

### NClass::InstantiateClass()

```mermaid
sequenceDiagram
    participant Caller as 调用者
    participant NClass as NClass
    participant NAPI as N-API
    participant Entity as Entity Object
    
    Caller->>NClass: InstantiateClass(name, args)
    
    NClass->>NAPI: napi_get_named_property()
    NAPI-->>NClass: constructor
    
    NClass->>NAPI: napi_new_instance()
    NAPI-->>NClass: instance
    
    NClass->>Entity: new T(args)
    Entity-->>NClass: entity pointer
    
    NClass->>NAPI: napi_wrap()
    NAPI-->>NClass: wrapped instance
    
    NClass-->>Caller: instance
```

**关键文件路径**:
1. `utils/filemgmt_libn/include/n_class.h`
2. `utils/filemgmt_libn/src/n_class.cpp`

## 文件监控调用链 (Watcher)

### createWatcher()

```mermaid
sequenceDiagram
    participant JS as JavaScript
    participant WatcherExp as WatcherNExporter
    participant WatcherEntity as WatcherEntity
    participant FsWatcher as FsFileWatcher
    participant Thread as Watcher Thread
    participant Inotify as inotify
    participant Kernel as Kernel
    
    JS->>WatcherExp: fs.createWatcher(path, events, callback)
    
    WatcherExp->>WatcherExp: CreateWatcher()
    WatcherExp->>WatcherExp: ValidateEvents()
    
    WatcherExp->>FsWatcher: FsFileWatcher::GetInstance()
    FsWatcher->>FsWatcher: AddWatcher(path, events, callback)
    
    alt 首次创建
        FsWatcher->>FsWatcher: Start()
        FsWatcher->>Thread: Create taskThread_
        Thread->>Inotify: inotify_init1()
        Thread->>Inotify: inotify_add_watch()
    end
    
    FsWatcher->>WatcherEntity: new WatcherEntity()
    WatcherEntity-->>WatcherExp: entity
    
    WatcherExp->>WatcherExp: NClass::InstantiateClass()
    WatcherExp-->>JS: Watcher object
    
    loop 事件监听
        Kernel->>Inotify: 文件系统事件
        Inotify-->>Thread: read(inotify_fd)
        Thread->>FsWatcher: ProcessEvent()
        FsWatcher->>JS: InvokeCallback(event)
    end
```

**关键文件路径**:
1. `interfaces/kits/js/src/mod_fs/class_watcher/watcher_n_exporter.cpp`
2. `interfaces/kits/js/src/mod_fs/class_watcher/fs_file_watcher.cpp`
3. `interfaces/kits/js/src/mod_fs/class_watcher/watcher_entity.h`

## HyperAIO 调用链

### HyperAio::Read()

```mermaid
sequenceDiagram
    participant Caller as 调用者
    participant HyperAio as HyperAio
    participant Impl as HyperAio::Impl
    participant Queue as Request Queue
    participant Harvest as Harvest Thread
    participant URING as io_uring
    participant Kernel as Kernel
    
    Caller->>HyperAio: Read(fd, buf, len, offset, callback)
    
    HyperAio->>Impl: ValidateReqNum()
    Impl-->>HyperAio: OK
    
    HyperAio->>Impl: SubmitReadRequest()
    Impl->>URING: io_uring_get_sqe()
    Impl->>URING: io_uring_prep_read()
    Impl->>URING: io_uring_submit()
    
    URING->>Kernel: 提交 IO 请求
    
    Kernel-->>URING: IO 完成
    
    URING->>Harvest: CQE 就绪
    Harvest->>Harvest: io_uring_peek_cqe()
    Harvest->>Impl: ProcessIoResult()
    Impl->>Caller: callback(result)
```

**关键文件路径**:
1. `interfaces/kits/hyperaio/include/hyperaio.h`
2. `interfaces/kits/hyperaio/src/hyperaio.cpp`

## 参考索引

| 调用链 | 入口文件 | 核心实现 | 辅助文件 |
|--------|----------|----------|----------|
| fs.open | `mod_fs/properties/open.cpp` | `open_core.cpp` | `remote_uri.cpp`, `file_entity.h` |
| fs.read | `mod_fs/class_file/file_n_exporter.cpp` | `Read()` | `n_async_work_promise.cpp` |
| fs.copy | `mod_fs/properties/copy.cpp` | `copy_core.cpp` | `trans_listener.cpp`, `task_signal.cpp` |
| fileio.open | `mod_fileio/properties/open.cpp` | `Open::Async()` | `prop_n_exporter.cpp` |
| Promise 异步 | `n_async_work_promise.cpp` | `Schedule()` | `n_async_context.h` |
| Watcher | `watcher_n_exporter.cpp` | `CreateWatcher()` | `fs_file_watcher.cpp` |
| HyperAIO | `hyperaio.cpp` | `Read/Write()` | `hyperaio.h` |
