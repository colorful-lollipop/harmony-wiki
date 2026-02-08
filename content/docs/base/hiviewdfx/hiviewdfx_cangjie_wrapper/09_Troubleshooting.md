# 常见问题与定位

## 概述

本文档收集 hiviewdfdfx_cangjie_wrapper 常见的构建、运行和调试问题，并提供定位路径和解决方案。

---

## 构建问题

### 问题1: 编译找不到模块

**现象**:
```
error: Cannot find module 'ohos.hilog'
```

**原因**: 模块未正确导入或依赖缺失

**解决步骤**:
1. 检查 `bundle.json` 中是否声明了依赖
2. 确认 `BUILD.gn` 配置正确
3. 清理并重新构建

**定位命令**:
```bash
# 1. 检查模块配置
cat bundle.json | grep -A 5 "sub_component"

# 2. 清理构建产物
rm -rf out/{product}/*
hb clean

# 3. 重新构建
hb build -f
```

**代码证据**: `bundle.json:31-36`

---

### 问题2: FFI 依赖缺失

**现象**:
```
error: undefined reference to 'FfiOHOSHiAppEventWrite'
```

**原因**: Native FFI 库未正确链接

**解决步骤**:
1. 检查 `external_deps` 配置
2. 确认依赖组件已构建
3. 检查 syscap 声明

**定位命令**:
```bash
# 1. 检查依赖配置
cat ohos/hiviewdfx/hi_app_event/BUILD.gn | grep external_deps

# 2. 检查依赖组件
cat bundle.json | grep -A 10 "deps"

# 3. 验证 FFI 库
ls -la out/{product}/libs/ | grep ffi
```

**代码证据**: `ohos/hiviewdfx/hi_app_event/BUILD.gn:42`

---

### 问题3: 跨平台构建失败

**现象**:
```
error: Cannot find '../../mock/ohos.hilog.cj'
```

**原因**: Windows/macOS 平台的 mock 文件路径错误

**解决步骤**:
1. 检查 mock 目录是否存在
2. 验证路径配置
3. 确认平台标识

**定位命令**:
```bash
# 1. 检查 mock 文件
ls -la mock/

# 2. 检查平台标识
echo "is_mac: $(uname)"
echo "is_mingw: $(echo $OSTYPE | grep mingw)"

# 3. 验证构建配置
cat ohos/hilog/BUILD.gn | grep -A 5 "is_mingw"
```

---

## 运行问题

### 问题4: 日志无法输出

**现象**: 调用 `Hilog.info()` 但日志不显示

**可能原因**:
1. 日志级别不可见
2. domain 或 tag 无效
3. 隐私开关关闭

**解决步骤**:
1. 检查日志级别
2. 验证 domain 和 tag
3. 检查隐私设置

**定位代码**:
```cj
import ohos.hilog.{Hilog, LogLevel}

// 1. 检查日志是否可打印
let isLoggable = Hilog.isLoggable(0xD002800, "TestTag", LogLevel.Info)
if (!isLoggable) {
    // 日志级别不可用
}

// 2. 使用有效 domain
// domain 范围: 0x0 - 0xFFFF

// 3. 检查标签长度
let tag = "TestTag"  // 最大 32 字节
```

**代码证据**: `ohos/hilog/hilog.cj:131-138`

---

### 问题5: 事件写入失败

**现象**: 调用 `HiAppEvent.write()` 抛出异常

**可能原因**:
1. 功能被禁用
2. 事件参数无效
3. 存储配额超限

**解决步骤**:
1. 检查配置
2. 验证事件参数
3. 清理存储

**定位代码**:
```cj
import ohos.hiviewdfx.hi_app_event.{HiAppEvent, ConfigOption}

// 1. 检查功能是否启用
try {
    HiAppEvent.configure(ConfigOption(disable: false))
} catch (e: BusinessException) {
    // 功能被禁用
}

// 2. 验证参数
// domain: 最多 32 字符，字母开头
// name: 最多 48 字符，字母/$开头
// params: 最多 32 个

// 3. 清理存储
HiAppEvent.clearData()
```

**错误码参考**:
- `11100001`: 功能禁用
- `11101001`: 无效事件域
- `11101002`: 无效事件名
- `11101003`: 参数数量超限

**代码证据**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:64-75`

---

### 问题6: 性能追踪无效

**现象**: 调用 `HiTraceMeter.startTrace()` 后 bytrace 看不到数据

**可能原因**:
1. 任务时间太短（< 3ms）
2. taskId 不匹配
3. bytrace 配置问题

**解决步骤**:
1. 延长任务时间
2. 验证 taskId 一致性
3. 检查 bytrace 配置

**定位代码**:
```cj
import ohos.hi_trace_meter.HiTraceMeter

// 1. 确保 startTrace 和 finishTrace 使用相同参数
HiTraceMeter.startTrace("network_request", 1001)
// ... 业务逻辑（建议 > 3ms）
HiTraceMeter.finishTrace("network_request", 1001)

// 2. 使用 bytrace 捕获
// bytrace --trace_format=binary -b 16384 -c 1 -t 5
```

**代码证据**: `ohos/hi_trace_meter/hi_trace_meter.cj:29-34`

---

### 问题7: 内存泄漏

**现象**: 长时间运行后内存持续增长

**可能原因**:
1. CString 未释放
2. AppEventPackageHolder 未释放
3. 事件订阅未清理

**解决步骤**:
1. 使用 try-with-resources
2. 清理 Watcher
3. 定期清理数据

**定位代码**:
```cj
// 1. 使用 asResource() 自动释放
unsafe {
    try (cName = LibC.mallocCString(name).asResource()) {
        // 操作完成后自动释放
    }
}

// 2. 清理 Watcher
HiAppEvent.removeWatcher(watcher)

// 3. 清理数据
HiAppEvent.clearData()
```

**代码证据**: `ohos/hilog/hilog.cj:156-158`

---

## 调试方法

### 日志调试

启用调试日志：

```cj
// 在应用启动时配置
Hilog.info(0, "Debug", "Module initialized", [])

// 检查特定模块
Hilog.info(domain, "HiAppEvent", "Event written: %{public}s", [eventName])
```

### 事件追踪

使用 bytrace 工具：

```bash
# 开启 HiTrace 追踪
bytrace --trace_format=binary -b 16384 -c 1 -t 30 --hi-trace ohos.hiviewdfx &

# 运行应用
hdc shell aa start -D 0 -n com.example.app

# 停止追踪并查看结果
bytrace --stop
```

### API 调试

使用 hilogcat 查看日志：

```bash
# 查看所有日志
hdc shell hilogcat

# 过滤特定标签
hdc shell hilogcat | grep "TestTag"

# 过滤特定级别
hdc shell hilogcat | grep -E "Error|Fatal"
```

---

## 性能问题

### 性能建议

| 场景 | 建议 |
|------|------|
| 高频日志 | 使用 `isLoggable()` 检查，避免无效日志 |
| 大量事件 | 使用 batch 上报，减少 write() 调用 |
| 性能追踪 | 追踪 > 3ms 的任务，避免过细粒度 |
| 事件订阅 | 及时 removeWatcher，避免资源泄漏 |

### 监控指标

```cj
// 1. 日志输出频率监控
// 通过 hilogcat 统计

// 2. 事件写入监控
// 检查 HiAppEvent 返回值

// 3. 内存使用监控
// 使用 systemview 或 perf
```

---

## 常见错误码

### HiAppEvent 错误码

| 错误码 | 含义 | 处理建议 |
|--------|------|---------|
| 11100001 | 功能禁用 | 检查 ConfigOption |
| 11101001 | 无效事件域 | 验证 domain 格式 |
| 11101002 | 无效事件名 | 验证 name 格式 |
| 11101003 | 参数过多 | 减少参数数量 |
| 11101004 | 字符串过长 | 缩短字符串 |
| 11102001 | 无效 Watcher 名 | 验证 name 格式 |
| 11103001 | 无效存储配额 | 检查 maxStorage |
| 11104001 | 无效大小值 | 检查 size 参数 |
| 11105001 | 参数错误 | 检查输入参数 |

**数据来源**: `ohos/hiviewdfx/hi_app_event/hi_app_event.cj:64-75`

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [04_External_API.md](04_External_API.md) | API 参考 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物 |
| [08_Security_Review.md](08_Security_Review.md) | 安全评审 |
