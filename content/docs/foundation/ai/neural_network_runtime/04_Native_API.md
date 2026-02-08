# 04. 对外 Native API

## 目的

本文档详细介绍 Neural Network Runtime 对外暴露的 Native API (C 接口)，包括 API 清单、参数说明、错误码和使用示例。

## 适用范围

- AI 框架开发者
- 应用开发者

## 重要说明

**本项目是纯 Native 库，没有 N-API (JavaScript API) 实现。**

如果需要 JavaScript 接口，需要通过上层 AI 框架（如 MindSpore Lite）间接使用。

## API 清单

### 1. 模型构建 API

| API | 功能 | 头文件 |
|-----|------|--------|
| `OH_NNModel_Construct()` | 创建模型实例 | neural_network_runtime.h |
| `OH_NNModel_AddTensorToModel()` | 添加张量到模型 | neural_network_runtime.h |
| `OH_NNModel_SetTensorData()` | 设置张量数据 | neural_network_runtime.h |
| `OH_NNModel_SetTensorQuantParams()` | 设置量化参数 | neural_network_runtime.h |
| `OH_NNModel_SetTensorType()` | 设置张量类型 | neural_network_runtime.h |
| `OH_NNModel_AddOperation()` | 添加算子 | neural_network_runtime.h |
| `OH_NNModel_SpecifyInputsAndOutputs()` | 指定输入输出 | neural_network_runtime.h |
| `OH_NNModel_Finish()` | 完成模型构建 | neural_network_runtime.h |
| `OH_NNModel_Destroy()` | 销毁模型 | neural_network_runtime.h |
| `OH_NNModel_GetAvailableOperations()` | 查询设备支持的算子 | neural_network_runtime.h |

### 2. 编译 API

| API | 功能 | 头文件 |
|-----|------|--------|
| `OH_NNCompilation_Construct()` | 从模型创建编译 | neural_network_core.h |
| `OH_NNCompilation_ConstructWithOfflineModelFile()` | 从离线模型文件创建 | neural_network_core.h |
| `OH_NNCompilation_ConstructWithOfflineModelBuffer()` | 从离线模型缓冲区创建 | neural_network_core.h |
| `OH_NNCompilation_ConstructForCache()` | 为缓存恢复创建 | neural_network_core.h |
| `OH_NNCompilation_SetDevice()` | 设置编译设备 | neural_network_core.h |
| `OH_NNCompilation_SetCache()` | 设置缓存路径 | neural_network_core.h |
| `OH_NNCompilation_SetPerformanceMode()` | 设置性能模式 | neural_network_core.h |
| `OH_NNCompilation_SetPriority()` | 设置优先级 | neural_network_core.h |
| `OH_NNCompilation_EnableFloat16()` | 启用 Float16 | neural_network_core.h |
| `OH_NNCompilation_AddExtensionConfig()` | 添加扩展配置 | neural_network_core.h |
| `OH_NNCompilation_ExportCacheToBuffer()` | 导出缓存到缓冲区 | neural_network_core.h |
| `OH_NNCompilation_ImportCacheFromBuffer()` | 从缓冲区导入缓存 | neural_network_core.h |
| `OH_NNCompilation_Build()` | 执行编译 | neural_network_core.h |
| `OH_NNCompilation_Destroy()` | 销毁编译实例 | neural_network_core.h |

### 3. 执行 API

| API | 功能 | 头文件 |
|-----|------|--------|
| `OH_NNExecutor_Construct()` | 创建执行器 | neural_network_core.h |
| `OH_NNExecutor_RunSync()` | 同步执行 | neural_network_core.h |
| `OH_NNExecutor_RunAsync()` | 异步执行（预留） | neural_network_core.h |
| `OH_NNExecutor_GetInputCount()` | 获取输入数量 | neural_network_core.h |
| `OH_NNExecutor_GetOutputCount()` | 获取输出数量 | neural_network_core.h |
| `OH_NNExecutor_CreateInputTensorDesc()` | 创建输入张量描述符 | neural_network_core.h |
| `OH_NNExecutor_CreateOutputTensorDesc()` | 创建输出张量描述符 | neural_network_core.h |
| `OH_NNExecutor_GetOutputShape()` | 获取输出形状 | neural_network_core.h |
| `OH_NNExecutor_GetInputDimRange()` | 获取输入维度范围 | neural_network_core.h |
| `OH_NNExecutor_SetOnRunDone()` | 设置完成回调（预留） | neural_network_core.h |
| `OH_NNExecutor_SetOnServiceDied()` | 设置服务死亡回调（预留） | neural_network_core.h |
| `OH_NNExecutor_Destroy()` | 销毁执行器 | neural_network_core.h |

### 4. 张量 API

| API | 功能 | 头文件 |
|-----|------|--------|
| `OH_NNTensorDesc_Create()` | 创建张量描述符 | neural_network_core.h |
| `OH_NNTensorDesc_SetName()` | 设置名称 | neural_network_core.h |
| `OH_NNTensorDesc_SetDataType()` | 设置数据类型 | neural_network_core.h |
| `OH_NNTensorDesc_SetShape()` | 设置形状 | neural_network_core.h |
| `OH_NNTensorDesc_SetFormat()` | 设置格式 | neural_network_core.h |
| `OH_NNTensorDesc_GetName()` | 获取名称 | neural_network_core.h |
| `OH_NNTensorDesc_GetDataType()` | 获取数据类型 | neural_network_core.h |
| `OH_NNTensorDesc_GetShape()` | 获取形状 | neural_network_core.h |
| `OH_NNTensorDesc_GetFormat()` | 获取格式 | neural_network_core.h |
| `OH_NNTensorDesc_GetElementCount()` | 获取元素数量 | neural_network_core.h |
| `OH_NNTensorDesc_GetByteSize()` | 获取字节大小 | neural_network_core.h |
| `OH_NNTensorDesc_Destroy()` | 销毁描述符 | neural_network_core.h |
| `OH_NNTensor_Create()` | 创建张量 | neural_network_core.h |
| `OH_NNTensor_CreateWithSize()` | 指定大小创建 | neural_network_core.h |
| `OH_NNTensor_CreateWithFd()` | 使用 fd 创建 | neural_network_core.h |
| `OH_NNTensor_GetTensorDesc()` | 获取描述符 | neural_network_core.h |
| `OH_NNTensor_GetDataBuffer()` | 获取数据缓冲区 | neural_network_core.h |
| `OH_NNTensor_GetFd()` | 获取文件描述符 | neural_network_core.h |
| `OH_NNTensor_GetSize()` | 获取大小 | neural_network_core.h |
| `OH_NNTensor_GetOffset()` | 获取偏移 | neural_network_core.h |
| `OH_NNTensor_Destroy()` | 销毁张量 | neural_network_core.h |

### 5. 设备 API

| API | 功能 | 头文件 |
|-----|------|--------|
| `OH_NNDevice_GetAllDevicesID()` | 获取所有设备 ID | neural_network_core.h |
| `OH_NNDevice_GetType()` | 获取设备类型 | neural_network_core.h |
| `OH_NNDevice_GetName()` | 获取设备名称 | neural_network_core.h |

### 6. 量化参数 API

| API | 功能 | 头文件 |
|-----|------|--------|
| `OH_NNQuantParam_Create()` | 创建量化参数 | neural_network_runtime.h |
| `OH_NNQuantParam_SetScales()` | 设置缩放因子 | neural_network_runtime.h |
| `OH_NNQuantParam_SetZeroPoints()` | 设置零点 | neural_network_runtime.h |
| `OH_NNQuantParam_SetNumBits()` | 设置位数 | neural_network_runtime.h |
| `OH_NNQuantParam_Destroy()` | 销毁量化参数 | neural_network_runtime.h |

## 关键类型定义

### 错误码 (OH_NN_ReturnCode)

```c
typedef enum {
    OH_NN_SUCCESS = 0,              // 成功
    OH_NN_FAILED = 1,               // 失败
    OH_NN_INVALID_PARAMETER = 2,    // 无效参数
    OH_NN_MEMORY_ERROR = 3,         // 内存错误
    OH_NN_OPERATION_FORBIDDEN = 4,  // 操作禁止
    OH_NN_NULL_PTR = 5,             // 空指针
    OH_NN_INVALID_FILE = 6,         // 无效文件
    OH_NN_UNAVALIDABLE_DEVICE = 7,  // 设备不可用（已废弃）
    OH_NN_INVALID_PATH = 8,         // 无效路径
    OH_NN_TIMEOUT = 9,              // 超时
    OH_NN_UNSUPPORTED = 10,         // 不支持
    OH_NN_CONNECTION_EXCEPTION = 11,// 连接异常
    OH_NN_SAVE_CACHE_EXCEPTION = 12,// 缓存保存异常
    OH_NN_DYNAMIC_SHAPE = 13,       // 动态形状
    OH_NN_UNAVAILABLE_DEVICE = 14,  // 设备不可用
} OH_NN_ReturnCode;
```

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:144-191`

### 数据类型 (OH_NN_DataType)

```c
typedef enum {
    OH_NN_UNKNOWN = 0,
    OH_NN_BOOL = 1,
    OH_NN_INT8 = 2,
    OH_NN_INT16 = 3,
    OH_NN_INT32 = 4,
    OH_NN_INT64 = 5,
    OH_NN_UINT8 = 6,
    OH_NN_UINT16 = 7,
    OH_NN_UINT32 = 8,
    OH_NN_UINT64 = 9,
    OH_NN_FLOAT16 = 10,
    OH_NN_FLOAT32 = 11,
    OH_NN_FLOAT64 = 12
} OH_NN_DataType;
```

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:290-317`

### 设备类型 (OH_NN_DeviceType)

```c
typedef enum {
    OH_NN_OTHERS = 0,       // 其他设备
    OH_NN_CPU = 1,          // CPU
    OH_NN_GPU = 2,          // GPU
    OH_NN_ACCELERATOR = 3,  // 专用加速器
} OH_NN_DeviceType;
```

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:273-282`

### 性能模式 (OH_NN_PerformanceMode)

```c
typedef enum {
    OH_NN_PERFORMANCE_NONE = 0,      // 无偏好
    OH_NN_PERFORMANCE_LOW = 1,       // 低功耗
    OH_NN_PERFORMANCE_MEDIUM = 2,    // 中等性能
    OH_NN_PERFORMANCE_HIGH = 3,      // 高性能
    OH_NN_PERFORMANCE_EXTREME = 4,   // 极致性能
} OH_NN_PerformanceMode;
```

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:108-119`

### 优先级 (OH_NN_Priority)

```c
typedef enum {
    OH_NN_PRIORITY_NONE = 0,     // 无优先级
    OH_NN_PRIORITY_LOW = 1,      // 低优先级
    OH_NN_PRIORITY_MEDIUM = 2,   // 中优先级
    OH_NN_PRIORITY_HIGH = 3,     // 高优先级
} OH_NN_Priority;
```

**证据**: `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h:127-136`

## 使用示例

### 完整推理流程

```c
#include "neural_network_runtime/neural_network_runtime.h"
#include "neural_network_runtime/neural_network_core.h"
#include <stdio.h>

int main() {
    // 1. 创建模型
    OH_NNModel *model = OH_NNModel_Construct();
    if (model == NULL) {
        printf("Failed to create model\n");
        return -1;
    }

    // 2. 添加张量（输入、权重、偏置、输出）
    // ... 添加张量代码 ...

    // 3. 添加算子（例如卷积）
    // ... 添加算子代码 ...

    // 4. 指定输入输出
    // ... 指定输入输出代码 ...

    // 5. 完成模型构建
    OH_NN_ReturnCode ret = OH_NNModel_Finish(model);
    if (ret != OH_NN_SUCCESS) {
        printf("Failed to finish model: %d\n", ret);
        OH_NNModel_Destroy(&model);
        return -1;
    }

    // 6. 获取可用设备
    size_t *deviceIDs = NULL;
    uint32_t deviceCount = 0;
    ret = OH_NNDevice_GetAllDevicesID(&deviceIDs, &deviceCount);
    if (ret != OH_NN_SUCCESS || deviceCount == 0) {
        printf("No available device\n");
        OH_NNModel_Destroy(&model);
        return -1;
    }
    size_t deviceID = deviceIDs[0];

    // 7. 创建编译配置
    OH_NNCompilation *compilation = OH_NNCompilation_Construct(model);
    if (compilation == NULL) {
        printf("Failed to create compilation\n");
        OH_NNModel_Destroy(&model);
        return -1;
    }

    // 8. 设置编译选项
    ret = OH_NNCompilation_SetDevice(compilation, deviceID);
    if (ret != OH_NN_SUCCESS) {
        printf("Failed to set device: %d\n", ret);
        OH_NNCompilation_Destroy(&compilation);
        OH_NNModel_Destroy(&model);
        return -1;
    }

    ret = OH_NNCompilation_SetPerformanceMode(compilation, OH_NN_PERFORMANCE_HIGH);
    if (ret != OH_NN_SUCCESS) {
        printf("Failed to set performance mode: %d\n", ret);
    }

    // 9. 执行编译
    ret = OH_NNCompilation_Build(compilation);
    if (ret != OH_NN_SUCCESS) {
        printf("Failed to build compilation: %d\n", ret);
        OH_NNCompilation_Destroy(&compilation);
        OH_NNModel_Destroy(&model);
        return -1;
    }

    // 10. 创建执行器
    OH_NNExecutor *executor = OH_NNExecutor_Construct(compilation);
    if (executor == NULL) {
        printf("Failed to create executor\n");
        OH_NNCompilation_Destroy(&compilation);
        OH_NNModel_Destroy(&model);
        return -1;
    }

    // 11. 创建输入输出张量
    // ... 创建张量代码 ...

    // 12. 填充输入数据
    // ... 填充数据代码 ...

    // 13. 执行推理
    ret = OH_NNExecutor_RunSync(executor, inputTensors, inputCount, outputTensors, outputCount);
    if (ret != OH_NN_SUCCESS) {
        printf("Failed to run: %d\n", ret);
    }

    // 14. 读取输出结果
    // ... 读取结果代码 ...

    // 15. 释放资源
    // ... 释放张量代码 ...
    OH_NNExecutor_Destroy(&executor);
    OH_NNCompilation_Destroy(&compilation);
    OH_NNModel_Destroy(&model);

    return 0;
}
```

## 调用链

### 模型构建调用链

```
OH_NNModel_Construct()
    └── new InnerModel()
        └── InnerModel::InnerModel()

OH_NNModel_AddTensorToModel()
    └── InnerModel::AddTensor()
        └── 创建 NNTensor 并添加到模型

OH_NNModel_AddOperation()
    └── InnerModel::AddOperation()
        └── OpsRegistry::GetOpsBuilder()
        └── OpsBuilder::Build()
        └── 添加到 LiteGraph

OH_NNModel_Finish()
    └── InnerModel::Build()
        └── 验证模型完整性
        └── 生成最终 LiteGraph
```

### 编译调用链

```
OH_NNCompilation_Construct()
    └── new NNCompiler(model)
        └── NNCompiler::NNCompiler()

OH_NNCompilation_SetDevice()
    └── NNCompiler::SetDevice()

OH_NNCompilation_Build()
    └── NNCompiler::Build()
        ├── NNBackend::CreateCompiler()
        ├── NNCompiler::IsSupportedModel()
        ├── Device::PrepareModel()
        │   └── HDIDeviceV2_1::PrepareModel()
        │       └── HDI Service IPC 调用
        └── 保存 PreparedModel
```

### 执行调用链

```
OH_NNExecutor_Construct()
    └── NNCompiler::CreateExecutor()
        └── new NNExecutor()

OH_NNTensor_Create()
    └── NNBackend::CreateTensor()
        └── new NNTensor2_0()

OH_NNExecutor_RunSync()
    └── NNExecutor::RunSync()
        ├── 验证输入
        ├── PreparedModel::Run()
        │   └── HDIPreparedModelV2_1::Run()
        │       └── HDI Service IPC 调用
        └── 更新输出维度
```

## 相关跳转

- [API 速查](appendix/API_Quick_Reference.md)
- [错误码对照](appendix/Error_Codes.md)
- [内部 API](05_Inner_API.md)
- [架构说明](02_Architecture.md)
