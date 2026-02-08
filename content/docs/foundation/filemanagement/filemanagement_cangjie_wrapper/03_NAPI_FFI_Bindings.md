# FFI/N-API 绑定文档

> 目的：记录 FFI 外部函数绑定、外部依赖接口、参数校验、错误处理
> 适用范围：集成开发者理解 FFI 接口、底层开发调试、性能优化
> 最后更新：2026-02-06

## 重要说明

**本仓库不包含直接的 N-API 代码**。这是一个**仓颉（Cangjie）语言封装层**，通过 FFI 调用外部 C/C++ 实现。

**实际的 N-API 绑定位于外部仓库**：
- `arkcompiler_cangjie_ark_interop` - 仓颉到 N-API 桥接层
- `filemanagement_file_api` - `cj_file_fs_ffi` 原生实现
- `filemanagement_app_file_service` - `cj_file_fileuri_ffi` 原生实现

---

## FFI 函数分类

### 1. 文件操作（File Operations）

**文件位置**：`ohos/file/fs/native.cj:99-166`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsOpen(path, mode)` | 100 | `RetDataI64` | 打开文件，返回 instanceId |
| `FfiOHOSFileFsDup(fd)` | 102 | `RetDataI64` | 复制文件描述符 |
| `FfiOHOSFILEFsGetFD(id)` | 104 | `Int32` | 获取文件描述符 |
| `FfiOHOSFILEFsGetPath(id)` | 106 | `CString` | 获取文件路径 |
| `FfiOHOSFILEFsGetName(id)` | 108 | `CString` | 获取文件名 |
| `FfiOHOSFILEFsTryLock(id, exclusive)` | 110 | `RetCode` | 应用文件锁 |
| `FfiOHOSFILEFsUnLock(id)` | 112 | `RetCode` | 释放文件锁 |
| `FfiOHOSFILEFsGetParent(id)` | 114 | `RetDataCString` | 获取父目录 |
| `FfiOHOSFileFsClose(file)` | 166 | `Int32` | 关闭文件 |

**Cangjie 封装**：`ohos/file/fs/file.cj:31-162`

---

### 2. 目录操作（Directory Operations）

**文件位置**：`ohos/file/fs/native.cj:118-125`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsMkdir(path, recursion, isTwoArgs)` | 118 | `Int32` | 创建目录 |
| `FfiOHOSFileFsRmdir(path)` | 120 | `Int32` | 删除目录 |
| `FfiOHOSFileFsMoveDir(src, dest, mode)` | 122 | `RetDataCArrConflictFiles` | 移动目录（冲突检测） |
| `FfiOHOSFileFsRename(oldFile, newFile)` | 124 | `Int32` | 重命名文件/目录 |
| `FfiOHOSFileFsUnlink(path)` | 126 | `Int32` | 删除文件 |
| `FfiOHOSFileFsListFile(path, options)` | 152 | `RetDataCArrStringN` | 列出文件（支持过滤） |

**Cangjie 封装**：`ohos/file/fs/cj_file_fs.cj`

---

### 3. 文件读写（File I/O）

**文件位置**：`ohos/file/fs/native.cj:130-137`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsRead(fd, buffer, size, length, offset)` | 130 | `RetDataI64` | 从文件读取数据 |
| `FfiOHOSFileFsReadCur(fd, buffer, size, length)` | 132 | `RetDataI64` | 从当前位置读取 |
| `FfiOHOSFileFsWrite(fd, buffer, length, offset, encoding)` | 134 | `RetDataI64` | 向文件写入数据 |
| `FfiOHOSFileFsWriteCur(fd, buffer, length, encoding)` | 136 | `RetDataI64` | 从当前位置写入 |

**Cangjie 封装**：`ohos/file/fs/cj_file_fs.cj:1339-1426`

---

### 4. 复制移动操作（Copy/Move Operations）

**文件位置**：`ohos/file/fs/native.cj:138-146`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsCopyDir(src, dest, mode)` | 138 | `RetDataCArrConflictFiles` | 复制目录 |
| `FfiOHOSFileFsCopyFile(src, dest, mode)` | 140 | `Int32` | 复制文件 |
| `FfiOHOSFileFsCopyFileSI(src, dest, mode)` | 142 | `Int32` | 从字符串复制到 FD |
| `FfiOHOSFileFsCopyFileIS(src, dest, mode)` | 144 | `Int32` | 从 FD 复制到字符串 |
| `FfiOHOSFileFsCopyFileII(src, dest, mode)` | 146 | `Int32` | 从 FD 复制到 FD |
| `FfiOHOSFileFsMoveFile(src, dest, mode)` | 148 | `Int32` | 移动文件 |

---

### 5. 文件元信息（File Metadata / Stat）

**文件位置**：`ohos/file/fs/native.cj:31-63`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsStatByID(file)` | 31 | `RetDataI64` | 通过 FD 获取 stat |
| `FfiOHOSFileFsStatByString(file)` | 33 | `RetDataI64` | 通过路径获取 stat |
| `FfiOHOSStatGetIno(id)` | 35 | `Int64` | 获取 inode |
| `FfiOHOSStatGetMode(id)` | 37 | `Int64` | 获取权限模式 |
| `FfiOHOSStatGetUid(id)` | 39 | `Int64` | 获取用户 ID |
| `FfiOHOSStatGetGid(id)` | 41 | `Int64` | 获取组 ID |
| `FfiOHOSStatGetSize(id)` | 43 | `Int64` | 获取文件大小 |
| `FfiOHOSStatGetAtime(id)` | 45 | `Int64` | 获取访问时间 |
| `FfiOHOSStatGetMtime(id)` | 47 | `Int64` | 获取修改时间 |
| `FfiOHOSStatGetCtime(id)` | 49 | `Int64` | 获取创建时间 |
| `FfiOHOSStatIsBlockDeviceV2(id)` | 51 | `RetDataBool` | 是否为块设备 |
| `FfiOHOSStatIsCharacterDeviceV2(id)` | 53 | `RetDataBool` | 是否为字符设备 |
| `FfiOHOSStatIsDirectoryV2(id)` | 55 | `RetDataBool` | 是否为目录 |
| `FfiOHOSStatIsFIFOV2(id)` | 57 | `RetDataBool` | 是否为 FIFO |
| `FfiOHOSStatIsFileV2(id)` | 59 | `RetDataBool` | 是否为文件 |
| `FfiOHOSStatIsSocketV2(id)` | 61 | `RetDataBool` | 是否为套接字 |
| `FfiOHOSStatIsSymbolicLinkV2(id)` | 63 | `RetDataBool` | 是否为符号链接 |

**Cangjie 封装**：`ohos/file/fs/stat.cj:31-287`

---

### 6. 流操作（Stream Operations）

**文件位置**：`ohos/file/fs/native.cj:66-80`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsCreateStream(path, mode)` | 66 | `RetDataI64` | 创建流对象 |
| `FfiOHOSFileFsFdopenStream(fd, mode)` | 68 | `RetDataI64` | 从 FD 创建流 |
| `FfiOHOSStreamClose(id)` | 70 | `RetCode` | 关闭流 |
| `FfiOHOSStreamFlush(id)` | 72 | `RetCode` | 刷新流 |
| `FfiOHOSStreamWriteCur(id, string, length, encoding)` | 74 | `RetDataI64` | 流写入（当前位置） |
| `FfiOHOSStreamWrite(id, string, length, offset, encoding)` | 76 | `RetDataI64` | 流写入（指定偏移） |
| `FfiOHOSStreamReadCur(id, buffer, size, length)` | 78 | `RetDataI64` | 流读取（当前位置） |
| `FfiOHOSStreamRead(id, buffer, size, length, offset)` | 80 | `RetDataI64` | 流读取（指定偏移） |

**Cangjie 封装**：`ohos/file/fs/stream.cj:33-264`

---

### 7. 随机访问文件（RandomAccessFile）

**文件位置**：`ohos/file/fs/native.cj:83-97`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsCreateRandomAccessFileByString(file, mode)` | 83 | `RetDataI64` | 从路径创建 |
| `FfiOHOSFileFsCreateRandomAccessFileByID(file, mode)` | 85 | `RetDataI64` | 从 FD 创建 |
| `FfiOHOSRandomAccessFileGetFd(id)` | 87 | `Int32` | 获取 FD |
| `FfiOHOSRandomAccessFileGetFPointer(id)` | 89 | `Int64` | 获取文件指针 |
| `FfiOHOSRandomAccessFileSetFilePointerSync(id, fd)` | 91 | `Unit` | 设置文件指针 |
| `FfiOHOSRandomAccessFileClose(id)` | 93 | `Unit` | 关闭对象 |
| `FfiOHOSRandomAccessFileWrite(id, buffer, length, offset)` | 95 | `RetDataI64` | 随机写入 |
| `FfiOHOSRandomAccessFileRead(id, buffer, length, offset)` | 97 | `RetDataI64` | 随机读取 |

**Cangjie 封装**：TODO（未完整阅读 `random_access_file.cj`）

---

### 8. 杂项操作（Miscellaneous Operations）

**文件位置**：`ohos/file/fs/native.cj:116-121, 154-178`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFileFsLstat(path)` | 116 | `RetDataI64` | 获取符号链接 stat |
| `FfiOHOSFileFsMkdtemp(prefix)` | 150 | `RetDataCString` | 创建临时目录 |
| `FfiOHOSFileFsLseek(fd, offset, whence)` | 154 | `RetDataI64` | 移动文件指针 |
| `FfiOHOSFileFsFdatasync(fd)` | 156 | `Int32` | 数据同步 |
| `FfiOHOSFileFsFsync(fd)` | 158 | `Int32` | 完整同步 |
| `FfiOHOSFileFsSymlink(target, srcPath)` | 160 | `Int32` | 创建符号链接 |
| `FfiOHOSFileFsTruncateByString(file, len)` | 162 | `Int32` | 截断文件（路径） |
| `FfiOHOSFileFsTruncateByFd(file, len)` | 164 | `Int32` | 截断文件（FD） |
| `FfiOHOSFileFsAccessExt(path, mode, flag)` | 128 | `RetDataBool` | 检查访问权限 |
| `FfiOHOSFileFsReadLines(path, encoding)` | 170 | `RetDataI64` | 逐行读取 |
| `FfiOHOSFileFsReaderIteratorNext(id)` | 172 | `RetReaderIteratorResult` | 迭代器下一行 |
| `FfiOHOSFileFsReadText(path, offset, hasLen, len, encoding)` | 174 | `RetDataCString` | 读取文本 |
| `FfiOHOSFileFsUtimes(path, mtime)` | 176 | `Int32` | 更新时间戳 |
| `FfiOHOSFileFsReleaseCString(str)` | 178 | `Unit` | 释放 C 字符串 |

---

### 9. File URI 操作（File URI Operations）

**文件位置**：`ohos/file/fileuri/file_uri.cj:25-33`

| FFI 函数 | 行号 | 参数 | 返回值 | 功能说明 |
|----------|------|------|--------|----------|
| `FfiOHOSFILEUriCreateUri(uriOrPath)` | 25 | `Int64` | 创建 FileUri 对象 |
| `FfiOHOSFILEUriGetPath(id)` | 27 | `CString` | 获取路径 |
| `FfiOHOSFILEUriGetName(id)` | 29 | `CString` | 获取文件名 |
| `FfiOHOSFILEUriToString(id)` | 31 | `CString` | 序列化为字符串 |
| `FfiOHOSFILEUriGetUriFromPath(path)` | 33 | `CString` | 路径转 URI |

**Cangjie 封装**：`ohos/file/fileuri/file_uri.cj:86-210`

---

## 参数校验与类型转换

### Cangjie → C 类型映射

| Cangjie 类型 | C 类型 | FFI 返回类型 | 说明 |
|-------------|---------|-------------|------|
| `String` | `CString` | - | 通过 `LibC.mallocCString()` 转换 |
| `Array<Byte>` | `CPointer<Byte>` | - | 通过 `acquireArrayRawData()` 转换 |
| `Int64` | `Int64` | - | 直接传递 |
| `Int32` | `Int32` | - | 直接传递 |
| `Bool` | `Bool` | - | 直接传递 |

### 返回值封装

| FFI 返回类型 | Cangjie 封装 | 说明 |
|-------------|-------------|------|
| `RetDataI64` | `match { case SUCCESS_CODE => return Stat(id); _ => throw BusinessException() }` | 检查 code 后返回数据或抛异常 |
| `RetDataCString` | `let ret = cStr.toString(); LibC.free(cStr); return ret` | 转换为 String 并释放 C 字符串 |
| `RetDataBool` | `return ret.data` | 直接返回 Bool 值 |
| `RetCode` | `if (code != SUCCESS_CODE) { throw BusinessException() }` | 仅检查 code，成功则返回 Unit |

**证据**：`ohos/file/fs/cj_file_fs.cj:955-1001`

---

## 同步/异步模式

### WorkerThread 标记

**定义**：`@!APILevel` 注解中的 `workerthread: true` 标记

**作用**：API 在后台线程执行，避免阻塞主线程

**标记的 API**（部分清单）：
- ✅ `FileIo.stat()` - cj_file_fs.cj:955
- ✅ `FileIo.createStream()` - cj_file_fs.cj:1053
- ✅ `FileIo.open()` - cj_file_fs.cj:1270
- ✅ `FileIo.read()` - cj_file_fs.cj:1339
- ✅ `FileIo.write()` - cj_file_fs.cj:1396
- ✅ `Stream.read()` - stream.cj:239
- ✅ `Stream.write()` - stream.cj:129

**同步 API**（无 workerthread 标记）：
- ⚠️ `File.dup()` - cj_file_fs.cj:1301（主线程执行）
- ⚠️ `File.fd` 属性 - file.cj:47（主线程执行）

---

## 错误码与异常处理

### BusinessException 封装

**定义**：所有 FFI 返回的错误码都通过 `BusinessException` 抛出

**证据**：`ohos/file/fs/cj_file_fs.cj:950-964`

```cangjie
// ohos/file/fs/cj_file_fs.cj:955-965
public static func stat(file: Int32): Stat {
        let cValue = unsafe { FfiOHOSFileFsStatByID(file) }
        match {
                case cValue.code == SUCCESS_CODE =>
                        return Stat(cValue.data)
                case _ =>
                        FS_LOG.error(getErrorInfo(cValue.code))
                        throw BusinessException(cValue.code, getErrorInfo(cValue.code))
        }
}
```

### 错误码范围

| 错误码范围 | 说明 | 来源 |
|-------------|------|------|
| 13900001-13900044 | 文件系统错误（ENOENT, EACCES, EIO 等） | FFI 返回 |
| 14300002 | URI 无效 | FileUri 模块 |

**完整错误码清单**：TODO（需阅读 `conflict_file_exception.cj`）

---

## FFI 调用链示例

### 文件读取完整链路

```
Cangjie App
    ↓ FileIo.read(fd, buffer, options)
    ↓ unsafe { FfiOHOSFileFsRead(fd, cBuffer, size, length, offset) }
    ↓ Native FFI 边界
    ↓ file_api:cj_file_fs_ffi (C++ 实现)
    ↓ OpenHarmony VFS / read() syscall
    ↓ 返回: RetDataI64(code, data=bytesRead)
    ↓ match { case SUCCESS_CODE => return; _ => throw }
    ↓ BusinessException(code, message)
    ↑ App 层捕获异常
```

**关键证据**：`ohos/file/fs/cj_file_fs.cj:1339-1363`

---

## 关键结论

1. **70+ FFI 函数**：`native.cj` 中声明了 70+ 个外部函数调用
2. **类型安全**：通过 `CString`、`CPointer` 等类型确保类型安全
3. **返回值封装**：统一使用 `RetData*` 结构封装返回值和错误码
4. **错误传播**：所有 FFI 返回通过 `BusinessException` 统一抛出
5. **WorkerThread 优先**：I/O 操作标记 `workerthread: true` 异步化

---

## 相关跳转

- [02_Architecture.md](02_Architecture.md) - 架构层次与数据流
- [04_Public_API.md](04_Public_API.md) - 对外 API 详细说明
- [05_Internal_API.md](05_Internal_API.md) - 内部接口设计
- [06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md) - 构建依赖
