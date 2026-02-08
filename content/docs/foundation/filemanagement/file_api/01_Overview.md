# File API 项目概览

本文档提供 File API 项目的全面概览，帮助读者快速理解项目定位、能力边界、运行环境，并提供快速开始指南。

## 1. 项目定位

### 1.1 一句话定义

**File API** 是 OpenHarmony 文件管理子系统的核心 N-API 封装层，为应用提供文件 I/O、目录管理、文件系统状态查询等能力，支持 JS、C、CJ、Rust、Native 五种编程接口。

### 1.2 在 OpenHarmony 中的位置

File API 位于 OpenHarmony 系统架构的应用框架层（Framework），是连接上层应用与底层文件系统的关键桥梁。

**证据来源**：`bundle.json:14`

```json
"subsystem": "filemanagement"
```

**证据说明**：File API 属于 `filemanagement` 子系统。

### 1.3 项目定位总结

| 维度 | 定位 |
|------|------|
| **类型** | N-API 插件 / 系统框架模块 |
| **运行域** | 应用框架（Framework） |
| **子系统** | filemanagement |
| **组件名** | file_api |
| **版本** | 4.0 |

### 1.4 核心职责

File API 的核心职责包括以下几个方面：

| 职责 | 说明 |
|------|------|
| **API 抽象** | 将底层 POSIX 接口和系统调用封装为易用的上层 API |
| **安全隔离** | 实现 URI 沙箱机制，确保应用只能访问授权文件 |
| **多语言支持** | 提供 JS、C、CJ、Rust、Native 五种编程接口 |
| **异步支持** | 支持 Promise 和 Callback 两种异步编程模型 |
| **性能优化** | 提供异步 I/O（hyperaio）和读取优化能力 |

## 2. 能力边界

### 2.1 支持的能力

File API 提供以下系统能力：

**证据来源**：`bundle.json:15-21`

```json
"syscap": [
  "SystemCapability.FileManagement.File.FileIO",
  "SystemCapability.FileManagement.File.FileIO.Lite",
  "SystemCapability.FileManagement.File.Environment",
  "SystemCapability.FileManagement.File.DistributedFile",
  "SystemCapability.FileManagement.File.Environment.FolderObtain"
]
```

| 能力名称 | 描述 | 支持的 API 模块 |
|----------|------|----------------|
| File.IO | 完整文件 I/O 操作能力 | @ohos.fileio、@ohos.file.fs |
| File.IO.Lite | 轻量级文件 I/O 能力 | @ohos.fileio（子集） |
| Environment | 环境信息获取能力 | @ohos.file.environment |
| DistributedFile | 分布式文件访问能力 | @ohos.file.fs（扩展） |
| FolderObtain | 文件夹获取能力 | @ohos.file.environment |

### 2.2 不支持的能力

File API 当前不支持以下能力：

| 不支持项 | 说明 | 替代方案 |
|----------|------|----------|
| 外部存储访问 | 禁止访问外部存储目录 | 使用分布式文件服务 |
| 任意路径访问 | 必须通过 URI 或沙箱路径访问 | 使用应用沙箱目录 |
| 网络文件系统 | 不直接支持 NFS 等网络协议 | 使用 dfs_service |
| 实时文件监控 | 不提供实时文件变化通知 | 使用 Watcher API |

### 2.3 功能模块

File API 提供以下功能模块：

| 模块 | JS 命名空间 | 功能描述 |
|------|-------------|----------|
| 文件 I/O | @ohos.fileio | 基础文件读写操作 |
| 文件系统 | @ohos.file.fs | 完整文件系统 API（基于 URI） |
| 文件哈希 | @ohos.file.hash | 文件哈希值计算 |
| 文件统计 | @ohos.file.statfs | 文件系统状态查询 |
| 统计信息 | @ohos.file.statvfs | 文件系统统计信息 |
| 环境信息 | @ohos.file.environment | 获取环境路径信息 |
| 安全标签 | @ohos.file.securityLabel | 文件安全标签操作 |
| 文档管理 | @ohos.file.document | 文档 URI 管理 |
| 沙箱文件 | @ohos.file.file | 沙箱内文件操作 |

## 3. 运行环境

### 3.1 系统依赖

File API 依赖以下系统组件：

**证据来源**：`bundle.json:30-56`

| 依赖类型 | 组件名 | 功能 |
|----------|--------|------|
| 框架 | ability_runtime | 应用框架运行时 |
| 框架 | bundle_framework | 包管理框架 |
| 权限 | access_token | 访问令牌管理 |
| 运行时 | runtime_core | 运行时核心 |
| 运行时 | napi | N-API 运行时 |
| 异步 | libuv | 异步 I/O |
| 异步 | liburing | 高性能 I/O（可选） |
| 日志 | hilog | 日志系统 |
| 安全 | openssl | 加密库 |
| 安全 | bounds_checking_function | 安全函数 |
| IPC | ipc | 进程间通信 |
| IPC | samgr | 服务管理 |
| 存储 | dfs_service | 分布式文件服务 |
| 存储 | app_file_service | 应用文件服务 |
| 账户 | os_account | 系统账户 |

### 3.2 操作系统支持

File API 支持以下操作系统类型：

**证据来源**：`bundle.json:26`

```json
"adapted_system_type": ["mini", "small", "standard"]
```

| 类型 | 说明 | 典型设备 |
|------|------|----------|
| mini | 轻量级系统 | 穿戴设备 |
| small | 标准系统 | 智能手机、平板 |
| standard | 全功能系统 | 智能电视、PC |

### 3.3 硬件要求

| 要求 | 说明 |
|------|------|
| **处理器** | ARMv7 及以上架构 |
| **内存** | 最小 256MB（mini），推荐 512MB 以上 |
| **存储** | 最小 4MB ROM，支持外部存储扩展 |

### 3.4 Feature 开关

File API 提供以下 Feature 开关：

**证据来源**：`file_api.gni:22-25`

```gni
declare_args() {
    file_api_read_optimize = false
    file_api_feature_hyperaio = false
}
```

| Feature | 默认值 | 说明 |
|---------|--------|------|
| file_api_read_optimize | false | 启用读取优化功能 |
| file_api_feature_hyperaio | false | 启用高性能异步 I/O 功能 |

## 4. 架构概览

### 4.1 整体架构

File API 采用分层架构设计，从上到下依次为：

```
┌─────────────────────────────────────┐
│           应用层（JS/C/CJ/Rust）       │
├─────────────────────────────────────┤
│           N-API 接口层                │
│  ┌───────────────────────────────┐   │
│  │ @ohos.fileio | @ohos.file.fs │   │
│  │ @ohos.hash | @ohos.environment │  │
│  └───────────────────────────────┘   │
├─────────────────────────────────────┤
│         LibN 抽象层                   │
│  ┌───────────────────────────────┐   │
│  │ NClass | NVal | NAsyncWork  │    │
│  └───────────────────────────────┘   │
├─────────────────────────────────────┤
│         文件系统封装层                 │
│  ┌───────────────────────────────┐   │
│  │ filemgmt_libfs | filemgmt_   │   │
│  │ libn | filemgmt_libhilog    │   │
│  └───────────────────────────────┘   │
├─────────────────────────────────────┤
│         系统调用层                    │
│  GLIBC | libuv | liburing | OpenSSL  │
├─────────────────────────────────────┤
│           内核层                      │
│      文件系统 | 权限管理 | IPC        │
└─────────────────────────────────────┘
```

### 4.2 模块关系

| 模块 | 依赖关系 | 说明 |
|------|----------|------|
| JS N-API | 依赖 LibN | 提供 JS 接口 |
| Native API | 直接使用 | 提供 C/Rust 接口 |
| LibN | 被所有模块使用 | N-API 抽象层 |
| 文件系统封装 | 依赖系统调用 | 平台相关抽象 |

### 4.3 线程模型

File API 的线程模型包括以下几个关键点：

| 线程类型 | 职责 | 说明 |
|----------|------|------|
| 主线程 | JS 调用入口 | N-API 回调在主线程执行 |
| libuv 线程池 | 异步 I/O | 文件系统操作在独立线程执行 |
| HyperAIO 线程 | 高性能异步 | 可选，专用异步 I/O 线程 |

## 5. 快速开始

### 5.1 环境准备

在开始使用 File API 之前，请确保满足以下环境要求：

| 要求 | 说明 |
|------|------|
| OpenHarmony SDK | 版本 4.0 及以上 |
| Node.js | 版本 14 及以上 |
| 开发工具 | DevEco Studio 或命令行工具 |

### 5.2 导入模块

#### 5.2.1 使用 @ohos.fileio

```javascript
// 导入 fileio 模块
import fileio from '@ohos.fileio';

try {
  // 创建文件流
  let stream = fileio.createStreamSync('/data/storage/el2/base/files/test.txt', 'w+');
  
  // 写入数据
  stream.writeSync('Hello, OpenHarmony!');
  
  // 读取数据
  stream.seekSync(0);
  let buffer = new ArrayBuffer(1024);
  let readLen = stream.readSync(buffer);
  console.log('Read length: ' + readLen);
  
  // 关闭流
  stream.closeSync();
} catch (error) {
  console.error('Error: ' + error.message);
}
```

#### 5.2.2 使用 @ohos.file.fs

```javascript
// 导入 fs 模块
import fs from '@ohos.file.fs';

try {
  // 使用 URI 打开文件（沙箱模式）
  let file = fs.openSync('internal://app/test.txt', fs.OpenMode.READ_WRITE | fs.OpenMode.CREAT);
  
  // 写入数据
  let writeLen = fs.writeSync(file.fd, 'Hello, OpenHarmony!');
  console.log('Write length: ' + writeLen);
  
  // 读取数据
  let buffer = new ArrayBuffer(1024);
  let readLen = fs.readSync(file.fd, buffer);
  console.log('Read length: ' + readLen);
  
  // 关闭文件
  fs.closeSync(file.fd);
} catch (error) {
  console.error('Error: ' + error.message);
}
```

### 5.3 编程模型

File API 支持三种编程模型：

| 编程模型 | 说明 | 适用场景 |
|----------|------|----------|
| **同步** | 方法名包含 Sync，直接返回结果 | 简单操作、小文件 |
| **Promise 异步** | 返回 Promise 对象 | 现代 JS 开发、链式调用 |
| **Callback 异步** | 最后一个参数为回调函数 | 传统回调风格、兼容性 |

#### 5.3.1 同步示例

```javascript
// 同步方式
let stat = fs.statSync('/data/storage/el2/base/files/test.txt');
console.log('File size: ' + stat.size);
```

#### 5.3.2 Promise 异步示例

```javascript
// Promise 方式
fs.stat('/data/storage/el2/base/files/test.txt')
  .then((stat) => {
    console.log('File size: ' + stat.size);
  })
  .catch((error) => {
    console.error('Error: ' + error.message);
  });
```

#### 5.3.3 Callback 异步示例

```javascript
// Callback 方式
fs.stat('/data/storage/el2/base/files/test.txt', (error, stat) => {
  if (error) {
    console.error('Error: ' + error.message);
  } else {
    console.log('File size: ' + stat.size);
  }
});
```

### 5.4 URI 沙箱机制

File API 使用 URI 沙箱机制来限制应用的文件访问范围。

#### 5.4.1 支持的 URI 前缀

| URI 前缀 | 目录类型 | 访问权限 |
|----------|----------|----------|
| internal://app/ | 应用私有目录 | 仅当前应用 |
| internal://cache/ | 缓存目录 | 仅当前应用 |
| internal://share/ | 共享目录 | 所有应用（需权限） |

#### 5.4.2 URI 示例

```javascript
// 访问应用私有目录
let file1 = fs.openSync('internal://app/files/data.txt', fs.OpenMode.READ);

// 访问缓存目录
let file2 = fs.openSync('internal://cache/temp.txt', fs.OpenMode.READ_WRITE);

// 访问共享目录（需要权限）
let file3 = fs.openSync('internal://share/documents/file.txt', fs.OpenMode.READ);
```

### 5.5 错误处理

File API 使用统一的错误码机制：

**证据来源**：`interfaces/kits/js/src/common/uni_error.h`

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| 13900001 | EPERM - 操作不允许 | 检查权限 |
| 13900002 | ENOENT - 文件不存在 | 检查路径 |
| 13900003 | ESRCH - 进程不存在 | 检查上下文 |
| 13900004 | EINTR - 系统调用中断 | 重试操作 |
| 13900005 | EIO - I/O 错误 | 检查存储状态 |
| 13900020 | EISDIR - 是目录而非文件 | 检查文件类型 |

### 5.6 下一步

完成快速开始后，您可以通过以下方式深入学习：

| 学习路径 | 推荐文档 |
|----------|----------|
| 理解架构设计 | [02_Architecture.md](02_Architecture.md) |
| 查看 API 详情 | [04_Interface.md](04_Interface.md) |
| 学习安全使用 | [05_AttackSurface.md](05_AttackSurface.md) |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [02_Architecture.md](02_Architecture.md) | 架构设计详解 |
| [04_Interface.md](04_Interface.md) | API 接口文档 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 |

---

**最后更新**：2026-02-07

**版本**：1.0