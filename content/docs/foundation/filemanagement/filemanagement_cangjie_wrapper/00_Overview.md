# 项目概览

> 目的：提供项目定位、边界、核心能力、运行环境、关键概念的快速了解
> 适用范围：新人快速理解项目、架构师评估系统定位
> 最后更新：2026-02-06

## 项目定位

### 核心定位

**filemanagement_cangjie_wrapper** 是 OpenHarmony 系统的**文件管理仓颉（Cangjie）语言 API 封装层**，为应用开发者提供：

- ✅ 安全的文件访问能力
- ✅ 完善的文件存储管理能力
- ✅ 文件 URI 处理能力

**目标用户**：使用 Cangjie 语言开发 OpenHarmony 应用的开发者

**系统定位**：
- 属于 **filemanagement** 子系统
- 部件名：**filemanagement_cangjie_wrapper**
- API 级别：**22+**

### 项目边界

**负责范围**：
- ✅ 基础文件操作（创建、删除、移动、重命名）
- ✅ 文件读写（流式 I/O、随机访问）
- ✅ 文件元信息获取（stat、权限、时间戳）
- ✅ 目录操作（列表、过滤、递归）
- ✅ 文件 URI 获取与路径转换
- ✅ 文件锁机制

**不负责范围**（对比 ArkTS API，README:61-70）：
- ❌ 端云同步管理功能
- ❌ 目录环境功能
- ❌ 文件哈希处理功能
- ❌ 选择器功能（文件选择器和保存器）
- ❌ 数据标签功能（安全等级）
- ❌ 文件系统空间统计
- ❌ 应用空间统计
- ❌ 文件分享功能

---

## 核心能力

### 1. 基础文件管理能力

**证据**：`ohos/file/fs/cj_file_fs.cj:927` - `FileIo` 静态类

**功能清单**：

| 类别 | 方法（示例） | 功能说明 |
|------|---------------|----------|
| **文件操作** | `open()`, `close()`, `rename()`, `unlink()` | 打开、关闭、重命名、删除文件 |
| **目录操作** | `mkdir()`, `rmdir()`, `listFile()` | 创建、删除、列出目录内容 |
| **复制移动** | `copyFile()`, `copyDir()`, `moveFile()`, `moveDir()` | 文件/目录的复制与移动 |
| **元信息** | `stat()`, `lstat()` | 获取文件详细信息 |
| **权限检查** | `access()` | 检查文件存在性与读写权限 |

### 2. 文件流 I/O 能力

**证据**：`ohos/file/fs/stream.cj:33` - `Stream` 类

**功能清单**：

| 方法 | 功能说明 | 支持的数据类型 |
|------|----------|--------------|
| `read()` | 从流读取数据 | `Array<Byte>`, `String` |
| `write()` | 向流写入数据 | `Array<Byte>`, `String` |
| `flush()` | 刷新缓冲区 | - |
| `close()` | 关闭流 | - |

**特性**：
- ✅ 支持同步/异步模式（注解 `workerthread: true`）
- ✅ 支持偏移量读写（`offset` 参数）
- ✅ 支持编码配置（`encoding` 参数，默认 UTF-8）

### 3. 文件元数据能力

**证据**：`ohos/file/fs/stat.cj:31` - `Stat` 类

**属性清单**（stat.cj:47-87）：

| 属性 | 类型 | 说明 |
|------|------|------|
| `ino` | `Int64` | 文件标识符（inode） |
| `mode` | `Int64` | 文件权限（八进制，rwxrwxrwx） |
| `uid` | `Int64` | 文件所有者 ID |
| `gid` | `Int64` | 文件所属组 ID |
| `size` | `Int64` | 文件大小（字节） |
| `atime` | `Int64` | 最后访问时间（Unix 时间戳） |
| `mtime` | `Int64` | 最后修改时间（Unix 时间戳） |
| `ctime` | `Int64` | 元数据修改时间（Unix 时间戳） |

**类型检查方法**（stat.cj:70-86）：

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `isBlockDevice()` | `Bool` | 是否为块设备 |
| `isCharacterDevice()` | `Bool` | 是否为字符设备 |
| `isDirectory()` | `Bool` | 是否为目录 |
| `isFile()` | `Bool` | 是否为普通文件 |
| `isFifo()` | `Bool` | 是否为命名管道（FIFO） |
| `isSocket()` | `Bool` | 是否为套接字 |
| `isSymbolicLink()` | `Bool` | 是否为符号链接 |

### 4. 文件 URI 能力

**证据**：`ohos/file/fileuri/file_uri.cj:96` - `getUriFromPath()` 函数

**功能清单**：

| 类/函数 | 功能说明 |
|----------|----------|
| `FileUri(uriOrPath: String)` | 构造文件 URI 对象 |
| `FileUri.path` (属性) | 获取 URI 的路径部分 |
| `FileUri.name` (属性) | 获取文件名 |
| `FileUri.toString()` (方法) | 将 URI 序列化为字符串 |
| `getUriFromPath(path: String)` | 从应用沙箱路径获取 URI |

---

## 运行环境

### 支持的平台

**证据**：`bundle.json:17-19`

```json
"adapted_system_type": [
        "standard"
]
```

| 平台类型 | 支持情况 | 说明 |
|----------|----------|------|
| **Standard** | ✅ 支持 | 标准设备（手机、平板等） |
| **Small** | ❌ 不支持 | 小型系统 |
| **Mini** | ❌ 不支持 | 轻量系统 |

### 系统要求

| 要求项 | 说明 | 证据 |
|--------|------|------|
| **API Level** | 22+ | 所有 `@!APILevel` 注解标注 `since: "22"` |
| **编码支持** | UTF-8/16 | README:58 约束条件 |
| **依赖组件** | 4 个外部组件 | bundle.json:22-28 |

---

## 关键概念

### 1. RemoteDataLite（远程数据轻量级）

**定义**：`cangjie_ark_interop` 提供的基类，用于管理跨进程边界的远程资源

**使用证据**：
- `File` 类：`ohos/file/fs/file.cj:31` - `public class File <: RemoteDataLite`
- `Stream` 类：`ohos/file/fs/stream.cj:33` - `public class Stream <: RemoteDataLite`
- `Stat` 类：`ohos/file/fs/stat.cj:31` - `public class Stat <: RemoteDataLite`
- `FileUri` 类：`ohos/file/fileuri/file_uri.cj:86` - `public class FileUri <: Uri`

**生命周期**：
```cangjie
init(instanceId: Int64) {
        super(instanceId)  // 继承远程 ID
}

~init() {
        releaseFFIData(myDataId)  // 自动释放远程资源
}
```

**作用**：
- ✅ 管理原生堆上的远程资源
- ✅ 自动资源清理（RAII 模式）
- ✅ 防止资源泄漏

### 2. FFI（Foreign Function Interface）

**定义**：Cangjie 语言提供的 C 语言互操作机制

**证据**：`ohos/file/fs/native.cj:22-178` - 70+ `foreign func` 声明

**作用**：
- ✅ 跨语言边界调用 C/C++ 函数
- ✅ 类型安全的参数传递
- ✅ 返回值封装（`RetDataI64`, `RetDataCString`, `RetDataBool`）

**示例**：
```cangjie
// ohos/file/fs/native.cj:100
foreign {
        func FfiOHOSFileFsOpen(path: CString, openMode: Int64): RetDataI64
}
```

### 3. WorkerThread（工作线程）

**定义**：标记 API 在工作线程执行（非主线程）

**证据**：`ohos/file/fs/cj_file_fs.cj:955` - `@!APILevel` 注解

```cangjie
@!APILevel[
        since: "22",
        syscap: "SystemCapability.FileManagement.File.FileIO",
        throwexception: true,
        workerthread: true  // 关键：在工作线程执行
]
public static func stat(file: String): Stat
```

**作用**：
- ✅ 避免阻塞主线程（UI 线程）
- ✅ 提升用户体验（I/O 操作异步化）

---

## 系统架构概要

**证据**：README:8-29 架构图

```
┌─────────────────────────────────────────────────────────────┐
│  Interface Layer（接口层）                                │
│  - Basic File Management API（基础文件管理 API）                 │
│  - File URI API（文件 URI API）                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Framework Layer（框架层）                               │
│  - Basic File Management Wrapper（文件管理封装）                 │
│  - File URI Wrapper（文件 URI 封装）                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Cangjie File Management Service Dependencies（服务依赖）        │
│  - File Access Interface（文件访问接口）                        │
│  - Public File Access Service（公共文件访问服务）            │
│  - cangjie_ark_interop（C 语言互操作）                 │
│  - Cangjie DFX（日志接口）                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 关键结论

1. **定位明确**：文件管理仓颉 API 封装层，提供安全、易用的文件访问能力
2. **能力聚焦**：聚焦基础文件操作，暂不支持云同步、文件分享等高级功能
3. **三层架构**：接口层 → 框架层 → FFI 桥接层 → 外部服务
4. **资源管理**：通过 `RemoteDataLite` 基类实现自动资源清理
5. **异步优先**：文件操作标记 `workerthread: true` 避免阻塞主线程

---

## 相关跳转

- [01_Directories_and_Modules.md](01_Directories_and_Modules.md) - 详细目录结构与模块职责
- [02_Architecture.md](02_Architecture.md) - 架构深入说明
- [04_Public_API.md](04_Public_API.md) - 对外 API 使用指南
