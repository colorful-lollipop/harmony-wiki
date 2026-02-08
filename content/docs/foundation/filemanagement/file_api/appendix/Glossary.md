# 附录 C: 术语表

## 架构术语

| 术语 | 英文 | 说明 |
|------|------|------|
| N-API | Node-API | Node.js 的 Native API，用于 C/C++ 扩展 |
| ANI | ArkTS Native Interface | ArkTS 的新一代 Native 调用接口 |
| FFI | Foreign Function Interface | 外部函数接口，用于跨语言调用 |
| LibN | Library for N-API | File API 项目的 N-API 封装框架 |
| LibFS | Library for FS | 文件系统工具库 |
| LibHilog | Library for HiLog | 日志工具库 |
| GN | Generate Ninja | 元构建系统，生成 Ninja 构建文件 |
| syscap | System Capability | 系统能力声明 |

## 文件系统术语

| 术语 | 英文 | 说明 |
|------|------|------|
| FD | File Descriptor | 文件描述符，内核中标识打开文件的整数 |
| URI | Uniform Resource Identifier | 统一资源标识符 |
| 沙箱 | Sandbox | 应用隔离的文件访问环境 |
| 符号链接 | Symbolic Link | 指向另一个文件的特殊文件类型 |
| 硬链接 | Hard Link | 指向相同 inode 的多个目录项 |
| inotify | inode notify | Linux 文件系统事件监控机制 |
| io_uring | I/O Uring | Linux 5.1+ 的高性能异步 IO 接口 |
| CQE | Completion Queue Event | io_uring 完成队列事件 |
| SQE | Submission Queue Event | io_uring 提交队列事件 |

## 编程模型术语

| 术语 | 英文 | 说明 |
|------|------|------|
| Promise | Promise | 异步编程模式，代表未来完成或失败的操作 |
| Callback | Callback | 回调函数，异步操作完成后调用 |
| Sync | Synchronous | 同步，阻塞直到操作完成 |
| Async | Asynchronous | 异步，非阻塞，通过回调或 Promise 返回结果 |
| RAII | Resource Acquisition Is Initialization | 资源获取即初始化，C++ 资源管理惯用法 |
| PIMPL | Pointer to Implementation | 指向实现的指针，隐藏实现细节 |
| FFI | Foreign Function Interface | 跨语言调用接口 |
| ABI | Application Binary Interface | 应用二进制接口 |

## OpenHarmony 术语

| 术语 | 英文 | 说明 |
|------|------|------|
| OHOS | OpenHarmony Operating System | 开放鸿蒙操作系统 |
| ArkTS | Ark TypeScript | OpenHarmony 的 TypeScript 超集 |
| HAP | HarmonyOS Ability Package | 应用包格式 |
| Syscap | System Capability | 系统能力 |
| AccessToken | Access Token | 访问令牌，权限管理 |
| BundleName | Bundle Name | 应用包名 |
| UID | User ID | 用户标识 |
| PID | Process ID | 进程标识 |
| SA | System Ability | 系统能力服务 |
| IPC | Inter-Process Communication | 进程间通信 |

## 安全术语

| 术语 | 英文 | 说明 |
|------|------|------|
| 路径遍历 | Path Traversal | 通过 `../` 访问父目录的攻击 |
| TOCTOU | Time-of-check to Time-of-use | 检查时与使用时状态不一致的竞态条件 |
| CFI | Control Flow Integrity | 控制流完整性保护 |
| PAC | Pointer Authentication Code | 指针认证码，ARM64 安全特性 |
| UBSan | Undefined Behavior Sanitizer | 未定义行为检测器 |
| ASan | Address Sanitizer | 地址检测器 |
| 沙箱逃逸 | Sandbox Escape | 突破应用沙箱限制 |

## 性能术语

| 术语 | 英文 | 说明 |
|------|------|------|
| 零拷贝 | Zero Copy | 避免数据在内核和用户空间间复制 |
| 异步 IO | Async I/O | 非阻塞 IO 操作 |
| 线程池 | Thread Pool | 复用线程执行异步任务 |
| 事件循环 | Event Loop | 处理异步事件的循环机制 |
| 防抖 | Debounce | 减少频繁触发 |
| 节流 | Throttle | 限制执行频率 |

## 代码结构术语

| 术语 | 英文 | 说明 |
|------|------|------|
| Entity | Entity | 实体类，封装 Native 资源 |
| Exporter | Exporter | 导出器，将 C++ 类/函数导出给 JS |
| Module | Module | 模块，一组相关功能的集合 |
| Property | Property | 属性，JS 对象的特性 |
| Method | Method | 方法，JS 对象可调用的函数 |
| Static | Static | 静态，属于类而非实例 |
| Instance | Instance | 实例，类的具体对象 |
| Constructor | Constructor | 构造函数，创建对象时调用 |
| Finalizer | Finalizer | 终结器，对象销毁时调用 |

## 缩写对照

| 缩写 | 全称 | 说明 |
|------|------|------|
| JS | JavaScript | 脚本语言 |
| TS | TypeScript | JavaScript 的超集 |
| CC | Cangjie | 仓颉编程语言 |
| RS | Rust | Rust 编程语言 |
| NDK | Native Development Kit | 原生开发套件 |
| API | Application Programming Interface | 应用编程接口 |
| SDK | Software Development Kit | 软件开发工具包 |
| SO | Shared Object | 共享库文件 |
| ABC | Ark Bytecode | Ark 字节码 |
| DSO | Dynamic Shared Object | 动态共享对象 |

## 目录结构术语

| 路径 | 说明 |
|------|------|
| `interfaces/kits/` | 对外接口实现 |
| `interfaces/kits/js/src/` | N-API / ANI 实现 |
| `interfaces/kits/native/` | Native C++ 接口 |
| `interfaces/kits/c/` | C NDK 接口 |
| `interfaces/kits/cj/` | Cangjie FFI |
| `interfaces/kits/rust/` | Rust FFI |
| `utils/` | 工具库 |
| `utils/filemgmt_libn/` | LibN 框架 |
| `utils/filemgmt_libfs/` | LibFS 工具库 |
| `utils/filemgmt_libhilog/` | 日志工具库 |

## 构建术语

| 术语 | 英文 | 说明 |
|------|------|------|
| Target | Target | 构建目标 |
| GN | Generate Ninja | 元构建系统 |
| Ninja | Ninja | 构建工具 |
| GNI | GN Include | GN 包含文件 |
| Dep | Dependency | 依赖 |
| Public Dep | Public Dependency | 公共依赖 |
| Config | Configuration | 配置 |
| Define | Preprocessor Define | 预处理器定义 |

## 参考

如需了解更多术语，请参考：
- [OpenHarmony 术语表](https://gitee.com/openharmony/docs)
- [Linux 文件系统术语](https://man7.org/linux/man-pages/)
- [N-API 文档](https://nodejs.org/api/n-api.html)
