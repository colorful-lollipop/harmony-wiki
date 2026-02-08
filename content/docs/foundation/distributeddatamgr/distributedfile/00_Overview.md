# 项目概览

> Distributed File（分布式文件）子系统 - 架构与能力

## 项目定位

### 子系统名称
- **英文**: Distributed File
- **中文**: 分布式文件
- **仓库**: `distributeddatamgr_file`
- **命名空间**: `@OHOS.distributedfile.fileio`, `@system.file`

### 核心能力

| 能力类别 | 功能描述 |
|----------|----------|
| **基础文件 API** | 文件创建/修改/访问、权限管理、绝对路径/文件描述符操作 |
| **基础目录 API** | 目录读取、文件类型识别 |
| **统计 API** | 文件大小、访问权限、修改时间获取 |
| **流式文件 API** | 基于路径或文件描述符的数据流读写 |
| **沙箱文件 API** | 基于 URI 的受限文件操作 |

### 适用范围

- **操作系统**: OpenHarmony
- **运行环境**: Stage 模型应用
- **编码支持**: UTF-8 / UTF-16
- **路径限制**: 不支持外部存储目录 URI

## 架构说明

### 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│              (@OHOS.distributedfile.fileio)                   │
├─────────────────────────────────────────────────────────────┤
│                     JS Engine Layer                           │
│            (JS API → C++ 转换引擎)                            │
├─────────────────────────────────────────────────────────────┤
│                        LibN 层                                │
│     (类型系统 / 内存管理 / 通用编程模型)                       │
├─────────────────────────────────────────────────────────────┤
│                    N-API Layer                               │
│              (napi_*, NAPI_MODULE)                            │
├─────────────────────────────────────────────────────────────┤
│                   Native Core                                 │
│              (文件 I/O 核心逻辑)                               │
├─────────────────────────────────────────────────────────────┤
│                   GLIBC Runtime                              │
│                    (POSIX I/O)                                │
├─────────────────────────────────────────────────────────────┤
│                    Hardware Layer                             │
└─────────────────────────────────────────────────────────────┘
```

**图 1**: Distributed File 子系统架构（来源: [README.md](../README.md)）

### 依赖关系

| 依赖方 | 提供的功能 |
|--------|------------|
| JS 引擎层 | JavaScript → C++ 转换能力 |
| 应用框架 | 应用目录路径解析 |
| GLIBC Runtime | POSIX I/O 系统调用 |

### 线程模型（推断）

基于 N-API 异步编程模型：

| 模式 | 线程 | 说明 |
|------|------|------|
| Sync API | 主线程 | 同步阻塞调用 |
| Callback | 线程池 | 异步回调模式 |
| Promise | 线程池 | 异步 Promise 模式 |

### 关键调用链

**同步 API**:
```
JS (accessSync) → JS Engine → N-API → LibN → GLIBC
```

**异步 API (Callback)**:
```
JS (createStream) → JS Engine → N-API → LibN → 线程池执行 → Callback → GLIBC
```

## 目录结构

> ⚠️ 基于 README 描述的标准结构，实际代码仓库可能不同

```
distributedfile/
├── figures/                     # 架构图等资源
│   └── distributed-file-subsystem-architecture.png
├── interfaces/                   # API 接口层
│   └── kits/                     # 对外暴露的 API
├── utils/                        # 公共组件
│   ├── filemgmt_libhilog/        # 日志组件
│   └── filemgmt_libn/            # 平台相关抽象库
├── services/                     # 服务层（可能在其他仓库）
├── BUILD.gn                      # 构建配置（可能在其他仓库）
└── ohos.build                    # OpenHarmony 构建配置
```

## 相关仓库

| 仓库 | 职责 |
|------|------|
| [distributeddatamgr_file](https://gitee.com/openharmony/distributeddatamgr_file) | 分布式文件核心实现 |
| [filemanagement_dfs_service](https://gitee.com/openharmony/filemanagement_dfs_service) | DFS 服务 |
| [filemanagement_user_file_service](https://gitee.com/openharmony/filemanagement_user_file_service) | 用户文件服务 |
| [filemanagement_storage_service](https://gitee.com/openharmony/filemanagement_storage_service) | 存储服务 |
| [filemanagement_app_file_service](https://gitee.com/openharmony/filemanagement_app_file_service) | 应用文件服务 |

## 约束与限制

### 编码限制
- ✅ 支持: UTF-8, UTF-16
- ❌ 不支持: 其他编码（GBK, ISO-8859-1 等）

### 路径限制
- ❌ 禁止: 外部存储目录 URI
- ✅ 支持: `internal://cache/`, `internal://app/`, `internal://share/`

### 权限限制
- 沙箱目录受应用隔离保护
- 跨应用文件访问需权限授权

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始文档（基于 README） |

## 参考

- [API 参考](01_APIs.md)
- [目录结构](02_Directory_Structure.md)
- [安全评审](04_Security_Review.md)
