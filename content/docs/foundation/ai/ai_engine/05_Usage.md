# AI Engine 使用指南

> 适用读者：需要快速上手 AI Engine 的开发者和集成者
> 依赖章节：建议先阅读 [01_项目概览](01_Overview.md) 和 [02_架构与数据流](02_Architecture.md)

---

## 概述

本章节提供 AI Engine 的快速开始指南，包括：
- 环境准备
- SDK 使用示例
- 插件开发示例
- 常见问题排查

---

## 1. 环境准备

### 1.1 系统要求

| 组件 | 版本要求 | 说明 |
|------|----------|------|
| OpenHarmony | Small 系统 | 轻量级系统（130KB ROM, ~337KB RAM） |
| SAMGR | 必须运行 | System Ability Manager 必须已启动 |
| C++ 标准 | C++14 | 最低 C++14 支持 |

### 1.2 依赖组件

**从 `bundle.json:20-30`**:
- `hilog` — 日志系统
- `utils_base` — 基础工具
- `ipc` — IPC 通信
- `samgr_lite` — System Ability Manager
- `bounds_checking_function` — 边界检查函数

**证据**: `bundle.json:20-30`

### 1.3 构建配置

**编译完整 AI Engine**：
```bash
# 设置编译路径
hb set -root /path/to/project

# 设置编译产品
hb set -p

# 编译 AI Engine（完整编译）
hb build -f

# 或仅编译 ai_engine 组件
hb build ai_engine
```

**产物位置**：
- 服务端：`$root_out_dir/ai_server`
- 客户端库：`$root_out_dir/libai_client.so`
- 插件库：`$root_out_dir/lib*.so`

**证据**: `README_zh.md:52-75`

---

## 2. SDK 使用示例

### 2.1 Keyword Spotting (KWS) SDK

#### 完整使用流程

**位置**: `interfaces/kits/asr/keyword_spotting/kws_sdk.h`

```cpp
#include "ai/kits/asr/keyword_spotting/kws_sdk.h"
#include "ai/kits/asr/keyword_spotting/kws_callback.h"

using namespace OHOS::AI;

// 自定义回调类
class MyKWSCallback : public KWSCallback {
public:
    void OnError(int32_t errorCode) override {
        printf("[KWS] Error: %d\n", errorCode);
    }
    
    void OnResult(const Array<int32_t> &result) override {
        printf("[KWS] Detected keyword at index: %d\n", result.data[0]);
    }
};

int main() {
    // 1. 创建 SDK 实例
    KWSSdk kwsSdk;
    
    // 2. 创建会话
    int32_t ret = kwsSdk.Create();
    if (ret != KWS_RETCODE_SUCCESS) {
        printf("[KWS] Create failed: %d\n", ret);
        return -1;
    }
    
    // 3. 设置回调
    std::shared_ptr<MyKWSCallback> callback = std::make_shared<MyKWSCallback>();
    ret = kwsSdk.SetCallback(callback);
    if (ret != KWS_RETCODE_SUCCESS) {
        printf("[KWS] SetCallback failed: %d\n", ret);
        kwsSdk.Destroy();
        return -1;
    }
    
    // 4. 准备音频数据（PCM 格式）
    Array<int16_t> audioData;
    audioData.data = audioBuffer;  // 假设已有 PCM 数据
    audioData.size = audioLength;  // 假设 4000 采样点
    
    // 5. 执行推理（可以多次调用）
    for (int i = 0; i < 10; ++i) {
        ret = kwsSdk.SyncExecute(audioData);
        if (ret != KWS_RETCODE_SUCCESS) {
            printf("[KWS] SyncExecute failed: %d\n", ret);
            break;
        }
        
        // 模拟处理下一块音频数据
        // audioData.data = nextChunk;
        // audioData.size = nextLength;
    }
    
    // 6. 销毁会话
    ret = kwsSdk.Destroy();
    if (ret != KWS_RETCODE_SUCCESS) {
        printf("[KWS] Destroy failed: %d\n", ret);
        return -1;
    }
    
    return 0;
}
```

**API 调用顺序**（必须遵守）：
```
Create() → SetCallback() → SyncExecute() [多次] → Destroy()
```

**违反顺序的后果**：
- ⚠️ `Create()` 后未调用 `Destroy()` → 内存泄漏
- ⚠️ `Destroy()` 后调用 `SyncExecute()` → 会话已关闭错误

**证据**: `interfaces/kits/asr/keyword_spotting/kws_sdk.h:72-107`

#### SDK 内部实现流程

**位置**: `services/client/algorithm_sdk/asr/keyword_spotting/source/kws_sdk_impl.cpp`

```cpp
// KWS SDK 内部实现（简化版）
int32_t KWSSdk::KWSSdkImpl::Create() {
    // 1. 调用核心客户端 API 初始化
    int32_t retCode = AieClientInit(configInfo_, clientInfo_, algorithmInfo_, nullptr);
    if (retCode != RETCODE_SUCCESS) {
        return KWS_RETCODE_INIT_ERROR;
    }
    
    // 2. 准备算法（加载插件）
    DataInfo inputInfo = { .data = nullptr, .length = 0 };
    DataInfo outputInfo = { .data = nullptr, .length = 0 };
    retCode = AieClientPrepare(clientInfo_, algorithmInfo_, inputInfo, outputInfo, nullptr);
    if (retCode != RETCODE_SUCCESS) {
        return KWS_RETCODE_INIT_ERROR;
    }
    
    // 3. 反序列化插件句柄
    retCode = PluginHelper::UnSerializeHandle(outputInfo, kwsHandle_);
    if (retCode != RETCODE_SUCCESS) {
        return KWS_RETCODE_UNSERIALIZATION_ERROR;
    }
    
    return KWS_RETCODE_SUCCESS;
}
```

**证据**: `services/client/algorithm_sdk/asr/keyword_spotting/source/kws_sdk_impl.cpp:198-241`

---

### 2.2 Image Classification (IC) SDK

#### 完整使用流程

**位置**: `interfaces/kits/cv/image_classification/ic_sdk.h`

```cpp
#include "ai/kits/cv/image_classification/ic_sdk.h"
#include "ai/kits/cv/image_classification/ic_callback.h"
#include "ai/kits/ai_datatype.h"

using namespace OHOS::AI;

// 自定义回调类
class MyICCallback : public IcCallback {
public:
    void OnError(IcRetCode errorCode) override {
        printf("[IC] Error: %d\n", errorCode);
    }
    
    void OnResult(const Array<int32_t> &result) override {
        printf("[IC] Classification result: %d (confidence: %d)\n", 
               result.data[0], result.data[1]);
    }
};

int main() {
    // 1. 创建 SDK 实例
    IcSdk icSdk;
    
    // 2. 建立连接
    int32_t ret = icSdk.Create();
    if (ret != IC_RETCODE_SUCCESS) {
        printf("[IC] Create failed: %d\n", ret);
        return -1;
    }
    
    // 3. 设置回调
    std::shared_ptr<MyICCallback> callback = std::make_shared<MyICCallback>();
    ret = icSdk.SetCallback(callback);
    if (ret != IC_RETCODE_SUCCESS) {
        printf("[IC] SetCallback failed: %d\n", ret);
        icSdk.Destroy();
        return -1;
    }
    
    // 4. 准备图像数据（BGR 格式）
    IcInput imageInput;
    imageInput.data = imageData;      // 假设已有图像数据
    imageInput.size = imageLength;    // width * height * 3 (BGR)
    
    // 5. 执行推理（可以多次调用）
    for (int i = 0; i < 5; ++i) {
        ret = icSdk.SyncExecute(imageInput);
        if (ret != IC_RETCODE_SUCCESS) {
            printf("[IC] SyncExecute failed: %d\n", ret);
            break;
        }
        
        // 模拟处理下一张图像
        // imageInput.data = nextImage;
    }
    
    // 6. 释放资源
    ret = icSdk.Destroy();
    if (ret != IC_RETCODE_SUCCESS) {
        printf("[IC] Destroy failed: %d\n", ret);
        return -1;
    }
    
    return 0;
}
```

**API 调用顺序**（必须遵守）：
```
Create() → SetCallback() → SyncExecute() [多次] → Destroy()
```

**证据**: `interfaces/kits/cv/image_classification/ic_sdk.h:71-106`

---

## 3. 插件开发示例

### 3.1 插件接口定义

**位置**: `services/server/plugin/i_plugin.h:34-105`

**关键方法**：
```cpp
class IPlugin {
public:
    // 元数据
    virtual const long long GetVersion() const = 0;
    virtual const char *GetName() const = 0;
    virtual const char *GetInferMode() const = 0;
    
    // 生命周期
    virtual int Prepare(long long transactionId, 
                   const DataInfo &inputInfo, 
                   DataInfo &outputInfo) = 0;
    virtual int Release(bool isFullUnload, 
                   long long transactionId, 
                   const DataInfo &inputInfo) = 0;
    
    // 推理
    virtual int SyncProcess(IRequest *request, 
                       IResponse *&response) = 0;
    virtual int AsyncProcess(IRequest *request, 
                         IPluginCallback *callback) = 0;
    
    // 配置
    virtual int SetOption(int optionType, const DataInfo &inputInfo) = 0;
    virtual int GetOption(int optionType, const DataInfo &inputInfo, 
                       DataInfo &outputInfo) = 0;
};
```

**注意**：
- ✅ `SyncProcess()` 和 `AsyncProcess()` 只需实现一个
- ✅ 未实现的方法返回 0（如异步插件可让 `SyncProcess()` 返回空实现）
- ✅ `Prepare()` 调用一次，用于初始化模型
- ✅ `SyncProcess()` 可被多次调用

**证据**: `services/server/plugin/i_plugin.h:34-105`

---

### 3.2 Keyword Spotting 插件示例

**位置**: `services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp`

**完整实现**（简化版）：
```cpp
#include "services/server/plugin/i_plugin.h"
#include "services/server/plugin/i_plugin_callback.h"
#include "services/common/protocol/data_channel/include/i_request.h"
#include "services/common/protocol/data_channel/include/i_response.h"

using namespace OHOS::AI;

class KWSPlugin : public IPlugin {
private:
    // 模型句柄（示例）
    intptr_t modelHandle_ = -1;
    
public:
    KWSPlugin() = default;
    ~KWSPlugin() override = default;
    
    // 元数据
    const long long GetVersion() const override {
        return ALGOTYPE_VERSION_KWS;  // 20001002
    }
    
    const char *GetName() const override {
        return ALGORITHM_NAME_KWS.c_str();  // "KWS"
    }
    
    const char *GetInferMode() const override {
        return DEFAULT_INFER_MODE.c_str();  // "SYNC"
    }
    
    // 初始化
    int Prepare(long long transactionId, 
              const DataInfo &inputInfo, 
              DataInfo &outputInfo) override {
        // 1. 加载 NNIE 模型
        modelHandle_ = NnieLoadModel("/storage/data/keyword_spotting.wk");
        if (modelHandle_ < 0) {
            return RETCODE_FAILURE;
        }
        
        // 2. 返回插件句柄给客户端
        // 实际实现需要序列化 handle 到 outputInfo
        return RETCODE_SUCCESS;
    }
    
    // 同步推理
    int SyncProcess(IRequest *request, IResponse *&response) override {
        // 1. 获取输入数据
        DataInfo inputInfo = request->GetMsg();
        
        // 2. 反序列化音频数据
        intptr_t handle = -1;
        uint32_t frameIndex = 0;
        Array<int16_t> audioData;
        // EncdecFacade::ProcessDecode(inputInfo, handle, frameIndex, audioData);
        
        // 3. 执行推理
        Array<int32_t> results;
        retCode = NnieInference(modelHandle_, audioData.data, audioData.size, 
                              results.data, &results.size);
        if (retCode != 0) {
            return RETCODE_FAILURE;
        }
        
        // 4. 创建响应
        response = IResponse::Create(request);
        response->SetResult(results);
        
        return RETCODE_SUCCESS;
    }
    
    // 异步推理（空实现，因为 KWS 是同步插件）
    int AsyncProcess(IRequest *request, IPluginCallback *callback) override {
        return RETCODE_SUCCESS;
    }
    
    // 释放
    int Release(bool isFullUnload, long long transactionId, 
              const DataInfo &inputInfo) override {
        // 卸载模型
        NnieUnloadModel(modelHandle_);
        modelHandle_ = -1;
        return RETCODE_SUCCESS;
    }
    
    // 配置
    int SetOption(int optionType, const DataInfo &inputInfo) override {
        // 设置 MFCC 参数等
        return RETCODE_SUCCESS;
    }
    
    int GetOption(int optionType, const DataInfo &inputInfo, 
                DataInfo &outputInfo) override {
        // 获取配置信息
        return RETCODE_SUCCESS;
    }
};

// 注册插件
PLUGIN_INTERFACE_IMPL(KWSPlugin);
```

**注册宏**：
```cpp
#define PLUGIN_INTERFACE_IMPL(PluginName) \
    extern "C" IPlugin* PLUGIN_INTERFACE() \
    { \
        return new PluginName(); \
    }
```

**证据**: `services/server/plugin/asr/keyword_spotting/source/kws_plugin.cpp`

---

### 3.3 插件集成到构建系统

**步骤 1**: 创建插件目录

```
services/server/plugin/my_new_plugin/
├── include/
│   └── my_plugin.h
└── source/
    └── my_plugin.cpp
```

**步骤 2**: 创建 BUILD.gn

**位置**: `services/server/plugin/my_new_plugin/BUILD.gn`

```gn
import("//build/lite/config/component/lite_component.gni")

# 插件共享库
shared_library("my_new_plugin") {
    sources = [
        "source/my_plugin.cpp",
    ]
    
    include_dirs = [
        "include",
        "//foundation/ai/ai_engine/services/common/protocol/include",
    ]
    
    deps = [
        "//foundation/ai/ai_engine/services/common:protocol",
        "//foundation/hiviewdfx/hilog:hilog_shared",
        "//foundation/hiviewdfx/hardware_nnie:engine_nnie_sdk",
        "//foundation/ai/ai_engine/services/common:plugin_helper",
        "//foundation/ai/ai_engine/services/common/utils:encdec",
        "//third_party/bounds_checking_function:libsec_shared",
    ]
    
    # 拷贝模型文件到输出目录
    copy("my_model.wk") {
        outputs = [ "$root_out_dir/data/my_model.wk" ]
    }
}
```

**步骤 3**: 添加到插件组件

**修改**: `services/server/plugin/BUILD.gn`

```gn
lite_component("plugin") {
    features = [
        "my_new_plugin",  # 添加新插件
        "asr",
        "cv",
    ]
}
```

**步骤 4**: 配置激活列表

**修改**: `services/ai_plugin_config.gni`

```gn
declare_args() {
    activate_plugin_list = [
        "my_new_plugin",  # 激活新插件
    ]
}
```

**证据**: `services/server/plugin/asr/keyword_spotting/BUILD.gn`（参考）

---

## 4. 错误处理

### 4.1 公共错误码

**位置**: `interfaces/kits/ai_retcode.h:22-35`

| 错误码 | 值 | 说明 | 处理建议 |
|--------|-----|------|---------|
| `AI_RETCODE_SUCCESS` | 0 | 操作成功，正常处理 |
| `AI_RETCODE_FAILURE` | -1 | 通用失败，检查日志 |
| `AI_RETCODE_INIT_ERROR` | 1001 | 初始化失败，检查 SAMGR 是否运行 |
| `AI_RETCODE_NULL_PARAM` | 1002 | 空指针参数，检查输入数据 |
| `AI_RETCODE_DUPLICATE_INIT_ERROR` | 1004 | 重复初始化，确保 `Destroy()` 后才再次 `Create()` |
| `AI_RETCODE_SERIALIZATION_ERROR` | 2001 | 序列化失败，检查输入数据格式 |
| `AI_RETCODE_UNSERIALIZATION_ERROR` | 2002 | 反序列化失败，检查数据版本匹配 |
| `AI_RETCODE_PLUGIN_EXECUTION_ERROR` | 3001 | 插件执行失败，检查插件日志 |
| `AI_RETCODE_PLUGIN_SESSION_ERROR` | 3002 | 插件会话错误，检查是否已 `Release()` |

**证据**: `interfaces/kits/ai_retcode.h:22-35`

### 4.2 错误处理最佳实践

```cpp
// SDK 初始化错误处理
int32_t ret = kwsSdk.Create();
if (ret != KWS_RETCODE_SUCCESS) {
    switch (ret) {
        case KWS_RETCODE_INIT_ERROR:
            printf("[KWS] Failed to init, check if SAMGR is running\n");
            break;
        case KWS_RETCODE_NULL_PARAM:
            printf("[KWS] Null parameter, check input data\n");
            break;
        default:
            printf("[KWS] Unknown error: %d\n", ret);
    }
    return -1;
}

// 推理执行错误处理
ret = kwsSdk.SyncExecute(audioData);
if (ret != KWS_RETCODE_SUCCESS) {
    switch (ret) {
        case KWS_RETCODE_SERIALIZATION_ERROR:
            printf("[KWS] Failed to serialize audio data\n");
            break;
        case KWS_RETCODE_PLUGIN_EXECUTION_ERROR:
            // 警告用户但不中断
            printf("[KWS] Plugin execution error, but continuing...\n");
            break;
        default:
            printf("[KWS] SyncExecute failed: %d\n", ret);
    }
    // 错误恢复：重试或使用默认值
    if (ret == KWS_RETCODE_PLUGIN_EXECUTION_ERROR) {
        // 使用上一帧结果或默认值
    }
}
```

**证据**: SDK 使用示例中的错误处理模式

---

## 5. 调试技巧

### 5.1 启用日志

**HILOG 宏**：
```cpp
#include "hilog/log.h"

#undef LOG_TAG
#define LOG_TAG "MyAIApp"

// 错误日志
HILOGE(LOG_TAG, "Failed to initialize: %{public}d", retCode);

// 信息日志
HILOGI(LOG_TAG, "Plugin loaded successfully");

// 调试日志
HILOGD(LOG_TAG, "Input size: %{public}zu", inputSize);
```

**证据**: HILOG 使用示例

### 5.2 常见问题排查

| 问题 | 可能原因 | 排查步骤 |
|------|----------|---------|
| `Create()` 返回 INIT_ERROR | SAMGR 未启动 | 1. 检查 `ps -ef | grep samgr`<br>2. 重启设备<br>3. 检查系统日志 |
| `SyncExecute()` 返回 NULL_PARAM | 输入数据为空 | 1. 检查 `data` 指针<br>2. 检查 `length` 是否为 0 |
| `SyncExecute()` 返回 PLUGIN_EXECUTION_ERROR | 插件未加载 | 1. 检查插件 .so 是否存在<br>2. 检查插件配置文件<br>3. 检查插件依赖 |
| 插件未被加载 | 路径映射错误 | 1. 检查 `plugin_label.cpp` 中的映射<br>2. 确认 `aid` 和 `version` 匹配 |
| 内存泄漏 | 未调用 `Destroy()` | 1. 使用工具检测内存泄漏<br>2. 确保 `Create()`/`Destroy()` 成对调用 |
| 回调未触发 | `SetCallback()` 未调用 | 1. 检查回调对象是否有效<br>2. 检查回调是否为空<br>3. 确认 SDK 生命周期 |

---

## 6. 性能优化建议

### 6.1 API 调用优化

| 优化点 | 建议 |
|--------|------|
| **避免频繁 Create/Destroy** | 复用 SDK 实例，只在需要时创建 |
| **批量推理** | 如果支持，一次调用 `SyncExecute()` 处理多个输入 |
| **异步模式** | 对于长时间推理，使用异步 API（如果插件支持） |
| **共享内存阈值** | 小于 200 字节的数据直接 IPC，避免共享内存开销 |

### 6.2 插件开发优化

| 优化点 | 建议 |
|--------|------|
| **模型预加载** | 在 `Prepare()` 中加载模型，避免首次推理延迟 |
| **内存池化** | 复用缓冲区，减少动态分配 |
| **批量处理** | 如果支持，一次 `SyncProcess()` 处理多个输入 |
| **多线程** | 使用 Engine 提供的线程池（服务端自动管理） |

---

## 7. 相关章节

- [01_项目概览](01_Overview.md) — 项目定位和能力边界
- [02_架构与数据流](02_Architecture.md) — 理解系统架构
- [03_代码地图](03_CodeMap.md) — 快速定位代码位置
- [04_对外接口文档](04_Interface.md) — 详细的 API 参考
- [06_攻击面分析](06_AttackSurface.md) — 了解安全风险
- [08_FAQ](08_FAQ.md) — 常见问题解答

---

## 证据要求

本章节所有使用示例均基于实际代码：

- ✅ 文件路径：完整的 SDK 头文件和实现文件路径
- ✅ 行号范围：关键方法所在的行号
- ✅ API 顺序：正确的生命周期调用顺序
- ✅ 错误码：所有相关错误码及其含义

**示例来源**：
- `README_zh.md` — 官方文档示例
- `interfaces/kits/` — SDK 头文件
- `services/client/algorithm_sdk/` — SDK 实现
- `services/server/plugin/` — 插件示例实现
