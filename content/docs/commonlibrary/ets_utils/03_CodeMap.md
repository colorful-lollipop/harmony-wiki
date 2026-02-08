# 目录结构与代码地图

> 顶层目录职责、核心文件定位、代码导航图

**文档版本**: 1.0  
**更新日期**: 2026-02-07  
**适用版本**: OpenHarmony ets_utils 组件

---

## 1. 顶层目录结构

```
commonlibrary/ets_utils/
├── base_sdk/                    # SDK 基础工具
│   ├── ets/                     # ArkTS 基础定义
│   └── transfer/                # 类型转换工具
│
├── js_api_module/               # 【API 模块】基础数据处理能力
│   ├── buffer/                  # Buffer/Blob 二进制数据操作
│   ├── convertxml/              # XML 格式转换
│   ├── fastbuffer/              # 高性能缓冲区
│   ├── uri/                     # URI 解析 (RFC 2396)
│   ├── url/                     # URL 解析 (WhatWG URL Standard)
│   └── xml/                     # XML 解析与序列化
│
├── js_concurrent_module/        # 【并发模块】多线程支持
│   ├── common/                  # 并发模块公共代码
│   │   └── helper/              # 辅助工具 (napi_helper, error_helper)
│   ├── taskpool/                # 任务池 (TaskPool API)
│   ├── utils/                   # 并发工具 (AsyncLock, Condition)
│   └── worker/                  # Worker 多线程 (Worker API)
│
├── js_sys_module/               # 【系统模块】系统级功能
│   ├── console/                 # 控制台输出 (console API)
│   ├── dfx/                     # DFX 调试工具
│   ├── process/                 # 进程管理 (Process API)
│   ├── test/                    # 系统模块测试 (已排除)
│   └── timer/                   # 定时器 (setTimeout/setInterval)
│
├── js_util_module/              # 【工具模块】实用工具类
│   ├── collections/             # 集合工具
│   ├── container/               # 容器类 (15+ 种)
│   │   ├── arraylist/           # 动态数组
│   │   ├── deque/               # 双端队列
│   │   ├── hashmap/             # 哈希映射
│   │   ├── hashset/             # 哈希集合
│   │   ├── lightweightmap/      # 轻量级映射
│   │   ├── lightweightset/      # 轻量级集合
│   │   ├── linkedlist/          # 链表
│   │   ├── list/                # 列表
│   │   ├── plainarray/          # 简单数组
│   │   ├── queue/               # 队列
│   │   ├── stack/               # 栈
│   │   ├── treemap/             # 树形映射
│   │   ├── treeset/             # 树形集合
│   │   └── vector/              # 向量
│   ├── json/                    # JSON 处理
│   ├── stream/                  # 流处理
│   └── util/                    # 工具类
│       ├── plugin/              # 插件
│       └── test/                # 测试 (已排除)
│
├── platform/                    # 【平台抽象层】跨平台支持
│   ├── android/                 # Android 平台适配
│   ├── default/                 # 默认实现
│   ├── ios/                     # iOS 平台适配
│   └── ohos/                    # OpenHarmony 平台适配
│
├── tools/                       # 开发工具
│
└── wiki/                        # 【本文档目录】
    ├── _work/                   # 工作文档
    └── appendix/                # 附录
```

---

## 2. 核心文件定位

### 2.1 入口文件

| 类别 | 文件路径 | 说明 |
|------|----------|------|
| **组件配置** | `bundle.json` | 组件描述、依赖、构建配置 |
| **构建配置** | `ets_utils_config.gni` | GN 变量定义 |
| **事件配置** | `hisysevent.yaml` | 事件监控配置 |

### 2.2 N-API 注册文件 (按模块)

| 模块 | 命名空间 | N-API 注册文件 | 行号 |
|------|---------|---------------|------|
| **URL** | `@ohos.url` | `js_api_module/url/native_module_url.cpp` | 1-50 |
| **URI** | `@ohos.uri` | `js_api_module/uri/native_module_uri.cpp` | 1-40 |
| **XML** | `@ohos.xml` | `js_api_module/xml/native_module_xml.cpp` | 1-45 |
| **Buffer** | `@ohos.buffer` | `js_api_module/buffer/native_module_buffer.cpp` | 1-60 |
| **ConvertXML** | `@ohos.convertxml` | `js_api_module/convertxml/native_module_convertxml.cpp` | 1-35 |
| **FastBuffer** | `@ohos.fastbuffer` | `js_api_module/fastbuffer/native_module_fastbuffer.cpp` | 1-40 |
| **Util** | `@ohos.util` | `js_util_module/util/native_module_util.cpp` | 1-80 |
| **JSON** | `@ohos.json` | `js_util_module/json/native_module_json.cpp` | 1-30 |
| **Collections** | `@ohos.collections` | `js_util_module/collections/native_module_collections.cpp` | 1-35 |
| **Stream** | `@ohos.stream` | `js_util_module/stream/native_module_stream.cpp` | 1-40 |
| **Process** | `@ohos.process` | `js_sys_module/process/native_module_process.cpp` | 1-45 |
| **Console** | `console` | `js_sys_module/console/console.cpp` | 1-50 |
| **Timer** | (内部) | `js_sys_module/timer/sys_timer.cpp` | 1-40 |
| **DFX** | (内部) | `js_sys_module/dfx/native_module_dfx.cpp` | 200-220 |
| **Worker** | `@ohos.worker` | `js_concurrent_module/worker/native_module_worker.cpp` | 30-50 |
| **TaskPool** | `@ohos.taskpool` | `js_concurrent_module/taskpool/native_module_taskpool.cpp` | 1-50 |
| **Concurrent** | `@ohos.concurrent` | `js_concurrent_module/utils/native_utils_module.cpp` | 1-40 |

### 2.3 核心实现文件

#### js_api_module (API 模块)

| 功能 | 头文件 | 实现文件 | 关键类 |
|------|--------|---------|--------|
| URL 解析 | `js_url.h` | `js_url.cpp` | `URL`, `URLSearchParams` |
| URI 解析 | `js_uri.h` | `js_uri.cpp` | `URI` |
| XML 解析 | `js_xml.h` | `js_xml.cpp` | `XmlPullParser`, `XmlSerializer` |
| Buffer | `js_buffer.h` | `js_buffer.cpp` | `Buffer` |
| Blob | `js_blob.h` | `js_blob.cpp` | `Blob` |
| ConvertXML | `js_convertxml.h` | `js_convertxml.cpp` | `ConvertXml` |
| URL 辅助 | `url_helper.h` | `url_helper.cpp` | URL 工具函数 |

#### js_util_module (工具模块)

| 功能 | 头文件 | 实现文件 | 关键类 |
|------|--------|---------|--------|
| TextEncoder | `js_textencoder.h` | `js_textencoder.cpp` | `TextEncoder` |
| TextDecoder | `js_textdecoder.h` | `js_textdecoder.cpp` | `TextDecoder` |
| Base64 | `js_base64.h` | `js_base64.cpp` | `Base64` |
| Types | `js_types.h` | `js_types.cpp` | `Types` |
| UUID | `js_uuid.h` | `js_uuid.cpp` | UUID 生成 |
| 容器 (15种) | `native_module_*.h` | `native_module_*.cpp` | List/Map/Set/... |

#### js_sys_module (系统模块)

| 功能 | 头文件 | 实现文件 | 关键类 |
|------|--------|---------|--------|
| Process | `js_process.h` | `js_process.cpp` | `Process` |
| ChildProcess | `js_childprocess.h` | `js_childprocess.cpp` | `ChildProcess` |
| Timer | `sys_timer.h` | `sys_timer.cpp` | `Timer` |
| Console | `console.h` | `console.cpp` | `Console` |
| Log | `log.h` | - | 日志工具 |
| DFX | `native_module_dfx.cpp` | - | DFX 工具 |

#### js_concurrent_module (并发模块)

| 功能 | 头文件 | 实现文件 | 关键类 |
|------|--------|---------|--------|
| Worker | `worker.h` | `worker.cpp` | `Worker` |
| WorkerRunner | `worker_runner.h` | `worker_runner.cpp` | `WorkerRunner` |
| MessageQueue | `message_queue.h` | `message_queue.cpp` | `MessageQueue` |
| TaskPool | `taskpool.h` | `taskpool.cpp` | `TaskPool` |
| Task | `task.h` | `task.cpp` | `Task` |
| TaskManager | `task_manager.h` | `task_manager.cpp` | `TaskManager` |
| AsyncLock | `async_lock.h` | `async_lock.cpp` | `AsyncLock` |
| AsyncLockManager | `async_lock_manager.h` | `async_lock_manager.cpp` | `AsyncLockManager` |
| ConditionVariable | `condition_variable.h` | `condition_variable.cpp` | `ConditionVariable` |

### 2.4 平台抽象层

| 平台 | 目录 | 关键文件 |
|------|------|----------|
| OpenHarmony | `platform/ohos/` | `util_helper.cpp`, `qos_helper.cpp` |
| Android | `platform/android/` | 平台适配 |
| iOS | `platform/ios/` | 平台适配 |
| Default | `platform/default/` | 默认实现 |

---

## 3. 代码导航图

### 3.1 JS API → Native 调用链

```
【URL 模块调用链示例】

ArkTS 代码:
    import url from '@ohos.url'
    let u = new URL('http://example.com')
    
         ↓
         
JS 入口 (TypeScript 声明):
    (类型定义在 SDK 中)
    
         ↓
         
N-API 注册:
    js_api_module/url/native_module_url.cpp:30
    └── napi_module_register(&g_urlModule)
    
         ↓
         
构造函数绑定:
    js_api_module/url/js_url.cpp:45
    └── URL::URL(napi_env, napi_callback_info)
    
         ↓
         
业务逻辑:
    js_api_module/url/js_url.cpp:67
    └── URL::ParseUrl(input, base)
    
         ↓
         
辅助函数:
    js_api_module/url/url_helper.cpp:34
    └── ParseAuthority()
```

### 3.2 Buffer 操作调用链

```
【Buffer 操作调用链示例】

ArkTS 代码:
    import buffer from '@ohos.buffer'
    let buf = buffer.alloc(1024)
    buf.write('Hello', 0, 5, 'utf-8')
    
         ↓
         
N-API 注册:
    js_api_module/buffer/native_module_buffer.cpp:42
    
         ↓
         
alloc 实现:
    js_api_module/buffer/js_buffer.cpp:156
    └── Buffer::Alloc(size, fill, encoding)
    
         ↓
         
write 实现:
    js_api_module/buffer/js_buffer.cpp:320
    └── Buffer::WriteString(str, offset, length, encoding)
    
         ↓
         
编码转换:
    js_api_module/buffer/converter.cpp:78
    └── ConvertStringToBytes()
```

### 3.3 Worker 多线程调用链

```
【Worker 调用链示例】

ArkTS 代码:
    import worker from '@ohos.worker'
    let w = new worker.Worker('worker.js')
    w.postMessage({data: 'test'})
    
         ↓
         
N-API 注册:
    js_concurrent_module/worker/native_module_worker.cpp:36
    └── nm_modname: "worker"
    
         ↓
         
Worker 构造:
    js_concurrent_module/worker/worker.cpp:156
    └── Worker::Worker(scriptURL, options)
    
         ↓
         
线程创建:
    js_concurrent_module/worker/worker.cpp:178
    └── CreateWorkerThread()
    
         ↓
         
消息发送:
    js_concurrent_module/worker/worker.cpp:320
    └── postMessage(message, transfer)
    
         ↓
         
消息队列:
    js_concurrent_module/worker/message_queue.cpp:67
    └── MessageQueue::Enqueue()
```

### 3.4 Process 系统调用链

```
【Process 调用链示例】

ArkTS 代码:
    import Process from '@ohos.process'
    Process.runCmd('ls -la')
    
         ↓
         
N-API 注册:
    js_sys_module/process/native_module_process.cpp:38
    
         ↓
         
runCmd 实现:
    js_sys_module/process/js_childprocess.cpp:89
    └── ChildProcess::RunCmd(command, options)
    
         ↓
         
进程创建:
    js_sys_module/process/js_childprocess.cpp:145
    └── fork() + execvp()
    
         ↓
         
平台适配:
    platform/ohos/process_helper.cpp
    └── 平台特定实现
```

---

## 4. 功能 → 文件映射表

### 4.1 按功能查找文件

| 功能需求 | 查找路径 | 关键文件 |
|---------|---------|----------|
| **URL 解析** | `js_api_module/url/` | `js_url.cpp`, `url_helper.cpp` |
| **XML 解析** | `js_api_module/xml/` | `js_xml.cpp` |
| **XML 转换** | `js_api_module/convertxml/` | `js_convertxml.cpp` |
| **Buffer 操作** | `js_api_module/buffer/` | `js_buffer.cpp`, `js_blob.cpp` |
| **容器类** | `js_util_module/container/*/` | `native_module_*.cpp` |
| **Text 编解码** | `js_util_module/util/` | `js_textencoder.cpp`, `js_textdecoder.cpp` |
| **Base64** | `js_util_module/util/` | `js_base64.cpp` |
| **JSON** | `js_util_module/json/` | `native_module_json.cpp` |
| **进程管理** | `js_sys_module/process/` | `js_process.cpp`, `js_childprocess.cpp` |
| **定时器** | `js_sys_module/timer/` | `sys_timer.cpp` |
| **控制台** | `js_sys_module/console/` | `console.cpp` |
| **Worker** | `js_concurrent_module/worker/` | `worker.cpp`, `worker_runner.cpp` |
| **TaskPool** | `js_concurrent_module/taskpool/` | `taskpool.cpp`, `task.cpp` |
| **AsyncLock** | `js_concurrent_module/utils/locks/` | `async_lock.cpp` |
| **Condition** | `js_concurrent_module/utils/condition/` | `condition_variable.cpp` |

### 4.2 按问题类型查找文件

| 问题类型 | 相关目录 | 关键文件 |
|---------|---------|----------|
| **内存问题** | `js_api_module/buffer/` | `js_buffer.cpp`, `js_blob.cpp` |
| **并发问题** | `js_concurrent_module/*/` | `worker.cpp`, `taskpool.cpp`, `async_lock.cpp` |
| **安全问题** | `js_sys_module/process/` | `js_process.cpp`, `js_childprocess.cpp` |
| **编码问题** | `js_util_module/util/` | `js_textencoder.cpp`, `js_textdecoder.cpp` |
| **解析问题** | `js_api_module/url/`, `js_api_module/xml/` | `js_url.cpp`, `js_xml.cpp` |
| **构建问题** | 各模块 `BUILD.gn` | 配置相关 |

---

## 5. 头文件依赖图

### 5.1 公共头文件

```
tools/
├── common_helper.h          # 通用辅助宏
├── ets_error.h              # 错误码定义
└── log.h                    # 日志工具

platform/
├── utils.h                  # 平台工具
├── process_helper.h         # 进程辅助
├── util_helper.h            # 工具辅助
└── qos_helper.h             # QoS 辅助
```

### 5.2 模块间依赖

```
js_concurrent_module/
└── common/helper/
    ├── napi_helper.h        # N-API 辅助 (被所有模块使用)
    ├── error_helper.h       # 错误处理辅助
    ├── concurrent_helper.h  # 并发辅助
    └── object_helper.h      # 对象操作辅助
```

---

## 6. 代码阅读指南

### 6.1 阅读顺序建议

```
【新人学习路线】

第 1 步: 了解整体
    ├── bundle.json (组件配置)
    ├── ets_utils_config.gni (构建变量)
    └── 本文档 (代码地图)

第 2 步: 选择一个简单模块
    └── js_api_module/url/ (URL 解析相对简单)
        ├── native_module_url.cpp (入口)
        ├── js_url.h (接口定义)
        └── js_url.cpp (实现)

第 3 步: 理解 N-API 模式
    └── 观察 napi_module_register 注册
    └── 观察 napi_type_tag 类型检查
    └── 观察参数解析模式

第 4 步: 深入复杂模块
    └── js_concurrent_module/worker/ (复杂示例)
        ├── worker.h (类定义)
        ├── worker.cpp (核心实现)
        └── worker_runner.cpp (线程运行)
```

### 6.2 关键代码模式

#### N-API 模块注册模式

```cpp
// 位置: 各 native_module_*.cpp 文件

static napi_module g_moduleName = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = InitFunction,
    .nm_modname = "module_name",
    .nm_priv = nullptr,
    .reserved = { 0 },
};

extern "C" __attribute__((constructor)) void RegisterModule()
{
    napi_module_register(&g_moduleName);
}
```

#### 类型安全检查模式

```cpp
// 位置: js_api_module/url/js_url.cpp:30

static const napi_type_tag urlTypeTag = {
    0x9499581afe1b47fd,  // lower
    0xb456ff59fad2b512   // upper
};

// 检查类型
bool CheckType(napi_env env, napi_value obj) {
    bool result = false;
    napi_check_object_type_tag(env, obj, &urlTypeTag, &result);
    return result;
}
```

---

*文档版本: 1.0*  
*最后更新: 2026-02-07*
