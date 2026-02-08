# 09. 常见问题与调试

## 目的

本文档介绍 Neural Network Runtime 的常见构建、运行问题及其定位方法。

## 适用范围

- 开发者
- 测试工程师
- 运维人员

## 构建问题

### 问题 1: 编译失败，找不到头文件

**现象**:
```
fatal error: 'neural_network_runtime/neural_network_runtime_type.h' file not found
```

**原因**: 头文件路径配置不正确

**解决方案**:
1. 检查 `BUILD.gn` 中的 `include_dirs` 配置
2. 确保依赖 `nnrt_public_config` 或 `nnrt_config`

**证据**: `frameworks/native/neural_network_core/BUILD.gn:45-52`

```gn
public_configs = [
  "//foundation/ai/neural_network_runtime/config:coverage_flags",
  ":nnrt_public_config",
]
```

### 问题 2: 链接错误，未定义符号

**现象**:
```
undefined reference to `OH_NNModel_Construct'
```

**原因**: 未链接 `libneural_network_runtime.so`

**解决方案**:
1. 在 `BUILD.gn` 中添加依赖:
```gn
deps = [
  "//foundation/ai/neural_network_runtime:nnrt_target",
]
```

2. 或只依赖 Core 库:
```gn
deps = [
  "//foundation/ai/neural_network_runtime:nncore_target",
]
```

### 问题 3: 驱动编译失败，找不到 HDI 接口

**现象**:
```
fatal error: 'nnrt/interfaces/innrt_device.h' file not found
```

**原因**: 缺少 HDI 接口依赖

**解决方案**:
1. 在 `BUILD.gn` 中添加外部依赖:
```gn
external_deps = [
  "drivers_interface_nnrt:libnnrt_stub_2.0",
]
```

## 运行问题

### 问题 1: 模型构建失败，返回 OH_NN_INVALID_PARAMETER

**现象**:
```c
OH_NN_ReturnCode ret = OH_NNModel_Finish(model);
// ret == OH_NN_INVALID_PARAMETER (2)
```

**可能原因**:
1. 算子参数数量不正确
2. 输入输出索引越界
3. 张量形状不匹配

**调试方法**:
1. 检查日志输出:
```bash
hilog | grep NNRt
```

2. 验证算子参数:
```c
// 检查算子参数数量
// 参考 neural_network_runtime_type.h 中的算子定义
```

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:326+`

### 问题 2: 编译失败，返回 OH_NN_UNAVALIDABLE_DEVICE

**现象**:
```c
OH_NN_ReturnCode ret = OH_NNCompilation_Build(compilation);
// ret == OH_NN_UNAVALIDABLE_DEVICE (7)
```

**可能原因**:
1. 设备未找到
2. HDI 服务未启动
3. 设备不支持模型中的算子

**调试方法**:
1. 检查设备列表:
```c
size_t* deviceIDs = NULL;
uint32_t deviceCount = 0;
OH_NNDevice_GetAllDevicesID(&deviceIDs, &deviceCount);
printf("Found %u devices\n", deviceCount);
```

2. 检查设备支持:
```c
const bool* isSupported = NULL;
uint32_t opCount = 0;
OH_NNModel_GetAvailableOperations(model, deviceID, &isSupported, &opCount);
for (uint32_t i = 0; i < opCount; i++) {
    printf("Op %u: %s\n", i, isSupported[i] ? "supported" : "not supported");
}
```

3. 检查 HDI 服务状态:
```bash
# 查看 nnrt_host 进程
ps -ef | grep nnrt_host

# 查看服务日志
hilog | grep nnrt
```

### 问题 3: 执行失败，返回 OH_NN_MEMORY_ERROR

**现象**:
```c
OH_NN_ReturnCode ret = OH_NNExecutor_RunSync(executor, ...);
// ret == OH_NN_MEMORY_ERROR (3)
```

**可能原因**:
1. 内存不足
2. 张量大小计算错误
3. 共享内存映射失败

**调试方法**:
1. 检查张量大小:
```c
size_t byteSize = 0;
OH_NNTensorDesc_GetByteSize(tensorDesc, &byteSize);
printf("Tensor byte size: %zu\n", byteSize);
```

2. 检查可用内存:
```bash
cat /proc/meminfo | grep MemAvailable
```

### 问题 4: 动态形状输出维度不正确

**现象**: 输出张量的形状与预期不符

**调试方法**:
1. 获取实际输出形状:
```c
int32_t* shape = NULL;
uint32_t shapeLength = 0;
OH_NNExecutor_GetOutputShape(executor, outputIndex, &shape, &shapeLength);
printf("Output shape: [");
for (uint32_t i = 0; i < shapeLength; i++) {
    printf("%d, ", shape[i]);
}
printf("]\n");
```

2. 获取输入维度范围:
```c
size_t *minDims = NULL, *maxDims = NULL, *shapeLen = NULL;
OH_NNExecutor_GetInputDimRange(executor, inputIndex, &minDims, &maxDims, &shapeLen);
```

## 性能问题

### 问题 1: 首次推理延迟高

**原因**: 模型编译和初始化需要时间

**解决方案**:
1. 使用模型缓存:
```c
OH_NNCompilation_SetCache(compilation, "/data/cache", 1);
```

2. 使用离线模型:
```c
OH_NNCompilation* compilation = OH_NNCompilation_ConstructWithOfflineModelFile("/data/model.offline");
```

### 问题 2: 推理性能不达标

**调试方法**:
1. 启用性能跟踪:
```c
// 编译时设置性能模式
OH_NNCompilation_SetPerformanceMode(compilation, OH_NN_PERFORMANCE_EXTREME);

// 启用 Float16
OH_NNCompilation_EnableFloat16(compilation, true);
```

2. 检查设备类型:
```c
OH_NN_DeviceType deviceType;
OH_NNDevice_GetType(deviceID, &deviceType);
// 确保使用 NPU/GPU 而非 CPU
```

## 调试技巧

### 启用详细日志

```c
// 在代码中设置日志级别
// 参考 common/log.h
#define LOG_LEVEL LOG_LEVEL_DEBUG
```

### 使用 hilog 查看日志

```bash
# 实时查看 NNRt 日志
hilog | grep NNRt

# 查看所有日志
hilog

# 过滤特定级别
hilog -l D  # Debug
hilog -l I  # Info
hilog -l W  # Warning
hilog -l E  # Error
```

### 检查进程状态

```bash
# 查看 nnrt_host 进程
ps -ef | grep nnrt

# 查看进程内存
smem -P nnrt

# 查看进程打开的文件
ls -l /proc/$(pidof nnrt_host)/fd
```

### 使用 strace 跟踪系统调用

```bash
# 跟踪应用进程
strace -p $(pidof your_app)

# 跟踪 nnrt_host
strace -p $(pidof nnrt_host)
```

### 检查 SELinux 日志

```bash
# 查看 SELinux 拒绝日志
dmesg | grep avc

# 查看 nnrt 相关 SELinux 日志
dmesg | grep -i nnrt
```

## 常见错误码速查

| 错误码 | 值 | 含义 | 常见原因 |
|--------|-----|------|----------|
| OH_NN_SUCCESS | 0 | 成功 | - |
| OH_NN_FAILED | 1 | 失败 | 内部错误 |
| OH_NN_INVALID_PARAMETER | 2 | 无效参数 | 参数越界、类型错误 |
| OH_NN_MEMORY_ERROR | 3 | 内存错误 | 内存不足、分配失败 |
| OH_NN_OPERATION_FORBIDDEN | 4 | 操作禁止 | 状态错误、重复操作 |
| OH_NN_NULL_PTR | 5 | 空指针 | 传入 NULL 指针 |
| OH_NN_INVALID_FILE | 6 | 无效文件 | 文件不存在、格式错误 |
| OH_NN_UNAVALIDABLE_DEVICE | 7 | 设备不可用 | 设备离线、不支持 |
| OH_NN_INVALID_PATH | 8 | 无效路径 | 路径不存在、无权限 |
| OH_NN_TIMEOUT | 9 | 超时 | 执行超时 |
| OH_NN_UNSUPPORTED | 10 | 不支持 | 功能未实现 |
| OH_NN_CONNECTION_EXCEPTION | 11 | 连接异常 | IPC 连接断开 |
| OH_NN_SAVE_CACHE_EXCEPTION | 12 | 缓存异常 | 缓存写入失败 |
| OH_NN_DYNAMIC_SHAPE | 13 | 动态形状 | 动态形状相关错误 |
| OH_NN_UNAVAILABLE_DEVICE | 14 | 设备不可用 | HDI 服务异常 |

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:144-191`

## 相关跳转

- [错误码对照](appendix/Error_Codes.md)
- [对外 API](04_Native_API.md)
- [安全风险评审](08_Security_Review.md)
