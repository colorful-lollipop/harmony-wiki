# 故障排查

## 常见问题与解决方案

### Q1: 事件写入返回 -6 (ERR_WRITE_IN_HIGH_FREQ)

**现象**:
```cpp
int ret = HiSysEventWrite(HiSysEvent::Domain::AAFWK, "test", HiSysEvent::EventType::BEHAVIOR);
// ret == -6
```

**原因**: 写入频率过高，触发频率限制。

**解决方案**:
1. 检查调用频率，避免在高频路径（如循环）中调用
2. 考虑合并多个事件为一次调用
3. 如需更高频率，参考 `HISYSEVENT_PERIOD` 和 `HISYSEVENT_THRESHOLD` 配置

**相关代码**:
- `hisysevent.h:267-276` - 频率控制逻辑
- `write_controller.h` - WriteController 类

---

### Q2: 事件写入返回 -7 (ERR_DOMAIN_MASKED)

**现象**:
```cpp
int ret = HiSysEventWrite(HiSysEvent::Domain::AAFWK, "test", HiSysEvent::EventType::BEHAVIOR);
// ret == -7
```

**原因**: 该 domain 被 `DOMAIN_MASKS` 宏屏蔽。

**解决方案**:
1. 检查代码中是否定义了 `DOMAIN_MASKS` 宏
2. 检查 BUILD.gn 中的编译标志
3. 使用未屏蔽的 domain 或自定义 domain

**相关代码**:
- `hisysevent.h:38-46` - DOMAIN_MASKS 定义
- `hisysevent.h:309-314` - 屏蔽域处理

---

### Q3: 事件写入返回 -1 (ERR_DOMAIN_NAME_INVALID)

**现象**:
```cpp
int ret = HiSysEvent::Write(__FUNCTION__, __LINE__, "Invalid_Domain!", "test", HiSysEvent::EventType::BEHAVIOR);
// ret == -1
```

**原因**: domain 名称格式无效。

**解决方案**:
1. Domain 只能包含 `0-9`, `a-z`, `A-Z`, `_`
2. 必须以字母开头
3. 长度不超过 16 字符
4. 使用预定义 domain 或遵循命名规范

**验证代码** (`def.h`):
```cpp
static constexpr unsigned int MAX_DOMAIN_LENGTH = 16;
```

---

### Q4: JS 调用报权限错误 (201)

**现象**:
```typescript
hiSysEvent.write({...})
// 抛出错误: Permission denied
```

**原因**: N-API 仅限系统应用使用。

**解决方案**:
1. 确认应用是否具有系统应用权限
2. 普通应用无法使用 HiSysEvent N-API
3. 如需普通应用使用，考虑通过系统服务间接调用

**相关代码**:
- `napi_hisysevent_js.cpp:70-73` - 权限检查

---

### Q5: 事件参数丢失

**现象**: 部分参数未被写入，查看日志发现警告。

**原因**: 参数校验失败，参数被忽略。

**常见原因**:
1. 参数名无效（不以字母开头，包含非法字符）
2. 参数名超过 48 字符
3. 参数值字符串超过 256KB
4. 数组超过 100 项
5. 总参数超过 128 个

**解决方案**:
检查参数是否符合限制。

**相关代码**:
- `def.h:44-55` - 参数错误码
- `hisysevent.h:367-376` - 参数校验逻辑

---

### Q6: 观察者收不到事件

**现象**: 添加 watcher 后，事件未触发回调。

**可能原因**:
1. 规则匹配条件过于严格
2. 事件未被正确发送
3. 回调函数未正确定义

**解决方案**:
1. 检查 `ListenerRule` 配置
2. 使用 `WHOLE_WORD` 全词匹配测试
3. 先写入简单事件验证流程

**示例**:
```typescript
// 使用最简单的规则测试
hiSysEvent.addWatcher({
    rules: [{
        domain: 'HIVIEWDFX',  // 确保有事件触发
        ruleType: hiSysEvent.RuleType.WHOLE_WORD
    }],
    onEvent: (event) => {
        console.info('Received:', event.domain, event.name);
    }
});

// 手动触发一个事件
hiSysEvent.write({
    domain: 'HIVIEWDFX',
    name: 'test_event',
    type: hiSysEvent.EventType.BEHAVIOR
});
```

---

### Q7: 导出文件路径错误

**现象**: `exportSysEvents` 返回空或错误路径。

**可能原因**:
1. 路径权限问题
2. 磁盘空间不足
3. 查询参数无效

**解决方案**:
1. 检查导出目录权限
2. 确认存储空间充足
3. 检查 `QueryArg` 参数是否正确

---

## 日志定位

### HiSysEvent 日志标签

| 模块 | LOG_TAG | LOG_DOMAIN |
|------|---------|------------|
| N-API | `NAPI_HISYSEVENT_JS` | `0xD002D08` |
| Core | `HISYSEVENT_CORE` | 见代码 |
| Transport | `HISYSEVENT_TRANS` | 见代码 |

### 查看日志

```bash
# 过滤 HiSysEvent 相关日志
hilog | grep -E "HISYSEVENT|NAPI_HISYSEVENT"

# 查看 N-API 日志
hilog | grep "NAPI_HISYSEVENT_JS"

# 实时监控（开发板）
hilog -w &
```

### 常见日志关键字

| 关键字 | 含义 |
|--------|------|
| `Write` | 事件写入 |
| `CheckLimitWritingEvent` | 频率检查 |
| `ERR_WRITE_IN_HIGH_FREQ` | 高频错误 |
| `ERR_DOMAIN_MASKED` | 域被屏蔽 |
| `AddListener` | 添加观察者 |
| `RemoveListener` | 移除观察者 |
| `Query` | 查询事件 |
| `Export` | 导出事件 |

---

## 调试技巧

### 1. 启用详细日志

```cpp
// 在代码中启用调试日志
#include "hilog/log.h"

HILOG_DEBUG(LOG_CORE, "Debug message");
HILOG_INFO(LOG_CORE, "Info message");
HILOG_WARN(LOG_CORE, "Warning message");
HILOG_ERROR(LOG_CORE, "Error message");
```

### 2. 检查事件是否发送成功

```cpp
int ret = HiSysEventWrite(HiSysEvent::Domain::AAFWK, "debug_event",
    HiSysEvent::EventType::BEHAVIOR, "test", 1);

if (ret < 0) {
    HILOG_ERROR(LOG_CORE, "Write failed with code: %{public}d", ret);
    // 根据错误码处理
} else {
    HILOG_INFO(LOG_CORE, "Write succeeded");
}
```

### 3. 使用 test 目录的单元测试

参考 `test/unittest/` 目录下的测试用例，了解正确用法。

```bash
# 运行单元测试（需要相应权限）
./run_tests.sh --test-module hisysevent
```

---

## 错误码速查表

| 错误码 | 常量名 | 描述 | 处理建议 |
|--------|--------|------|----------|
| 0 | SUCCESS | 成功 | - |
| -1 | ERR_DOMAIN_NAME_INVALID | 无效 domain | 检查域名格式 |
| -2 | ERR_EVENT_NAME_INVALID | 无效事件名 | 检查事件名格式 |
| -3 | ERR_DOES_NOT_INIT | 未初始化 | 检查初始化流程 |
| -4 | ERR_OVER_SIZE | 超出大小 | 检查数据大小 |
| -5 | ERR_SEND_FAIL | 发送失败 | 检查 IPC 连接 |
| -6 | ERR_WRITE_IN_HIGH_FREQ | 高频写入 | 降低写入频率 |
| -7 | ERR_DOMAIN_MASKED | 域被屏蔽 | 检查 DOMAIN_MASKS |
| -8 | ERR_EMPTY_EVENT | 空事件 | 检查参数 |
| -9 | ERR_RAW_DATA_WROTE_EXCEPTION | 原始数据异常 | 检查参数编码 |

| 错误码 | 常量名 | 描述 | 处理建议 |
|--------|--------|------|----------|
| 1 | ERR_KEY_NAME_INVALID | 无效参数名 | 检查参数名 |
| 2 | ERR_VALUE_LENGTH_TOO_LONG | 值太长 | 缩短字符串 |
| 3 | ERR_KEY_NUMBER_TOO_MUCH | 参数太多 | 减少参数 |
| 4 | ERR_ARRAY_TOO_MUCH | 数组太长 | 减少数组项 |
| 5 | ERR_VALUE_INVALID | 无效值 | 检查值类型 |

---

## 相关链接

- [概览](00_Overview.md) - 项目整体介绍
- [架构说明](01_Architecture.md) - 内部架构
- [N-API 参考](02_N-API.md) - JS/ArkTS 接口
- [C++ API 参考](03_CPP_API.md) - Native 接口
- [构建与编译](04_Build.md) - 编译配置
- [安全风险评审](05_Security.md) - 安全注意事项
