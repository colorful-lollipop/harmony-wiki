# 05. 内部 API

## 目的

本文档介绍 Neural Network Runtime 的内部 API，包括模块接口、依赖方向、稳定性和可替换点。

## 适用范围

- 框架开发者
- 需要扩展 NNRt 的开发者

## 内部 API 清单

### 1. 内部模型加载 API

**头文件**: `interfaces/innerkits/c/neural_network_runtime_inner.h`

| API | 功能 | 稳定性 |
|-----|------|--------|
| `OH_NNModel_BuildFromLiteGraph()` | 从 LiteGraph 加载模型 | 不稳定 |
| `OH_NNModel_SetInputsAndOutputsInfo()` | 设置 MetaGraph 输入输出 | 不稳定 |
| `OH_NNModel_BuildFromMetaGraph()` | 从 MetaGraph 加载模型 | 不稳定 |
| `OH_NNModel_HasCache()` | 检查缓存是否存在 | 不稳定 |
| `OH_NN_GetDeviceID()` | 获取设备 ID | 不稳定 |
| `OH_NN_IsSupportAIPP()` | 检查 AIPP 支持 | 不稳定 |
| `OH_NNExecutor_RunSyncWithAipp()` | 带 AIPP 的同步执行 | 不稳定 |
| `CacheInfoGetCrc16()` | 计算 CRC16 校验和 | 不稳定 |

**证据**: `interfaces/innerkits/c/neural_network_runtime_inner.h:75-204`

## Core 层内部接口

### Backend 抽象类

**文件**: `frameworks/native/neural_network_core/backend.h`

```cpp
class Backend {
public:
    virtual ~Backend() = default;

    // 设备信息查询
    virtual const std::string& GetBackendName() = 0;
    virtual const std::string& GetBackendVersion() = 0;
    
    // 创建对象
    virtual std::shared_ptr<Compiler> CreateCompiler() = 0;
    virtual std::shared_ptr<Executor> CreateExecutor() = 0;
    virtual std::shared_ptr<Tensor> CreateTensor() = 0;
    
    // 设备查询
    virtual OH_NN_ReturnCode GetDeviceName(std::string& name) = 0;
    virtual OH_NN_ReturnCode GetVendorName(std::string& name) = 0;
    virtual OH_NN_ReturnCode GetVersion(std::string& version) = 0;
    virtual OH_NN_ReturnCode GetDeviceType(OH_NN_DeviceType& deviceType) = 0;
    virtual OH_NN_ReturnCode GetDeviceStatus(DeviceStatus& status) = 0;
};
```

**稳定性**: 稳定（Core 层接口）

### Compiler 抽象类

**文件**: `frameworks/native/neural_network_core/compiler.h`

```cpp
class Compiler {
public:
    virtual ~Compiler() = default;

    // 编译配置
    virtual OH_NN_ReturnCode SetModel(std::shared_ptr<const InnerModel> model) = 0;
    virtual OH_NN_ReturnCode SetModel(const void* modelBuffer, size_t modelSize) = 0;
    virtual OH_NN_ReturnCode SetCacheDir(const std::string& cacheDir, uint32_t version) = 0;
    virtual OH_NN_ReturnCode SetPerformanceMode(OH_NN_PerformanceMode performance) = 0;
    virtual OH_NN_ReturnCode SetPriority(OH_NN_Priority priority) = 0;
    virtual OH_NN_ReturnCode EnableFloat16(bool isFloat16) = 0;
    
    // 编译执行
    virtual OH_NN_ReturnCode Build() = 0;
    
    // 创建执行器
    virtual std::shared_ptr<Executor> CreateExecutor() = 0;
    
    // 缓存导出
    virtual OH_NN_ReturnCode ExportCacheToBuffer(uint8_t* buffer, size_t length, size_t* modelSize) = 0;
    virtual OH_NN_ReturnCode ImportCacheFromBuffer(const uint8_t* buffer, size_t modelSize) = 0;
};
```

**稳定性**: 稳定（Core 层接口）

### Executor 抽象类

**文件**: `frameworks/native/neural_network_core/executor.h`

```cpp
class Executor {
public:
    virtual ~Executor() = default;

    // 执行推理
    virtual OH_NN_ReturnCode RunSync(const std::vector<std::shared_ptr<Tensor>>& inputs,
                                     const std::vector<std::shared_ptr<Tensor>>& outputs) = 0;
    virtual OH_NN_ReturnCode RunAsync(const std::vector<std::shared_ptr<Tensor>>& inputs,
                                      const std::vector<std::shared_ptr<Tensor>>& outputs,
                                      void* userData) = 0;
    
    // 回调设置
    virtual OH_NN_ReturnCode SetOnRunDone(NN_OnRunDone onRunDone) = 0;
    virtual OH_NN_ReturnCode SetOnServiceDied(NN_OnServiceDied onServiceDied) = 0;
    
    // 输出信息
    virtual OH_NN_ReturnCode GetOutputShape(uint32_t outputIndex, int32_t** shape, uint32_t* shapeLength) = 0;
};
```

**稳定性**: 稳定（Core 层接口）

## Runtime 层内部接口

### Device 抽象类

**文件**: `frameworks/native/neural_network_runtime/device.h`

```cpp
class Device {
public:
    Device() = default;
    virtual ~Device() = default;

    // 设备信息
    virtual OH_NN_ReturnCode GetDeviceName(std::string& name) = 0;
    virtual OH_NN_ReturnCode GetVendorName(std::string& name) = 0;
    virtual OH_NN_ReturnCode GetVersion(std::string& version) = 0;
    virtual OH_NN_ReturnCode GetDeviceType(OH_NN_DeviceType& deviceType) = 0;
    virtual OH_NN_ReturnCode GetDeviceStatus(DeviceStatus& status) = 0;
    
    // 能力查询
    virtual OH_NN_ReturnCode GetSupportedOperation(std::shared_ptr<const mindspore::lite::LiteGraph> model,
                                                   std::vector<bool>& ops) = 0;
    virtual OH_NN_ReturnCode IsFloat16PrecisionSupported(bool& isSupported) = 0;
    virtual OH_NN_ReturnCode IsPerformanceModeSupported(bool& isSupported) = 0;
    virtual OH_NN_ReturnCode IsPrioritySupported(bool& isSupported) = 0;
    virtual OH_NN_ReturnCode IsDynamicInputSupported(bool& isSupported) = 0;
    virtual OH_NN_ReturnCode IsModelCacheSupported(bool& isSupported) = 0;
    
    // 模型准备
    virtual OH_NN_ReturnCode PrepareModel(std::shared_ptr<const mindspore::lite::LiteGraph> model,
                                          const ModelConfig& config,
                                          std::shared_ptr<PreparedModel>& preparedModel) = 0;
    virtual OH_NN_ReturnCode PrepareModelFromModelCache(const std::vector<Buffer>& modelCache,
                                                        const ModelConfig& config,
                                                        std::shared_ptr<PreparedModel>& preparedModel,
                                                        bool& isUpdatable) = 0;
    
    // 内存管理
    virtual void* AllocateBuffer(size_t length) = 0;
    virtual OH_NN_ReturnCode ReleaseBuffer(const void* buffer) = 0;
    virtual OH_NN_ReturnCode AllocateBuffer(size_t length, int& fd) = 0;
    virtual OH_NN_ReturnCode ReleaseBuffer(int fd, size_t length) = 0;
};
```

**稳定性**: 中等（Runtime 层接口，可能随 HDI 版本变化）

**证据**: `frameworks/native/neural_network_runtime/device.h:32-76`

### PreparedModel 抽象类

**文件**: `frameworks/native/neural_network_runtime/prepared_model.h`

```cpp
class PreparedModel {
public:
    PreparedModel() = default;
    virtual ~PreparedModel() = default;

    // 执行推理
    virtual OH_NN_ReturnCode Run(const std::vector<IOTensor>& inputs,
                                 const std::vector<IOTensor>& outputs,
                                 std::vector<std::vector<int32_t>>& outputsDims,
                                 std::vector<bool>& isOutputBufferEnough) = 0;
    
    // 获取输入维度范围
    virtual OH_NN_ReturnCode GetInputDimRanges(std::vector<std::vector<uint32_t>>& minInputDims,
                                               std::vector<std::vector<uint32_t>>& maxInputDims) = 0;
    
    // 导出缓存
    virtual OH_NN_ReturnCode ExportModelCache(std::vector<Buffer>& modelCache) = 0;
};
```

**稳定性**: 中等（Runtime 层接口）

**证据**: `frameworks/native/neural_network_runtime/prepared_model.h`

### InnerModel 类

**文件**: `frameworks/native/neural_network_runtime/inner_model.h`

```cpp
class InnerModel {
public:
    // 构建模型
    OH_NN_ReturnCode AddTensor(const std::shared_ptr<NNTensor>& tensor);
    OH_NN_ReturnCode AddOperation(OpsType opsType,
                                  const std::vector<uint32_t>& params,
                                  const std::vector<uint32_t>& inputs,
                                  const std::vector<uint32_t>& outputs);
    OH_NN_ReturnCode SpecifyInputsAndOutputs(const std::vector<uint32_t>& inputs,
                                             const std::vector<uint32_t>& outputs);
    OH_NN_ReturnCode Build();
    
    // 从 LiteGraph/MetaGraph 构建
    OH_NN_ReturnCode BuildFromLiteGraph(const mindspore::lite::LiteGraph* graph);
    OH_NN_ReturnCode BuildFromMetaGraph(const mindspore::schema::MetaGraphT* graph);
    
    // 获取模型信息
    std::shared_ptr<const mindspore::lite::LiteGraph> GetLiteGraph() const;
    const std::vector<std::shared_ptr<NNTensor>>& GetTensors() const;
    
    // 查询设备支持
    OH_NN_ReturnCode GetSupportedOperation(size_t deviceID, std::vector<bool>& ops) const;
};
```

**稳定性**: 低（内部实现细节，可能变化）

## 模块依赖关系

### 依赖方向图

```
interfaces/kits/c/ (对外 API)
         │
         │ 使用
         ▼
frameworks/native/neural_network_runtime/ (Runtime 实现)
         │
         ├── 使用 ──► interfaces/innerkits/c/ (内部 API)
         │
         ├── 实现 ──► frameworks/native/neural_network_core/ (Core 抽象)
         │                │
         │                │ 使用
         │                ▼
         │           common/ (公共组件)
         │
         ├── 使用 ──► drivers_interface_nnrt (HDI 接口)
         │
         └── 使用 ──► mindspore (MindIR)
```

### 依赖规则

1. **Core 层** 不依赖 Runtime 层
2. **Runtime 层** 依赖 Core 层
3. **对外 API** 依赖 Runtime 层
4. **内部 API** 被 Runtime 层使用
5. **HDI 接口** 由 Runtime 层使用

## 接口稳定性分级

| 级别 | 说明 | 接口示例 |
|------|------|----------|
| **稳定** | 不会变化，可放心依赖 | Core 层抽象接口 (Backend, Compiler, Executor) |
| **中等** | 可能随版本变化，需关注更新 | Runtime 层接口 (Device, PreparedModel) |
| **不稳定** | 可能随时变化，不建议依赖 | 内部 API (BuildFromLiteGraph, AIPP 接口) |

## 可替换点

### 1. 后端设备替换

通过实现 `Device` 和 `PreparedModel` 接口，可以添加新的后端设备支持。

```cpp
// 自定义设备实现
class MyDevice : public Device {
    // 实现 Device 接口
};

class MyPreparedModel : public PreparedModel {
    // 实现 PreparedModel 接口
};

// 注册到 BackendManager
BackendManager::GetInstance().RegisterBackend("MyBackend", []() {
    return std::make_shared<MyBackend>();
});
```

### 2. 算子扩展

通过继承 `OpsBuilder` 并注册到 `OpsRegistry`，可以添加新的算子支持。

```cpp
// 自定义算子构建器
class MyOpBuilder : public OpsBuilder {
    OH_NN_ReturnCode Build(...) override {
        // 实现算子构建逻辑
    }
};

// 注册算子
REGISTER_OPS(MyOpBuilder, OH_NN_OPS_MY_OP);
```

### 3. HDI 版本适配

通过实现不同版本的 `HDIDevice` 和 `HDIPreparedModel`，可以适配新的 HDI 版本。

```cpp
// HDI v3.0 适配
class HDIDeviceV3_0 : public Device {
    // 实现 v3.0 适配
};
```

## 相关跳转

- [对外 API](04_Native_API.md)
- [架构说明](02_Architecture.md)
- [目录结构](03_Directory_Structure.md)
