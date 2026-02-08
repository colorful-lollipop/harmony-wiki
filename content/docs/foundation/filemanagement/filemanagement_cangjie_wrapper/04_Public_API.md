# 对外 Cangjie API 文档

> 目的：提供对外的 Cangjie API 清单、参数、返回值、使用示例
> 适用范围：仓颉开发者使用文件管理 API
> 最后更新：2026-02-06

## API 入口点

### Kit 层导入

**证据**：`kit/CoreFileKit/index.cj:18-21`

```cangjie
package kit.CoreFileKit

public import ohos.file.fs.*
public import ohos.file.fileuri.*
```

**使用方式**：
```cangjie
import kit.CoreFileKit.*

// 直接使用 ohos.file.fs 和 ohos.file.fileuri
let file = FileIo.open("/data/storage/el2/base/files/test.txt")
```

---

## FileIo 静态类 API

### 文件操作

#### open()

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `open()` | `FileIo.open(path, mode!)` | cj_file_fs.cj:1270 | 打开文件 | `path: String` - 文件路径<br/>`mode: Int64` - 打开模式 | `File` | 同步 |
| `dup()` | `FileIo.dup(fd)` | cj_file_fs.cj:1301 | 复制 FD | `fd: Int32` - 文件描述符 | `File` | 同步 |

**示例**：
```cangjie
import kit.CoreFileKit.*

// 以读写模式打开文件
let file = FileIo.open("/data/storage/el2/base/files/test.txt", OpenMode.READ_WRITE)

// 以只读模式打开
let fileReadOnly = FileIo.open("/data/storage/el2/base/files/test.txt", OpenMode.READ_ONLY)
```

#### 文件信息

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `stat()` | `FileIo.stat(file)` | cj_file_fs.cj:992 | 通过路径获取 stat | `file: String` - 文件路径 | `Stat` | 异步 |
| `stat()` | `FileIo.stat(file)` | cj_file_fs.cj:955 | 通过 FD 获取 stat | `file: Int32` - 文件描述符 | `Stat` | 异步 |
| `lstat()` | `FileIo.lstat(path)` | cj_file_fs.cj:1158 | 获取符号链接 stat | `path: String` - 文件路径 | `Stat` | 异步 |
| `access()` | `FileIo.access(path, mode!, flag!)` | cj_file_fs.cj:1202 | 检查访问权限 | `path: String`<br/>`mode: AccessModeType`<br/>`flag: AccessFlagType` | `Bool` | 异步 |

**示例**：
```cangjie
// 检查文件是否存在且可读写
if (FileIo.access("/data/storage/el2/base/files/test.txt",
    AccessModeType.ReadWrite,
    AccessFlagType.Local)) {
    // 文件存在且有读写权限
}
```

#### 文件删除与重命名

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `unlink()` | `FileIo.unlink(path)` | （需查找） | 删除文件 | `path: String` - 文件路径 | `Unit` | 同步 |
| `rename()` | `FileIo.rename(oldFile, newFile)` | （需查找） | 重命名文件 | `oldFile: String`<br/>`newFile: String` | `Unit` | 同步 |

#### 文件读写

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `read()` | `FileIo.read(fd, buffer, options!)` | cj_file_fs.cj:1339 | 从文件读取 | `fd: Int32`<br/>`buffer: Array<Byte>`<br/>`options: ReadOptions` | `Int64` | 异步 |
| `write()` | `FileIo.write(fd, buffer, options!)` | cj_file_fs.cj:1396 | 向文件写入 | `fd: Int32`<br/>`buffer: Array<Byte>`<br/>`options: WriteOptions` | `Int64` | 异步 |

**示例**：
```cangjie
let file = FileIo.open("/data/storage/el2/base/files/test.txt", OpenMode.READ_WRITE)

// 写入数据
let buffer = Array<Byte>("Hello World".getBytes())
FileIo.write(file.fd, buffer, WriteOptions(encoding: "utf-8"))

// 读取数据
let readBuffer = Array<Byte>(1024)
let bytesRead = FileIo.read(file.fd, readBuffer)
```

#### 目录操作

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `mkdir()` | `FileIo.mkdir(path, recursion!)` | （需查找） | 创建目录 | `path: String`<br/>`recursion: Bool` - 是否递归创建 | `Int32` | 同步 |
| `rmdir()` | `FileIo.rmdir(path)` | （需查找） | 删除目录 | `path: String` - 目录路径 | `Int32` | 同步 |
| `listFile()` | `FileIo.listFile(path, options!)` | （需查找） | 列出文件 | `path: String`<br/>`options: ListFileOptions` | `Array<String>` | 异步 |
| `copyFile()` | `FileIo.copyFile(src, dest, mode!)` | （需查找） | 复制文件 | `src: String`<br/>`dest: String`<br/>`mode: Int32` | `Unit` | 同步 |
| `copyDir()` | `FileIo.copyDir(src, dest, mode!)` | （需查找） | 复制目录 | `src: String`<br/>`dest: String`<br/>`mode: Int32` | `Array<ConflictFiles>` | 同步 |
| `moveFile()` | `FileIo.moveFile(src, dest, mode!)` | （需查找） | 移动文件 | `src: String`<br/>`dest: String`<br/>`mode: Int32` | `Unit` | 同步 |
| `moveDir()` | `FileIo.moveDir(src, dest, mode!)` | （需查找） | 移动目录 | `src: String`<br/>`dest: String`<br/>`mode: Int32` | `Array<ConflictFiles>` | 同步 |

#### 流操作

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `createStream()` | `FileIo.createStream(path, mode)` | cj_file_fs.cj:1053 | 创建流对象 | `path: String`<br/>`mode: String` - "r", "w", "a", "r+", "w+", "a+" | `Stream` | 异步 |
| `fdopenStream()` | `FileIo.fdopenStream(fd, mode)` | cj_file_fs.cj:1121 | 从 FD 创建流 | `fd: Int32`<br/>`mode: String` | `Stream` | 异步 |

#### 其他操作

| API 名称 | 签名 | 行号 | 功能 | 参数 | 返回值 | 同步/异步 |
|----------|------|------|------|------|-----------|-----------|
| `createRandomAccessFile()` | `FileIo.createRandomAccessFile(path, mode!)` | （需查找） | 创建随机访问文件 | `path: String`<br/>`mode: Int64` | `RandomAccessFile` | 同步 |
| `readLines()` | `FileIo.readLines(path, options!)` | （需查找） | 逐行读取文件 | `path: String`<br/>`options: Options` | `ReaderIterator` | 异步 |
| `readText()` | `FileIo.readText(path, offset, hasLen, len, encoding)` | （需查找） | 读取文本 | `path: String`<br/>`offset: Int64`<br/>`hasLen: Bool`<br/>`len: Int64`<br/>`encoding: String` | `String` | 异步 |
| `lseek()` | `FileIo.lseek(fd, offset, whence)` | （需查找） | 移动文件指针 | `fd: Int32`<br/>`offset: Int64`<br/>`whence: WhenceType` | `Int64` | 同步 |
| `fsync()` | `FileIo.fsync(fd)` | （需查找） | 数据同步 | `fd: Int32` | `Int32` | 同步 |
| `fdatasync()` | `FileIo.fdatasync(fd)` | （需查找） | 数据同步 | `fd: Int32` | `Int32` | 同步 |
| `utimes()` | `FileIo.utimes(path, mtime)` | （需查找） | 更新时间戳 | `path: String`<br/>`mtime: Float64` | `Int32` | 同步 |
| `mkdtemp()` | `FileIo.mkdtemp(prefix)` | （需查找） | 创建临时目录 | `prefix: String` | `String` | 同步 |
| `symlink()` | `FileIo.symlink(target, srcPath)` | （需查找） | 创建符号链接 | `target: String`<br/>`srcPath: String` | `Int32` | 同步 |

---

## File 对象 API

### 属性

| 属性 | 类型 | 行号 | 功能 | 证据 |
|------|------|------|------|------|
| `fd` | `Int32` | file.cj:47 | 获取文件描述符 | `FfiOHOSFILEFsGetFD(getID())` |
| `path` | `String` | file.cj:60 | 获取文件路径 | `FfiOHOSFILEFsGetPath(getID())` |
| `name` | `String` | file.cj:76 | 获取文件名 | `FfiOHOSFILEFsGetName(getID())` |

### 方法

| 方法 | 签名 | 行号 | 功能 | 同步/异步 |
|------|------|------|------|-----------|
| `tryLock(exclusive!)` | `File.tryLock(exclusive!)` | file.cj:102 | 应用文件锁 | 同步 |
| `unlock()` | `File.unlock()` | file.cj:125 | 释放文件锁 | 同步 |
| `getParent()` | `File.getParent()` | file.cj:146 | 获取父目录 | 异步 |

---

## Stream 对象 API

### 方法

| 方法 | 签名 | 行号 | 功能 | 同步/异步 |
|------|------|------|------|-----------|
| `read(buffer, options!)` | `Stream.read(buffer, options!)` | stream.cj:239 | 从流读取 | 异步 |
| `write(buffer, options!)` | `Stream.write(buffer, options!)` | stream.cj:129 | 向流写入 | 异步 |
| `flush()` | `Stream.flush()` | stream.cj:90 | 刷新缓冲区 | 异步 |
| `close()` | `Stream.close()` | stream.cj:58 | 关闭流 | 异步 |

---

## Stat 对象 API

### 属性

| 属性 | 类型 | 行号 | 功能 | 证据 |
|------|------|------|------|------|
| `ino` | `Int64` | stat.cj:47 | inode 编号 | `FfiOHOSStatGetIno(getID())` |
| `mode` | `Int64` | stat.cj:72 | 权限模式（八进制） | `FfiOHOSStatGetMode(getID())` |
| `uid` | `Int64` | stat.cj:85 | 用户 ID | `FfiOHOSStatGetUid(getID())` |
| `gid` | `Int64` | stat.cj:98 | 组 ID | `FfiOHOSStatGetGid(getID())` |
| `size` | `Int64` | stat.cj:111 | 文件大小（字节） | `FfiOHOSStatGetSize(getID())` |
| `atime` | `Int64` | stat.cj:126 | 访问时间 | `FfiOHOSStatGetAtime(getID())` |
| `mtime` | `Int64` | stat.cj:140 | 修改时间 | `FfiOHOSStatGetMtime(getID())` |
| `ctime` | `Int64` | stat.cj:154 | 创建时间 | `FfiOHOSStatGetCtime(getID())` |

### 类型检查方法

| 方法 | 签名 | 行号 | 功能 | 返回值 |
|------|------|------|------|--------|
| `isBlockDevice()` | `Stat.isBlockDevice()` | stat.cj:170 | 是否为块设备 | `Bool` |
| `isCharacterDevice()` | `Stat.isCharacterDevice()` | stat.cj:189 | 是否为字符设备 | `Bool` |
| `isDirectory()` | `Stat.isDirectory()` | stat.cj:207 | 是否为目录 | `Bool` |
| `isFile()` | `Stat.isFile()` | stat.cj:243 | 是否为普通文件 | `Bool` |
| `isFifo()` | `Stat.isFifo()` | stat.cj:225 | 是否为 FIFO | `Bool` |
| `isSocket()` | `Stat.isSocket()` | stat.cj:261 | 是否为套接字 | `Bool` |
| `isSymbolicLink()` | `Stat.isSymbolicLink()` | stat.cj:279 | 是否为符号链接 | `Bool` |

---

## FileUri 对象 API

### 构造函数

| 构造函数 | 签名 | 行号 | 功能 |
|----------|------|------|------|
| `FileUri(uriOrPath)` | `FileUri(uriOrPath)` | file_uri.cj:103 | 从 URI 或路径构造 FileUri |

### 属性

| 属性 | 类型 | 行号 | 功能 | 证据 |
|------|------|------|------|------|
| `path` | `String` | file_uri.cj:132 | 获取路径 | `FfiOHOSFILEUriGetPath(getID())` |
| `name` | `String` | file_uri.cj:154 | 获取文件名 | `FfiOHOSFILEUriGetName(getID())` |

### 方法

| 方法 | 签名 | 行号 | 功能 | 同步/异步 |
|------|------|------|------|-----------|
| `toString()` | `FileUri.toString()` | file_uri.cj:174 | 序列化为 URI 字符串 | 异步 |

### 工具函数

| 函数 | 签名 | 行号 | 功能 | 异步 |
|------|------|------|------|-----------|
| `getUriFromPath(path)` | `getUriFromPath(path)` | file_uri.cj:196 | 从路径获取 URI | 异步 |

**示例**：
```cangjie
import kit.CoreFileKit.*

// 从路径获取 URI
let uri = getUriFromPath("/data/storage/el2/base/files/test.txt")
// uri: "file://com.example.app/data/storage/el2/base/files/test.txt"

// 构造 FileUri 对象
let fileUri = FileUri("/data/storage/el2/base/files/test.txt")
let path = fileUri.path  // 获取路径
let name = fileUri.name  // 获取文件名
```

---

## 配置类型 API

### OpenMode（文件打开模式）

**证据**：`ohos/file/fs/cj_file_fs.cj:88-180`

| 常量 | 值 | 功能说明 |
|------|-----|----------|
| `READ_ONLY` | `0o0` | 只读 |
| `WRITE_ONLY` | `0o1` | 只写 |
| `READ_WRITE` | `0o2` | 读写 |
| `CREATE` | `0o100` | 不存在则创建 |
| `TRUNC` | `0o1000` | 打开时截断 |
| `APPEND` | `0o2000` | 追加模式 |
| `NONBLOCK` | `0o4000` | 非阻塞 |
| `DIR` | `0o200000` | 目录 |
| `NOFOLLOW` | `0o400000` | 不跟随符号链接 |
| `SYNC` | `0o4010000` | 同步 I/O |

### AccessModeType（访问模式）

**证据**：`ohos/file/fs/cj_file_fs.cj:461-508`

| 常量 | 值 | 功能说明 |
|------|-----|----------|
| `Exist` | 0 | 检查文件存在 |
| `Write` | 2 | 检查写权限 |
| `Read` | 4 | 检查读权限 |
| `ReadWrite` | 6 | 检查读写权限 |

### WhenceType（偏移类型）

**证据**：`ohos/file/fs/cj_file_fs.cj:542-581`

| 常量 | 值 | 功能说明 |
|------|-----|----------|
| `SeekSet` | 0 | 从文件开始 |
| `SeekCur` | 1 | 从当前位置 |
| `SeekEnd` | 2 | 从文件末尾 |

---

## 关键结论

1. **统一入口**：通过 `kit.CoreFileKit` 导入使用所有文件管理 API
2. **静态类模式**：`FileIo` 提供静态方法作为工具类
3. **对象封装**：`File`, `Stream`, `Stat`, `FileUri` 封装资源句柄
4. **WorkerThread 优先**：I/O 操作标记为异步执行，避免阻塞主线程
5. **错误处理统一**：所有 API 通过 `BusinessException` 抛出错误

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览与核心能力
- [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) - FFI 函数详细说明
- [05_Internal_API.md](05_Internal_API.md) - 内部接口设计
