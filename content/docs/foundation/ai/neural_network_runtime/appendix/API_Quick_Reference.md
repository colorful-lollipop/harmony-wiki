# 附录 C: API 速查

## 模型构建 API

| API | 功能 | 返回值 |
|-----|------|--------|
| `OH_NNModel* OH_NNModel_Construct(void)` | 创建模型 | 模型句柄或 NULL |
| `OH_NN_ReturnCode OH_NNModel_AddTensorToModel(OH_NNModel* model, const NN_TensorDesc* tensorDesc)` | 添加张量 | 错误码 |
| `OH_NN_ReturnCode OH_NNModel_SetTensorData(OH_NNModel* model, uint32_t index, const void* dataBuffer, size_t length)` | 设置张量数据 | 错误码 |
| `OH_NN_ReturnCode OH_NNModel_AddOperation(OH_NNModel* model, OH_NN_OperationType op, const OH_NN_UInt32Array* paramIndices, const OH_NN_UInt32Array* inputIndices, const OH_NN_UInt32Array* outputIndices)` | 添加算子 | 错误码 |
| `OH_NN_ReturnCode OH_NNModel_SpecifyInputsAndOutputs(OH_NNModel* model, const OH_NN_UInt32Array* inputIndices, const OH_NN_UInt32Array* outputIndices)` | 指定输入输出 | 错误码 |
| `OH_NN_ReturnCode OH_NNModel_Finish(OH_NNModel* model)` | 完成模型构建 | 错误码 |
| `void OH_NNModel_Destroy(OH_NNModel** model)` | 销毁模型 | 无 |

## 编译 API

| API | 功能 | 返回值 |
|-----|------|--------|
| `OH_NNCompilation* OH_NNCompilation_Construct(const OH_NNModel* model)` | 从模型创建编译 | 编译句柄或 NULL |
| `OH_NNCompilation* OH_NNCompilation_ConstructWithOfflineModelFile(const char* modelPath)` | 从离线模型文件创建 | 编译句柄或 NULL |
| `OH_NNCompilation* OH_NNCompilation_ConstructWithOfflineModelBuffer(const void* modelBuffer, size_t modelSize)` | 从离线模型缓冲区创建 | 编译句柄或 NULL |
| `OH_NN_ReturnCode OH_NNCompilation_SetDevice(OH_NNCompilation* compilation, size_t deviceID)` | 设置编译设备 | 错误码 |
| `OH_NN_ReturnCode OH_NNCompilation_SetCache(OH_NNCompilation* compilation, const char* cachePath, uint32_t version)` | 设置缓存路径 | 错误码 |
| `OH_NN_ReturnCode OH_NNCompilation_SetPerformanceMode(OH_NNCompilation* compilation, OH_NN_PerformanceMode performanceMode)` | 设置性能模式 | 错误码 |
| `OH_NN_ReturnCode OH_NNCompilation_SetPriority(OH_NNCompilation* compilation, OH_NN_Priority priority)` | 设置优先级 | 错误码 |
| `OH_NN_ReturnCode OH_NNCompilation_EnableFloat16(OH_NNCompilation* compilation, bool enableFloat16)` | 启用 Float16 | 错误码 |
| `OH_NN_ReturnCode OH_NNCompilation_Build(OH_NNCompilation* compilation)` | 执行编译 | 错误码 |
| `void OH_NNCompilation_Destroy(OH_NNCompilation** compilation)` | 销毁编译实例 | 无 |

## 执行 API

| API | 功能 | 返回值 |
|-----|------|--------|
| `OH_NNExecutor* OH_NNExecutor_Construct(OH_NNCompilation* compilation)` | 创建执行器 | 执行器句柄或 NULL |
| `OH_NN_ReturnCode OH_NNExecutor_RunSync(OH_NNExecutor* executor, NN_Tensor* inputTensor[], size_t inputCount, NN_Tensor* outputTensor[], size_t outputCount)` | 同步执行 | 错误码 |
| `OH_NN_ReturnCode OH_NNExecutor_GetOutputShape(OH_NNExecutor* executor, uint32_t outputIndex, int32_t** shape, uint32_t* shapeLength)` | 获取输出形状 | 错误码 |
| `void OH_NNExecutor_Destroy(OH_NNExecutor** executor)` | 销毁执行器 | 无 |

## 张量 API

| API | 功能 | 返回值 |
|-----|------|--------|
| `NN_TensorDesc* OH_NNTensorDesc_Create()` | 创建张量描述符 | 描述符句柄或 NULL |
| `OH_NN_ReturnCode OH_NNTensorDesc_SetDataType(NN_TensorDesc* tensorDesc, OH_NN_DataType dataType)` | 设置数据类型 | 错误码 |
| `OH_NN_ReturnCode OH_NNTensorDesc_SetShape(NN_TensorDesc* tensorDesc, const int32_t* shape, size_t shapeLength)` | 设置形状 | 错误码 |
| `OH_NN_ReturnCode OH_NNTensorDesc_SetFormat(NN_TensorDesc* tensorDesc, OH_NN_Format format)` | 设置格式 | 错误码 |
| `OH_NN_ReturnCode OH_NNTensorDesc_Destroy(NN_TensorDesc** tensorDesc)` | 销毁描述符 | 错误码 |
| `NN_Tensor* OH_NNTensor_Create(size_t deviceID, NN_TensorDesc* tensorDesc)` | 创建张量 | 张量句柄或 NULL |
| `void* OH_NNTensor_GetDataBuffer(const NN_Tensor* tensor)` | 获取数据缓冲区 | 缓冲区指针或 NULL |
| `OH_NN_ReturnCode OH_NNTensor_Destroy(NN_Tensor** tensor)` | 销毁张量 | 错误码 |

## 设备 API

| API | 功能 | 返回值 |
|-----|------|--------|
| `OH_NN_ReturnCode OH_NNDevice_GetAllDevicesID(size_t** deviceIDs, uint32_t* deviceCount)` | 获取所有设备 ID | 错误码 |
| `OH_NN_ReturnCode OH_NNDevice_GetType(size_t deviceID, OH_NN_DeviceType* deviceType)` | 获取设备类型 | 错误码 |
| `OH_NN_ReturnCode OH_NNDevice_GetName(size_t deviceID, const char** name)` | 获取设备名称 | 错误码 |

## 类型定义

### 错误码

```c
OH_NN_SUCCESS = 0
OH_NN_FAILED = 1
OH_NN_INVALID_PARAMETER = 2
OH_NN_MEMORY_ERROR = 3
OH_NN_OPERATION_FORBIDDEN = 4
OH_NN_NULL_PTR = 5
OH_NN_INVALID_FILE = 6
OH_NN_UNAVALIDABLE_DEVICE = 7
OH_NN_INVALID_PATH = 8
OH_NN_TIMEOUT = 9
OH_NN_UNSUPPORTED = 10
OH_NN_CONNECTION_EXCEPTION = 11
OH_NN_SAVE_CACHE_EXCEPTION = 12
OH_NN_DYNAMIC_SHAPE = 13
OH_NN_UNAVAILABLE_DEVICE = 14
```

### 数据类型

```c
OH_NN_UNKNOWN = 0
OH_NN_BOOL = 1
OH_NN_INT8 = 2
OH_NN_INT16 = 3
OH_NN_INT32 = 4
OH_NN_INT64 = 5
OH_NN_UINT8 = 6
OH_NN_UINT16 = 7
OH_NN_UINT32 = 8
OH_NN_UINT64 = 9
OH_NN_FLOAT16 = 10
OH_NN_FLOAT32 = 11
OH_NN_FLOAT64 = 12
```

### 设备类型

```c
OH_NN_OTHERS = 0
OH_NN_CPU = 1
OH_NN_GPU = 2
OH_NN_ACCELERATOR = 3
```

### 性能模式

```c
OH_NN_PERFORMANCE_NONE = 0
OH_NN_PERFORMANCE_LOW = 1
OH_NN_PERFORMANCE_MEDIUM = 2
OH_NN_PERFORMANCE_HIGH = 3
OH_NN_PERFORMANCE_EXTREME = 4
```

### 优先级

```c
OH_NN_PRIORITY_NONE = 0
OH_NN_PRIORITY_LOW = 1
OH_NN_PRIORITY_MEDIUM = 2
OH_NN_PRIORITY_HIGH = 3
```

## 头文件

```c
#include "neural_network_runtime/neural_network_runtime_type.h"  // 类型定义
#include "neural_network_runtime/neural_network_runtime.h"       // 模型构建 API
#include "neural_network_runtime/neural_network_core.h"          // 编译执行 API
```

## 链接库

```gn
deps = [
  "//foundation/ai/neural_network_runtime:nnrt_target",
]
```

## 相关跳转

- [对外 API](../04_Native_API.md)
- [错误码对照](Error_Codes.md)
