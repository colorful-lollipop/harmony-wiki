# @ohos.file.fs 模块

## 简介

`@ohos.file.fs` 是 OpenHarmony 推荐的文件系统 API 模块，提供完整的文件操作能力。

## 模块信息

| 属性 | 值 |
|------|-----|
| 模块名 | `file.fs` |
| 产物 | `libfs.z.so` |
| 位置 | `/system/lib/module/file/` |
| Syscap | `SystemCapability.FileManagement.File.FileIO` |

## API 清单

### 全局函数

#### open / openSync

```javascript
// Promise 异步
function open(path: string, mode?: number): Promise<File>

// Callback 异步
function open(path: string, mode: number, callback: AsyncCallback<File>): void

// 同步
function openSync(path: string, mode?: number): File
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/open.cpp`

**调用链**: `PropNExporter::Open` → `Open::Async/Sync` → `uv_fs_open()`

#### read / readSync

```javascript
function read(fd: number, buffer: ArrayBuffer, options?: { offset?: number, length?: number, position?: number }): Promise<number>
function readSync(fd: number, buffer: ArrayBuffer, options?: { offset?: number, length?: number, position?: number }): number
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/class_file/file_n_exporter.cpp`

#### write / writeSync

```javascript
function write(fd: number, buffer: ArrayBuffer | string, options?: { offset?: number, length?: number, position?: number, encoding?: string }): Promise<number>
function writeSync(fd: number, buffer: ArrayBuffer | string, options?: { offset?: number, length?: number, position?: number, encoding?: string }): number
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/class_file/file_n_exporter.cpp`

#### close / closeSync

```javascript
function close(fd: number): Promise<void>
function closeSync(fd: number): void
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/close.cpp`

#### copy / copySync

```javascript
function copy(src: string, dest: string, options?: { progressCallback?: ProgressCallback, signal?: TaskSignal }): Promise<void>
function copySync(src: string, dest: string): void
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/copy_core.cpp`

**特性**: 支持进度回调、取消信号

#### move / moveSync

```javascript
function move(src: string, dest: string): Promise<void>
function moveSync(src: string, dest: string): void
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/move_core.cpp`

#### mkdir / mkdirSync

```javascript
function mkdir(path: string, options?: { recursive?: boolean, mode?: number }): Promise<void>
function mkdirSync(path: string, options?: { recursive?: boolean, mode?: number }): void
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/mkdir_core.cpp`

#### rmdir / rmdirSync

```javascript
function rmdir(path: string): Promise<void>
function rmdirSync(path: string): void
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/rmdir_core.cpp`

#### access / accessSync

```javascript
function access(path: string, mode?: number): Promise<void>
function accessSync(path: string, mode?: number): void
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/properties/access_core.cpp`

#### stat / statSync / lstat / lstatSync

```javascript
function stat(path: string): Promise<Stat>
function statSync(path: string): Stat
function lstat(path: string): Promise<Stat>
function lstatSync(path: string): Stat
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/class_stat/stat_n_exporter.cpp`

#### createStream / createStreamSync

```javascript
function createStream(path: string, mode: string): Promise<Stream>
function createStreamSync(path: string, mode: string): Stream
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/class_stream/stream_n_exporter.cpp`

#### createWatcher

```javascript
function createWatcher(path: string, events: number, callback: WatcherCallback): Watcher
```

**C++ 实现**: `interfaces/kits/js/src/mod_fs/class_watcher/watcher_n_exporter.cpp`

**特性**: 基于 inotify，支持文件变化监控

## 类定义

### File 类

```javascript
interface File {
    readonly fd: number;
    readonly path: string;
    
    read(buffer: ArrayBuffer, options?: ReadOptions): Promise<number>;
    readSync(buffer: ArrayBuffer, options?: ReadOptions): number;
    
    write(buffer: ArrayBuffer | string, options?: WriteOptions): Promise<number>;
    writeSync(buffer: ArrayBuffer | string, options?: WriteOptions): number;
    
    close(): Promise<void>;
    closeSync(): void;
}
```

**C++ 实体**: `interfaces/kits/js/src/mod_fs/class_file/file_entity.h:31`

```cpp
struct FileEntity {
    std::unique_ptr<FDGuard> fd_;
    std::string path_;
    // ...
};
```

### Stream 类

```javascript
interface Stream {
    read(buffer: ArrayBuffer): Promise<number>;
    readSync(buffer: ArrayBuffer): number;
    
    write(buffer: ArrayBuffer | string): Promise<number>;
    writeSync(buffer: ArrayBuffer | string): number;
    
    flush(): Promise<void>;
    flushSync(): void;
    
    close(): Promise<void>;
    closeSync(): void;
}
```

**C++ 实体**: `interfaces/kits/js/src/mod_fs/class_stream/stream_entity.h:23`

### Stat 类

```javascript
interface Stat {
    readonly size: number;
    readonly mode: number;
    readonly uid: number;
    readonly gid: number;
    readonly atime: number;
    readonly mtime: number;
    readonly ctime: number;
    
    isFile(): boolean;
    isDirectory(): boolean;
    isSymbolicLink(): boolean;
}
```

**C++ 实体**: `interfaces/kits/js/src/mod_fs/class_stat/stat_entity.h:30`

### Watcher 类

```javascript
interface Watcher {
    stop(): void;
}
```

**C++ 实体**: `interfaces/kits/js/src/mod_fs/class_watcher/watcher_entity.h:84`

## 常量

### OpenMode

| 常量 | 值 | 说明 |
|------|-----|------|
| `READ_ONLY` | 0o0 | 只读打开 |
| `WRITE_ONLY` | 0o1 | 只写打开 |
| `READ_WRITE` | 0o2 | 读写打开 |
| `CREATE` | 0o100 | 文件不存在时创建 |
| `TRUNC` | 0o1000 | 截断文件 |
| `APPEND` | 0o2000 | 追加模式 |
| `NONBLOCK` | 0o4000 | 非阻塞 |
| `DIR` | 0o200000 | 目录 |
| `NOFOLLOW` | 0o400000 | 不跟随符号链接 |
| `SYNC` | 0o4010000 | 同步 IO |

**C++ 定义**: `interfaces/kits/js/src/mod_fs/module.cpp:56-60`

```cpp
InitOpenMode(env, exports);  // 初始化 OpenMode 常量
```

## 调用链示例

### fs.open() 完整调用链

详见 [附录 A: 关键调用链](../appendix/Callgraphs.md#fsopen---promise-模式)

## 权限要求

| 功能 | 权限 | 说明 |
|------|------|------|
| 基本文件操作 | 无 | 应用沙箱内 |
| 系统目录 | `ohos.permission.FILE_ACCESS_MANAGER` | 需系统应用 |

## 示例代码

### 基本文件操作

```javascript
import fs from '@ohos.file.fs';

// 写入文件
async function writeFile() {
    let file = await fs.open('test.txt', fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
    let buffer = new ArrayBuffer(1024);
    let view = new Uint8Array(buffer);
    view.set([0x48, 0x65, 0x6c, 0x6c, 0x6f]); // "Hello"
    
    await fs.write(file.fd, buffer);
    await fs.close(file.fd);
}

// 读取文件
async function readFile() {
    let file = await fs.open('test.txt', fs.OpenMode.READ_ONLY);
    let buffer = new ArrayBuffer(1024);
    let readLen = await fs.read(file.fd, buffer);
    
    let view = new Uint8Array(buffer);
    let text = String.fromCharCode.apply(null, view.slice(0, readLen));
    console.log(text);
    
    await fs.close(file.fd);
}

// 复制文件带进度
async function copyWithProgress() {
    let signal = new fs.TaskSignal();
    
    await fs.copy('src.txt', 'dst.txt', {
        progressCallback: (receivedSize, totalSize) => {
            let percent = (receivedSize / totalSize * 100).toFixed(2);
            console.log(`Progress: ${percent}%`);
        },
        signal: signal
    });
}

// 文件监控
function watchFile() {
    let watcher = fs.createWatcher('/path/to/watch', fs.EVENT_CREATE | fs.EVENT_DELETE, (events) => {
        console.log('File changed:', events);
    });
    
    // 停止监控
    watcher.stop();
}
```

## 参考文档

| 文档 | 路径 |
|------|------|
| 接口声明文件 | `interfaces/kits/js/src/mod_fs/` |
| 构建配置 | `interfaces/kits/js/BUILD.gn:137-275` |
| OpenHarmony 官方文档 | [js-apis-file-fs.md](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-file-fs.md) |
