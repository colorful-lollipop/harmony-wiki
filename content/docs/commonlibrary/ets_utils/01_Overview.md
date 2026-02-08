# 组件概览

> 了解 ets_utils 组件的定位、边界与核心能力

## 1. 组件定位

### 1.1 在 OpenHarmony 中的位置

```
OpenHarmony 系统
├── 应用框架层
│   ├── ArkTS 运行时
│   └── ets_utils ← 所在位置
├── 系统服务层
│   ├── 基础服务
│   └── 增强服务
└── 内核层
```

### 1.2 职责边界

**ets_utils 负责**：
- 提供 ArkTS 标准库扩展 API
- 实现 Node.js 兼容的工具类
- 封装系统级操作（进程、线程）
- 提供数据处理能力（XML、JSON、Buffer）

**ets_utils 不负责**：
- UI 渲染（由 ArkUI 负责）
- 硬件抽象（由 Device SDK 负责）
- 权限管理（由系统框架负责）

### 1.3 核心能力矩阵

| 能力类别 | 子模块 | 主要功能 |
|---------|--------|----------|
| **数据处理** | js_api_module | URL 解析、XML 序列化、Buffer 操作 |
| **容器存储** | js_util_module | List、Map、Set 等 15+ 容器 |
| **系统交互** | js_sys_module | 进程管理、定时器、控制台 |
| **并发编程** | js_concurrent_module | Worker 线程、Taskpool 任务池 |

## 2. 运行环境

### 2.1 系统要求

- **操作系统**: OpenHarmony Standard (标准系统)
- **运行时**: ArkTS 运行时 (ArkCompiler)
- **依赖**: N-API (Native API)

### 2.2 依赖关系

```
ets_utils 依赖:
├── ace_engine        # UI 框架基础
├── napi              # Native API 桥接
├── hilog            # 日志系统
├── ipc              # 进程间通信
├── samgr            # 系统能力管理
├── icu              # 国际化
├── libxml2          # XML 解析
├── openssl          # 加密库
└── ffrt             # 快任务运行时
```

### 2.3 SysCap 要求

```json
{
  "syscap": [
    "SystemCapability.Utils.Lang"
  ]
}
```

## 3. 关键概念

### 3.1 N-API 桥接机制

```
ArkTS 应用
    ↓ (JS 调用)
N-API 桥接层 (native_module_*.cpp)
    ↓ (Native 调用)
Native 实现 (C++)
    ↓ (系统调用)
OpenHarmony 系统服务
```

### 3.2 Type Tag 安全

项目使用 `napi_type_tag` 进行运行时类型检查：

```cpp
static const napi_type_tag bufferTypeTag = {
    0xaf0e0e7de1c249bc,  // lower
    0xb510ff1f3587c69f   // upper
};

// 使用示例
napi_unwrap_s(env, thisVar, &bufferTypeTag, reinterpret_cast<void**>(&buffer));
```

### 3.3 字节码编译链

```
.ts/.js 源文件
    ↓ (build_ts_js.py)
.js 中间文件
    ↓ (es2abc)
.abc 字节码
    ↓ (打包)
.hap 应用包
```

## 4. 模块职责

### 4.1 js_api_module - 数据处理

| 子模块 | 职责 | 关键类 |
|--------|------|--------|
| url | URL 解析构造 | URL, URLSearchParams |
| uri | URI 标准化 | Uri |
| xml | XML 解析序列化 | XmlSerializer, XmlPullParser |
| buffer | 二进制缓冲区 | Buffer, Blob |
| convertxml | XML 转换 | ConvertXml |

### 4.2 js_util_module - 工具集

| 子模块 | 职责 | 示例 |
|--------|------|------|
| container | 容器类 | ArrayList, HashMap, TreeSet |
| util | 工具类 | TextEncoder, LruBuffer, Scope |
| json | JSON 处理 | JSON.parse, JSON.stringify |
| stream | 流处理 | ReadableStream, WritableStream |

### 4.3 js_sys_module - 系统交互

| 子模块 | 职责 | 关键 API |
|--------|------|----------|
| process | 进程管理 | pid, kill(), runCmd() |
| timer | 定时器 | setTimeout, setInterval |
| console | 控制台输出 | console.log, console.error |
| dfx | 调试工具 | hiTraceMeter |

### 4.4 js_concurrent_module - 并发编程

| 子模块 | 职责 | 关键类 |
|--------|------|--------|
| worker | 多线程 | Worker, parentPort |
| taskpool | 任务池 | Taskpool, Task |
| utils | 并发工具 | AsyncLock, ConditionVariable |

## 5. 特性开关

### 5.1 编译开关

| 开关 | 说明 | 默认值 |
|------|------|--------|
| `ets_utils_stacksize_low_enable` | 低功耗栈大小模式 | false |

### 5.2 安全特性

- **CFI (Control Flow Integrity)**: 控制流完整性检查
- **PAC (Pointer Authentication)**: 返回地址指针认证

## 6. 相关文档

- [README.md](./README.md) - 项目概述
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [03_API_js_api_module.md](./03_API_js_api_module.md) - API 参考

---

*文档版本: 1.0*
*最后更新: 2026-02-06*
