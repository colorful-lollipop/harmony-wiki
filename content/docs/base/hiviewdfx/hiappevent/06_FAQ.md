# 常见问题与解决方案

## 6.1 构建相关问题

### 6.1.1 编译依赖缺失

**问题描述**：

```
error: dependency 'napi' not found
error: dependency 'hilog' not found
```

**问题原因**：HiAppEvent 依赖的 OpenHarmony 子系统模块未被包含在当前产品配置中。

**解决方案**：

1. 检查产品配置文件（通常位于 `build/lite/product/` 目录下）
2. 确认以下子系统已被包含：
   - hiviewdfx（DFX 子系统）
   - ability（能力子系统）
   - bundle（包管理子系统）
   - utils（工具子系统）

3. 修改产品配置文件，添加缺失的子系统：
   ```json
   {
     "subsystem": [
       "hiviewdfx",
       "ability",
       "bundle",
       "utils"
     ]
   }
   ```

**定位路径**：

| 检查项 | 文件路径 |
|-------|---------|
| bundle.json | `/base/hiviewdfx/hiappevent/bundle.json` |
| 产品配置 | `build/lite/product/{product_name}.json` |
| 构建日志 | `out/{build_dir}/build.log` |

### 6.1.2 头文件找不到

**问题描述**：

```
fatal error: 'hiappevent/hiappevent.h' file not found
```

**问题原因**：NDK 头文件未被正确包含到编译路径中。

**解决方案**：

1. 检查 ndk/BUILD.gn 中的 include_dirs 配置
2. 确认以下路径已包含：
   ```gn
   include_dirs = [
     "$hiappevent_native_path/libhiappevent/include",
     "$hiappevent_native_path/ndk/include",
     "//base/hiviewdfx/hiappevent/interfaces/native/kits/include",
   ]
   ```

3. 如果是独立使用 NDK，确保 NDK 包已正确安装：
   ```bash
   # 检查 NDK 头文件路径
   ls ${NDK_PATH}/include/hiappevent/
   ```

### 6.1.3 链接错误

**问题描述**：

```
undefined reference to 'OH_HiAppEvent_Write'
```

**问题原因**：链接器未能找到 HiAppEvent 库。

**解决方案**：

1. 检查 BUILD.gn 中的 deps 配置：
   ```gn
   deps = [ "//base/hiviewdfx/hiappevent/frameworks/native/libhiappevent:libhiappevent_base" ]
   ```

2. 确保编译产物路径在链接器的搜索路径中：
   ```bash
   # 手动链接时添加 -L 和 -l 参数
   g++ -L/out/.../lib -lhiappevent_base ...
   ```

## 6.2 运行相关问题

### 6.2.1 事件未写入

**问题描述**：

调用 write() 接口后，事件日志中没有相应记录。

**问题原因**：可能的原因包括打点功能被禁用、配置错误、存储路径问题等。

**排查步骤**：

1. 检查打点开关配置：
   ```javascript
   // 查看当前配置
   hiAppEvent.configure({ disable: false })
   ```

2. 检查事件日志目录权限：
   ```bash
   # 查看事件日志目录
   ls -la /data/log/hiappevent/
   
   # 检查目录权限
   ls -ld /data/log/hiappevent/
   ```

3. 检查应用是否有写入权限：
   ```javascript
   // 检查错误码
   hiAppEvent.write("test", hiAppEvent.EventType.FAULT, {}, (err) => {
       if (err) {
           console.error(`错误码: ${err.code}`)
       }
   })
   ```

**错误码对照**：

| 错误码 | 含义 | 排查方向 |
|-------|------|---------|
| 0 | 成功 | 无需排查 |
| 正整数 | 部分参数无效 | 检查 keyValues 参数 |
| -1 | 参数校验失败 | 检查 eventName 和 type |
| -2 | 存储失败 | 检查存储路径和权限 |

### 6.2.2 存储配额问题

**问题描述**：

事件日志文件不断增长，存储空间不足。

**问题原因**：未配置存储配额限制或配额已耗尽。

**解决方案**：

1. 配置存储配额：
   ```javascript
   // 设置存储配额为 10MB
   hiAppEvent.configure({ maxStorage: '10M' })
   ```

2. 手动清理旧日志：
   ```bash
   # 查看当前存储使用情况
   du -sh /data/log/hiappevent/
   
   # 手动删除旧日志文件
   rm -rf /data/log/hiappevent/*.old
   ```

3. 启用自动清理：
   ```bash
   # 检查清理配置
   cat /data/service/el2/100/hiappevent/hiappevent.cfg
   ```

### 6.2.3 Watcher 回调不触发

**问题描述**：

注册了事件观察者，但回调函数从未被调用。

**问题原因**：可能的原因包括过滤条件配置错误、回调注册失败、事件未被正确分发等。

**排查步骤**：

1. 检查 Watcher 创建和配置：
   ```javascript
   // 创建 Watcher
   let watcher = hiAppEvent.createWatcher('testWatcher')
   
   // 配置过滤条件
   watcher.setAppEventFilter({
       domain: 'hiappevent',
       eventTypes: [hiAppEvent.EventType.BEHAVIOR]
   })
   
   // 添加 Watcher
   hiAppEvent.addWatcher(watcher)
   ```

2. 检查过滤条件：
   - 确保 domain 和 eventTypes 与预期触发的事件匹配
   - 检查是否有事件被过滤

3. 检查日志：
   ```bash
   # 查看 HiLog 日志
   hilog | grep -i "hiappevent"
   ```

## 6.3 调试相关问题

### 6.3.1 开启调试日志

**问题描述**：

需要查看 HiAppEvent 的详细运行日志。

**解决方案**：

1. 启用 HiLog 调试：
   ```bash
   # 设置日志级别为 DEBUG
   hilog --set-level debug
   ```

2. 过滤 HiAppEvent 相关日志：
   ```bash
   # 过滤指定标签的日志
   hilog | grep -E "(Napi|HiAppEvent|HiAppEventWatcher)"
   ```

3. 在代码中添加调试日志：
   ```cpp
   #undef LOG_DOMAIN
   #define LOG_DOMAIN 0xD002D07
   
   #undef LOG_TAG
   #define LOG_TAG "HiAppEventDebug"
   
   HILOG_DEBUG(LOG_CORE, "Debug event: %{public}s", eventName.c_str());
   ```

### 6.3.2 事件日志格式解析

**问题描述**：

需要解析和理解事件日志文件的格式。

**日志格式示例**：

```json
{
  "domain": "hiappevent",
  "name": "user_login",
  "type": 4,
  "time": 1640995200000,
  "pid": 12345,
  "tid": 67890,
  "uid": 10000,
  "bundleName": "com.example.myapp",
  "params": {
    "user_id": "U123456",
    "login_type": "password"
  }
}
```

**字段说明**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| domain | string | 事件领域 |
| name | string | 事件名称 |
| type | int | 事件类型（1-FAULT、2-STATISTIC、3-SECURITY、4-BEHAVIOR） |
| time | long | 时间戳（毫秒） |
| pid | int | 进程 ID |
| tid | int | 线程 ID |
| uid | int | 用户 ID |
| bundleName | string | 应用包名 |
| params | object | 事件参数 |

### 6.3.3 使用命令行工具测试

**问题描述**：

需要通过命令行测试 HiAppEvent 功能。

**解决方案**：

```bash
# 进入设备 shell
hdc shell

# 创建测试事件
hiappevent_test write --domain test --name test_event --type 4 --params '{"key":"value"}'

# 查看当前配置
hiappevent_test config --list

# 清除所有事件数据
hiappevent_test clear
```

## 6.4 性能相关问题

### 6.4.1 事件写入性能优化

**问题描述**：

事件写入接口调用过于频繁，影响应用性能。

**优化建议**：

1. **批量写入**：将多个相关事件合并为一个事件：
   ```javascript
   // 优化前：多次调用
   hiAppEvent.write("event1", type, {...})
   hiAppEvent.write("event2", type, {...})
   hiAppEvent.write("event3", type, {...})
   
   // 优化后：合并为一个事件
   hiAppEvent.write("batch_event", type, {
       "event1_data": {...},
       "event2_data": {...},
       "event3_data": {...}
   })
   ```

2. **异步调用**：始终使用异步模式，避免阻塞主线程：
   ```javascript
   // 推荐：Promise 模式
   hiAppEvent.write("event", type, {...})
       .then(() => console.log('写入完成'))
       .catch(err => console.error('写入失败'))
   ```

3. **节流控制**：高频场景下使用节流：
   ```javascript
   let lastWriteTime = 0
   const MIN_INTERVAL = 1000 // 1秒
   
   function throttledWrite(event) {
       const now = Date.now()
       if (now - lastWriteTime >= MIN_INTERVAL) {
           hiAppEvent.write(event)
           lastWriteTime = now
       }
   }
   ```

### 6.4.2 存储性能优化

**问题描述**：

事件存储导致 I/O 性能问题。

**优化建议**：

1. **调整存储配额**：避免存储过大导致 I/O 压力：
   ```javascript
   // 根据设备性能设置合适的配额
   hiAppEvent.configure({ maxStorage: '5M' })  // 低端设备
   hiAppEvent.configure({ maxStorage: '20M' }) // 高端设备
   ```

2. **减少参数数量**：每个事件的参数数量影响序列化性能：
   ```javascript
   // 优化参数数量
   hiAppEvent.write("event", type, {
       "essential_param1": value1,  // 保留必要参数
       // "optional_param": value2,   // 移除可选参数
   })
   ```

## 6.5 兼容性相关问题

### 6.5.1 API 版本兼容

**问题描述**：

在不同版本的 OpenHarmony 上运行时报错。

**解决方案**：

1. **API 7 vs API 9+ 差异**：
   ```javascript
   // API 7
   import hiAppEvent from '@ohos.hiappevent'
   
   // API 9+
   import hiAppEvent from '@ohos.hiAppEvent'
   ```

2. **特性可用性检查**：
   ```javascript
   // 检查 Watcher 功能是否可用（API 12+）
   if (typeof hiAppEvent.createWatcher === 'function') {
       // 使用 Watcher 功能
   }
   ```

3. **条件导入**：
   ```javascript
   try {
       // 尝试导入新版 API
       const hiAppEvent = require('@ohos.hiappevent')
   } catch (e) {
       // 回退到旧版 API
       const hiAppEvent = require('@ohos.hiAppEvent')
   }
   ```

### 6.5.2 NDK 兼容性

**问题描述**：

Native 应用链接 NDK 库时遇到兼容性问题。

**解决方案**：

1. **确保 ABI 匹配**：
   ```bash
   # 检查目标 ABI
   file libhiappevent_ndk.z.so
   
   # arm64-v8a 适用于 64 位设备
   # armeabi-v7a 适用于 32 位设备
   ```

2. **正确链接 STL**：
   ```bash
   # 使用共享 STL
   g++ -L/path/to/stl -lgnustl_shared app.cpp -lhiappevent_ndk
   ```

## 6.6 问题定位工具

### 6.6.1 日志查看

```bash
# 查看 HiAppEvent 所有日志
hilog | grep -i "HiAppEvent"

# 查看实时日志（持续输出）
hilog -T HiAppEvent

# 保存日志到文件
hilog > hiappevent.log &

# 按级别过滤
hilog -L Error  # 只看错误日志
```

### 6.6.2 事件监控

```bash
# 监控事件写入
hdc shell "cat /data/log/hiappevent/events/*"

# 监听事件目录变化
hdc shell "inotifywait -m /data/log/hiappevent/"
```

### 6.6.3 配置检查

```bash
# 查看当前配置
hdc shell "cat /data/service/el2/100/hiappevent/hiappevent.cfg"

# 查看存储使用情况
hdc shell "du -sh /data/log/hiappevent/"
```

## 6.7 提交问题反馈

当遇到无法解决的问题时，请按以下模板提交问题反馈：

```
【问题标题】简洁描述问题现象

【问题环境】
- 设备型号：
- OpenHarmony 版本：
- API 版本：
- HiAppEvent 版本：

【问题描述】
详细描述问题现象，包括：
1. 预期行为
2. 实际行为
3. 重现步骤

【复现概率】
- 必现 / 偶现（复现频率：）

【日志信息】
粘贴相关日志（脱敏后）

【尝试过的解决方案】
1. 方案一：xxx
2. 方案二：xxx

【附加信息】
其他可能有帮助的信息
```

**反馈渠道**：
- 代码仓库 Issue：https://gitee.com/openharmony/hiviewdfx_hiappevent/issues
