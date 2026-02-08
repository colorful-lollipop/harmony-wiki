# VPE 视频处理引擎架构设计

本文档详细描述 VPE 视频处理引擎的架构设计，包括组件图、数据流、线程模型和关键时序。

---

## 1 整体架构

### 1.1 架构分层

VPE 采用分层架构设计，从上到下分为四个层次：

```mermaid
graph TD
    subgraph "应用层"
        A1[JS/TS 应用]
        A2[C/C++ 应用]
        A3[ArkTS 应用]
    end
    
    subgraph "接口层"
        I1[N-API]
        I2[C API]
        I3[Inner API]
        I4[IPC]
    end
    
    subgraph "框架层"
        F1[算法框架]
        F2[插件管理]
        F3[资源管理]
    end
    
    subgraph "算法层"
        P1[色彩空间转换]
        P2[细节增强]
        P3[元数据生成]
        P4[HDR 增强]
    end
    
    A1 --> I1
    A2 --> I2
    A3 --> I4
    
    I1 --> I2
    I2 --> I3
    I3 --> I4
    
    I4 --> F1
    I4 --> F2
    
    F1 --> P1
    F1 --> P2
    F1 --> P3
    F1 --> P4
    
    F2 --> P1
    F2 --> P2
    F2 --> P3
    F2 --> P4
```

**证据位置**：`README.md:12-119` ✅

### 1.2 模块职责

| 层级 | 模块 | 职责 | 证据位置 |
|------|------|------|---------|
| **应用层** | JS/TS API | 提供 ArkTS/JS 调用接口 | `detail_enhance_napi.cpp:314-327` ✅ |
| **应用层** | C API | 提供 C 语言调用接口 | `image_processing.h` ✅ |
| **接口层** | Inner API | 系统内部模块间调用 | `interfaces/inner_api/` ✅ |
| **接口层** | IPC | 跨进程服务通信 | `IVideoProcessingServiceManager.idl` ✅ |
| **框架层** | 算法框架 | 统一调度各算法模块 | `framework/algorithm/` ✅ |
| **框架层** | 插件管理 | 动态算法插件注册加载 | `extension_manager.cpp` ✅ |
| **算法层** | 各算法模块 | 具体算法实现 | `framework/algorithm/*/` ✅ |

---

## 2 组件图

### 2.1 核心组件

```mermaid
graph TB
    subgraph "应用进程"
        APP[应用层]
        NAPI[N-API 封装]
        CAPI[C API 封装]
    end
    
    subgraph "framework.so"
        FWK[算法框架]
        PLG[插件管理]
        EXT[扩展实现]
    end
    
    subgraph "services/"
        SA[VideoProcessingServer SA]
        FACT[算法工厂]
    end
    
    subgraph "预编译库"
        EVE[EVE 算法库]
        AISR[AI 超分库]
        AIHDR[AI HDR 库]
        SKIA[Skia 软件库]
    end
    
    APP --> NAPI
    APP --> CAPI
    NAPI --> FWK
    CAPI --> FWK
    FWK --> PLG
    FWK --> EXT
    FWK --> SA
    SA --> FACT
    PLG --> EVE
    PLG --> AISR
    PLG --> AIHDR
    EXT --> SKIA
```

### 2.2 数据流组件

```mermaid
graph LR
    subgraph "输入"
        IN1[SurfaceBuffer]
        IN2[PixelMap]
        IN3[模型文件]
    end
    
    subgraph "处理"
        PROC[VideoProcessingServer]
        ALGO[算法模块]
    end
    
    subgraph "输出"
        OUT1[SurfaceBuffer]
        OUT2[PixelMap]
        OUT3[元数据]
    end
    
    IN1 --> PROC
    IN2 --> PROC
    IN3 --> PROC
    PROC --> ALGO
    ALGO --> OUT1
    ALGO --> OUT2
    ALGO --> OUT3
```

---

## 3 数据流

### 3.1 图像处理数据流

```mermaid
sequenceDiagram
    participant App as 应用
    participant CAPI as Image Processing CAPI
    participant FWK as 框架层
    participant ALGO as 算法模块
    
    App->>CAPI: OH_ImageProcessing_Create()
    CAPI->>FWK: 创建处理实例
    FWK->>ALGO: 初始化算法<br>SetParameter()
    
    App->>CAPI: OH_ImageProcessing_EnhanceDetail()
    CAPI->>FWK: 处理请求
    FWK->>ALGO: Process(input, output)
    ALGO-->>FWK: 处理结果
    FWK-->>CAPI: 返回结果
    CAPI-->>App: 返回 PixelMap
    
    App->>CAPI: OH_ImageProcessing_Destroy()
    CAPI->>FWK: 销毁实例
    FWK->>ALGO: 清理资源
```

**证据位置**：`image_processing.h:150-308` ✅

### 3.2 视频处理数据流

```mermaid
sequenceDiagram
    participant Decoder as 视频解码器
    participant VPE as Video Processing
    participant SA as SA 服务
    participant ALGO as 算法模块
    
    Decoder->>VPE: GetSurface() 获取输入 Surface
    VPE->>SA: 注册回调、配置参数
    SA->>ALGO: 初始化算法
    
    loop 处理帧
        Decoder->>VPE: 解码输出到 Surface
        VPE->>SA: Process(input)
        SA->>ALGO: ProcessFrame()
        ALGO-->>SA: 处理完成
        SA-->>VPE: OnNewOutputBuffer
        VPE->>Decoder: RenderOutputBuffer()
    end
    
    VPE->>SA: Stop()
    SA->>ALGO: 清理资源
```

**证据位置**：`video_processing.h:120-244` ✅

### 3.3 插件加载数据流

```mermaid
sequenceDiagram
    participant Sys as 系统
    participant EM as ExtensionManager
    participant DL as 动态加载器
    participant Plugin as 算法插件
    
    Sys->>EM: Initialize()
    EM->>DL: dlopen(libpath)
    DL->>Plugin: 加载库
    Plugin-->>DL: GetCreator()
    DL-->>EM: 返回函数指针
    EM->>Plugin: RegisterExtensions()
    Plugin-->>EM: 注册能力
    EM->>EM: 添加到插件列表
    
    Note over EM, Plugin: 插件生命周期
    EM->>DL: dlsym() 获取函数
    DL-->>EM: 返回符号地址
    EM->>Plugin: CreateInstance()
    Plugin-->>EM: 返回算法实例
```

**证据位置**：`video_processing_algorithm_factory.cpp:63-90` ✅

---

## 4 线程模型

### 4.1 线程划分

```mermaid
graph TB
    subgraph "应用进程"
        T1[主线程<br>API 调用]
        T2[工作线程<br>数据处理]
    end
    
    subgraph "SA 进程"
        T3[主线程<br>IPC 处理]
        T4[算法线程池<br>并行处理]
    end
    
    T1 --> T2
    T2 -->|IPC| T3
    T3 --> T4
```

### 4.2 线程职责

| 线程 | 职责 | 证据位置 |
|------|------|---------|
| **应用主线程** | API 调用、N-API 回调 | `detail_enhance_napi.cpp:91-117` ✅ |
| **SA 主线程** | IPC 请求处理、生命周期管理 | `video_processing_server.cpp` ✅ |
| **算法线程池** | 实际算法计算、资源密集型任务 | `services/algorithm/` ✅ |

### 4.3 线程同步机制

**使用到的同步原语**：

| 同步类型 | 使用场景 | 证据位置 |
|---------|---------|---------|
| **std::mutex** | N-API 实例保护 | `detail_enhance_napi.cpp:94` ✅ |
| **std::lock_guard** | 临界区保护 | `detail_enhance_napi.cpp:121` ✅ |
| **sptr** | IPC 远程对象管理 | `video_processing_client.h` ✅ |
| **回调机制** | 异步结果通知 | `video_processing_callback_impl.cpp` ✅ |

**示例代码**：
```cpp
// detail_enhance_napi.cpp:94-99
std::lock_guard<std::mutex> lock(lock_);
if (mDetailEnh != nullptr) {
    napi_get_boolean(env, true, &result);
    return result;
}
```

---

## 5 关键时序

### 5.1 N-API 初始化时序

```mermaid
sequenceDiagram
    participant JS as JS 引擎
    participant NM as N-API 模块
    participant NAPI as DetailEnhanceNapi
    participant FWK as DetailEnhancerImage
    
    JS->>NM: load('multimedia.detailEnhancer')
    NM->>NM: napi_module_register()
    NM->>NM: Init()
    
    Note over NM: 模块初始化
    NM->>NAPI: Init()
    NAPI->>FWK: DetailEnhancerImage::Create()
    FWK-->>NAPI: 返回实例
    NAPI-->>NM: 初始化完成
```

**证据位置**：`detail_enhance_napi.cpp:302-327` ✅

### 5.2 SA 服务加载时序

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant SAMgr as SA Manager
    participant SA as VideoProcessingServer
    participant Factory as 算法工厂
    
    Note over Client: 首次调用
    Client->>SAMgr: GetSystemAbility()
    SAMgr-->>Client: SA 未启动
    
    Client->>SAMgr: LoadSystemAbility()
    SAMgr->>SA: OnStart()
    SA->>SA: Publish()
    SA->>Factory: Initialize()
    Factory->>Factory: 加载插件
    
    SA-->>SAMgr: 启动完成
    SAMgr-->>Client: 返回 SA 代理
    
    Note over Client, SA: 正常 IPC 调用
    Client->>SA: Create()
    SA->>Factory: CreateAlgorithm()
    Factory-->>SA: 返回算法实例
    SA-->>Client: clientID
```

**证据位置**：
- `video_processing_client.cpp:139-185` ✅
- `video_processing_server.cpp:41` ✅

### 5.3 图像处理完整时序

```mermaid
sequenceDiagram
    participant App as 应用
    participant NAPI as N-API 层
    participant Base as NativeBase
    participant Impl as 算法实现
    participant Buffer as SurfaceBuffer
    
    App->>NAPI: EnhanceDetail(width, height, pixelmap)
    NAPI->>NAPI: 参数解析
    NAPI->>Base: 获取 PixelMap
    Base->>Buffer: 获取输入 SurfaceBuffer
    Buffer-->>Base: 返回 Buffer
    
    NAPI->>Base: 创建输出 PixelMap
    Base->>Buffer: 分配输出 SurfaceBuffer
    Buffer-->>Base: 返回 Buffer
    
    NAPI->>Impl: Process(input, output)
    Impl->>Impl: 算法计算
    Impl-->>NAPI: 处理完成
    
    NAPI->>NAPI: 构造返回值
    NAPI-->>App: 返回 PixelMap
```

**证据位置**：`detail_enhance_napi.cpp:217-251` ✅

---

## 6 依赖方向

### 6.1 模块依赖图

```mermaid
graph TB
    subgraph "接口层"
        JSAPI[N-API JS]
        CAPI[C API]
    end
    
    subgraph "框架层"
        FWK[核心框架]
        EXT[扩展管理]
    end
    
    subgraph "服务层"
        SA[SA 服务]
        FACT[算法工厂]
    end
    
    subgraph "算法层"
        CSC[色彩空间]
        DE[细节增强]
        MET[元数据]
        HDR[HDR]
    end
    
    subgraph "依赖库"
        SKIA[Skia]
        EVE[EVE]
        AISR[AI SR]
    end
    
    JSAPI --> CAPI
    CAPI --> FWK
    FWK --> EXT
    FWK --> SA
    SA --> FACT
    
    FACT --> CSC
    FACT --> DE
    FACT --> MET
    FACT --> HDR
    
    DE --> SKIA
    DE --> EVE
    DE --> AISR
```

### 6.2 依赖关系说明

| 依赖方向 | 说明 | 证据位置 |
|---------|------|---------|
| 接口 → 框架 | CAPI 依赖核心框架 | `framework/BUILD.gn:335` ✅ |
| 框架 → 服务 | IPC 调用 SA 服务 | `services/BUILD.gn` ✅ |
| 服务 → 算法 | 工厂创建算法实例 | `video_processing_algorithm_factory.cpp` ✅ |
| 算法 → 扩展 | 插件动态加载 | `extension_manager.cpp:51` ✅ |

### 6.3 循环依赖检测

**当前架构无循环依赖** ✅

| 模块对 | 是否存在循环 | 说明 |
|-------|-------------|------|
| CAPI ↔ 框架 | ❌ 无 | CAPI 调用框架，框架不回调 CAPI |
| 框架 ↔ 服务 | ❌ 无 | IPC 单向调用 |
| 服务 ↔ 算法 | ❌ 无 | 工厂模式创建 |

---

## 7 跨进程通信

### 7.1 IPC 架构

```mermaid
graph TB
    subgraph "应用进程"
        C[Client Proxy]
    end
    
    subgraph "IPC Layer"
        B[Binder Driver]
    end
    
    subgraph "SA 进程"
        S[Server Stub]
        H[Handler]
    end
    
    C -->|IPC Call| B
    B -->|IPC Call| S
    S --> H
```

### 7.2 IDL 接口定义

**接口文件**：`services/IVideoProcessingServiceManager.idl`

| 方法 | 功能 | 方向 |
|------|------|------|
| `LoadInfo` | 加载模型信息 | Request |
| `Create` | 创建算法实例 | Request |
| `Destroy` | 销毁算法实例 | Request |
| `SetParameter` | 设置参数 | Request |
| `GetParameter` | 获取参数 | Request |
| `UpdateMetadata` | 更新元数据 | Request |
| `Process` | 执行处理 | Request |
| `ComposeImage` | 图像合成 | Request |
| `DecomposeImage` | 图像分解 | Request |

**证据位置**：`services/IVideoProcessingServiceManager.idl` ✅

### 7.3 SA 配置

| 配置项 | 值 | 证据位置 |
|--------|-----|---------|
| SA ID | 0x00010256 (66134) | `vpe_sa_constants.h:26` ✅ |
| 进程名 | video_processing_service | `video_processing_service.cfg` ✅ |
| 运行用户 | media | `video_processing_service.cfg` ✅ |
| SELinux 标签 | u:r:video_processing_service:s0 | `video_processing_service.cfg` ✅ |

---

## 8 插件扩展机制

### 8.1 插件注册流程

```mermaid
flowchart TD
    A[开始] --> B[ExtensionManager Initialize]
    B --> C{dlopen 库文件}
    C -->|成功| D[获取 GetCreator]
    C -->|失败| E[记录日志<br>使用内置算法]
    D --> F[调用 GetCreator]
    F --> G[获取注册函数]
    G --> H[调用注册函数]
    H --> I[注册插件能力]
    I --> J[添加到插件列表]
    J --> K[返回插件列表]
```

**证据位置**：`extension_manager.cpp:40-80` ✅

### 8.2 插件类型

| 插件类型 | 基类 | 功能 | 证据位置 |
|---------|------|------|---------|
| 色彩空间转换 | ColorSpaceConverterBase | 色彩空间转换 | `colorspace_converter_base.h` ✅ |
| 细节增强 | DetailEnhancerBase | 超分/锐化 | `detail_enhancer_base.h` ✅ |
| 元数据生成 | MetadataGeneratorBase | 元数据生成 | `metadata_generator_base.h` ✅ |

### 8.3 内置插件

| 插件名称 | 类型 | 算法来源 | 证据位置 |
|---------|------|---------|---------|
| Skia | DetailEnhancer | Skia 软件渲染 | `skia_impl.cpp` ✅ |
| EVE | DetailEnhancer | EVE AI 引擎 | 预编译库 |
| AISR | DetailEnhancer | AI 超分 | 预编译库 |
| AIHDR | HDR 增强 | AI HDR 引擎 | 预编译库 |

---

## 9 相关文档链接

| 文档 | 说明 |
|------|------|
| [index.md](./index.md) | 项目概览 |
| [NAPI_Reference.md](./NAPI_Reference.md) | API 参考 |
| [Build_System.md](./Build_System.md) | 构建系统 |
| [Artifacts.md](./Artifacts.md) | 编译产物 |
| [Security_Review.md](./Security_Review.md) | 安全评审 |

---

## 10 更新日志

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，包含完整架构设计 |
