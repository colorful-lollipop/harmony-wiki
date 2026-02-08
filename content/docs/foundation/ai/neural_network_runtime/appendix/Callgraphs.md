# 附录 A: 关键调用链

## 模型构建调用链

```
OH_NNModel_Construct() [neural_network_runtime.cpp]
    └── new InnerModel() [inner_model.cpp]
        └── InnerModel::InnerModel()
            └── m_liteGraph = std::make_shared<LiteGraph>()

OH_NNModel_AddTensorToModel(tensorDesc) [neural_network_runtime.cpp]
    └── InnerModel::AddTensor(tensor) [inner_model.cpp]
        ├── validation: ValidateTensorDesc()
        ├── tensor = std::make_shared<NNTensor>()
        └── m_tensors.emplace_back(tensor)

OH_NNModel_AddOperation(op, params, inputs, outputs) [neural_network_runtime.cpp]
    └── InnerModel::AddOperation(...) [inner_model.cpp]
        ├── OpsRegistry::GetInstance() [ops_registry.cpp]
        ├── OpsRegistry::GetOpsBuilder(op) [ops_registry.cpp]
        ├── OpsBuilder::Build(...) [ops/*_builder.cpp]
        │   ├── 验证参数数量和类型
        │   ├── 创建 MindIR Primitive
        │   └── 设置算子属性
        └── 添加到 m_liteGraph

OH_NNModel_Finish() [neural_network_runtime.cpp]
    └── InnerModel::Build() [inner_model.cpp]
        ├── 验证模型完整性
        ├── 验证输入输出
        └── m_isBuild = true
```

## 编译调用链

```
OH_NNCompilation_Construct(model) [neural_network_core.cpp]
    └── new NNCompiler(model) [nncompiler.cpp]
        ├── 保存 model 引用
        └── m_backend = BackendManager::GetBackend(deviceID)

OH_NNCompilation_SetDevice(deviceID) [neural_network_core.cpp]
    └── NNCompiler::SetDevice(deviceID) [nncompiler.cpp]
        ├── BackendManager::GetInstance() [backend_manager.cpp]
        ├── BackendManager::GetBackend(deviceID) [backend_manager.cpp]
        └── m_backend = backend

OH_NNCompilation_Build() [neural_network_core.cpp]
    └── NNCompiler::Build() [nncompiler.cpp]
        ├── m_backend->GetSupportedOperation(model) [nnbackend.cpp]
        │   └── Device::GetSupportedOperation(liteGraph) [device.h]
        │       └── HDIDeviceV2_1::GetSupportedOperation() [hdi_device_v2_1.cpp]
        │           └── m_iDevice->GetSupportedOperation() [IPC]
        ├── m_backend->CreateCompiler() [nnbackend.cpp]
        │   └── return shared_from_this()
        ├── Device::PrepareModel(liteGraph, config, preparedModel) [device.h]
        │   └── HDIDeviceV2_1::PrepareModel() [hdi_device_v2_1.cpp]
        │       ├── LiteGraphToHdiModel(liteGraph, hdiModel) [lite_graph_to_hdi_model_v2_1.cpp]
        │       └── m_iDevice->PrepareModel(hdiModel, config, iPreparedModel) [IPC]
        │           └── nnrt_host 进程: NnrtDeviceService::PrepareModel() [nnrt_device_service.cpp]
        │               ├── 验证模型
        │               ├── 编译模型
        │               └── preparedModel = new PreparedModelService(...) [prepared_model_service.cpp]
        ├── m_preparedModel = preparedModel
        └── SaveToCacheFile() [nncompiled_cache.cpp] (如果设置了缓存)
```

## 执行调用链

```
OH_NNExecutor_Construct(compilation) [neural_network_core.cpp]
    └── NNCompiler::CreateExecutor() [nncompiler.cpp]
        └── new NNExecutor(m_preparedModel, ...) [nnexecutor.cpp]
            ├── 保存 preparedModel
            ├── 创建输入输出 TensorDesc
            └── 启动自动卸载线程

OH_NNTensor_Create(deviceID, tensorDesc) [neural_network_core.cpp]
    └── NNBackend::CreateTensor(deviceID) [nnbackend.cpp]
        └── new NNTensor2_0(deviceID) [nntensor.cpp]
            ├── 分配共享内存
            └── 映射缓冲区

OH_NNExecutor_RunSync(executor, inputs, outputs) [neural_network_core.cpp]
    └── NNExecutor::RunSync(inputs, outputs) [nnexecutor.cpp]
        ├── 验证输入张量
        ├── 检查输入维度范围
        ├── PreparedModel::Run(inputs, outputs, ...) [prepared_model.h]
        │   └── HDIPreparedModelV2_1::Run(...) [hdi_prepared_model_v2_1.cpp]
        │       ├── 转换张量为 HDI IOTensor
        │       └── m_hdiPreparedModel->Run(iInputs, iOutputs, ...) [IPC]
        │           └── nnrt_host 进程: PreparedModelService::Run() [prepared_model_service.cpp]
        │               ├── 验证输入
        │               ├── 执行推理 (MindSpore Lite)
        │               └── 填充输出
        ├── 更新输出张量维度
        └── 重置自动卸载定时器
```

## 设备发现调用链

```
OH_NNDevice_GetAllDevicesID(deviceIDs, deviceCount) [neural_network_core.cpp]
    └── BackendManager::GetInstance() [backend_manager.cpp]
        └── BackendManager::GetAllBackendsID() [backend_manager.cpp]
            ├── 首次调用时初始化
            │   └── RegisterBackendByHdi() [register_hdi_device_v2_1.cpp]
            │       ├── V2_1::INnrtDevice::Get() [HDI Proxy]
            │       ├── iDevice->GetDeviceName() [IPC]
            │       └── BackendManager::RegisterBackend(name, creator) [backend_manager.cpp]
            └── return m_backendIDs
```

## 后端注册调用链

```
// 库加载时自动注册
static BackendRegistrar g_backendRegistrarNNRT(NNRT_BACKEND_NAME, []() {
    return std::make_shared<NNBackend>();
}); [backend_registrar.cpp]

BackendRegistrar::BackendRegistrar(name, creator) [backend_registrar.cpp]
    └── BackendManager::GetInstance().RegisterBackend(name, creator) [backend_manager.cpp]
        ├── 验证 backend 有效性
        ├── m_backends[backendID] = backend
        └── m_backendNames[backendID] = name
```

## 算子构建调用链

```
// 以 Add 算子为例
InnerModel::AddOperation(OH_NN_OPS_ADD, ...)
    └── OpsRegistry::GetInstance()->GetOpsBuilder(OH_NN_OPS_ADD) [ops_registry.cpp]
        └── return m_opsBuilder[OH_NN_OPS_ADD] (AddBuilder 实例)

AddBuilder::Build(params, inputs, outputs, allTensors) [ops/add_builder.cpp]
    ├── 验证输入输出数量
    ├── 获取参数值
    │   └── OpsBuilder::GetParamValue(...) [ops_builder.cpp]
    ├── 创建 MindIR Primitive
    │   └── std::make_shared<Add>()
    ├── 设置激活类型 (如果有)
    └── return primitive
```

## 内存管理调用链

```
OH_NNTensor_Create(deviceID, tensorDesc) [neural_network_core.cpp]
    └── NNTensor::Create(deviceID, tensorDesc) [nntensor.cpp]
        ├── OH_NNTensorDesc_GetByteSize(tensorDesc, &size)
        ├── Device::AllocateTensorBuffer(size, tensorDesc) [device.h]
        │   └── HDIDeviceV2_1::AllocateTensorBuffer(...) [hdi_device_v2_1.cpp]
        │       ├── m_iDevice->AllocateBuffer(length, buffer) [IPC]
        │       └── MemoryManager::GetInstance().MapMemory(fd, length) [memory_manager.cpp]
        │           ├── mmap(nullptr, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0)
        │           └── m_memorys[buffer] = Memory{fd, buffer, length}
        └── 保存 buffer 到 NNTensor

OH_NNTensor_Destroy(tensor) [neural_network_core.cpp]
    └── NNTensor::~NNTensor() [nntensor.cpp]
        ├── Device::ReleaseBuffer(buffer) [device.h]
        │   └── MemoryManager::GetInstance().UnMapMemory(buffer) [memory_manager.cpp]
        │       ├── munmap(buffer, length)
        │       └── m_memorys.erase(buffer)
        └── 释放 tensor 资源
```

## 缓存调用链

```
OH_NNCompilation_SetCache(compilation, cachePath, version) [neural_network_core.cpp]
    └── NNCompiler::SetCacheDir(cachePath, version) [nncompiler.cpp]
        └── m_cacheDir = cachePath
        └── m_cacheVersion = version

OH_NNCompilation_Build() [neural_network_core.cpp]
    └── NNCompiler::Build() [nncompiler.cpp]
        ├── CheckCacheExist() [nncompiled_cache.cpp]
        │   └── 检查缓存文件是否存在
        ├── 如果缓存存在:
        │   └── LoadFromCacheFile() [nncompiled_cache.cpp]
        │       ├── 读取缓存文件
        │       ├── 验证 CRC 校验和
        │       └── Device::PrepareModelFromModelCache(...) [device.h]
        └── 如果缓存不存在:
            ├── 正常编译
            └── SaveToCacheFile() [nncompiled_cache.cpp]
                ├── 序列化模型
                ├── 计算 CRC 校验和
                └── 写入缓存文件
```

## 相关跳转

- [对外 API](04_Native_API.md)
- [内部 API](05_Inner_API.md)
- [架构说明](02_Architecture.md)
