# 常见问题与调试指南

## 8.1 构建问题

### 问题 1：编译找不到头文件

**错误信息**:
```
fatal error: 'ipc_skeleton.h' file not found
```

**解决方案**:
1. 检查是否正确引入依赖
2. 在 `BUILD.gn` 中添加：
   ```gn
   external_deps = [
     "ipc:ipc_core",
   ]
   ```

**证据**: `bundle.json:58-60` - SDK 依赖配置

### 问题 2：链接错误 - undefined reference

**错误信息**:
```
undefined reference to `OHOS::IPCSkeleton::GetCallingPid()'
```

**解决方案**:
1. 检查是否链接了正确的库
2. 在 `BUILD.gn` 中添加：
   ```gn
   deps = [
     "//foundation/communication/ipc/interfaces/innerkits/ipc_core:ipc_core",
   ]
   ```

### 问题 3：Feature Flag 未启用

**错误信息**:
```
error: 'FFRT_IPC_ENABLE' is not defined
```

**解决方案**:
在产品配置中启用 FFRT 支持，或在 `config.gni` 中设置：
```gn
resourceschedule_ffrt_support = true
```

**证据**: `config.gni:24`

## 8.2 运行时问题

### 问题 1：IPC 调用无响应（卡死）

**症状**:
- `SendRequest()` 调用后无返回
- 进程挂起

**排查步骤**:

1. **检查是否死锁**
   ```bash
   # 查看线程堆栈
   hdc shell "cat /proc/<pid>/stack"
   ```

2. **检查线程池配置**
   ```cpp
   // 确认 IPC 线程池是否耗尽
   IPCSkeleton::SetMaxWorkThreadNum(16);  // 增加线程数
   ```

3. **检查同步调用超时**
   ```cpp
   MessageOption option;
   option.setWaitTime(5000);  // 设置 5 秒超时
   ```

4. **日志追踪**
   ```cpp
   # 启用 IPC 追踪（如果已编译）
   hilog -D IPC_DEBUG
   ```

### 问题 2：返回错误码 -1

**错误码**: `-1` = `ERR_UNKNOWN`

**常见原因**:
| 原因 | 排查方法 |
|------|----------|
| 接口令牌不匹配 | 检查 `WriteInterfaceToken()` / `ReadInterfaceToken()` |
| 远程对象已死亡 | 检查 `IsObjectDead()` |
| 序列化失败 | 检查 `WriteXXX()` 返回值 |
| 参数错误 | 检查参数类型和大小 |

**调试代码**:
```cpp
int MyStub::OnRemoteRequest(uint32_t code, MessageParcel &data,
                            MessageParcel &reply, MessageOption &option) {
    // 添加调试日志
    ZLOGI(IPC_DEBUG) << "Received code: " << code;
    
    // 校验接口令牌
    std::u16string token = data.ReadInterfaceToken();
    if (token != GetDescriptor()) {
        ZLOGE(IPC_DEBUG) << "Token mismatch!";
        return -1;
    }
    
    // 处理请求
    // ...
}
```

### 问题 3：数据读取不完整

**症状**:
- `ReadInt32()` 返回错误值
- `ReadString()` 返回空字符串

**排查步骤**:

1. **检查写入完整性**
   ```cpp
   // 发送端
   bool ok = data.WriteInt32(100);
   if (!ok) {
       ZLOGE("WriteInt32 failed!");
   }
   ```

2. **检查数据位置**
   ```cpp
   // 确保在读取前 rewind（如果需要）
   data.RewindRead();
   ```

3. **检查数据大小**
   ```cpp
   size_t size = data.GetDataSize();
   ZLOGI("Data size: %{public}zu", size);
   ```

### 问题 4：跨设备 RPC 连接失败

**症状**:
- DBinder 跨设备调用失败
- 返回网络错误

**排查步骤**:

1. **检查 DSoftBus 服务**
   ```bash
   # 检查 DSoftBus 服务状态
   hdc shell "hibusctl status"
   ```

2. **检查网络连接**
   ```bash
   # 测试设备间网络连通性
   ping <device_ip>
   ```

3. **检查 DBinder 服务**
   ```cpp
   // 确保 DBinder 服务已启动
   auto dbinder = DBinderService::GetInstance();
   if (dbinder == nullptr) {
       ZLOGE("DBinder not initialized!");
   }
   ```

4. **检查设备认证**
   ```bash
   # 检查设备是否在同一网络
   deviceManager list
   ```

## 8.3 调试方法

### 日志打印

```cpp
#include "hilog/log.h"

// 定义日志标签
#undef LOG_TAG
#define LOG_TAG "MyIPC"

void MyIPCFunction() {
    // 不同级别日志
    ZLOGI(LOG_TAG) << "Info log";
    ZLOGW(LOG_TAG) << "Warning log";
    ZLOGE(LOG_TAG) << "Error log";
}
```

**日志级别**:
| 级别 | 宏 | 用途 |
|------|-----|------|
| DEBUG | `ZLOGD` | 调试信息 |
| INFO | `ZLOGI` | 普通信息 |
| WARN | `ZLOGW` | 警告 |
| ERROR | `ZLOGE` | 错误 |
| FATAL | `ZLOGF` | 致命错误 |

### 追踪调用链

```cpp
#include "hitrace/hitrace_meter.h"

// 开始追踪
uint64_t traceId = HiTraceBegin("MyFunction", HITRACE_FLAG_DEFAULT);

// 执行操作
// ...

// 结束追踪
HiTraceEnd(traceId);
```

### 打印 MessageParcel 内容

```cpp
void DumpMessageParcel(const MessageParcel &data) {
    size_t size = data.GetDataSize();
    ZLOGI("Parcel size: %{public}zu", size);
    
    // 获取只读数据指针
    const uint8_t *buffer = data.GetData();
    for (size_t i = 0; i < size && i < 64; i++) {
        printf("%02x ", buffer[i]);
    }
    printf("\n");
}
```

## 8.4 调试工具

### 命令行工具

| 工具 | 用途 |
|------|------|
| `hdc` | 设备连接调试 |
| `hilog` | 查看日志 |
| `hiperf` | 性能分析 |
| `hidumper` | 系统状态转储 |

### 使用示例

```bash
# 查看 IPC 相关日志
hdc shell "hilog | grep -i ipc"

# 查看进程堆栈
hdc shell "cat /proc/<pid>/stack"

# 转储系统信息
hidumper -s IPC

# 查看 Binder 状态
cat /sys/kernel/debug/binder/state
```

### 代码调试

```cpp
// 条件编译调试代码
#ifdef DEBUG_IPC
void DebugDump(const MessageParcel &data) {
    // 调试代码
}
#else
inline void DebugDump(const MessageParcel &) {}
#endif
```

## 8.5 性能优化

### 减少序列化开销

```cpp
// ❌ 低效: 多次 Write 调用
data.WriteInt32(a);
data.WriteInt32(b);
data.WriteInt32(c);

// ✅ 高效: 批量写入
struct MyData {
    int32_t a;
    int32_t b;
    int32_t c;
};
data.WriteRawData(&data, sizeof(MyData));
```

### 使用共享内存

```cpp
// 对于 > 64KB 的数据，使用 Ashmem
auto ashmem = Ashmem::CreateAshmem(size);
ashmem.WriteToAshmem(largeBuffer, size, 0);
data.WriteAshmem(ashmem);
```

### 异步调用

```cpp
// 避免阻塞主线程
MessageOption option;
option.setFlags(MessageOption::TF_ASYNC);

// 使用回调
proxy->SendRequest(code, data, reply, option,
    [](AsyncResult result) {
        // 处理结果
    });
```

---

*证据来源*:
- `bundle.json` - SDK 依赖配置
- `config.gni` - Feature Flags 配置
- `README_zh.md` - 使用说明与示例
- Phase 1 全局扫描结果
