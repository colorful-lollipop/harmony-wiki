# 附录 D: 错误码对照

## 错误码列表

| 错误码 | 值 | 名称 | 说明 | 常见场景 |
|--------|-----|------|------|----------|
| 0 | 0 | `OH_NN_SUCCESS` | 操作成功 | - |
| 1 | 1 | `OH_NN_FAILED` | 操作失败 | 内部错误 |
| 2 | 2 | `OH_NN_INVALID_PARAMETER` | 无效参数 | 参数越界、类型错误、形状不匹配 |
| 3 | 3 | `OH_NN_MEMORY_ERROR` | 内存错误 | 内存不足、分配失败、映射失败 |
| 4 | 4 | `OH_NN_OPERATION_FORBIDDEN` | 操作禁止 | 状态错误、重复操作、接口未实现 |
| 5 | 5 | `OH_NN_NULL_PTR` | 空指针 | 传入 NULL 指针 |
| 6 | 6 | `OH_NN_INVALID_FILE` | 无效文件 | 文件不存在、格式错误、权限不足 |
| 7 | 7 | `OH_NN_UNAVALIDABLE_DEVICE` | 设备不可用（已废弃） | 设备离线、不支持算子 |
| 8 | 8 | `OH_NN_INVALID_PATH` | 无效路径 | 路径不存在、权限不足 |
| 9 | 9 | `OH_NN_TIMEOUT` | 超时 | 执行超时、IPC 超时 |
| 10 | 10 | `OH_NN_UNSUPPORTED` | 不支持 | 功能未实现、设备不支持 |
| 11 | 11 | `OH_NN_CONNECTION_EXCEPTION` | 连接异常 | IPC 连接断开、服务死亡 |
| 12 | 12 | `OH_NN_SAVE_CACHE_EXCEPTION` | 缓存异常 | 缓存写入失败、校验失败 |
| 13 | 13 | `OH_NN_DYNAMIC_SHAPE` | 动态形状 | 动态形状相关错误 |
| 14 | 14 | `OH_NN_UNAVAILABLE_DEVICE` | 设备不可用 | HDI 服务异常、设备离线 |

## 错误码详情

### OH_NN_SUCCESS (0)

**说明**: 操作成功完成。

**处理**: 无需处理，继续后续操作。

### OH_NN_FAILED (1)

**说明**: 通用失败错误。

**常见原因**:
- 内部逻辑错误
- 未预期的异常

**处理**: 查看日志获取详细信息。

### OH_NN_INVALID_PARAMETER (2)

**说明**: 传入的参数无效。

**常见原因**:
- 参数值为 NULL（但不应为 NULL）
- 数值参数越界
- 张量形状不匹配
- 算子参数数量不正确
- 输入输出索引越界

**处理**: 检查 API 调用参数，参考文档确认参数范围。

**示例**:
```c
// 错误：张量形状不匹配
int32_t shape[] = {1, 2, 3};
OH_NNTensorDesc_SetShape(desc, shape, 0);  // shapeLength 为 0，错误

// 正确
int32_t shape[] = {1, 2, 3};
OH_NNTensorDesc_SetShape(desc, shape, 3);  // shapeLength 为 3
```

### OH_NN_MEMORY_ERROR (3)

**说明**: 内存相关错误。

**常见原因**:
- 系统内存不足
- 共享内存分配失败
- 内存映射失败
- 张量大小计算溢出

**处理**: 
- 检查系统可用内存
- 减小模型大小或批量大小
- 检查张量形状是否合理

### OH_NN_OPERATION_FORBIDDEN (4)

**说明**: 当前状态下不允许执行该操作。

**常见原因**:
- 模型已构建完成，再次调用构建接口
- 编译已完成，再次调用编译接口
- 异步执行接口未实现

**处理**: 检查调用时序，确保在正确状态下调用。

**示例**:
```c
// 错误：重复调用 Finish
OH_NNModel_Finish(model);
OH_NNModel_Finish(model);  // 返回 OH_NN_OPERATION_FORBIDDEN

// 正确
OH_NNModel_Finish(model);
// 不再调用 Finish
```

### OH_NN_NULL_PTR (5)

**说明**: 传入空指针。

**常见原因**:
- 必需的参数为 NULL
- 未初始化的句柄

**处理**: 检查所有参数是否已正确初始化。

### OH_NN_INVALID_FILE (6)

**说明**: 文件无效。

**常见原因**:
- 文件不存在
- 文件格式错误
- 文件权限不足
- 文件被损坏

**处理**: 检查文件路径、权限和完整性。

### OH_NN_UNAVALIDABLE_DEVICE (7) [已废弃]

**说明**: 设备不可用（API 11 后使用 OH_NN_UNAVAILABLE_DEVICE）。

**常见原因**:
- 设备离线
- 设备不支持模型中的算子
- HDI 服务未启动

**处理**: 使用 `OH_NN_UNAVAILABLE_DEVICE` 替代。

### OH_NN_INVALID_PATH (8)

**说明**: 路径无效。

**常见原因**:
- 缓存路径不存在
- 路径权限不足
- 路径包含非法字符

**处理**: 确保路径存在且有写权限。

### OH_NN_TIMEOUT (9)

**说明**: 操作超时。

**常见原因**:
- 推理执行时间过长
- IPC 调用超时
- 设备响应超时

**处理**: 检查设备状态，可能需要重启服务。

### OH_NN_UNSUPPORTED (10)

**说明**: 功能不支持。

**常见原因**:
- 设备不支持该功能
- API 版本不支持
- 算子不支持

**处理**: 检查设备能力和 API 版本。

### OH_NN_CONNECTION_EXCEPTION (11)

**说明**: 连接异常。

**常见原因**:
- HDI 服务进程崩溃
- IPC 连接断开
- 设备驱动异常

**处理**: 检查 nnrt_host 进程状态，可能需要重新编译模型。

### OH_NN_SAVE_CACHE_EXCEPTION (12)

**说明**: 缓存保存异常。

**常见原因**:
- 磁盘空间不足
- 缓存文件写入失败
- 缓存校验失败

**处理**: 检查磁盘空间和缓存目录权限。

### OH_NN_DYNAMIC_SHAPE (13)

**说明**: 动态形状相关错误。

**常见原因**:
- 动态形状输入未设置具体值
- 输出形状计算错误

**处理**: 确保动态维度已正确设置。

### OH_NN_UNAVAILABLE_DEVICE (14)

**说明**: 设备不可用。

**常见原因**:
- HDI 服务异常
- 设备离线
- 设备被占用

**处理**: 检查设备状态，重启 HDI 服务。

## 错误处理示例

```c
#include "neural_network_runtime/neural_network_runtime_type.h"
#include <stdio.h>

const char* GetErrorString(OH_NN_ReturnCode code) {
    switch (code) {
        case OH_NN_SUCCESS: return "Success";
        case OH_NN_FAILED: return "Failed";
        case OH_NN_INVALID_PARAMETER: return "Invalid parameter";
        case OH_NN_MEMORY_ERROR: return "Memory error";
        case OH_NN_OPERATION_FORBIDDEN: return "Operation forbidden";
        case OH_NN_NULL_PTR: return "Null pointer";
        case OH_NN_INVALID_FILE: return "Invalid file";
        case OH_NN_UNAVALIDABLE_DEVICE: return "Device unavailable (deprecated)";
        case OH_NN_INVALID_PATH: return "Invalid path";
        case OH_NN_TIMEOUT: return "Timeout";
        case OH_NN_UNSUPPORTED: return "Unsupported";
        case OH_NN_CONNECTION_EXCEPTION: return "Connection exception";
        case OH_NN_SAVE_CACHE_EXCEPTION: return "Save cache exception";
        case OH_NN_DYNAMIC_SHAPE: return "Dynamic shape";
        case OH_NN_UNAVAILABLE_DEVICE: return "Device unavailable";
        default: return "Unknown error";
    }
}

void HandleError(OH_NN_ReturnCode code, const char* operation) {
    if (code != OH_NN_SUCCESS) {
        printf("Error in %s: %s (code %d)\n", operation, GetErrorString(code), code);
        
        // 根据错误码采取不同处理
        switch (code) {
            case OH_NN_MEMORY_ERROR:
                printf("Suggestion: Check available memory\n");
                break;
            case OH_NN_INVALID_PARAMETER:
                printf("Suggestion: Check API parameters\n");
                break;
            case OH_NN_UNAVAILABLE_DEVICE:
                printf("Suggestion: Check device status and restart nnrt_host\n");
                break;
            case OH_NN_CONNECTION_EXCEPTION:
                printf("Suggestion: Recompile the model\n");
                break;
        }
    }
}

// 使用示例
int main() {
    OH_NNModel* model = OH_NNModel_Construct();
    if (model == NULL) {
        HandleError(OH_NN_MEMORY_ERROR, "OH_NNModel_Construct");
        return -1;
    }
    
    // ... 其他操作 ...
    
    OH_NN_ReturnCode ret = OH_NNModel_Finish(model);
    HandleError(ret, "OH_NNModel_Finish");
    
    return 0;
}
```

## 相关跳转

- [API 速查](API_Quick_Reference.md)
- [对外 API](../04_Native_API.md)
- [常见问题](../09_Troubleshooting.md)
