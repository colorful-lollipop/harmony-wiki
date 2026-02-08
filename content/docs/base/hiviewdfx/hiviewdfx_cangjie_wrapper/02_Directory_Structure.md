# 目录结构与模块职责

## 整体目录结构

```
base/hiviewdfx/hiviewdfx_cangjie_wrapper/
├── figures/                          # 文档资源
│   └── hiviewdfx_cangjie_wrapper_architecture*.png  # 架构图
├── kit/                              # Kit 化代码（对外接口）
│   └── PerformanceAnalysisKit/       # Performance Analysis Kit
│       ├── index.cj                  # 包入口，重导出所有模块
│       └── BUILD.gn                  # Kit 构建配置
├── ohos/                             # Ohos 模块代码
│   ├── hilog/                        # HiLog 模块
│   │   ├── hilog.cj                  # HiLog 实现
│   │   ├── hilog_channel.cj          # 渠道相关实现
│   │   └── BUILD.gn                  # 构建配置
│   ├── hi_trace_meter/               # HiTraceMeter 模块
│   │   ├── hi_trace_meter.cj         # 性能打点实现
│   │   └── BUILD.gn                  # 构建配置
│   └── hiviewdfx/                    # DFX 模块
│       └── hi_app_event/             # HiAppEvent 模块
│           ├── app_event_package_holder.cj  # 事件包持有者
│           ├── cj_event.cj           # 事件类型定义
│           ├── cj_event_ffi.cj       # FFI 绑定
│           ├── cj_hiappevent_log.cj  # 日志辅助
│           ├── hi_app_event.cj       # HiAppEvent 主类
│           ├── utils.cj              # 工具函数
│           └── BUILD.gn              # 构建配置
├── mock/                             # Mock 代码（跨平台支持）
│   ├── ohos.hilog.cj
│   ├── ohos.hi_trace_meter.cj
│   ├── ohos.hiviewdfx.cj
│   └── ohos.hiviewdfx.hi_app_event.cj
├── test/                             # 测试代码（本文档不涉及）
├── BUILD.gn                          # 根构建入口
├── bundle.json                       # 组件配置
├── README.md                         # 项目说明（英文）
└── README_zh.md                      # 项目说明（中文）
```

**数据来源**: `README.md:37-51` 及代码目录探索

---

## 模块职责

### kit/PerformanceAnalysisKit

**职责**: 提供统一的 Kit 入口，聚合所有 DFX 能力。

**代码位置**: `kit/PerformanceAnalysisKit/index.cj:18-22`

**功能**:
- 重导出 `ohos.hi_trace_meter.*`
- 重导出 `ohos.hiviewdfx.hi_app_event.*`
- 重导出 `ohos.hilog.*`

**依赖**:
- `ohos.hi_trace_meter`
- `ohos.hilog`
- `ohos.hiviewdfx.hi_app_event`

**数据来源**: `kit/PerformanceAnalysisKit/BUILD.gn:22-26`

---

### ohos/hilog

**职责**: 提供 Cangjie 语言的 HiLog 日志能力。

**代码位置**: `ohos/hilog/`

**文件说明**:

| 文件 | 职责 | 代码行数 |
|------|------|---------|
| `hilog.cj` | Hilog 类和 LogLevel 枚举实现 | 340 行 |
| `hilog_channel.cj` | 日志渠道相关功能 | - |
| `BUILD.gn` | 构建配置 | 40 行 |

**核心类型**:

| 类型 | 职责 | 位置 |
|------|------|------|
| `LogLevel` | 日志级别枚举 | `hilog.cj:36-93` |
| `Hilog` | 日志操作主类 | `hilog.cj:104-276` |

**外部依赖**:
- `hilog:libhilog` (Native 库)
- `cangjie_ark_interop:ohos.business_exception`
- `cangjie_ark_interop:ohos.labels`

**数据来源**: `ohos/hilog/BUILD.gn:30-35`

---

### ohos/hi_trace_meter

**职责**: 提供 Cangjie 语言的 HiTraceMeter 性能追踪能力。

**代码位置**: `ohos/hi_trace_meter/`

**文件说明**:

| 文件 | 职责 | 代码行数 |
|------|------|---------|
| `hi_trace_meter.cj` | HiTraceMeter 类实现 | 85 行 |
| `BUILD.gn` | 构建配置 | 39 行 |

**核心类型**:

| 类型 | 职责 | 位置 |
|------|------|------|
| `HiTraceMeter` | 性能打点主类 | `hi_trace_meter.cj:39-84` |

**外部依赖**:
- `hitrace:cj_hitracemeter_ffi` (FFI 库)
- `cangjie_ark_interop:ohos.ffi`
- `cangjie_ark_interop:ohos.labels`

**内部依赖**:
- `ohos.hilog` (日志支持)

**数据来源**: `ohos/hi_trace_meter/BUILD.gn:27-34`

---

### ohos/hiviewdfx/hi_app_event

**职责**: 提供 Cangjie 语言的 HiAppEvent 事件打点和订阅能力。

**代码位置**: `ohos/hiviewdfx/hi_app_event/`

**文件说明**:

| 文件 | 职责 | 代码行数 |
|------|------|---------|
| `hi_app_event.cj` | HiAppEvent 主类 | 340 行 |
| `cj_event.cj` | 事件类型定义 | 1190 行 |
| `app_event_package_holder.cj` | 事件包持有者 | 148 行 |
| `cj_event_ffi.cj` | FFI 绑定 | - |
| `cj_hiappevent_log.cj` | 日志辅助 | - |
| `utils.cj` | 工具函数 | - |
| `BUILD.gn` | 构建配置 | 47 行 |

**核心类型**:

| 类型 | 职责 | 位置 |
|------|------|------|
| `HiAppEvent` | 事件操作主类 | `hi_app_event.cj:55-339` |
| `EventType` | 事件类型枚举 | `cj_event.cj:36-101` |
| `EventValueType` | 事件值类型枚举 | `cj_event.cj:110-228` |
| `Domain` | 事件域常量 | `cj_event.cj:237-246` |
| `Event` | 事件名称常量 | `cj_event.cj:258-299` |
| `Param` | 参数名称常量 | `cj_event.cj:308-333` |
| `AppEventInfo` | 事件信息类 | `cj_event.cj:357-461` |
| `ConfigOption` | 配置选项类 | `cj_event.cj:476-522` |
| `Processor` | 数据处理器类 | `cj_event.cj:587-735` |
| `Watcher` | 事件观察者类 | `cj_event.cj:950-1029` |
| `AppEventPackageHolder` | 事件包持有者 | `app_event_package_holder.cj:40-147` |

**外部依赖**:
- `hiappevent:cj_hiappevent_ffi` (FFI 库)
- `cangjie_ark_interop:ohos.business_exception`
- `cangjie_ark_interop:ohos.ffi`
- `cangjie_ark_interop:ohos.labels`

**内部依赖**:
- `ohos.hilog` (日志支持)

**数据来源**: `ohos/hiviewdfx/hi_app_event/BUILD.gn:34-42`

---

## 模块依赖关系图

```
kit.PerformanceAnalysisKit
│
├── ohos.hi_trace_meter
│   └── ohos.hilog (间接依赖)
│
├── ohos.hilog
│   └── cangjie_ark_interop (business_exception, labels)
│       └── hilog:libhilog (Native)
│
└── ohos.hiviewdfx.hi_app_event
    ├── ohos.hilog (日志支持)
    └── cangjie_ark_interop (business_exception, ffi, labels)
        └── hiappevent:cj_hiappevent_ffi (Native)

hitrace:cj_hitracemeter_ffi (Native)
```

---

## 跨平台支持

项目支持 Windows 和 macOS 平台的开发测试，使用 mock 代码：

**数据来源**: `ohos/hilog/BUILD.gn:21-28`

```gn
if (is_mingw || is_mac) {
  sources = [ "../../mock/ohos.hilog.cj" ]
} else {
  sources = [ "hilog.cj", "hilog_channel.cj" ]
}
```

Mock 模块列表:
- `mock/ohos.hilog.cj`
- `mock/ohos.hi_trace_meter.cj`
- `mock/ohos.hiviewdfx.cj`
- `mock/ohos.hiviewdfx.hi_app_event.cj`

---

## 不包含的目录

以下目录**不**在本 Wiki 文档的范围内：

| 目录 | 说明 | 排除原因 |
|------|------|---------|
| `test/` | 测试用例代码 | 测试内容不涉及业务逻辑 |
| `figures/` | 架构图资源 | 文档资源，非代码 |

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目概览 |
| [03_Architecture.md](03_Architecture.md) | 架构说明 |
| [04_External_API.md](04_External_API.md) | API 参考 |
| [06_GN_Build.md](06_GN_Build.md) | 构建系统 |
