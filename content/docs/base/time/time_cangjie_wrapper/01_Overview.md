# 01 - 项目概览

**目的**: 帮助新人在5分钟内理解项目定位、用途和快速上手  
**适用范围**: 仓颉应用开发者、技术评估人员  
**关键词**: time_cangjie_wrapper, SystemDateTime, FFI, OpenHarmony

---

## 一句话定义

**time_cangjie_wrapper** 是 OpenHarmony 上为仓颉(Cangjie)语言应用提供系统时间和时区访问能力的官方SDK封装层，通过FFI机制桥接底层 `time_service` 系统服务。

---

## 能力边界

### 本项目能做什么 ✅

| 功能 | 描述 | 适用场景 |
|------|------|----------|
| **获取系统时间** | 获取从Unix纪元(1970-01-01)到当前的时间戳 | 日志记录、数据排序、超时判断 |
| **获取运行时间** | 获取设备从启动至今的运行时长 | 性能统计、运行监控 |
| **获取系统时区** | 获取当前系统设置的时区标识符 | 时间显示、多时区转换 |

### 本项目不能做什么 ❌

| 功能 | 说明 | 替代方案 |
|------|------|----------|
| **设置系统时间** | 不提供修改系统时间的能力 | 使用系统设置应用 |
| **设置系统时区** | 不提供修改系统时区的能力 | 使用系统设置应用 |
| **定时器操作** | 不支持创建/启动/停止/销毁定时器 | 使用仓颉语言标准库定时器 |

### 与ArkTS API的能力对比

| 能力 | ArkTS API | 仓颉API(本项目) |
|------|-----------|-----------------|
| 获取系统时间 | ✅ | ✅ |
| 获取系统时区 | ✅ | ✅ |
| 设置系统时间 | ✅ | ❌ |
| 设置系统时区 | ✅ | ❌ |
| 定时器操作 | ✅ | ❌ |

**证据**: `README.md:51-58`

---

## 运行环境

### 系统要求

| 属性 | 要求 |
|------|------|
| **系统类型** | OpenHarmony Standard设备 |
| **API级别** | API 22+ |
| **系统能力** | SystemCapability.MiscServices.Time |

### 依赖组件

```
本项目(time_cangjie_wrapper)
├── cangjie_ark_interop      # 仓颉互操作框架
│   ├── ohos.business_exception  # 异常定义
│   ├── ohos.ffi                 # FFI基础设施
│   └── ohos.labels              # API注解
├── hiviewdfx_cangjie_wrapper # 日志服务
│   └── ohos.hilog               # 日志接口
└── time_service              # 底层时间服务
    └── cj_system_date_time_ffi  # C++ FFI实现
```

**证据**: `bundle.json:26-30`, `ohos/system_date_time/BUILD.gn:30-37`

---

## 快速开始

### 1. 添加依赖

在仓颉项目的 `cjpm.toml` 中添加依赖：

```toml
[dependencies]
ohos.system_date_time = "*"
```

### 2. 导入模块

```cangjie
import ohos.system_date_time.*
```

### 3. 使用API

#### 获取系统时间

```cangjie
// 获取毫秒级时间戳
let timeMs = SystemDateTime.getTime()
println("当前时间戳(毫秒): ${timeMs}")

// 获取纳秒级时间戳
let timeNs = SystemDateTime.getTime(isNanoseconds: true)
println("当前时间戳(纳秒): ${timeNs}")
```

**证据**: `ohos/system_date_time/system_date_time.cj:61-65`

#### 获取系统运行时间

```cangjie
// 获取系统启动至今的总时间（含深度睡眠）
let startupTime = SystemDateTime.getUptime(TimeType.Startup)
println("系统已运行(毫秒): ${startupTime}")

// 获取系统活跃时间（不含深度睡眠）
let activeTime = SystemDateTime.getUptime(TimeType.Active)
println("系统活跃时间(毫秒): ${activeTime}")
```

**证据**: `ohos/system_date_time/system_date_time.cj:78-82`

#### 获取系统时区

```cangjie
try {
    let timezone = SystemDateTime.getTimezone()
    println("系统时区: ${timezone}")
    // 输出示例: "Asia/Shanghai"
} catch (e: BusinessException) {
    println("获取时区失败: ${e.message}")
}
```

**证据**: `ohos/system_date_time/system_date_time.cj:95-101`

### 4. 错误处理

所有API都可能抛出 `BusinessException`：

```cangjie
import ohos.business_exception.BusinessException

try {
    let time = SystemDateTime.getTime()
} catch (e: BusinessException) {
    // 错误码 16000050: 内部错误
    println("错误码: ${e.code}, 消息: ${e.message}")
}
```

**证据**: `ohos/system_date_time/cj_date_time_error.cj:34-42`

---

## 项目特征

### 技术特征

| 特征 | 说明 |
|------|------|
| **编程语言** | Cangjie (仓颉) - 华为自研编程语言 |
| **互操作机制** | FFI (Foreign Function Interface) |
| **构建系统** | GN (Generate Ninja) |
| **架构定位** | Framework层封装 |
| **API稳定性** | Beta阶段 |

### 代码规模

| 指标 | 数值 |
|------|------|
| **核心源文件** | 3个 (.cj文件) |
| **核心代码行数** | ~200行 |
| **公共API数量** | 3个方法 + 1个枚举 |

### 资源占用

| 资源 | 占用 |
|------|------|
| **ROM** | 120KB |
| **RAM** | 88KB |

**证据**: `bundle.json:23-24`

---

## 架构定位

本项目在OpenHarmony架构中的位置：

```
┌─────────────────────────────────────────┐
│  应用层 (Application)                    │
│  - 仓颉应用代码                          │
│  - 调用 SystemDateTime.getTime()         │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  框架层 (Framework)                      │
│  - 本项目: time_cangjie_wrapper          │
│  - FFI封装层                             │
└──────────────┬──────────────────────────┘
               │ FFI调用
┌──────────────▼──────────────────────────┐
│  系统服务层 (System Service)             │
│  - time_service                          │
│  - 实际时间/时区管理                      │
└─────────────────────────────────────────┘
```

更多架构细节请参考 [02_Architecture.md](./02_Architecture.md)。

---

## 相关资源

### 官方文档

- [仓颉时间时区API参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-system_date_time.md)
- [仓颉时间时区开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/system_date_time/cj-system_data_time.md)

### 相关仓库

- [time_service](https://gitcode.com/openharmony/time_time_service) - 底层时间服务
- [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - 仓颉互操作框架
- [hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper) - 日志封装

### 内部Wiki

- [02_Architecture.md](./02_Architecture.md) - 架构分析
- [04_Interface.md](./04_Interface.md) - 接口详细文档
- [05_AttackSurface.md](./05_AttackSurface.md) - 攻击面分析

---

## 常见问题

### Q: 为什么不支持设置系统时间？

**A**: 设置系统时间是敏感操作，需要系统级权限。当前版本为Beta阶段，仅提供安全的读取操作。如需设置时间，请引导用户使用系统设置应用。

### Q: 与ArkTS的时间API有什么区别？

**A**: 功能上等价，都是访问底层time_service。区别在于编程语言：
- ArkTS API: 用于ArkTS/JS开发
- 仓颉API(本项目): 用于Cangjie开发

### Q: 获取的时间戳是本地时间还是UTC？

**A**: 获取的是Unix时间戳，基于UTC。如需本地时间显示，结合 `getTimezone()` 进行时区转换。

### Q: Beta阶段是否可用于生产？

**A**: Beta表示API可能在未来版本调整。建议关注版本更新说明，但当前功能已稳定可用。

---

## 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-07 | v1.0 | 初始版本，基于代码分析创建 |

---

**下一步**: 阅读 [02_Architecture.md](./02_Architecture.md) 了解FFI架构设计
