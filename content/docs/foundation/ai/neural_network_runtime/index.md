# Neural Network Runtime 概览

## 项目定位

**Neural Network Runtime (NNRt)** 是 OpenHarmony 的神经网络运行时，作为连接上层 AI 推理框架与底层 AI 加速芯片的桥梁。

```
┌─────────────────────────────────────────────────────────────┐
│                    AI Application (JS/TS)                   │
├─────────────────────────────────────────────────────────────┤
│              AI Inference Framework                          │
│         (MindSpore Lite / TensorFlow Lite)                  │
├─────────────────────────────────────────────────────────────┤
│           Neural Network Runtime (本仓库)                    │
│    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│    │   Core API  │    │   Runtime   │    │   HDI Client│   │
│    └─────────────┘    └─────────────┘    └─────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                    HDF IPC (Binder)                          │
├─────────────────────────────────────────────────────────────┤
│              Device Driver (HDI Service)                     │
│              (NPU / DSP / GPU / CPU)                         │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

### 1. 跨芯片推理

- 支持多种 AI 加速芯片（NPU、DSP、GPU、CPU）
- 统一的 C API 接口，屏蔽底层硬件差异
- 通过 HDI (Hardware Device Interface) 与芯片驱动通信

### 2. MindIR 统一中间表示

- 使用 MindSpore Lite 的 LiteGraph 作为中间表示
- 减少模型转换开销，提高传输效率
- 支持从 LiteGraph 和 MetaGraph 直接加载模型

### 3. 模型编译与缓存

- 在线模型编译（从算子图编译）
- 离线模型加载（预编译模型文件）
- 模型缓存机制，加速后续加载

### 4. 性能优化

- 支持 Float16 推理
- 性能模式设置（低功耗/中等/高/极致）
- 任务优先级设置
- 动态形状支持

## 运行环境

### 系统要求

- **OpenHarmony 版本**: 3.2+ (API 9+)
- **系统类型**: Standard System
- **SysCap**: `SystemCapability.AI.NeuralNetworkRuntime`

### 资源占用

- **ROM**: 1024 KB
- **RAM**: 2048 KB

### 依赖组件

```json
{
  "deps": {
    "components": [
      "c_utils",
      "drivers_interface_nnrt",
      "hdf_core",
      "hilog",
      "hitrace",
      "ipc",
      "mindspore",
      "init",
      "json",
      "jsoncpp",
      "eventhandler",
      "openssl"
    ]
  }
}
```

## 关键概念

### 1. Model (模型)

表示一个神经网络模型，由张量和算子组成。

```c
OH_NNModel *model = OH_NNModel_Construct();
// 添加张量、算子，构建模型拓扑
OH_NNModel_Finish(model);
```

### 2. Compilation (编译)

将模型编译为可在特定设备上执行的格式。

```c
OH_NNCompilation *compilation = OH_NNCompilation_Construct(model);
OH_NNCompilation_SetDevice(compilation, deviceID);
OH_NNCompilation_Build(compilation);
```

### 3. Executor (执行器)

执行编译后的模型进行推理。

```c
OH_NNExecutor *executor = OH_NNExecutor_Construct(compilation);
OH_NNExecutor_RunSync(executor, inputs, inputCount, outputs, outputCount);
```

### 4. Tensor (张量)

多维数组，是模型的基本数据单元。

```c
NN_TensorDesc *desc = OH_NNTensorDesc_Create();
OH_NNTensorDesc_SetDataType(desc, OH_NN_FLOAT32);
OH_NNTensorDesc_SetShape(desc, shape, shapeLength);
NN_Tensor *tensor = OH_NNTensor_Create(deviceID, desc);
```

### 5. Backend (后端)

抽象的设备后端，封装了特定硬件的能力。

### 6. HDI (Hardware Device Interface)

硬件设备接口，定义了 NNRt 与芯片驱动之间的通信协议。

## 版本信息

- **当前版本**: 4.0
- **API 版本**: 9 (基础), 11 (扩展)
- **HDI 版本**: 1.0, 2.0, 2.1 (同时支持)

## 相关资源

- [官方 API 文档](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/reference/apis-neural-network-runtime-kit)
- [HDI 接口定义](https://gitee.com/openharmony/drivers_interface/tree/master/nnrt)
- [开发指南](../neural-network-runtime-guidelines.md)

## 下一步

- [了解架构设计](02_Architecture.md)
- [查看目录结构](03_Directory_Structure.md)
- [学习 API 使用](04_Native_API.md)
