# File API 对外接口文档

本文档提供 File API 的完整接口参考，包括 N-API、C API、Native API 的详细说明。所有接口均提供参数、返回值、错误码和使用示例。

## 1. 接口概览

### 1.1 接口类型总览

File API 提供以下类型的编程接口：

| 接口类型 | 命名空间 | 主要用途 | 语言支持 |
|----------|----------|----------|----------|
| **N-API（JS）** | @ohos.fileio, @ohos.file.fs | 文件 I/O 操作 | JavaScript, TypeScript |
| **N-API（C）** | libfileio, libenvironment | C 语言接口 | C |
| **CJ FFI** | cj.fs | Cangjie 语言接口 | CJ |
| **Native** | remote_uri, hyperaio | 原生接口 | C++ |
| **Rust** | rust_file | Rust 接口 | Rust |

### 1.2 系统能力映射

| 接口模块 | 需要的系统能力 |
|----------|----------------|
| @ohos.fileio | SystemCapability.FileManagement.File.FileIO |
| @ohos.file.fs | SystemCapability.FileManagement.File.FileIO |
| @ohos.file.hash | SystemCapability.FileManagement.File.FileIO |
| @ohos.file.environment | SystemCapability.FileManagement.File.Environment |
| @ohos.file.statvfs | SystemCapability.FileManagement.File.FileIO |
| @ohos.file.securityLabel | SystemCapability.FileManagement.File.FileIO |
| @ohos.file.document | SystemCapability.FileManagement.File.FileIO |

---

## 2. JS API 模块

### 2.1 @ohos.fileio 模块

@ohos.fileio 是传统的文件 I/O 模块，提供基础的同步和异步文件操作。

**模块注册**：`interfaces/kits/js/src/mod_fileio/module.cpp`

**模块标识**：`fileio`

#### 2.1.1 文件操作

| JS API | C++ 入口 | 参数 | 返回 | 同步/异步 |
|--------|----------|------|------|-----------|
| openSync() | `OpenSync()` | path: string, flags: number | number (fd) | 同步 |
| open() | `Open()` | path: string, number | Promise\<number\> | Promise |
| closeSync() | `CloseSync()` | fd: number | void | 同步 |
| close() | `Close()` | fd: number | Promise\<void\> | Promise |
| readSync() | `ReadSync()` | fd: number, buffer: ArrayBuffer, offset: number, length: number, position: number | number | 同步 |
| read() | `Read()` | fd: number, buffer: ArrayBuffer, options: object | Promise\<ReadResult\> | Promise |
| writeSync() | `WriteSync()` | fd: number, buffer: ArrayBuffer, offset: number, length: number, position: number | number | 同步 |
| write() | `Write()` | fd: number, buffer: ArrayBuffer, options: object | Promise\<WriteResult\> | Promise |

**openSync 使用示例**：

```javascript
import fileio from '@ohos.fileio';

try {
  // 以读写模式打开文件，不存在则创建
  let fd = fileio.openSync('/data/storage/el2/base/test.txt', 0o2 | 0o100);
  console.log('File descriptor: ' + fd);
  
  // 写入数据
  let writeLen = fileio.writeSync(fd, 'Hello, OpenHarmony!');
  console.log('Written bytes: ' + writeLen);
  
  // 读取数据
  let buffer = new ArrayBuffer(1024);
  let readLen = fileio.readSync(fd, buffer, 0, 1024, 0);
  console.log('Read bytes: ' + readLen);
  
  // 关闭文件
  fileio.closeSync(fd);
} catch (error) {
  console.error('Error: ' + error.message);
}
```

**open 使用示例**：

```javascript
import fileio from '@ohos.fileio';

try {
  // 异步打开文件
  fileio.open('/data/storage/el2/base/test.txt', 0o2 | 0o100)
    .then((fd) => {
      console.log('File descriptor: ' + fd);
      return fileio.read(fd, new ArrayBuffer(1024), { offset: 0, length: 1024, position: 0 });
    })
    .then((readResult) => {
      console.log('Read bytes: ' + readResult.bytesRead);
      console.log('Data: ' + String.fromCharCode(...new Uint8Array(readResult.buffer)));
    })
    .catch((error) => {
      console.error('Error: ' + error.message);
    });
} catch (error) {
  console.error('Sync error: ' + error.message);
}
```

#### 2.1.2 文件属性操作

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| statSync() | 同步获取文件状态 | path: string | Stat |
| stat() | 异步获取文件状态 | path: string | Promise\<Stat\> |
| fstatSync() | 同步获取文件描述符状态 | fd: number | Stat |
| fstat() | 异步获取文件描述符状态 | fd: number | Promise\<Stat\> |
| accessSync() | 同步检查文件访问权限 | path: string, mode: number | boolean |
| access() | 异步检查文件访问权限 | path: string, mode: number | Promise\<boolean\> |
| chmodSync() | 同步修改文件权限 | path: string, mode: number | void |
| chmod() | 异步修改文件权限 | path: string, mode: number | Promise\<void\> |
| chownSync() | 同步修改文件所有者 | path: string, uid: number, gid: number | void |
| chown() | 异步修改文件所有者 | path: string, uid: number, gid: number | Promise\<void\> |

#### 2.1.3 目录操作

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| mkdirSync() | 同步创建目录 | path: string, mode: number | void |
| mkdir() | 异步创建目录 | path: string, mode: number | Promise\<void\> |
| rmdirSync() | 同步删除目录 | path: string | void |
| rmdir() | 异步删除目录 | path: string | Promise\<void\> |
| openDirSync() | 同步打开目录 | path: string | Dir |
| openDir() | 异步打开目录 | path: string | Promise\<Dir\> |
| readDirSync() | 同步读取目录项 | dir: Dir | Dirent[] |
| readDir() | 异步读取目录项 | dir: Dir | Promise\<Dirent[]\> |

#### 2.1.4 流操作

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| createStreamSync() | 同步创建流 | path: string, mode: string | Stream |
| createStream() | 异步创建流 | path: string, mode: string | Promise\<Stream\> |
| fdopenStreamSync() | 同步从文件描述符创建流 | fd: number, mode: string | Stream |
| fdopenStream() | 异步从文件描述符创建流 | fd: number, mode: string | Promise\<Stream\> |
| Stream.readSync() | 同步读取流数据 | buffer: ArrayBuffer, length: number | number |
| Stream.read() | 异步读取流数据 | buffer: ArrayBuffer, options: object | Promise\<ReadResult\> |
| Stream.writeSync() | 同步写入流数据 | buffer: ArrayBuffer, length: number | number |
| Stream.write() | 异步写入流数据 | buffer: ArrayBuffer, options: object | Promise\<WriteResult\> |
| Stream.closeSync() | 同步关闭流 | void | void |
| Stream.close() | 异步关闭流 | void | Promise\<void\> |

#### 2.1.5 其他操作

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| renameSync() | 同步重命名文件 | oldPath: string, newPath: string | void |
| rename() | 异步重命名文件 | oldPath: string, newPath: string | Promise\<void\> |
| unlinkSync() | 同步删除文件 | path: string | void |
| unlink() | 异步删除文件 | path: string | Promise\<void\> |
| symlinkSync() | 同步创建符号链接 | target: string, srcPath: string | void |
| symlink() | 异步创建符号链接 | target: string, srcPath: string | Promise\<void\> |
| linkSync() | 同步创建硬链接 | target: string, srcPath: string | void |
| link() | 异步创建硬链接 | target: string, srcPath: string | Promise\<void\> |
| truncateSync() | 同步截断文件 | path: string, len: number | void |
| truncate() | 异步截断文件 | path: string, len: number | Promise\<void\> |
| fsyncSync() | 同步同步文件数据 | fd: number | void |
| fsync() | 异步同步文件数据 | fd: number | Promise\<void\> |
| fdatasyncSync() | 同步同步文件数据（不含元数据） | fd: number | void |
| fdatasync() | 异步同步文件数据（不含元数据） | fd: number | Promise\<void\> |

### 2.2 @ohos.file.fs 模块

@ohos.file.fs 是基于 URI 的文件系统 API，提供更安全的沙箱文件访问。

**模块注册**：`interfaces/kits/js/src/mod_fs/module.cpp`

**模块标识**：`fs`

#### 2.2.1 文件打开与关闭

| JS API | 功能描述 | 参数 | 返回 | 同步/异步 |
|--------|----------|------|------|-----------|
| openSync() | 同步打开文件 | uri: string, mode: number | File | 同步 |
| open() | 异步打开文件 | uri: string, mode: number | Promise\<File\> | Promise |
| closeSync() | 同步关闭文件 | fd: number | void | 同步 |
| close() | 异步关闭文件 | fd: number | Promise\<void\> | Promise |

**openSync 使用示例**：

```javascript
import fs from '@ohos.file.fs';

try {
  // 打开沙箱内文件（内部存储）
  let file = fs.openSync('internal://app/test.txt', fs.OpenMode.READ | fs.OpenMode.CREAT);
  
  console.log('File fd: ' + file.fd);
  console.log('File name: ' + file.name);
  
  // 写入数据
  let writeLen = fs.writeSync(file.fd, 'Hello, OpenHarmony!');
  console.log('Written bytes: ' + writeLen);
  
  // 关闭文件
  fs.closeSync(file.fd);
} catch (error) {
  console.error('Error: ' + error.message);
}
```

#### 2.2.2 文件读写

| JS API | 功能描述 | 参数 | 返回 | 同步/异步 |
|--------|----------|------|------|-----------|
| readSync() | 同步读取 | fd: number, buffer: ArrayBuffer, offset: number, length: number, position: number | number | 同步 |
| read() | 异步读取 | fd: number, buffer: ArrayBuffer, options: object | Promise\<ReadResult\> | Promise |
| writeSync() | 同步写入 | fd: number, buffer: ArrayBuffer, offset: number, length: number, position: number | number | 同步 |
| write() | 异步写入 | fd: number, buffer: ArrayBuffer, options: object | Promise\<WriteResult\> | Promise |

#### 2.2.3 文件属性

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| statSync() | 同步获取文件状态 | uri: string | Stat |
| stat() | 异步获取文件状态 | uri: string | Promise\<Stat\> |
| fstatSync() | 同步获取文件描述符状态 | fd: number | Stat |
| fstat() | 异步获取文件描述符状态 | fd: number | Promise\<Stat\> |

#### 2.2.4 目录操作

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| mkdirSync() | 同步创建目录 | uri: string, mode: number | void |
| mkdir() | 异步创建目录 | uri: string, mode: number | Promise\<void\> |
| rmdirSync() | 同步删除目录 | uri: string | void |
| rmdir() | 异步删除目录 | uri: string | Promise\<void\> |
| listFileSync() | 同步列出文件 | uri: string, options: object | File[] |
| listFile() | 异步列出文件 | uri: string, options: object | Promise\<File[]\> |

#### 2.2.5 文件操作

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| renameSync() | 同步重命名 | oldUri: string, newUri: string | void |
| rename() | 异步重命名 | oldUri: string, newUri: string | Promise\<void\> |
| unlinkSync() | 同步删除文件 | uri: string | void |
| unlink() | 异步删除文件 | uri: string | Promise\<void\> |
| copySync() | 同步复制文件 | srcUri: string, destUri: string | void |
| copy() | 异步复制文件 | srcUri: string, destUri: string | Promise\<void\> |
| moveSync() | 同步移动文件 | srcUri: string, destUri: string | void |
| move() | 异步移动文件 | srcUri: string, destUri: string | Promise\<void\> |
| truncateSync() | 同步截断文件 | uri: string, len: number | void |
| truncate() | 异步截断文件 | uri: string, len: number | Promise\<void\> |

#### 2.2.6 权限常量

**证据来源**：`interfaces/kits/js/src/mod_fs/class_file/fs_file.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/class_file/fs_file.cpp
// 权限常量定义
class File {
public:
    static constexpr int OpenMode = {
        READ = 0o0,     // 只读
        WRITE = 0o1,     // 只写
        READ_WRITE = 0o2, // 读写
        CREAT = 0o100,   // 创建
        TRUNC = 0o1000,  // 截断
        APPEND = 0o2000, // 追加
    };
};
```

| 常量名 | 值 | 说明 |
|--------|-----|------|
| OpenMode.READ | 0o0 | 只读模式 |
| OpenMode.WRITE | 0o1 | 只写模式 |
| OpenMode.READ_WRITE | 0o2 | 读写模式 |
| OpenMode.CREAT | 0o100 | 文件不存在则创建 |
| OpenMode.TRUNC | 0o1000 | 截断文件（清空） |
| OpenMode.APPEND | 0o2000 | 追加模式 |

### 2.3 @ohos.file.hash 模块

计算文件的哈希值。

**模块注册**：`interfaces/kits/js/src/mod_hash/module.cpp`

**模块标识**：`hash`

| JS API | 功能描述 | 参数 | 返回 | 同步/异步 |
|--------|----------|------|------|-----------|
| hashSync() | 同步计算哈希 | uri: string, algorithm: string | string | 同步 |
| hash() | 异步计算哈希 | uri: string, algorithm: string | Promise\<string\> | Promise |
| streamHashSync() | 同步流哈希 | stream: Stream, algorithm: string | string | 同步 |
| streamHash() | 异步流哈希 | stream: Stream, algorithm: string | Promise\<string\> | Promise |

**使用示例**：

```javascript
import hash from '@ohos.file.hash';

try {
  // 计算 MD5 哈希
  let md5 = hash.hashSync('internal://app/test.txt', 'md5');
  console.log('MD5: ' + md5);
  
  // 计算 SHA256 哈希
  let sha256 = hash.hashSync('internal://app/test.txt', 'sha256');
  console.log('SHA256: ' + sha256);
} catch (error) {
  console.error('Error: ' + error.message);
}
```

### 2.4 @ohos.file.environment 模块

获取环境路径信息。

**模块注册**：`interfaces/kits/js/src/mod_environment/environment_napi.cpp`

**模块标识**：`environment`

| JS API | 功能描述 | 返回 |
|--------|----------|------|
| getStorageDir() | 获取应用存储目录路径 | string |
| getCacheDir() | 获取应用缓存目录路径 | string |
| getDataDir() | 获取应用数据目录路径 | string |
| getDistributedDir() | 获取分布式文件目录路径 | string |
| getExternalStorageDir() | 获取外部存储目录路径（如果支持） | string |

**使用示例**：

```javascript
import environment from '@ohos.file.environment';

console.log('Storage dir: ' + environment.getStorageDir());
console.log('Cache dir: ' + environment.getCacheDir());
console.log('Data dir: ' + environment.getDataDir());
console.log('Distributed dir: ' + environment.getDistributedDir());
```

### 2.5 @ohos.file.statvfs 模块

获取文件系统统计信息。

**模块注册**：`interfaces/kits/js/src/mod_statvfs/statvfs_napi.cpp`

**模块标识**：`statvfs`

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| statvfsSync() | 同步获取文件系统状态 | path: string | Statvfs |
| statvfs() | 异步获取文件系统状态 | path: string | Promise\<Statvfs\> |

**Statvfs 属性**：

| 属性 | 类型 | 说明 |
|------|------|------|
| f_bsize | number | 文件系统块大小 |
| f_frsize | number | 片段大小 |
| f_blocks | number | 文件系统总块数 |
| f_bfree | number | 可用块数 |
| f_bavail | number | 非特权用户可用块数 |
| f_files | number | 总索引节点数 |
| f_ffree | number | 空闲索引节点数 |
| f_favail | number | 非特权用户可用索引节点数 |
| f_flag | number | 挂载标志 |
| f_namemax | number | 最大文件名长度 |

### 2.6 @ohos.file.securityLabel 模块

管理文件安全标签。

**模块注册**：`interfaces/kits/js/src/mod_securitylabel/securitylabel_napi.cpp`

**模块标识**：`securitylabel`

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| setSecurityLabelSync() | 同步设置安全标签 | uri: string, label: string | void |
| setSecurityLabel() | 异步设置安全标签 | uri: string, label: string | Promise\<void\> |
| getSecurityLabelSync() | 同步获取安全标签 | uri: string | string |
| getSecurityLabel() | 异步获取安全标签 | uri: string | Promise\<string\> |
| changeModeSync() | 同步修改安全模式 | uri: string, mode: number | void |
| changeMode() | 异步修改安全模式 | uri: string, mode: number | Promise\<void\> |

### 2.7 @ohos.file.document 模块

管理文档 URI。

**模块注册**：`interfaces/kits/js/src/mod_document/document_napi.cpp`

**模块标识**：`document`

| JS API | 功能描述 | 参数 | 返回 |
|--------|----------|------|------|
| createDocumentSync() | 同步创建文档 | uri: string | boolean |
| createDocument() | 异步创建文档 | uri: string | Promise\<boolean\> |
| deleteDocumentSync() | 同步删除文档 | uri: string | boolean |
| deleteDocument() | 异步删除文档 | uri: string | Promise\<boolean\> |
| copyDocumentSync() | 同步复制文档 | srcUri: string, destUri: string | boolean |
| copyDocument() | 异步复制文档 | srcUri: string, destUri: string | Promise\<boolean\> |
| moveDocumentSync() | 同步移动文档 | srcUri: string, destUri: string | boolean |
| moveDocument() | 异步移动文档 | srcUri: string, destUri: string | Promise\<boolean\> |

---

## 3. 编程模型

### 3.1 同步模型

同步 API 会阻塞调用线程直到操作完成。

**适用场景**：
- 简单的文件操作
- 小文件读写
- 启动/配置阶段

**示例**：

```javascript
import fs from '@ohos.file.fs';

try {
  // 同步打开
  let file = fs.openSync('internal://app/config.json', fs.OpenMode.READ);
  
  // 同步读取
  let buffer = new ArrayBuffer(1024);
  let readLen = fs.readSync(file.fd, buffer, 0, 1024, 0);
  
  // 同步关闭
  fs.closeSync(file.fd);
  
  console.log('Read ' + readLen + ' bytes');
} catch (error) {
  console.error('Error: ' + error.message);
}
```

### 3.2 Promise 异步模型

Promise API 返回 Promise 对象，适用于现代 JavaScript 开发。

**适用场景**：
- UI 交互
- 复杂业务逻辑
- 需要链式调用

**示例**：

```javascript
import fs from '@ohos.file.fs';

try {
  fs.open('internal://app/large_file.txt', fs.OpenMode.READ)
    .then((file) => {
      let buffer = new ArrayBuffer(4096);
      return fs.read(file.fd, buffer, { offset: 0, length: 4096, position: 0 })
        .then((result) => {
          console.log('Read ' + result.bytesRead + ' bytes');
          return file;
        });
    })
    .then((file) => {
      return fs.close(file.fd);
    })
    .then(() => {
      console.log('Done');
    })
    .catch((error) => {
      console.error('Error: ' + error.message);
    });
} catch (error) {
  console.error('Sync error: ' + error.message);
}

// 使用 async/await
async function readFile() {
  try {
    let file = await fs.open('internal://app/data.txt', fs.OpenMode.READ);
    let buffer = new ArrayBuffer(1024);
    let result = await fs.read(file.fd, buffer, { offset: 0, length: 1024, position: 0 });
    await fs.close(file.fd);
    return result;
  } catch (error) {
    console.error('Error: ' + error.message);
  }
}
```

### 3.3 Callback 异步模型

Callback API 接受最后一个参数为回调函数。

**适用场景**：
- 传统回调风格代码
- 需要兼容旧代码

**示例**：

```javascript
import fs from '@ohos.file.fs';

try {
  fs.open('internal://app/data.txt', fs.OpenMode.READ, (err, file) => {
    if (err) {
      console.error('Open error: ' + err.message);
      return;
    }
    
    let buffer = new ArrayBuffer(1024);
    fs.read(file.fd, buffer, { offset: 0, length: 1024, position: 0 }, (err, result) => {
      if (err) {
        console.error('Read error: ' + err.message);
        fs.close(file.fd, (err) => {
          console.error('Close error: ' + err.message);
        });
        return;
      }
      
      console.log('Read ' + result.bytesRead + ' bytes');
      
      fs.close(file.fd, (err) => {
        if (err) {
          console.error('Close error: ' + err.message);
        } else {
          console.log('Done');
        }
      });
    });
  });
} catch (error) {
  console.error('Sync error: ' + error.message);
}
```

---

## 4. 错误码参考

### 4.1 通用错误码

**证据来源**：`interfaces/kits/js/src/common/uni_error.h`

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 13900001 | EPERM | 操作不允许 |
| 13900002 | ENOENT | 文件或目录不存在 |
| 13900003 | ESRCH | 进程不存在 |
| 13900004 | EINTR | 系统调用被中断 |
| 13900005 | EIO | I/O 错误 |
| 13900006 | ENXIO | 设备或地址不存在 |
| 13900007 | E2BIG | 参数列表过长 |
| 13900008 | ENOEXEC | 执行格式错误 |
| 13900009 | EBADF | 文件描述符错误 |
| 13900010 | ECHILD | 子进程不存在 |
| 13900011 | EAGAIN | 资源暂时不可用 |
| 13900012 | ENOMEM | 内存不足 |
| 13900013 | EACCES | 权限不足 |
| 13900014 | EFAULT | 地址错误 |
| 13900015 | ENOTBLK | 不是块设备 |
| 13900016 | EBUSY | 设备或资源忙 |
| 13900017 | EEXIST | 文件已存在 |
| 13900018 | EXDEV | 跨设备链接 |
| 13900019 | ENODEV | 设备不存在 |
| 13900020 | ENOTDIR | 不是目录 |
| 13900021 | EISDIR | 是目录 |
| 13900022 | EINVAL | 参数无效 |
| 13900023 | ENFILE | 文件表溢出 |
| 13900024 | EMFILE | 打开文件过多 |
| 13900025 | ENOTTY | 不是终端设备 |
| 13900026 | ETXTBSY | 文本文件忙 |
| 13900027 | EFBIG | 文件过大 |
| 13900028 | ENOSPC | 设备无空间 |
| 13900029 | ESPIPE | 无效的seek |
| 13900030 | EROFS | 只读文件系统 |
| 13900031 | EMLINK | 链接过多 |
| 13900032 | EPIPE | 管道破裂 |
| 13900033 | EDOM | 数学参数超出范围 |
| 13900034 | ERANGE | 结果超出范围 |
| 13900035 | EDEADLK | 资源死锁 |
| 13900036 | ENAMETOOLONG | 文件名过长 |
| 13900037 | ENOLCK | 无可用锁 |
| 13900038 | ENOTEMPTY | 目录非空 |
| 13900039 | ELOOP | 符号链接过多 |
| 13900040 | EONE | 未知错误 |

### 4.2 File API 特定错误

| 错误码 | 常量名 | 说明 | 处理建议 |
|--------|--------|------|----------|
| 13900041 | EFILE_EXISTS | 文件已存在（创建模式） | 检查文件是否存在 |
| 13900042 | E_DIR_NOT_EMPTY | 目录非空 | 删除目录内容后重试 |
| 13900043 | E_INVALID_URI | 无效的 URI | 检查 URI 格式 |
| 13900044 | E_OUT_OF_SANDBOX | 超出沙箱范围 | 使用沙箱内 URI |
| 13900045 | E_PERMISSION_DENIED | 权限被拒绝 | 检查权限配置 |
| 13900046 | E_TOO_MANY_LINKS | 链接过多 | 清理链接后重试 |

---

## 5. C API 参考

### 5.1 文件操作 C API

**头文件**：`interfaces/kits/c/fileio/fileio.h`

| C 函数 | 功能描述 | 参数 | 返回值 |
|--------|----------|------|--------|
| OH_FileIO_Open | 打开文件 | const char* path, int flags | int (fd) 或负值错误 |
| OH_FileIO_Close | 关闭文件 | int fd | int (0 成功) |
| OH_FileIO_Read | 读取数据 | int fd, void* buf, size_t len | ssize_t |
| OH_FileIO_Write | 写入数据 | int fd, const void* buf, size_t len | ssize_t |
| OH_FileIO_Stat | 获取文件状态 | const char* path, struct stat* st | int (0 成功) |
| OH_FileIO_Unlink | 删除文件 | const char* path | int (0 成功) |
| OH_FileIO_Mkdir | 创建目录 | const char* path, mode_t mode | int (0 成功) |
| OH_FileIO_Rmdir | 删除目录 | const char* path | int (0 成功) |

### 5.2 环境 C API

**头文件**：`interfaces/kits/c/environment/environment.h`

| C 函数 | 功能描述 | 参数 | 返回值 |
|--------|----------|------|--------|
| OH_Environment_GetStorageDir | 获取存储目录 | char* buffer, size_t size | int (0 成功) |
| OH_Environment_GetCacheDir | 获取缓存目录 | char* buffer, size_t size | int (0 成功) |
| OH_Environment_GetDataDir | 获取数据目录 | char* buffer, size_t size | int (0 成功) |

---

## 6. Native API 参考

### 6.1 Remote URI API

**头文件**：`interfaces/kits/native/remote_uri/remote_uri.h`

| Native 函数 | 功能描述 | 参数 |
|-------------|----------|------|
| OH_RemoteUri_Parse | 解析 URI | const char* uri, UriInfo* info |
| OH_RemoteUri_ConvertToPath | URI 转路径 | const char* uri, char* path, size_t size |
| OH_RemoteUri_Verify | 验证 URI | const char* uri, bool* valid |

### 6.2 HyperAIO API

**头文件**：`interfaces/kits/hyperaio/hyperaio.h`

| Native 函数 | 功能描述 | 参数 |
|-------------|----------|------|
| OH_HyperAio_Init | 初始化异步 I/O | HyperAioCtx* ctx |
| OH_HyperAio_Read | 异步读取 | HyperAioCtx* ctx, int fd, void* buf, size_t len, off_t offset |
| OH_HyperAio_Write | 异步写入 | HyperAioCtx* ctx, int fd, const void* buf, size_t len, off_t offset |
| OH_HyperAio_Wait | 等待完成 | HyperAioCtx* ctx, HyperAioResult* result |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 |

---

**最后更新**：2026-02-07

**版本**：1.0