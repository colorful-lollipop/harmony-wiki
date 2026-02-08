# 代码地图

本文档提供 `request_cangjie_wrapper` 的目录结构说明和核心代码定位索引。

---

## 3.1 目录结构

```
base/request/request_cangjie_wrapper/
├── figures/                          # 架构图等资源
│   ├── request_cangjie_wrapper_architecture_en.png
│   └── request_cangjie_wrapper_architecture_zh.png
│
├── ohos/request/                     # 核心实现目录
│   ├── BUILD.gn                      # 构建配置 (1,421 bytes)
│   ├── agent.cj                      # 主要 API 实现 (61,865 bytes)
│   ├── error.cj                      # 错误定义 (1,496 bytes)
│   └── ffi.cj                        # FFI 接口封装 (20,995 bytes)
│
├── test/                             # 测试目录 (不计入分析)
│   └── request/
│
├── BUILD.gn                          # 根构建入口
├── bundle.json                       # 组件配置
├── LICENSE                           # Apache 2.0
├── README.md                         # 英文说明
└── README_zh.md                     # 中文说明
```

---

## 3.2 文件职责

| 文件 | 行数 | 职责 | 关键类/函数 |
|------|------|------|-------------|
| **agent.cj** | ~1830 | 公共 API 实现 | Task, Config, Progress, EventCallbackType |
| **ffi.cj** | ~820 | FFI 互操作 | CConfig, CProgress, FFI 函数声明 |
| **error.cj** | ~50 | 错误处理 | ERROR_CODE_MAP, getErrorMsg |

---

## 3.3 核心代码定位

### Task 类 (任务主入口)

| 功能 | 行号 | 代码位置 |
|------|------|----------|
| 类定义 | 1661-1702 | `public class Task` |
| 构造函数 | 1690-1693 | `init(tid: String, config: Config)` |
| 进度回调订阅 | 1756-1787 | `on(event: Progress, callback)` |
| 完成回调订阅 | 1756-1787 | `on(event: Completed, callback)` |
| 响应头回调 | 1715-1742 | `on(event: Response, callback)` |
| 取消订阅 | 1801-1829 | `off(event, callback)` |
| 启动任务 | [FFI 调用] | `FfiOHOSRequestTaskStart` |
| 暂停任务 | [FFI 调用] | `FfiOHOSRequestTaskPause` |
| 恢复任务 | [FFI 调用] | `FfiOHOSRequestTaskResume` |
| 停止任务 | [FFI 调用] | `FfiOHOSRequestTaskStop` |
| 析构函数 | 1696-1702 | `~init()` |

### Config 类 (任务配置)

| 功能 | 行号 | 代码位置 |
|------|------|----------|
| 类定义 | 578-932 | `public class Config` |
| action 字段 | 586 | `action: Action` |
| url 字段 | 596 | `url: String` (最大 8192 字符) |
| saveas 字段 | 687 | `saveas: String` |
| headers 字段 | 661 | `headers: HashMap<String, String>` |
| data 字段 | 672 | `data: ?ConfigData` |
| network 字段 | 696 | `network: Network` |
| token 字段 | 804 | `token: ?String` |
| 构造函数 | 868-905 | `init(action, url, ...)` |
| C 结构体构造 | 907-931 | `init(v: CConfig)` |

### FileSpec 类 (文件规格)

| 功能 | 行号 | 代码位置 |
|------|------|----------|
| 类定义 | 342-433 | `public class FileSpec` |
| path 字段 | 358 | `path: String` |
| mimeType 字段 | 368 | `mimeType: ?String` |
| filename 字段 | 380 | `filename: ?String` |
| extras 字段 | 390 | `extras: HashMap<String, String>` |
| 构造函数 | 404-414 | `init(path, mimeType, ...)` |
| C 结构体构造 | 416-432 | `init(v: CFileSpec)` |

### Progress 类 (进度信息)

| 功能 | 行号 | 代码位置 |
|------|------|----------|
| 类定义 | 1076-1130 | `public class Progress` |
| state 字段 | 1084 | `state: State` |
| index 字段 | 1092 | `index: UInt32` |
| processed 字段 | 1100 | `processed: Int64` |
| sizes 字段 | 1108 | `sizes: Array<Int64>` |
| extras 字段 | 1121 | `extras: HashMap<String, String>` |
| C 结构体构造 | 1123-1129 | `init(v: CProgress)` |

### 枚举类型定义

| 枚举 | 行号 | 用途 |
|------|------|------|
| EventCallbackType | 48-132 | 事件类型 |
| Action | 136-181 | 任务类型 (Upload/Download) |
| Mode | 186-239 | 运行模式 (Background/Foreground) |
| Network | 244-300 | 网络类型 |
| State | 942-1059 | 任务状态 |
| Faults | 1197-1288 | 失败原因 |

### FFI 函数声明 (ffi.cj)

| 函数 | 行号 | 用途 |
|------|------|------|
| FfiOHOSRequestCreateTask | 808 | 创建任务 |
| FfiOHOSRequestRemoveTask | 810 | 移除任务 |
| FfiOHOSRequestTaskStart | 800 | 启动任务 |
| FfiOHOSRequestTaskPause | 802 | 暂停任务 |
| FfiOHOSRequestTaskResume | 804 | 恢复任务 |
| FfiOHOSRequestTaskStop | 806 | 停止任务 |
| FfiOHOSRequestShowTask | 814 | 查询任务信息 |
| FfiOHOSRequestTouchTask | 816 | 带 token 查询 |
| FfiOHOSRequestSearchTask | 818 | 搜索任务 |
| FfiOHOSRequestTaskProgressOn | 796 | 注册回调 |
| FfiOHOSRequestTaskProgressOff | 798 | 取消回调 |

---

## 3.4 代码导航图

### 新建任务

```
1. 创建 Config 对象
   → agent.cj:578 (Config 类定义)
   → agent.cj:868 (Config 构造函数)

2. 创建 Task 对象
   → agent.cj:1661 (Task 类定义)
   → agent.cj:1690 (Task 构造函数)

3. FFI 调用创建任务
   → ffi.cj:808 (FfiOHOSRequestCreateTask)
   → ffi.cj:396 (CConfig 结构体)
```

### 订阅进度事件

```
1. 调用 Task.on()
   → agent.cj:1756 (on 方法)

2. 参数验证
   → agent.cj:1757-1761 (事件类型检查)

3. FFI 回调注册
   → ffi.cj:796 (FfiOHOSRequestTaskProgressOn)

4. 回调管理
   → agent.cj:1622 (EventManage 类)
   → agent.cj:1572 (RequestEvent 类)
```

### 处理进度回调

```
1. 接收 CProgress
   → ffi.cj:610 (CProgress 结构体)

2. 转换为 Progress
   → agent.cj:1123 (Progress 构造函数)

3. 提取状态
   → agent.cj:1024-1026 (State.parse)

4. 提取进度数据
   → agent.cj:1125-1128 (processed, sizes)
```

### 错误处理

```
1. 获取错误码
   → agent.cj:31-34 (错误码常量)

2. 转换为错误消息
   → error.cj:32 (getErrorMsg 函数)

3. 抛出异常
   → error.cj:40-47 (getErrorMsg 实现)
```

---

## 3.5 关键数据流路径

### 配置创建路径

```
Cangjie Config 构造
    → agent.cj:868
    → CConfig 结构体分配
    → ffi.cj:421 (init(config: Config))
    → FFI 函数调用
```

### 回调注册路径

```
Task.on(event, callback)
    → agent.cj:1756
    → wrapper 函数封装
    → ffi.cj:796 (FfiOHOSRequestTaskProgressOn)
    → request 服务注册
```

### 资源释放路径

```
Task ~init()
    → agent.cj:1696
    → FfiOHOSRequestFreeTask()
    → CConfig free()
    → agent.cj:468 (free 方法)
```

---

## 3.6 内部类定位

| 类名 | 访问级别 | 行号 | 用途 |
|------|----------|------|------|
| RequestEvent | internal | 1572 | 单个事件回调管理 |
| EventManage | internal | 1622 | 事件回调管理器 |
| Notification | @Hide | 37 | 通知配置 |
| MinSpeed | @Hide | 40 | 最小速度配置 |
| Timeout | @Hide | 43 | 超时配置 |
