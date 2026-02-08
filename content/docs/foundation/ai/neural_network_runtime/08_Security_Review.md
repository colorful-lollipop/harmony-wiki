# 08. 安全风险评审

## 目的

本文档基于代码证据对 Neural Network Runtime 进行安全风险评审，识别攻击面、信任边界和可被利用点。

## 适用范围

- 安全工程师
- 架构师
- 代码审计人员

## 威胁模型

### 系统架构与信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  不信任域: 应用层 (Application)                                              │
│  - 第三方应用                                                                │
│  - 可能包含恶意代码                                                          │
├─────────────────────────────────────────────────────────────────────────────┤ ← 信任边界 1
│  半信任域: AI 框架层                                                         │
│  - MindSpore Lite / TensorFlow Lite                                         │
│  - 验证输入数据                                                              │
├─────────────────────────────────────────────────────────────────────────────┤ ← 信任边界 2
│  信任域: Neural Network Runtime (本仓库)                                     │
│  - 参数校验                                                                  │
│  - 内存管理                                                                  │
│  - IPC 通信                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤ ← 信任边界 3
│  信任域: HDF IPC 层                                                          │
│  - Binder IPC                                                                │
│  - SELinux 访问控制                                                          │
├─────────────────────────────────────────────────────────────────────────────┤ ← 信任边界 4
│  信任域: 芯片驱动层 (nnrt_host 进程)                                         │
│  - HDI Service                                                               │
│  - 硬件访问                                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 攻击面清单

### 1. API 输入攻击面

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| 模型数据 | `OH_NNModel_AddTensorToModel()` | 高 |
| 算子参数 | `OH_NNModel_AddOperation()` | 高 |
| 文件路径 | `OH_NNCompilation_ConstructWithOfflineModelFile()` | 中 |
| 缓存路径 | `OH_NNCompilation_SetCache()` | 中 |
| 缓冲区数据 | `OH_NNTensor_GetDataBuffer()` | 中 |
| 扩展配置 | `OH_NNCompilation_AddExtensionConfig()` | 中 |

### 2. IPC 攻击面

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| HDI 调用 | `INnrtDevice::PrepareModel()` | 高 |
| 模型执行 | `IPreparedModel::Run()` | 高 |
| 内存分配 | `INnrtDevice::AllocateBuffer()` | 中 |
| 缓存导出 | `IPreparedModel::ExportModelCache()` | 中 |

### 3. 文件系统攻击面

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| 缓存文件读写 | `NNCompiledCache` 类 | 中 |
| 离线模型加载 | `PrepareOfflineModel()` | 中 |
| 共享内存 fd | `OH_NNTensor_CreateWithFd()` | 中 |

## 可被利用点分析

### 风险 1: 张量数据未充分验证

**证据**: `frameworks/native/neural_network_runtime/inner_model.cpp`

```cpp
// 添加张量时仅检查基本参数
OH_NN_ReturnCode InnerModel::AddTensor(const std::shared_ptr<NNTensor>& tensor) {
    // 检查张量是否为空
    if (tensor == nullptr) {
        LOGE("AddTensor failed, tensor is nullptr.");
        return OH_NN_NULL_PTR;
    }
    
    // 检查张量是否已添加
    auto it = std::find(m_tensors.begin(), m_tensors.end(), tensor);
    if (it != m_tensors.end()) {
        LOGW("AddTensor warning, tensor has been added.");
        return OH_NN_SUCCESS;
    }
    
    // 添加到列表，未验证数据内容
    m_tensors.emplace_back(tensor);
    return OH_NN_SUCCESS;
}
```

**触发路径**:
```
OH_NNModel_AddTensorToModel() -> InnerModel::AddTensor() -> 添加恶意张量数据
```

**影响**: 恶意构造的张量数据可能导致后续算子执行时内存访问异常

**修复建议**:
1. 增加张量数据内容的验证（如检查数值范围）
2. 在算子构建阶段增加参数交叉验证
3. 对常量张量数据进行 CRC 校验

### 风险 2: 文件路径遍历

**证据**: `frameworks/native/neural_network_runtime/nncompiler.cpp`

```cpp
// 设置缓存目录，未充分验证路径
OH_NN_ReturnCode NNCompiler::SetCacheDir(const std::string& cacheDir, uint32_t version) {
    // 仅检查空字符串
    if (cacheDir.empty()) {
        LOGE("SetCacheDir failed, cacheDir is empty.");
        return OH_NN_INVALID_PARAMETER;
    }
    
    // 未检查路径遍历攻击 (../)
    m_cacheDir = cacheDir;
    m_cacheVersion = version;
    return OH_NN_SUCCESS;
}
```

**触发路径**:
```
OH_NNCompilation_SetCache() -> NNCompiler::SetCacheDir() -> 写入任意路径
```

**影响**: 可能覆盖系统文件或写入敏感目录

**修复建议**:
1. 使用 `realpath()` 解析并验证路径
2. 限制缓存目录必须在应用沙箱内
3. 检查路径不包含 `..` 或符号链接

### 风险 3: 整数溢出

**证据**: `frameworks/native/neural_network_runtime/ops/ops_builder.cpp`

```cpp
// 计算缓冲区大小可能存在溢出
size_t OpsBuilder::CalculateBufferSize(const std::vector<int32_t>& shape, OH_NN_DataType dataType) {
    size_t elementCount = 1;
    for (auto dim : shape) {
        elementCount *= dim;  // 可能溢出
    }
    
    size_t typeSize = GetTypeSize(dataType);
    return elementCount * typeSize;  // 可能溢出
}
```

**触发路径**:
```
AddOperation() -> OpsBuilder::Build() -> CalculateBufferSize() -> 整数溢出
```

**影响**: 可能导致内存分配不足，后续访问越界

**修复建议**:
1. 使用饱和乘法检查溢出
2. 限制张量维度大小
3. 分配前验证计算结果

### 风险 4: 共享内存 fd 未验证

**证据**: `frameworks/native/neural_network_runtime/nntensor.cpp`

```cpp
// 使用 fd 创建张量，未验证 fd 有效性
OH_NN_ReturnCode NNTensor::CreateFromFd(int fd, size_t size, size_t offset) {
    // 仅检查 fd >= 0
    if (fd < 0) {
        LOGE("CreateFromFd failed, fd is invalid.");
        return OH_NN_INVALID_PARAMETER;
    }
    
    // 未验证 fd 指向的内存是否合法
    m_fd = fd;
    m_size = size;
    m_offset = offset;
    
    // 映射内存
    m_buffer = mmap(nullptr, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, offset);
    // ...
}
```

**触发路径**:
```
OH_NNTensor_CreateWithFd() -> NNTensor::CreateFromFd() -> 映射恶意 fd
```

**影响**: 可能映射到不应该访问的内存区域

**修复建议**:
1. 验证 fd 是否由 NNRt 分配
2. 检查 fd 对应的内存大小是否匹配
3. 使用 `fstat()` 验证 fd 类型

### 风险 5: IPC 反序列化风险

**证据**: `frameworks/native/neural_network_runtime/hdi_device_v2_1.cpp`

```cpp
// 从 HDI 接收模型数据
OH_NN_ReturnCode HDIDeviceV2_1::PrepareModel(...) {
    // 转换 LiteGraph 到 HDI Model
    V2_1::Model hdiModel;
    OH_NN_ReturnCode ret = LiteGraphToHdiModel(model, hdiModel);
    
    // 调用 HDI 服务
    int32_t hdiRet = m_iDevice->PrepareModel(hdiModel, config, preparedModel);
    // 未充分验证返回的 preparedModel
}
```

**触发路径**:
```
Compilation::Build() -> HDIDevice::PrepareModel() -> IPC 调用 -> 接收恶意响应
```

**影响**: 恶意驱动可能返回构造的 PreparedModel，导致后续执行时崩溃或被控制

**修复建议**:
1. 验证返回的 PreparedModel 句柄有效性
2. 增加 IPC 调用超时机制
3. 对 IPC 数据进行签名验证

### 风险 6: 日志信息泄露

**证据**: `frameworks/native/neural_network_runtime/nnexecutor.cpp`

```cpp
// 执行时记录详细日志
OH_NN_ReturnCode NNExecutor::RunSync(...) {
    LOGI("RunSync start, executorId=%{public}zu", m_executorId);
    
    for (size_t i = 0; i < inputs.size(); ++i) {
        // 可能记录敏感数据信息
        LOGD("Input %{public}zu: shape=[%{public}s]", i, shapeStr.c_str());
    }
    // ...
}
```

**触发路径**:
```
OH_NNExecutor_RunSync() -> NNExecutor::RunSync() -> 记录日志
```

**影响**: 可能泄露模型结构、输入形状等敏感信息

**修复建议**:
1. 减少 DEBUG 日志中的敏感信息
2. 使用 %{private} 标记敏感字段
3. 生产环境禁用 DEBUG 日志

### 风险 7: 竞态条件

**证据**: `frameworks/native/neural_network_runtime/memory_manager.cpp`

```cpp
// 内存管理器的映射操作
void* MemoryManager::MapMemory(int fd, size_t length) {
    std::lock_guard<std::mutex> lock(m_mtx);
    
    // 检查是否已存在
    auto it = m_memorys.find(key);
    if (it != m_memorys.end()) {
        return it->second.data;
    }
    
    // 映射新内存
    void* data = mmap(nullptr, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    
    // 保存映射关系
    m_memorys[key] = Memory{fd, data, length};
    return data;
}
```

**触发路径**:
```
多线程同时调用 AllocateBuffer/ReleaseBuffer -> 竞态条件
```

**影响**: 可能导致内存重复释放或泄漏

**修复建议**:
1. 确保所有操作都在锁保护下
2. 使用原子操作管理引用计数
3. 添加线程安全检查

## 安全检查范围与局限性

### 已检查范围

1. ✅ API 参数校验
2. ✅ 内存分配和释放
3. ✅ 文件路径处理
4. ✅ IPC 通信
5. ✅ 日志记录
6. ✅ 线程安全

### 未检查范围

1. ❌ HDI Service 实现（在驱动仓库）
2. ❌ 芯片固件安全
3. ❌ 硬件层面的侧信道攻击
4. ❌ 模型文件格式解析（MindIR）

### 局限性说明

1. 本评审仅基于 `foundation/ai/neural_network_runtime` 仓库代码
2. 依赖的第三方库（如 MindSpore Lite）未进行安全评审
3. 驱动层实现的安全依赖于芯片厂商

## 修复建议汇总

| 优先级 | 风险项 | 建议修复措施 |
|--------|--------|-------------|
| 高 | 文件路径遍历 | 路径规范化验证 |
| 高 | 整数溢出 | 增加溢出检查 |
| 中 | 共享内存 fd | fd 来源验证 |
| 中 | IPC 反序列化 | 响应数据验证 |
| 中 | 日志泄露 | 敏感信息脱敏 |
| 低 | 张量数据验证 | 增加数据校验 |
| 低 | 竞态条件 | 完善锁保护 |

## 相关跳转

- [架构说明](02_Architecture.md)
- [对外 API](04_Native_API.md)
- [常见问题](09_Troubleshooting.md)
