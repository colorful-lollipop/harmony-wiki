# 项目概览

本文档介绍 `request_cangjie_wrapper` 项目的定位、能力边界、运行环境及快速开始指南。

---

## 1.1 项目定义

**一句话定义**: 为 OpenHarmony 上使用 Cangjie 语言开发的应用提供上传下载能力的 N-API 封装层。

**详细说明**: `request_cangjie_wrapper` 是 OpenHarmony 上传下载子系统的 Cangjie 语言封装，基于底层的 `request` 服务实现。它为 Cangjie 开发者提供完整的文件上传/下载能力，涵盖任务创建、移除、暂停、启动等核心操作，同时支持订阅任务进度、成功、失败等状态变更。

**证据来源**:
- `bundle.json:3-4`: "The request_cangjie_wrapper is a Cangjie API encapsulated on OpenHarmony based on the capabilities of the upload and download subsystem."

**项目信息**:
| 属性 | 值 |
|------|-----|
| 项目名称 | @ohos/request_cangjie_wrapper |
| 版本 | 6.1 |
| 子系统 | request |
| 适用设备 | standard (标准设备) |
| ROM 占用 | 400KB |
| RAM 占用 | 332KB |

---

## 1.2 能力边界

### 支持的功能

| 功能类别 | 具体能力 | 证据来源 |
|----------|----------|----------|
| **任务管理** | 创建上传/下载任务 | agent.cj:1661 (Task 类) |
| | 查询任务信息 | agent.cj:1814 (TaskInfo 类) |
| | 移除指定任务 | ffi.cj:810 (FfiOHOSRequestRemoveTask) |
| | 按条件搜索任务 | ffi.cj:818 (FfiOHOSRequestSearchTask) |
| **生命周期控制** | 启动任务 | ffi.cj:800 (FfiOHOSRequestTaskStart) |
| | 停止任务 | ffi.cj:806 (FfiOHOSRequestTaskStop) |
| | 暂停任务 | ffi.cj:802 (FfiOHOSRequestTaskPause) |
| | 恢复任务 | ffi.cj:804 (FfiOHOSRequestTaskResume) |
| **状态订阅** | 订阅进度事件 | agent.cj:1756 (Task.on Progress) |
| | 订阅完成事件 | agent.cj:1756 (Task.on Completed) |
| | 订阅失败事件 | agent.cj:1756 (Task.on Failed) |
| | 订阅响应头 | agent.cj:1715 (Task.on Response) |
| **断点续传** | 设置起始偏移 (begins) | agent.cj:754 (Config.begins) |
| | 设置结束偏移 (ends) | agent.cj:766 (Config.ends) |
| | 设置任务索引 (index) | agent.cj:742 (Config.index) |
| **持久化** | 任务状态持久化 | README.md:20 |
| | token 机制隔离 | agent.cj:799 (Config.token) |

### 不支持的功能

| 功能 | 说明 | 替代方案 |
|------|------|----------|
| 完整 HTTP/HTTPS 接口 | 不提供底层 HTTP 请求能力 | 推荐使用 netmanager |
| 任务限速 | 无法设置每秒传输字节上限 | 无 |
| 订阅失败原因 | 无法获取任务失败具体原因 | 通过 Faults 枚举判断大类 |
| 订阅等待原因 | 无法获取任务等待原因 | 无 |
| 非文件数据单元 | 请求数据必须为文件形式 | 调用方自行封装 |

**证据来源**:
- README.md:67-75: "Compared to APIs provided by ArkTS, the following functions are not currently supported"

---

## 1.3 运行环境

### 系统依赖

```
OpenHarmony 系统
├── 应用层 (App) - Cangjie 应用
├── 框架层 (Framework)
│   └── request_cangjie_wrapper (本文档対象) ← N-API 封装
├── 系统服务层 (System Services)
│   └── request (底层上传下载服务)
└── 内核层 (Kernel)
```

### 依赖组件

| 依赖组件 | 用途 | 证据来源 |
|----------|------|----------|
| `request` | 底层上传下载 FFI 接口 | BUILD.gn:40 |
| `cangjie_ark_interop` | Cangjie 注解类、FFI、异常体系 | BUILD.gn:31-34 |
| `arkui_cangjie_wrapper` | 基础类型定义、CString 转换 | BUILD.gn:35 |
| `hiviewdfx_cangjie_wrapper` | 日志接口 (hilog) | BUILD.gn:36 |
| `ability_cangjie_wrapper` | Ability/Application 上下文 | BUILD.gn:37 |

### 权限要求

使用 `request` 服务需要申请以下权限：

| 权限名称 | 用途 | 申请方式 |
|----------|------|----------|
| `ohos.permission.INTERNET` | 允许应用访问网络 | 在 config.json 中声明 |
| `ohos.permission.WRITE_MEDIA` | 允许应用写入媒体文件 | 在 config.json 中声明 |
| `ohos.permission.READ_MEDIA` | 允许应用读取媒体文件 | 在 config.json 中声明 |

**证据来源**:
- README.md:63-66: "To use the request service, you need to apply for the following permissions"

### API Level 要求

- 最低 API Level: 22
- 系统能力 (Syscap): `SystemCapability.Request.FileTransferAgent`

**证据来源**:
- agent.cj:49-52: `@!APILevel[since: "22", syscap: "SystemCapability.Request.FileTransferAgent"]`

---

## 1.4 快速开始

### 环境准备

1. **申请权限**: 在应用的 `config.json` 中添加权限声明

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET"
      },
      {
        "name": "ohos.permission.WRITE_MEDIA"
      },
      {
        "name": "ohos.permission.READ_MEDIA"
      }
    ]
  }
}
```

2. **导入模块**: 在 Cangjie 代码中导入 request 模块

```cj
import ohos.request.{Task, Config, Action, Mode}
```

### 创建下载任务

```cj
// 1. 创建任务配置
let config = Config(
    action: Action.Download,
    url: "https://example.com/file.zip",
    saveas: "./downloads/file.zip",
    overwrite: true
)

// 2. 创建任务
let task = Task("my-task-id", config)

// 3. 订阅进度事件
task.on(.Progress) { progress =>
    let percent = (progress.processed * 100) / progress.sizes[0]
    println("下载进度: ${percent}%")
}

// 4. 订阅完成事件
task.on(.Completed) { _ =>
    println("下载完成!")
}

// 5. 启动任务
task.start()
```

### 创建上传任务

```cj
// 1. 创建文件规格
let fileSpec = FileSpec(
    path: "./documents/image.png"
)

// 2. 创建配置
let config = Config(
    action: Action.Upload,
    url: "https://example.com/upload",
    data: ConfigData.FormItems([
        FormItem(
            name: "file",
            value: FormItemValue.FileItem(fileSpec)
        )
    ])
)

// 3. 创建并启动任务
let task = Task("upload-task", config)
task.start()
```

### 任务状态查询

```cj
// 通过 tid 查询任务信息
let taskInfo = TaskInfo(tid: "task-id")
println("任务状态: ${taskInfo.progress.state}")
println("已下载: ${taskInfo.progress.processed} 字节")
println("文件大小: ${taskInfo.progress.sizes[0]} 字节")
```

### 事件类型

| 事件类型 | 触发时机 | 回调参数 |
|----------|----------|----------|
| `Progress` | 下载/上传进度更新 | `Progress` |
| `Completed` | 任务完成 | `Progress` |
| `Failed` | 任务失败 | `Progress` |
| `Pause` | 任务暂停 | `Progress` |
| `Resume` | 任务恢复 | `Progress` |
| `Remove` | 任务移除 | `Progress` |
| `Response` | 收到响应头 | `HttpResponse` |

---

## 1.5 约束与限制

### 使用约束

| 约束项 | 说明 | 证据来源 |
|--------|------|----------|
| HTTP HEAD 方法 | 下载服务器必须支持 HTTP HEAD 方法 | README.md:69 |
| 文件存在检查 | 下载时若文件已存在会验证失败 | README.md:72 |
| 多文件上传策略 | 所有文件上传成功才算任务成功 | README.md:73 |
| 数据单元格式 | 请求数据必须为文件形式 | README.md:67 |

### 已知限制

1. **跨平台差异**: Windows/Mac 平台使用 mock 实现，功能受限
2. **Token 安全**: Token 一旦创建无法通过查询获取，需妥善保管
3. **路径格式**: 仅支持特定路径格式，不支持任意路径

---

## 1.6 相关资源

### 外部文档

- [Upload/Download API 参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/BasicServicesKit/cj-apis-request-agent.md)
- [Upload/Download 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/basic-services/request/cj-app-file-upload-download.md)

### 相关仓库

- [request_request](https://gitcode.com/openharmony/request_request/blob/master/README.md) - 底层上传下载服务
- [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/README.md) - Cangjie 互操作层
