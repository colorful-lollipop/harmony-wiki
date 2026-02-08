# 安全风险评审

## 概述

本章节对 HiSysEvent 组件进行安全风险分析，识别潜在的 attack surface、trust boundary 和可被利用点，并提供修复建议。

## 攻击面清单

| 攻击面 | 类型 | 入口 | 受影响模块 |
|--------|------|------|-----------|
| N-API 接口 | 网络/本地 | JS 调用 | hisysevent_napi |
| C++ API | 本地 | 链接库 | libhisysevent |
| IPC 通信 | 进程间 | socket | libhisyseventmanager |
| 参数输入 | 数据 | API 参数 | libhisysevent |
| 事件订阅 | 回调 | 观察者模式 | libhisyseventmanager |

## 信任边界

```
┌─────────────────────────────────────────────────────────┐
│                     用户空间                            │
│  ┌─────────────────────────────────────────────────┐  │
│  │              受信任边界 (Trust Boundary)          │  │
│  │  ┌─────────────┐  ┌─────────────┐              │  │
│  │  │  JS/ArkTS   │  │  C++ 应用   │              │  │
│  │  │  (系统应用)  │  │  (特权进程) │              │  │
│  │  └──────┬──────┘  └──────┬──────┘              │  │
│  │         │                │                       │  │
│  │         ▼                ▼                       │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  libhisysevent_napi / libhisysevent        │  │  │
│  │  └────────────────┬────────────────────────────┘  │  │
│  │                   │ IPC                           │  │
│  │                   ▼                               │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  libhisyseventmanager → IPC → SysEventImpl  │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────┘  │
│                         │                             │
│                         ▼                             │
│              ┌────────────────────┐                     │
│              │   HDF / 内核驱动   │  (高特权)          │
│              └────────────────────┘                     │
└─────────────────────────────────────────────────────────┘
```

## 可被利用点分析

### 1. 高频写入攻击

| 项目 | 详情 |
|------|------|
| **风险等级** | 中 |
| **证据位置** | `hisysevent.h:267-276`, `write_controller.h` |
| **触发条件** | 短时间内大量调用 Write() |
| **影响** | 可能导致系统资源耗尽（CPU/内存/网络） |

**代码证据** (`hisysevent.h:267-276`):
```cpp
ControlParam param = {
#ifdef HISYSEVENT_PERIOD
    HISYSEVENT_PERIOD,
#else
    HISYSEVENT_DEFAULT_PERIOD,
#endif
#ifdef HISYSEVENT_THRESHOLD
    HISYSEVENT_THRESHOLD
#else
    HISYSEVENT_DEFAULT_THRESHOLD
#endif
};
uint64_t timeStamp = WriteController::CheckLimitWritingEvent(param, ...);
if (timeStamp == INVALID_TIME_STAMP) {
    return ERR_WRITE_IN_HIGH_FREQ;  // 高频限制返回 -6
}
```

**当前缓解措施**:
- ✅ 使用 `WriteController::CheckLimitWritingEvent()` 限制写入频率
- ✅ 返回 `ERR_WRITE_IN_HIGH_FREQ` 错误码

**修复建议**:
- ⚠️ 建议在服务端也增加速率限制
- ⚠️ 建议增加写入配额管理
- ⚠️ 建议对非特权进程增加更严格的限制

---

### 2. 大数据写入攻击

| 项目 | 详情 |
|------|------|
| **风险等级** | 中 |
| **证据位置** | `def.h:57-63`, `hisysevent.h:538-543` |
| **触发条件** | 单次写入超大数据（字符串 > 256KB，数组 > 100 项） |
| **影响** | 内存占用过大，可能导致 OOM |

**代码证据** (`def.h:57-63`):
```cpp
static constexpr unsigned int MAX_STRING_LENGTH = 256 * 1024;  // 256KB
static constexpr unsigned int MAX_ARRAY_SIZE = 100;
static constexpr unsigned int MAX_PARAM_NUMBER = 128;
static constexpr unsigned int MAX_DATA_SIZE = 384 * 1024;  // 384KB
```

**当前缓解措施**:
- ✅ 参数校验：`CheckValue()` 检查字符串长度
- ✅ 参数校验：`CheckArraySize()` 检查数组大小
- ✅ 参数校验：`UpdateAndCheckKeyNumIsOver()` 检查参数数量

**修复建议**:
- ⚠️ 建议在服务端也进行相同的数据大小校验
- ⚠️ 建议对单次写入设置更严格的全局上限
- ⚠️ 建议考虑流式写入方式

---

### 3. 路径遍历风险（导出功能）

| 项目 | 详情 |
|------|------|
| **风险等级** | 中 |
| **证据位置** | `napi_hisysevent_querier.cpp`, `hisysevent_manager.cpp` |
| **触发条件** | 导出路径被恶意构造 |
| **影响** | 可能写入任意位置，覆盖系统文件 |

**当前缓解措施**:
- ⚠️ **未发现明确的路径校验逻辑**（需要进一步确认）

**修复建议**:
- ✅ 导出路径应限制在指定目录（如 `/data/log/`）
- ✅ 使用 `realpath()` 规范化路径
- ✅ 检查路径前缀是否在白名单内
- ✅ 使用 `O_CREAT | O_EXCL` 避免覆盖

---

### 4. 整数溢出风险

| 项目 | 详情 |
|------|------|
| **风险等级** | 低 |
| **证据位置** | `hisysevent_c.h:87` (`size_t arraySize`) |
| **触发条件** | 参数中 `arraySize` 被构造为恶意值 |
| **影响** | 可能的内存访问越界 |

**代码证据** (`hisysevent_c.h:83-89`):
```cpp
struct HiSysEventParam {
    char name[MAX_LENGTH_OF_PARAM_NAME];
    HiSysEventParamType t;
    HiSysEventParamValue v;
    size_t arraySize;  // 无符号整数，但需防溢出
};
```

**当前缓解措施**:
- ⚠️ 未发现对 `arraySize` 的上界校验

**修复建议**:
- ✅ 在使用 `arraySize` 前进行范围校验
- ✅ 考虑使用 `SafeInt` 或手动溢出检查

---

### 5. 回调函数拒绝服务

| 项目 | 详情 |
|------|------|
| **风险等级** | 中 |
| **证据位置** | `napi_hisysevent_listener.cpp`, `js_callback_manager.cpp` |
| **触发条件** | 回调函数执行时间过长或抛出异常 |
| **影响** | JS 线程阻塞，事件处理延迟 |

**代码证据** (napi_hisysevent_listener.cpp):
```cpp
// 监听器回调
void NapiHiSysEventListener::OnEvent(...)
{
    napi_call_function(env_, context_->ref, ...);  // 回调 JS 函数
}
```

**当前缓解措施**:
- ⚠️ 未发现超时机制或异常处理

**修复建议**:
- ✅ 建议对回调函数设置超时时间
- ✅ 建议在 `napi_call_function` 外层包装 try-catch
- ✅ 考虑使用 `napi_create_async_work` 将回调移到工作队列

---

### 6. 权限提升风险

| 项目 | 详情 |
|------|------|
| **风险等级** | 低 |
| **证据位置** | `napi_hisysevent_js.cpp:70-73` |
| **触发条件** | 非系统应用尝试调用 N-API |
| **影响** | 权限拒绝（当前已有检查） |

**代码证据** (`napi_hisysevent_js.cpp:70-73`):
```cpp
if (!NapiHiSysEventUtil::IsSystemAppCall()) {
    NapiHiSysEventUtil::ThrowSystemAppPermissionError(env);
    return nullptr;
}
```

**当前缓解措施**:
- ✅ 所有 N-API 函数入口都有系统应用权限检查
- ✅ 权限错误会抛出 `ThrowSystemAppPermissionError`

**修复建议**:
- ✅ 建议考虑增加更细粒度的权限控制（如按 domain 授权）
- ⚠️ C++ 接口目前无权限检查，仅限内部子系统使用

---

### 7. 敏感信息泄露

| 项目 | 详情 |
|------|------|
| **风险等级** | 低 |
| **证据位置** | `hisysevent.cpp`, `raw_data_encoder.cpp` |
| **触发条件** | 事件参数包含敏感信息 |
| **影响** | 敏感数据写入日志，可能泄露 |

**当前缓解措施**:
- ⚠️ **未发现敏感信息过滤机制**

**修复建议**:
- ⚠️ 建议提供敏感字段自动过滤功能
- ⚠️ 建议在文档中明确标注哪些信息不应写入
- ⚠️ 考虑与安全模块集成，自动检测敏感数据

---

## 风险等级汇总

| 风险点 | 等级 | 已有缓解 | 建议措施 |
|--------|------|----------|----------|
| 高频写入攻击 | 中 | ✅ 频率控制 | 服务端限速、配额管理 |
| 大数据写入攻击 | 中 | ✅ 参数校验 | 服务端校验、全局上限 |
| 路径遍历风险 | 中 | ⚠️ 未确认 | 路径白名单、realpath |
| 整数溢出风险 | 低 | ⚠️ 未确认 | 范围校验 |
| 回调拒绝服务 | 中 | ⚠️ 未确认 | 超时、异常处理 |
| 权限提升风险 | 低 | ✅ 权限检查 | 细粒度控制 |
| 敏感信息泄露 | 低 | ⚠️ 未确认 | 自动过滤 |

## 安全最佳实践

### 对开发者的建议

1. **避免写入敏感数据**: 不要在事件参数中写入密码、Token、个人隐私信息
2. **控制写入频率**: 避免在高频路径（如渲染循环）中调用 Write()
3. **使用编译期 domain**: 推荐使用 `HiSysEventWrite(domain, ...)` 宏，支持域屏蔽
4. **处理错误码**: 总是检查返回值，处理错误情况

### 对系统集成者的建议

1. **限制导出目录**: 配置文件导出路径限制在安全目录
2. **监控异常**: 监控高频错误码（如 -6 高频写入）
3. **定期审计**: 定期检查事件参数是否包含敏感信息
4. **更新规则**: 根据实际使用情况调整频率限制参数

## 相关链接

- [概览](00_Overview.md) - 项目整体介绍
- [架构说明](01_Architecture.md) - 内部架构
- [N-API 参考](02_N-API.md) - JS/ArkTS 接口
- [C++ API 参考](03_CPP_API.md) - Native 接口
- [故障排查](06_Troubleshooting.md) - 常见问题
