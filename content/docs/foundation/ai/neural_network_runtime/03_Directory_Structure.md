# 03. 目录结构与模块职责

## 目的

本文档介绍 Neural Network Runtime 的目录结构、各模块的职责和关键文件位置。

## 适用范围

- 新加入的开发者
- 需要定位代码的维护者

## 顶层目录结构

```
foundation/ai/neural_network_runtime/
├── common/                    # 公共组件
├── config/                    # 构建配置
├── example/                   # 开发示例
│   ├── deep_learning_framework/  # AI 框架开发示例
│   └── drivers/               # 芯片驱动开发示例
├── figures/                   # 文档图片
├── frameworks/                # 框架实现
│   └── native/                # Native 实现
│       ├── neural_network_core/    # Core 抽象层
│       └── neural_network_runtime/ # Runtime 实现层
├── interfaces/                # API 接口
│   ├── innerkits/             # 内部 API
│   └── kits/                  # 对外 API
├── test/                      # 测试代码 (本文档不详细展开)
├── BUILD.gn                   # 根构建文件
├── bundle.json                # 组件配置
└── neural-network-runtime-guidelines.md  # 开发指南
```

## 模块详细说明

### 1. common/ - 公共组件

**职责**: 提供跨模块使用的公共工具和数据结构。

```
common/
├── log.h                      # 日志宏定义
├── mindir.h                   # MindIR 相关定义
├── nn_types.h                 # 公共类型定义
├── nn_tensor.h                # 张量公共定义
├── utils.h                    # 工具函数
└── ...
```

**关键文件**:
- `log.h` - 基于 hilog 的日志宏
- `mindir.h` - MindIR 模型定义

### 2. config/ - 构建配置

**职责**: 提供构建时的配置选项。

```
config/
└── BUILD.gn                   # 覆盖率等编译选项配置
```

**关键配置**:
- `neural_network_runtime_coverage` - 代码覆盖率开关

### 3. example/ - 开发示例

#### 3.1 deep_learning_framework/ - AI 框架开发示例

**职责**: 展示如何在 AI 框架中集成 NNRt。

```
example/deep_learning_framework/
├── tflite/                    # TensorFlow Lite 集成示例
│   ├── delegates/nnrt_delegate/  # NNRT Delegate 实现
│   ├── label_classify/        # 图像分类示例
│   ├── nnrt/                  # NNRT 直接调用示例
│   └── tools/                 # 工具脚本
└── cmake_build/               # CMake 构建示例
```

**关键示例**:
- `tflite/delegates/nnrt_delegate/` - TensorFlow Lite 的 NNRT Delegate 实现
- `tflite/nnrt/` - 直接使用 NNRt API 的示例

#### 3.2 drivers/ - 芯片驱动开发示例

**职责**: 展示如何实现 HDI 芯片驱动。

```
example/drivers/
├── README_zh.md               # 驱动开发指南
├── nnrt/
│   ├── v1_0/                  # HDI v1.0 示例
│   │   ├── BUILD.gn
│   │   └── hdi_cpu_service/   # CPU 后端服务实现
│   │       ├── include/       # 头文件
│   │       └── src/           # 源文件
│   └── v2_0/                  # HDI v2.0 示例
│       ├── BUILD.gn
│       └── hdi_cpu_service/   # CPU 后端服务实现
│           ├── include/
│           └── src/
```

**关键文件**:
- `nnrt/v2_0/hdi_cpu_service/src/nnrt_device_service.cpp` - 设备服务实现
- `nnrt/v2_0/hdi_cpu_service/src/prepared_model_service.cpp` - 模型服务实现
- `nnrt/v2_0/hdi_cpu_service/src/nnrt_device_driver.cpp` - 驱动入口

### 4. frameworks/native/neural_network_core/ - Core 抽象层

**职责**: 定义 NNRt 的核心抽象接口，供上层使用。

```
frameworks/native/neural_network_core/
├── BUILD.gn                   # 构建配置
├── backend.h/cpp              # Backend 抽象基类
├── backend_manager.h/cpp      # 后端管理器
├── backend_registrar.h/cpp    # 后端注册器
├── compilation.h              # 编译配置结构
├── compiler.h/cpp             # Compiler 抽象基类
├── cpp_type.h                 # C++ 类型定义
├── executor.h/cpp             # Executor 抽象基类
├── neural_network_core.cpp    # Core 层初始化
├── nn_tensor.h                # 张量定义
├── nnrt_client.h/cpp          # NNRT 客户端
├── tensor.h/cpp               # Tensor 抽象基类
├── tensor_desc.h/cpp          # 张量描述符
├── utils.h/cpp                # 工具函数
└── validation.h/cpp           # 参数验证
```

**关键类**:
- `Backend` - 后端抽象基类
- `Compiler` - 编译器抽象基类
- `Executor` - 执行器抽象基类
- `BackendManager` - 后端管理器单例

**产物**: `libneural_network_core.so`

### 5. frameworks/native/neural_network_runtime/ - Runtime 实现层

**职责**: 实现 NNRt 的核心功能，包括模型管理、编译、执行、设备适配等。

```
frameworks/native/neural_network_runtime/
├── BUILD.gn                   # 构建配置
├── neural_network_runtime.cpp # 对外 API 实现
├── neural_network_runtime_compat.cpp  # 兼容层
│
├── inner_model.h/cpp          # 内部模型表示
├── nnbackend.h/cpp            # Backend 实现
├── nncompiler.h/cpp           # Compiler 实现
├── nnexecutor.h/cpp           # Executor 实现
├── nntensor.h/cpp             # 张量实现
├── nn_tensor.h                # 张量定义
├── quant_param.h/cpp          # 量化参数
│
├── device.h                   # 设备抽象基类
├── prepared_model.h           # 预编译模型抽象基类
├── cpp_type.h                 # C++ 类型定义
├── transform.h/cpp            # 数据转换
├── memory_manager.h/cpp       # 内存管理器
│
├── hdi_device_v1_0.h/cpp      # HDI v1.0 设备适配
├── hdi_device_v2_0.h/cpp      # HDI v2.0 设备适配
├── hdi_device_v2_1.h/cpp      # HDI v2.1 设备适配
├── hdi_prepared_model_v1_0.h/cpp  # HDI v1.0 模型适配
├── hdi_prepared_model_v2_0.h/cpp  # HDI v2.0 模型适配
├── hdi_prepared_model_v2_1.h/cpp  # HDI v2.1 模型适配
├── hdi_returncode_utils_v2_1.h    # HDI 错误码转换
├── register_hdi_device_v1_0.cpp   # HDI v1.0 注册
├── register_hdi_device_v2_0.cpp   # HDI v2.0 注册
├── register_hdi_device_v2_1.cpp   # HDI v2.1 注册
│
├── lite_graph_to_hdi_model_v1_0.h/cpp  # LiteGraph 转 HDI v1.0
├── lite_graph_to_hdi_model_v2_0.h/cpp  # LiteGraph 转 HDI v2.0
├── lite_graph_to_hdi_model_v2_1.h/cpp  # LiteGraph 转 HDI v2.1
│
├── ops_builder.h/cpp          # 算子构建器基类
├── ops_registry.h/cpp         # 算子注册表
├── ops/                       # 算子实现目录
│   ├── abs_builder.h/cpp
│   ├── add_builder.h/cpp
│   ├── conv2d_builder.h/cpp
│   ├── ... (110+ 个算子)
│   └── ops_validation.cpp
│
└── nncompiled_cache.h/cpp     # 编译缓存
```

**关键类**:
- `NNBackend` - Backend 实现
- `NNCompiler` - Compiler 实现
- `NNExecutor` - Executor 实现
- `InnerModel` - 内部模型表示
- `NNTensor` - 张量实现
- `Device` - 设备抽象
- `PreparedModel` - 预编译模型抽象
- `MemoryManager` - 内存管理器单例
- `OpsRegistry` - 算子注册表单例
- `HDIDeviceV2_1` - HDI 设备适配
- `HDIPreparedModelV2_1` - HDI 模型适配

**产物**: `libneural_network_runtime.so`

### 6. interfaces/ - API 接口

#### 6.1 interfaces/kits/c/ - 对外 API

**职责**: 对外暴露的 C API 接口。

```
interfaces/kits/c/
└── neural_network_runtime/
    ├── neural_network_runtime_type.h   # 类型定义和枚举
    ├── neural_network_runtime.h        # 模型构建 API
    └── neural_network_core.h           # 编译和执行 API
```

**关键 API**:
- 模型构建: `OH_NNModel_Construct()`, `OH_NNModel_AddOperation()`, `OH_NNModel_Finish()`
- 编译: `OH_NNCompilation_Construct()`, `OH_NNCompilation_Build()`
- 执行: `OH_NNExecutor_Construct()`, `OH_NNExecutor_RunSync()`
- 张量: `OH_NNTensor_Create()`, `OH_NNTensor_GetDataBuffer()`

#### 6.2 interfaces/innerkits/c/ - 内部 API

**职责**: 内部使用的 C API 接口，不对外公开。

```
interfaces/innerkits/c/
└── neural_network_runtime_inner.h    # 内部 API
```

**关键 API**:
- `OH_NNModel_BuildFromLiteGraph()` - 从 LiteGraph 加载
- `OH_NNModel_BuildFromMetaGraph()` - 从 MetaGraph 加载
- `OH_NNModel_HasCache()` - 检查缓存

## 模块依赖关系

```
interfaces/kits/c/ (对外 API)
         │
         ▼
frameworks/native/neural_network_runtime/ (Runtime 实现)
         │
         ├── interfaces/innerkits/c/ (内部 API)
         │
         ▼
frameworks/native/neural_network_core/ (Core 抽象)
         │
         ▼
common/ (公共组件)
         │
         ▼
HDI Interface (drivers_interface_nnrt)
         │
         ▼
Chip Driver (HDI Service)
```

## 关键文件索引

### 按功能分类

| 功能 | 文件路径 |
|------|----------|
| **模型构建** | `interfaces/kits/c/neural_network_runtime/neural_network_runtime.h` |
| **编译执行** | `interfaces/kits/c/neural_network_runtime/neural_network_core.h` |
| **类型定义** | `interfaces/kits/c/neural_network_runtime/neural_network_runtime_type.h` |
| **后端管理** | `frameworks/native/neural_network_core/backend_manager.h` |
| **设备抽象** | `frameworks/native/neural_network_runtime/device.h` |
| **算子注册** | `frameworks/native/neural_network_runtime/ops_registry.h` |
| **内存管理** | `frameworks/native/neural_network_runtime/memory_manager.h` |
| **HDI 适配** | `frameworks/native/neural_network_runtime/hdi_device_v2_1.h` |

### 按模块分类

| 模块 | 主要文件 |
|------|----------|
| **Core 层** | `frameworks/native/neural_network_core/*.h` |
| **Runtime 层** | `frameworks/native/neural_network_runtime/*.h` |
| **算子层** | `frameworks/native/neural_network_runtime/ops/*_builder.h` |
| **对外 API** | `interfaces/kits/c/neural_network_runtime/*.h` |
| **内部 API** | `interfaces/innerkits/c/neural_network_runtime_inner.h` |

## 相关跳转

- [架构说明](02_Architecture.md)
- [对外 API](04_Native_API.md)
- [内部 API](05_Inner_API.md)
