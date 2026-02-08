# API 接口差异

## 概述

MindSpore Lite OH 版本相对于上游主要在 **系统集成层面** 有差异，核心推理 API 保持一致。本文档记录 OH 版本特有的 API 和配置变更。

---

## OH 新增 API

### 1. HI App Event 集成

**头文件**: `src/common/hi_app_event/hi_app_event.h`

**功能**: 集成 OpenHarmony 应用事件系统

#### API 列表

```c
// 初始化 App Event
int HiAppEvent_Init(void);

// 绑定推理事件到应用
int HiAppEvent_BindInferenceEvent(const char *event_name);

// 记录推理开始事件
int HiAppEvent_RecordInferenceStart(const char *model_name);

// 记录推理结束事件
int HiAppEvent_RecordInferenceEnd(const char *model_name, float duration_ms);

// 记录推理错误事件
int HiAppEvent_RecordInferenceError(const char *model_name, int error_code);

// 销毁 App Event
void HiAppEvent_Destroy(void);
```

#### 使用示例

```c
#include "hi_app_event/hi_app_event.h"

int main() {
    // 初始化
    HiAppEvent_Init();
    
    // 记录推理事件
    HiAppEvent_RecordInferenceStart("resnet50");
    
    // 执行推理
    // ...
    
    HiAppEvent_RecordInferenceEnd("resnet50", 45.6f);
    
    // 清理
    HiAppEvent_Destroy();
    return 0;
}
```

---

### 2. OH 上下文配置

**头文件**: `src/litert/cxx_api/context.h`

**新增配置项**:

```cpp
// OH 特定设备配置
enum class OHDeviceType {
  kOHDeviceTypeCPU = 0,
  kOHDeviceTypeNNRT = 1,  // 新增: NNRT 设备
};

// OH 上下文配置
class OHContext : public Context {
 public:
  // 设置 NNRT 设备优先级
  void SetNNRTPriority(int priority);
  
  // 获取 NNRT 设备能力
  std::vector<NNRTCapability> GetNNRTCapabilities();
  
  // 设置 QoS 策略
  void SetQoS(QoSType qos_type);
};
```

---

### 3. Taihe/ANI 接口

**功能**: Ark Native Interface (ANI) 绑定，支持 ArkTS/JS 调用

**头文件**: `src/litert/taihe/ability_delegator/taihe.h`

```cpp
namespace mindspore::taihe {

// Taihe 会话
class TaiheSession {
 public:
  // 创建会话
  static std::shared_ptr<TaiheSession> Create(const std::string &model_path);
  
  // 设置输入
  int SetInput(const std::string &tensor_name, const void *data, size_t size);
  
  // 执行推理
  int Predict();
  
  // 获取输出
  std::vector<Tensor> GetOutputs();
  
  // 释放资源
  void Destroy();
};

}  // namespace mindspore::taihe
```

---

## OH 特定行为变更

### 1. 日志系统切换

| 配置 | 上游行为 | OH 行为 |
|------|----------|---------|
| **日志输出** | GLOG | hilog |
| **日志级别** | GLOG 配置 | OH 日志级别 |
| **日志格式** | GLOG 格式 | OH 标准格式 |

#### 配置宏

```c
// 启用 OH 日志 (默认)
#define ENABLE_HI_APP_EVENT 1

// 禁用 GLOG
#define MSLITE_ENABLE_RUNTIME_GLOG 0
```

---

### 2. 内存管理

**变更**: OH 版本使用 OH 特有的内存分配器

```c
// OH 内存分配
void* OH_Malloc(size_t size);
void OH_Free(void *ptr);

// 替换标准分配
#define malloc OH_Malloc
#define free OH_Free
```

---

### 3. 线程模型

**变更**: 使用 OH 线程池替代标准线程

```c
// OH 线程配置
typedef struct {
  int32_t core_num;           // 核心数
  int32_t priority;           // 优先级
  bool bind_to_core;          // 绑定核心
  OH_ThreadPolicy policy;     // 调度策略
} OHThreadConfig;

// 设置推理线程
int MSContextSetThreadConfig(MSContextHandle context, OHThreadConfig *config);
```

---

### 4. 设备发现

**新增**: NNRT 设备自动发现

```c
// 枚举可用 NNRT 设备
int EnumerateNNRTDevices(NNRTDeviceInfo **devices, int32_t *count);

// 设备信息
typedef struct {
  char name[64];
  char vendor[64];
  int32_t version;
  int32_t compute_units;
  float frequency;
} NNRTDeviceInfo;
```

---

## 上游兼容 API

以下 API 与上游 MindSpore Lite 完全兼容：

### 核心推理 API

```c
// 上下文
MSContextHandle MSContextCreate(void);
void MSContextDestroy(MSContextHandle *context);
void MSContextSetDeviceTarget(MSContextHandle context, MSDeviceType target);

// 张量
MSTensorHandle MSTensorCreate(const char *name, MSDataType data_type,
                               const int64_t *shape, size_t shape_size);
void MSTensorDestroy(MSTensorHandle *tensor);
void *MSTensorGetData(MSTensorHandle tensor);
size_t MSTensorGetSize(MSTensorHandle tensor);

// 模型
MSModelHandle MSModelCreate(void);
MSStatus MSModelLoadFromFile(MSModelHandle model, const char *model_path);
MSStatus MSModelLoadFromMem(MSModelHandle model, const void *model_data, size_t data_size);
void MSModelDestroy(MSModelHandle *model);
MSTensorHandle MSModelGetInputByIndex(MSModelHandle model, size_t index);
MSTensorHandle MSModelGetOutputByIndex(MSModelHandle model, size_t index);
MSStatus MSModelPredict(MSModelHandle model);

// 会话
MSSessionHandle MSSessionCreate(MSModelHandle model);
MSSessionHandle MSSessionCreateAndCompile(MSModelHandle model, const MSGraph *graph);
MSStatus MSSessionPredict(MSSessionHandle session, const MSTensorHandleArray *inputs,
                          MSTensorHandleArray *outputs);
void MSSessionDestroy(MSSessionHandle *session);
```

---

## 已禁用功能

| 功能 | 上游状态 | OH 状态 | 原因 |
|------|----------|---------|------|
| GPU 后端 | 可用 | 禁用 | OH 不支持 |
| Ascend 后端 | 可用 | 禁用 | 硬件限制 |
| MindData | 可用 | 部分禁用 | 依赖缺失 |
| 云端推理 | 可用 | 禁用 | 场景不适用 |
| 模型转换工具 | 可用 | 禁用 | 不嵌入镜像 |

---

## 配置差异

### CMake vs BUILD.gn 配置对照

| CMake 选项 | BUILD.gn 定义 | OH 状态 |
|------------|---------------|---------|
| `MSLITE_ENABLE_GPU` | - | 禁用 |
| `MSLITE_ENABLE_ASCEND` | - | 禁用 |
| `MSLITE_ENABLE_MINDDATA` | - | 部分 |
| `MSLITE_ENABLE_CONVERTER` | - | 禁用 |
| `MSLITE_ENABLE_TOOLS` | - | 禁用 |
| `MSLITE_ENABLE_SERVER` | - | 禁用 |
| `MSLITE_ENABLE_NPU` | - | NNRT 替代 |

---

## 迁移指南

### 从上游 MindSpore Lite 迁移

#### 1. 日志替换

**上游代码**:
```c
#include "glog/logging.h"

LOG(INFO) << "Inference started";
```

**OH 适配**:
```c
#include "hilog/log.h"

OH_LOG_INFO(LOG_APP) << "Inference started";
```

#### 2. 设备配置

**上游代码**:
```c
context->SetDeviceType(kGPU);
```

**OH 适配**:
```c
context->SetDeviceType(kCPU);  // GPU 不可用
// 或使用 NNRT
context->SetDeviceType(kNNRT);
```

#### 3. 模型加载

**上游代码**:
```c
model->LoadFromFile("model.ms");
```

**OH 代码**: 兼容，无需修改

---

## API 版本历史

| API 版本 | OH 版本 | 变更内容 |
|----------|---------|----------|
| 1.0 | 3.0 | 初始版本 |
| 2.0 | 3.1 | 新增 NNRT 支持、新增 Taihe |

---

## 未来兼容性

### 计划中的 API 变更

| 功能 | 计划版本 | 描述 |
|------|----------|------|
| 多模型推理 | 3.2 | 并发推理支持 |
| 模型缓存 | 3.2 | 运行时缓存 |
| 异步推理 | 3.2 | 非阻塞推理 |

---

## 相关文档

- [MindSpore Lite C++ API](https://www.mindspore.cn/lite/api/en/master/api_cpp/)
- [OH 日志系统](https://gitee.com/openharmony/hilog)
- [NNRT API](https://gitee.com/openharmony/neural_network_runtime)
