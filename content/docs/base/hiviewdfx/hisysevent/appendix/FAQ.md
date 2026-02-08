# 常见问题（FAQ）

## 概述

本文档收集了 HiSysEvent 使用过程中最常见的问题及其解决方案。按照问题类型分为：集成问题、构建问题、运行时问题、性能问题和安全相关问题。每个问题都提供了问题描述、可能原因、解决方案和预防措施。

---

## 第一部分：集成问题

### Q1：如何在应用中集成 HiSysEvent？

**问题描述**：开发者不清楚如何在应用中正确集成和使用 HiSysEvent。

**解决方案**：

**步骤 1：声明系统能力**

在应用的 `module.json` 或 `config.json` 中声明使用 HiSysEvent 能力：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "SystemCapability.HiviewDFX.HiSysEvent"
      }
    ]
  }
}
```

**步骤 2：添加编译依赖**

```gn
# BUILD.gn
external_deps = [ "hisysevent:libhisysevent" ]
```

**步骤 3：包含头文件并调用**

```cpp
#include "hisysevent.h"

// 写入事件
HiSysEvent::Write(
    HiSysEvent::Domain::APPEXECFWK,
    "app_start",
    HiSysEvent::EventType::BEHAVIOR,
    "app_name", appName.c_str()
);
```

**JavaScript/ArkTS 集成**：

```typescript
import hiSysEvent from '@ohos.hiSysEvent';

hiSysEvent.write({
    domain: 'APPEXECFWK',
    name: 'app_start',
    type: hiSysEvent.EventType.BEHAVIOR,
    params: {
        app_name: 'com.example.app'
    }
});
```

**预防措施**：确保应用签名正确，权限声明完整。

---

### Q2：如何选择合适的事件类型？

**问题描述**：开发者不确定应该使用哪种 EventType。

**解决方案**：

根据事件的性质选择合适的事件类型：

| 事件类型 | 使用场景 | 示例 |
|----------|----------|------|
| **FAULT** | 系统或应用故障 | 进程崩溃、ANR、异常终止 |
| **STATISTIC** | 性能统计指标 | CPU 使用率、内存占用、启动耗时 |
| **SECURITY** | 安全相关事件 | 认证失败、权限变更、敏感操作 |
| **BEHAVIOR** | 用户行为或系统行为 | 页面跳转、应用启动、功能使用 |

**代码示例**：

```cpp
// 故障事件 - 进程崩溃
HiSysEvent::Write(
    HiSysEvent::Domain::AAFWK,
    "process_crash",
    HiSysEvent::EventType::FAULT,
    "pid", pid,
    "reason", crashReason
);

// 统计事件 - 启动耗时
HiSysEvent::Write(
    HiSysEvent::Domain::APPEXECFWK,
    "app_cold_startup",
    HiSysEvent::EventType::STATISTIC,
    "duration_ms", duration,
    "app_name", appName
);

// 安全事件 - 认证失败
HiSysEvent::Write(
    HiSysEvent::Domain::SECURITY,
    "auth_failure",
    HiSysEvent::EventType::SECURITY,
    "user_id", userId,
    "reason", "password_error"
);

// 行为事件 - 应用启动
HiSysEvent::Write(
    HiSysEvent::Domain::APPEXECFWK,
    "app_start",
    HiSysEvent::EventType::BEHAVIOR,
    "app_name", appName,
    "entry_type", entryType
);
```

---

### Q3：如何定义自定义事件域？

**问题描述**：开发者需要使用预定义域之外的自定义域。

**解决方案**：

自定义域命名规则（`README.md:52`）：

| 约束 | 要求 |
|------|------|
| 最大长度 | 16 字符 |
| 字符集 | 数字（0-9）、大小写字母（a-z、A-Z）、下划线（_） |
| 首字符 | 必须为字母 |

**代码示例**：

```cpp
// 使用自定义域（推荐：以下划线开头区分系统域）
HiSysEvent::Write(
    "MY_CUSTOM_DOMAIN",  // 自定义域
    "custom_event",
    HiSysEvent::EventType::BEHAVIOR,
    "custom_param", "value"
);

// 预定义域与自定义域对比
// 预定义域：AAFWK, APPEXECFWK, BMS, etc.
// 自定义域：MY_APP, _CUSTOM_, etc.
```

**注意事项**：
1. 推荐使用下划线前缀区分自定义域和系统域
2. 避免与系统域命名冲突
3. 建议在文档中记录自定义域的使用规范

---

## 第二部分：构建问题

### Q4：构建时提示缺少 samgr 依赖

**错误信息**：

```
error: dependency 'samgr' not found
```

**问题原因**：
1. 子系统未正确配置
2. 构建环境未初始化
3. 产品配置未包含 hiviewdfx 子系统

**解决方案**：

```bash
# 步骤 1：初始化构建环境
source build/build.sh

# 步骤 2：设置构建目标
hb set

# 步骤 3：选择包含 HiSysEvent 的产品
# 选择如：HiHope_DAYU200

# 步骤 4：执行构建
hb build -f

# 或直接使用 GN 构建
gn gen out/ohos-arm64 --args="target_os=\"ohos\" target_cpu=\"arm64\""
ninja -C out/ohos-arm64 libhisysevent.z.so
```

**预防措施**：确保正确执行环境初始化脚本。

---

### Q5：头文件找不到错误

**错误信息**：

```
fatal error: 'hisysevent.h' file not found
```

**问题原因**：
1. 头文件路径未添加到 include_dirs
2. 依赖声明不完整
3. 构建缓存过期

**解决方案**：

```gn
# 方案 1：检查 BUILD.gn 配置
hisysevent_dep = "//base/hiviewdfx/hisysevent/interfaces/native/innerkits/hisysevent"

static_library("my_component") {
    sources = [ "src/*.cpp" ]
    
    include_dirs = [
        # 添加 HiSysEvent 头文件路径
        "//base/hiviewdfx/hisysevent/interfaces/native/innerkits/hisysevent/include"
    ]
    
    deps = [ hisysevent_dep ]
}
```

**清理并重新构建**：

```bash
# 清理构建缓存
rm -rf out/*

# 重新生成构建文件
gn gen out/ohos-arm64

# 执行构建
ninja -C out/ohos-arm64
```

---

### Q6：Rust 构建错误

**错误信息**：

```
error: could not find native support library for target `aarch64-unknown-linux-ohos`
```

**问题原因**：
1. Rust 目标平台未安装
2. 工具链配置错误
3. Cargo.toml 配置不完整

**解决方案**：

```bash
# 步骤 1：添加 OpenHarmony Rust 目标
rustup target add aarch64-unknown-linux-ohos

# 步骤 2：检查工具链配置
rustup show

# 步骤 3：更新依赖
cd interfaces/rust/innerkits
cargo update

# 步骤 4：重新构建
cargo build --target aarch64-unknown-linux-ohos
```

**验证目标**：

```bash
# 列出已安装的目标
rustup target list --installed

# 应输出：
# aarch64-unknown-linux-ohos
# x86_64-unknown-linux-ohos
```

---

## 第三部分：运行时问题

### Q7：HiSysEvent::Write 返回错误码 -1

**返回值**：`-1`（ERR_PERMISSION_DENIED）

**问题原因**：
1. 应用未声明 `SystemCapability.HiviewDFX.HiSysEvent` 能力
2. 应用签名不正确
3. 系统服务未启动

**解决方案**：

```cpp
#include "hisysevent.h"

int result = HiSysEvent::Write(
    HiSysEvent::Domain::APPEXECFWK,
    "test_event",
    HiSysEvent::EventType::BEHAVIOR,
    "key", "value"
);

if (result != 0) {
    // 错误处理
    switch (result) {
        case ERR_PERMISSION_DENIED:
            // 检查权限配置
            break;
        case ERR_SERVICE_UNAVAILABLE:
            // 服务未启动，稍后重试
            break;
        default:
            // 其他错误
            break;
    }
}
```

**检查权限配置**：

```json
// module.json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "SystemCapability.HiviewDFX.HiSysEvent"
      }
    ]
  }
}
```

---

### Q8：返回错误码 -2（超过速率限制）

**返回值**：`-2`（ERR_RATE_LIMITED）

**问题原因**：写入频率超过系统限制（默认每分钟 10000 次）

**解决方案**：

```cpp
// 方案 1：降低写入频率
void ThrottledWrite() {
    static constexpr uint64_t MIN_INTERVAL_MS = 10;  // 至少间隔 10ms
    static uint64_t lastWriteTime = 0;
    
    uint64_t now = GetCurrentTimeMs();
    if (now - lastWriteTime < MIN_INTERVAL_MS) {
        return;  // 跳过
    }
    
    HiSysEvent::Write(...);
    lastWriteTime = now;
}

// 方案 2：批量写入
void BatchWrite() {
    std::vector<HiSysEvent> events;
    
    // 收集事件
    events.push_back(CreateEvent1());
    events.push_back(CreateEvent2());
    
    // 批量发送（如果支持）
    HiSysEvent::BatchWrite(events);
}
```

**预防措施**：
1. 避免在高频循环中调用 Write
2. 使用批量写入接口
3. 实现客户端节流

---

### Q9：事件未显示在日志中

**问题描述**：调用 Write 成功，但无法在日志中找到对应事件。

**可能原因**：
1. 事件被过滤
2. 查询条件错误
3. 事件已过期被清理

**排查步骤**：

```bash
# 步骤 1：检查系统日志
hilog | grep HiSysEvent

# 步骤 2：使用 hilogtool 查看
hilogtool --view --pid=$(pgrep your_app)

# 步骤 3：确认事件域和名称正确
```

**代码排查**：

```cpp
// 添加调试日志
HiLog::Info(LABEL, "Writing event: %{public}s.%{public}s",
            domain.c_str(), eventName.c_str());

int result = HiSysEvent::Write(...);

HiLog::Info(LABEL, "Write result: %{public}d", result);
```

---

### Q10：N-API 调用崩溃

**问题描述**：JavaScript/ArkTS 调用 hiSysEvent.write() 导致应用崩溃。

**可能原因**：
1. 参数格式错误
2. 内存访问违规
3. 类型不匹配

**解决方案**：

```typescript
// 方案 1：添加参数验证
function safeWrite() {
    try {
        // 验证参数
        if (!domain || !name) {
            console.error('Invalid parameters');
            return;
        }
        
        hiSysEvent.write({
            domain: domain,
            name: name,
            type: hiSysEvent.EventType.BEHAVIOR,
            params: {
                // 验证所有参数值
                ...(params || {})
            }
        });
    } catch (e) {
        console.error('HiSysEvent error:', e);
    }
}

// 方案 2：使用 Promise 包装
async function writeEvent(options) {
    return new Promise((resolve, reject) => {
        try {
            const result = hiSysEvent.write(options);
            resolve(result);
        } catch (e) {
            reject(e);
        }
    });
}
```

---

## 第四部分：性能问题

### Q11：Write 调用延迟过高

**问题描述**：HiSysEvent::Write() 调用耗时过长，影响应用性能。

**优化建议**：

```cpp
// 优化 1：异步写入
class AsyncWriter {
public:
    void WriteAsync(const HiSysEvent& event) {
        // 提交到工作队列
        workQueue_.Push([event]() {
            event.Write();
        });
    }
};

// 优化 2：批量处理
void BatchWrite(const std::vector<HiSysEvent>& events) {
    for (const auto& event : events) {
        // 批量处理减少系统调用
    }
    HiSysEvent::BatchWrite(events);
}

// 优化 3：使用 Easy API（更轻量）
OH_HiSysEvent_WriteEasy(
    "DOMAIN", "event", 4,  // BEHAVIOR
    "key", "value"
);
```

**性能基准**（参考值）：

| API | 平均耗时 | P99 耗时 |
|-----|----------|----------|
| 同步 Write | ~100μs | ~500μs |
| 异步 Write | ~10μs | ~50μs |
| Easy Write | ~50μs | ~200μs |

---

### Q12：内存占用过高

**问题描述**：长时间运行后 HiSysEvent 占用内存持续增长。

**排查方法**：

```bash
# 检查内存占用
adb shell dumpsys meminfo | grep hisysevent

# 查看对象数量
adb shell cat /proc/$(pgrep hisysevent)/status | grep Threads
```

**解决方案**：

```cpp
// 确保正确释放资源
class EventWriter {
public:
    ~EventWriter() {
        // 清理未发送的事件
        FlushPendingEvents();
    }
    
    void FlushPendingEvents() {
        while (!pendingQueue_.Empty()) {
            auto event = pendingQueue_.Pop();
            event->Write();
            delete event;  // 及时释放
        }
    }
};
```

---

## 第五部分：安全问题

### Q13：如何防止事件数据泄露？

**问题描述**：担心敏感信息随事件写入被泄露。

**解决方案**：

```cpp
// 方案 1：使用脱敏处理
std::string MaskSensitiveData(const std::string& input) {
    if (input.length() <= 4) {
        return "****";
    }
    return input.substr(0, 2) + "****" + input.substr(input.length() - 2);
}

// 敏感信息脱敏
HiSysEvent::Write(
    HiSysEvent::Domain::SECURITY,
    "user_login",
    HiSysEvent::EventType::BEHAVIOR,
    "user_id", userId,
    "action", "login",
    "password", "****",  // 绝不应写入真实密码
    "error_code", errorCode
);

// 方案 2：使用 SECURITY 事件类型审计
HiSysEvent::Write(
    HiSysEvent::Domain::SECURITY,
    "sensitive_operation",
    HiSysEvent::EventType::SECURITY,
    "operation_type", "data_access",
    "resource_id", resourceId,
    "result", MaskSensitiveData(result)
);
```

**最佳实践**：
1. 绝不在事件参数中写入密码、密钥等敏感数据
2. 使用 SECURITY 类型记录安全相关事件
3. 实施数据脱敏策略

---

### Q14：如何验证事件来源？

**问题描述**：需要确认事件确实来自可信源。

**解决方案**：

```cpp
// 使用调用方信息验证
class EventValidator {
public:
    static bool IsFromTrustedSource() {
        uint32_t callerUid = GetCallingUid();
        uint32_t callerPid = GetCallingPid();
        
        // 检查 UID
        auto it = trustedUids_.find(callerUid);
        if (it == trustedUids_.end()) {
            return false;
        }
        
        return true;
    }
    
private:
    static std::set<uint32_t> trustedUids_;
};

std::set<uint32_t> EventValidator::trustedUids_ = {
    1000,  // system
    10000, // system service
};
```

---

## 第六部分：调试技巧

### Q15：如何启用调试日志？

**解决方案**：

```cpp
// 启用调试日志
#include "hilog/log.h"

static constexpr HiLogLabel LABEL = {
    LOG_CORE, 0xD002D00, "HiSysEvent"
};

void EnableDebugLogging() {
    // 通过环境变量控制
    // HI_SYS_EVENT_DEBUG=1
}

// 使用条件日志
#ifdef HISYSEVENT_DEBUG
HiLog::Debug(LABEL, "Debug info: %{public}s", debugInfo.c_str());
#endif
```

**命令行方式**：

```bash
# 设置日志级别
hilog -v debug

# 过滤 HiSysEvent 日志
hilog | grep -i "HiSysEvent\|hisysevent"
```

---

### Q16：如何使用命令行工具调试？

**解决方案**：

```bash
# 查看 HiSysEvent 服务状态
hisysevent_tool status

# 查询事件
hisysevent_tool query --domain APPEXECFWK --limit 100

# 监听实时事件
hisysevent_tool listen --domain AAFWK

# 导出事件到文件
hisysevent_tool export --output /data/log/events.txt
```

---

## 附录：错误码速查表

| 错误码 | 宏定义 | 说明 | 处理建议 |
|--------|--------|------|----------|
| 0 | ERR_OK | 成功 | 无 |
| -1 | ERR_INVALID_PARAM | 参数无效 | 检查参数格式 |
| -2 | ERR_PERMISSION_DENIED | 权限不足 | 检查 SysCap 声明 |
| -3 | ERR_WRITE_FAILED | 写入失败 | 检查 Socket 连接 |
| -4 | ERR_RATE_LIMITED | 速率限制 | 降低写入频率 |
| -5 | ERR_BUFFER_FULL | 缓冲区满 | 等待后重试 |
| -6 | ERR_SERVICE_UNAVAILABLE | 服务不可用 | 检查 SA 服务 |
| -7 | ERR_ENCODING_FAILED | 编码失败 | 检查参数值 |
| -8 | ERR_MEMORY_ALLOC | 内存分配失败 | 检查内存使用 |

---

*文档版本：1.0*
*创建时间：2026-02-07*
*最后更新：2026-02-07*
