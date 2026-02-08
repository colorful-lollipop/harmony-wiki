# 内部接口文档

> 目的：描述内部模块接口、依赖方向、稳定性、可替换点
> 适用范围：架构师、贡献者、代码审查人员
> 最后更新：2026-02-06

## 模块接口总览

### 接口分类

| 模块 | 对外接口 | 内部接口 | 稳定性 | 可替换性 |
|------|----------|----------|--------|----------|
| **Kit 层** | `ohos.file.fs.*`, `ohos.file.fileuri.*` | - | ✅ 稳定 | ❌ 不建议 |
| **FS 模块** | `FileIo` 静态类<br/>`File`, `Stream`, `Stat` 类 | FFI 绑定 | ✅ 稳定 | ⚠️ 谨慎（需同步 FFI） |
| **URI 模块** | `FileUri`, `Uri`, `getUriFromPath` | FFI 绑定 | ✅ 稳定 | ⚠️ 谨慎（需同步 FFI） |

---

## Kit 层接口

### kit.CoreFileKit

**职责**：API 聚合层，控制对外导出

**证据**：`kit/CoreFileKit/index.cj:18-21`

```cangjie
package kit.CoreFileKit

public import ohos.file.fs.*
public import ohos.file.fileuri.*
```

**稳定性评估**：✅ **稳定**（公开 API）

**理由**：
1. 对外 API，向后兼容
2. 无内部实现细节
3. 仅控制导出，不包含业务逻辑

**依赖方向**：→ `ohos.file.fs`, `ohos.file.fileuri`

**可替换性**：❌ **不可替换**（唯一对外入口点）

---

## FS 模块接口

### ohos.file.fs.FiIo

**职责**：静态工具类，提供文件系统操作

**证据**：`ohos/file/fs/cj_file_fs.cj:927`

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.File.FileIO"
]
public class FileIo {
        protected init() {}

        // 40+ 静态方法...
}
```

**稳定性评估**：✅ **稳定**（框架层核心类）

**理由**：
1. 封装完整文件系统操作
2. 业务逻辑在框架层实现
3. FFI 绑定在底层

**依赖方向**：
- → `ohos.ffi`（FFI 类型与函数）
- → `ohos.business_exception`（错误处理）
- → `ohos.hilog`（日志）
- → `file_api:cj_file_fs_ffi`（外部依赖）

**可替换性**：⚠️ **谨慎替换**（需同步 FFI 接口）

**可替换点**：
- ❌ 不建议替换整个 `FileIo` 类（对内逻辑耦合深）
- ⚠️ 可替换单个方法（需保持 FFI 兼容性）

---

### ohos.file.fs.File

**职责**：已打开文件的句柄对象

**证据**：`ohos/file/fs/file.cj:31`

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.File.FileIO"
]
public class File <: RemoteDataLite {
        init(instanceId: Int64) {
                super(instanceId)
        }

        ~init() {
                releaseFFIData(myDataId)
        }

        public prop fd: Int32 { get() { ... } }
        public prop path: String { get() { ... } }
        public prop name: String { get() { ... } }
}
```

**稳定性评估**：✅ **稳定**（核心资源管理类）

**理由**：
1. 继承 `RemoteDataLite`（系统基类）
2. RAII 模式（自动资源释放）
3. 属性仅读（设计简单）

**依赖方向**：
- → `ohos.ffi.RemoteDataLite`（继承）
- → `ohos.business_exception`（错误处理）

**可替换性**：⚠️ **谨慎替换**（需保持 RemoteDataLite 兼容）

**可替换点**：
- ⚠️ 可扩展新属性（需同步 FFI）
- ❌ 不建议修改资源管理逻辑

---

### ohos.file.fs.Stream

**职责**：缓冲流 I/O 对象

**证据**：`ohos/file/fs/stream.cj:33`

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.File.FileIO"
]
public class Stream <: RemoteDataLite {
        init(instanceId: Int64) {
                super(instanceId)
        }

        public func read(buffer: Array<Byte>, options!: ReadOptions): Int64
        public func write(buffer: Array<Byte>, options!: WriteOptions): Int64
        public func flush(): Unit
        public func close(): Unit
}
```

**稳定性评估**：✅ **稳定**（核心 I/O 类）

**理由**：
1. 继承 `RemoteDataLite`（系统基类）
2. 接口清晰（read/write/flush/close）
3. WorkerThread 模式（异步执行）

**依赖方向**：
- → `ohos.ffi.RemoteDataLite`（继承）
- → `ohos.business_exception`（错误处理）
- → `ohos.hilog`（日志）

**可替换性**：⚠️ **谨慎替换**（需保持 FFI 兼容）

**可替换点**：
- ⚠️ 可扩展新的 I/O 方法（需同步 FFI）
- ❌ 不建议修改缓冲区管理逻辑

---

### ohos.file.fs.Stat

**职责**：文件元数据对象

**证据**：`ohos/file/fs/stat.cj:31`

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.File.FileIO"
]
public class Stat <: RemoteDataLite {
        public prop ino: Int64 { get() { ... } }
        public prop mode: Int64 { get() { ... } }
        public prop uid: Int64 { get() { ... } }
        public prop gid: Int64 { get() { ... } }
        public prop size: Int64 { get() { ... } }
        // ... 9 个属性
}
```

**稳定性评估**：✅ **稳定**（元数据只读类）

**理由**：
1. 继承 `RemoteDataLite`（系统基类）
2. 所有属性只读（不可变）
3. 无修改方法（设计简单）

**依赖方向**：
- → `ohos.ffi.RemoteDataLite`（继承）

**可替换性**：⚠️ **可扩展**（添加新的元数据属性）

**可替换点**：
- ✅ 可扩展新属性（需同步 FFI）
- ✅ 可扩展新的类型检查方法

---

## URI 模块接口

### ohos.file.fileuri.FileUri

**职责**：文件 URI 对象

**证据**：`ohos/file/fileuri/file_uri.cj:86`

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.AppFileService"
]
public class FileUri <: Uri {
        public init(uriOrPath: String) { ... }

        public override prop path: String { get() { ... } }
        public prop name: String { get() { ... } }
        public override func toString(): String { ... }
}
```

**稳定性评估**：✅ **稳定**（功能明确）

**理由**：
1. 继承抽象 `Uri` 基类
2. 接口简单（path/name/toString）
3. 无复杂状态管理

**依赖方向**：
- → `ohos.ffi.RemoteDataLite`（继承）
- → `ohos.business_exception`（错误处理）

**可替换性**：⚠️ **谨慎替换**（需同步 FFI）

**可替换点**：
- ⚠️ 可扩展新 URI 方案（需同步 FFI）
- ❌ 不建议修改路径解析逻辑

### ohos.file.fileuri.Uri

**职责**：URI 抽象基类

**证据**：`ohos/file/fileuri/file_uri.cj:46`

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.AppFileService"
]
protected abstract class Uri <: RemoteDataLite & ToString {
        protected abstract class UriImpl { }

        public open prop path: String { get() { throw ... } }
        public open func toString(): String { throw ... }
}
```

**稳定性评估**：✅ **稳定**（抽象基类）

**理由**：
1. 定义抽象接口
2. 抛出异常表示不支持的抽象方法

**可替换性**：❌ **不可替换**（系统抽象）

---

## FFI 层接口

### ohos.file.fs.native

**职责**：外部函数声明（70+ FFI 函数）

**证据**：`ohos/file/fs/native.cj:22-178`

```cangjie
package ohos.file.fs

import ohos.ffi.{RetDataI64, RetDataCString, RetDataBool, RetCode}

foreign {
        func FfiOHOSFileFsOpen(path: CString, openMode: Int64): RetDataI64
        func FfiOHOSFileFsRead(fd: Int32, buffer: CPointer<Byte>, size: Int64, length: UIntNative, offset: Int64): RetDataI64
        // ... 70+ 个外部函数声明
}
```

**稳定性评估**：✅ **稳定**（FFI 绑定层）

**理由**：
1. 纯声明层，无实现
2. 类型安全的外部函数签名
3. 外部依赖变更影响大

**依赖方向**：
- → `file_api:cj_file_fs_ffi`（外部 C++ 库）

**可替换性**：❌ **不可替换**（外部依赖契约）

---

## 依赖关系图

### Cangjie 依赖链

```
kit.CoreFileKit
    ↓ import
ohos.file.fs
    ↓ cj_external_deps
├── cangjie_ark_interop:ohos.ffi
│   └── RemoteDataLite (基类)
├── cangjie_ark_interop:ohos.business_exception
│   └── BusinessException (错误处理)
├── cangjie_ark_interop:ohos.labels
│   └── @!APILevel (API 标注)
└── hiviewdfx_cangjie_wrapper:ohos.hilog
    └── HilogChannel (日志)
    ↓ external_deps
file_api:cj_file_fs_ffi
    └── FfiOHOSFileFs* (70+ 函数)
```

### 依赖方向说明

| 层次 | 依赖类型 | 方向 | 稳定性 |
|------|----------|------|--------|
| **Kit 层** | `import` | → 内部模块 | ✅ 稳定 |
| **框架层** | `cj_external_deps` | → 外部 Cangjie 库 | ✅ 稳定 |
| **FFI 层** | `external_deps` | → 外部 C++ 库 | ⚠️ 谨慎（外部变更） |
| **服务层** | C++ 静态库 | ← FFI 声明 | ❌ 外部（不可控） |

---

## 接口稳定性分级

### 稳定性定义

| 级别 | 定义 | 示例 | 变更影响 |
|------|------|------|----------|
| ✅ **稳定** | 对外 API、核心类、无计划变更 | `FileIo`, `File`, `FileUri` | 需向后兼容 |
| ⚠️ **谨慎** | FFI 绑定、内部实现 | `native.cj` | 可能破坏兼容性 |
| ❌ **不可替换** | 系统抽象、唯一入口 | `kit.CoreFileKit`, `RemoteDataLite` | 破坏架构 |

### 可替换点清单

| 模块/类 | 可替换性 | 替换建议 |
|----------|----------|----------|
| `kit.CoreFileKit` | ❌ 不建议 | 唯一入口点，不建议修改 |
| `FileIo` | ⚠️ 谨慎 | 可扩展新方法，需同步 FFI |
| `File` | ⚠️ 谨慎 | 可扩展新属性，需同步 FFI |
| `Stream` | ⚠️ 谨慎 | 可扩展新 I/O 方法，需同步 FFI |
| `Stat` | ✅ 可扩展 | 可扩展新元数据属性 |
| `FileUri` | ⚠️ 谨慎 | 可扩展新 URI 方案，需同步 FFI |
| `Uri` | ❌ 不建议 | 系统抽象基类 |
| `RemoteDataLite` | ❌ 不可替换 | 系统基类 |

---

## 接口设计模式

### 1. 静态工具类模式

**示例**：`FileIo` 类

**证据**：`ohos/file/fs/cj_file_fs.cj:927`

```cangjie
public class FileIo {
        protected init() {}  // 禁止实例化

        public static func open(path: String, mode!: Int64): File
        public static func stat(file: String): Stat
        // ... 40+ 静态方法
}
```

**特点**：
- ✅ 无状态
- ✅ 工具方法集合
- ✅ 禁止 `new FileIo()`

### 2. RAII 资源管理模式

**示例**：`File`, `Stream`, `Stat` 类

**证据**：`ohos/file/fs/file.cj:32-38`

```cangjie
public class File <: RemoteDataLite {
        init(instanceId: Int64) {
                super(instanceId)  // 获取资源 ID
        }

        ~init() {
                releaseFFIData(myDataId)  // 自动释放
        }
}
```

**特点**：
- ✅ 构造时获取资源
- ✅ 析构时自动释放
- ✅ 防止资源泄漏

### 3. Option 模式

**示例**：`WriteOptions`, `ReadOptions`, `ListFileOptions` 类

**证据**：`ohos/file/fs/cj_file_fs.cj:309-363`

```cangjie
public class WriteOptions <: Options {
        public var offset: Option<Int64>
        public var length: Option<UIntNative>

        public init(
                length!: Option<UIntNative> = None,
                offset!: Option<Int64> = None,
                encoding!: String = "utf-8"
        ) {
                super(encoding: encoding)
                this.length = length
                this.offset = offset
        }
}
```

**特点**：
- ✅ 配置对象化
- ✅ 参数可选（`Option<T>`）
- ✅ 默认值支持

---

## 关键结论

1. **三层接口**：Kit（稳定）→ 框架层（稳定）→ FFI 层（谨慎）
2. **依赖清晰**：Cangjie → 外部 Cangjie 库 → 外部 C++ 库
3. **RAII 模式**：所有资源对象通过 `RemoteDataLite` 自动管理
4. **可扩展点**：`Stat`、`FileIo` 可扩展新方法/属性
5. **不可替换点**：`kit.CoreFileKit`、`RemoteDataLite`、`Uri` 基类

---

## 相关跳转

- [01_Directories_and_Modules.md](01_Directories_and_Modules.md) - 目录结构与模块职责
- [02_Architecture.md](02_Architecture.md) - 架构层次
- [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) - FFI 函数清单
- [06_GN_Targets_and_Build.md](06_GN_Targets_and_Build.md) - 构建依赖
