# 目录结构与模块职责

> 目的：描述项目代码组织方式、各目录/模块的职责与边界
> 适用范围：新人理解项目结构、开发者查找代码位置、架构师分析模块依赖
> 最后更新：2026-02-06

## 目录结构（排除测试）

```
filemanagement_cangjie_wrapper/
├── figures/                           # 架构图与文档图片
├── kit/CoreFileKit/                   # Kit 层：对外的 API 聚合
│   └── index.cj                      # Kit 入口点，导出所有公开接口
├── ohos/file/                          # 框架层：核心实现
│   ├── file_package.cj               # ohos.file 包声明
│   ├── fileuri/                      # File URI 模块
│   │   └── file_uri.cj               # Uri/FileUri 类实现
│   └── fs/                           # 文件系统模块
│       ├── cj_file_fs.cj             # FileIo 静态类与配置类型
│       ├── file.cj                   # File 类（已打开文件句柄）
│       ├── stream.cj                 # Stream 类（缓冲流 I/O）
│       ├── stat.cj                   # Stat 类（文件元数据）
│       ├── random_access_file.cj     # RandomAccessFile 类（随机访问）
│       ├── native.cj                 # FFI 外部函数声明（70+ 函数）
│       └── conflict_file_exception.cj# 错误码定义
├── mock/                              # Mock 实现层（测试桩）
│   ├── ohos.file.fs.cj               # Mock 文件系统
│   ├── ohos.file.fileuri.cj          # Mock 文件 URI
│   └── ohos.file.cj                  # Mock 包
└── test/                              # 测试目录（本文档忽略）
    ├── filemanagement/               # 文件管理测试
    └── file_uri/                     # 文件 URI 测试
```

## 模块职责详解

### 1. Kit 层 (`kit/CoreFileKit/`)

**职责**：对外 API 聚合层，提供统一入口点

**关键文件**：
- `index.cj` (行 18-21) - Kit 入口

**证据**：
```cangjie
// kit/CoreFileKit/index.cj:20-21
package kit.CoreFileKit

public import ohos.file.fs.*
public import ohos.file.fileuri.*
```

**导出内容**：
- `ohos.file.fs.*` - 所有文件系统操作（FileIo、File、Stream、Stat 等）
- `ohos.file.fileuri.*` - 文件 URI 操作（FileUri、Uri、getUriFromPath）

**稳定性**：✅ 稳定（公开 API，向后兼容）

---

### 2. 框架层 (`ohos/file/`)

#### 2.1 FS 模块 (`ohos/file/fs/`)

**职责**：文件系统操作、流 I/O、元数据查询

**核心类**：

| 类 | 文件 | 行号 | 职责 | 继承 |
|----|------|------|------|------|
| **FileIo** | cj_file_fs.cj | 静态工具类，提供 40+ 文件操作静态方法 | - |
| **File** | file.cj:31 | 已打开文件的句柄，提供 fd、path、name 属性和文件锁 | `RemoteDataLite` |
| **Stream** | stream.cj:33 | 缓冲流 I/O 对象，提供 read/write/flush/close | `RemoteDataLite` |
| **Stat** | stat.cj:31 | 文件元数据对象，提供 ino、mode、uid、gid、size 等 9 个属性 | `RemoteDataLite` |
| **RandomAccessFile** | random_access_file.cj | 随机访问文件对象（TODO: 完整分析） | `RemoteDataLite` |

**FFI 绑定**：
- `native.cj` - 70+ `foreign func FfiOHOS*` 声明（行 22-178）

**稳定性**：✅ 稳定（框架层核心实现）

---

#### 2.2 URI 模块 (`ohos/file/fileuri/`)

**职责**：文件 URI 路径转换与获取

**核心类**：

| 类 | 文件 | 行号 | 职责 | 继承 |
|----|------|------|------|------|
| **Uri** | file_uri.cj:46 | URI 抽象基类，定义 toString/ path 抽象接口 | `RemoteDataLite & ToString` |
| **FileUri** | file_uri.cj:86 | 具体文件 URI 实现，提供 path/name/toString | `Uri` |

**导出函数**：
- `getUriFromPath(path: String): String` (file_uri.cj:96) - 从路径获取 URI

**稳定性**：✅ 稳定（功能明确）

---

### 3. Mock 层 (`mock/`)

**职责**：单元测试桩实现，隔离外部依赖

**关键文件**：
- `ohos.file.fs.cj` - Mock 文件系统操作
- `ohos.file.fileuri.cj` - Mock 文件 URI
- `ohos.file.cj` - Mock 包

**证据**（BUILD.gn:21-22）：
```python
# ohos/file/fs/BUILD.gn:21-22
if (is_mingw || is_mac){
    sources = [
        "../../../mock/ohos.file.fs.cj",
        "conflict_file_exception.cj",
        "native.cj"
    ]
}
```

**稳定性**：⚠️ 不稳定（测试专用，不用于生产）

---

### 4. 测试层 (`test/`)

**职责**：单元测试与集成测试

**说明**：按本文档约束，不引用测试内容作为业务证据

---

## 模块依赖关系

### 依赖方向图

```
kit/CoreFileKit
    ↓ cj_deps
ohos.file.fs
ohos.file.fileuri
    ↓ cj_external_deps
cangjie_ark_interop (ohos.ffi, ohos.business_exception, ohos.labels)
hiviewdfx_cangjie_wrapper (ohos.hilog)
    ↓ external_deps (Native C++ FFI)
file_api:cj_file_fs_ffi
app_file_service:cj_file_fileuri_ffi
```

### 依赖说明

**Kit 层依赖**（kit/CoreFileKit/BUILD.gn:22-24）：
```python
cj_deps = [
        "../../ohos/file/fs:ohos.file.fs",
        "../../ohos/file/fileuri:ohos.file.fileuri",
]
```

**FS 模块依赖**（ohos/file/fs/BUILD.gn:39-44）：
```python
cj_external_deps = [
        "cangjie_ark_interop:ohos.business_exception",
        "cangjie_ark_interop:ohos.ffi",
        "cangjie_ark_interop:ohos.labels",
        "hiviewdfx_cangjie_wrapper:ohos.hilog",
]
external_deps = [ "file_api:cj_file_fs_ffi" ]
```

**URI 模块依赖**（ohos/file/fileuri/BUILD.gn:27-32）：
```python
cj_external_deps = [
        "cangjie_ark_interop:ohos.business_exception",
        "cangjie_ark_interop:ohos.ffi",
        "cangjie_ark_interop:ohos.labels",
        "hiviewdfx_cangjie_wrapper:ohos.hilog",
]
external_deps = [ "app_file_service:cj_file_fileuri_ffi" ]
```

---

## 模块边界与职责边界

### 职责边界

| 模块 | 负责 | 不负责 |
|------|------|--------|
| **Kit 层** | 对外 API 聚合、导出控制 | 业务逻辑实现（委托给框架层） |
| **FS 模块** | 文件系统操作、I/O、元数据 | URI 处理（由 URI 模块负责） |
| **URI 模块** | URI 路径转换 | 实际文件操作（调用 FFI） |
| **FFI 层** | 外部函数声明 | 实际 C++ 实现（在外部仓库） |

### 模块间通信

- **Kit → 框架层**：通过 `import` 语句（直接引用）
- **框架层 → FFI**：通过 `foreign func` 声明（FFI 调用）
- **框架层 → 外部服务**：通过 FFI 函数跨语言边界

---

## 关键结论

1. **三层架构**：Kit（对接口）→ 框架层（实现）→ FFI（桥接）→ 外部服务（C++）
2. **核心入口**：`kit.CoreFileKit` 是唯一对外的 API 聚合点（index.cj:18-21）
3. **模块独立**：FS 和 URI 模块功能独立，通过 Kit 层组合使用
4. **Mock 层隔离**：通过 `is_mingw || is_mac` 平台条件切换 Mock 实现
5. **FFI 依赖**：所有外部 C++ 接口通过 `external_deps` 声明在 BUILD.gn 中

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览与核心能力
- [02_Architecture.md](02_Architecture.md) - 架构说明与数据流
- [04_Public_API.md](04_Public_API.md) - 对外 API 详细说明
- [05_Internal_API.md](05_Internal_API.md) - 内部接口设计
