# 内部架构与 Inner API

## 概述

本文档描述 OpenHarmony HDI 的内部架构，包括模块依赖方向、线程模型、生命周期管理和 Inner API 约定。

---

## 架构分层

### 分层结构

```
┌─────────────────────────────────────────────────────┐
│                   上层服务层                          │
│              (Framework Services)                     │
├─────────────────────────────────────────────────────┤
│                   HDI 代理层                         │
│                 (Client Proxy)                       │
│     调用接口: I<Module>Interface + Callback         │
├─────────────────────────────────────────────────────┤
│                   IPC 框架                           │
│              (IPC Skeleton/Proxy)                    │
│     生成代码: lib<module>_client_*.so               │
├─────────────────────────────────────────────────────┤
│                   HDI 存根层                         │
│                (Server Stub)                         │
│     生成代码: lib<module>_stub_*.so                  │
├─────────────────────────────────────────────────────┤
│                   驱动实现层                         │
│            (drivers_peripheral)                      │
│     实现接口: 继承自动生成的接口类并实现业务逻辑        │
├─────────────────────────────────────────────────────┤
│                   HDF 框架                           │
│            (drivers_framework)                       │
│     驱动加载、服务管理、设备树解析                     │
└─────────────────────────────────────────────────────┘
```

---

## 模块依赖方向

### 依赖原则

- 上层模块可以依赖下层模块
- 同层模块间尽量避免循环依赖
- 跨层依赖通过接口抽象进行解耦

### 典型依赖链

```
Audio 服务
    ↓ 依赖
IAudioAdapter (HDI 接口)
    ↓ 由 hdi() 编译生成
Audio Proxy/Stub 代码
    ↓ 被调用
Audio 驱动实现 (drivers_peripheral)
    ↓ 依赖
HDF 核心框架 (drivers_framework)
    ↓ 依赖
IPC 框架 + 内核
```

### 模块间依赖声明

**BUILD.gn 中的依赖声明**:

```gni
hdi("camera") {
  module_name = "camera_service"
  sources = [...]
  
  # 内部序列化依赖
  sequenceable_pub_deps = [
    "../sequenceable/buffer_producer:libbuffer_producer_sequenceable_1.0",
  ]
  
  # 外部图形表面依赖
  sequenceable_ext_deps = [
    "graphic_surface:buffer_handle",
    "graphic_surface:surface",
  ]
  
  language = "cpp"
}
```

---

## 线程模型

### 同步调用模式

在 IPC 模式下，方法调用通常是同步的：

```cpp
// 客户端线程
sptr<IModule> module = IModule::Get();
int32_t ret = module->Method(inParam, outParam);  // 阻塞等待返回
// 继续执行
```

### 异步回调模式

使用回调接口实现异步通知：

```cpp
// 1. 注册回调
class MyCallback : public IModuleCallback {
    void OnEvent(EventType event) override {
        // 在回调线程执行
    }
};

auto callback = new MyCallback();
module->Register(callback);

// 2. 业务继续执行...
```

### 直通模式 (Passthrough)

在 `mode = "passthrough"` 模式下，调用直接在驱动进程执行：

```idl
// IInputInterfaces.idl (input 模式为 passthrough)
interface IInputInterfaces {
    ScanInputDevice([out] DevDesc[] staArr);
    RegisterReportCallback([in] unsigned int devIndex, [in] IInputCallback callback);
};
```

**特点**:
- 无 IPC 开销
- 同步阻塞调用
- 适用于对延迟敏感的场景

---

## 生命周期管理

### 服务端生命周期

```
┌─────────────────────────────────────────────────────┐
│  HDF 框架加载驱动                                    │
│      ↓                                              │
│  实例化驱动服务对象 (IModuleService)                 │
│      ↓                                              │
│  发布服务到 ServiceManager                          │
│      ↓                                              │
│  等待客户端请求                                       │
│      ↓                                              │
│  HDF 框架卸载驱动                                    │
│      ↓                                              │
│  释放资源                                           │
└─────────────────────────────────────────────────────┘
```

### 客户端生命周期

```cpp
// 1. 获取服务代理
sptr<IModule> module = IModule::Get();

// 2. 使用服务
module->Method(...);

// 3. 释放代理（引用计数管理）
module = nullptr;  // 引用计数减 1
```

### 引用计数

HDI 使用智能指针管理对象生命周期：

```cpp
// sptr 是 Strong Pointer (强引用)
sptr<IModule> module = IModule::Get();

// 当 sptr 离开作用域时，引用计数自动减 1
// 引用计数为 0 时，对象被销毁
```

---

## Inner API 约定

### API 级别标记

使用 `innerapi_tags` 标记不同级别的 API：

**示例** (`sensor/v3_0/BUILD.gn:30-33`):

```gni
innerapi_tags = [
  "chipsetsdk",           # 芯片 SDK 级别
  "platformsdk_indirect",  # 平台 SDK 间接依赖
]
```

### API 可见性

| 标签 | 含义 | 使用范围 |
|-----|------|---------|
| 无标记 | 公共 API | 所有模块可用 |
| `chipsetsdk` | 芯片 SDK API | 仅芯片相关模块 |
| `platformsdk_indirect` | 间接 SDK API | 平台内部使用 |

### 头文件导出

**bundle.json 中的 inner_kits 配置**:

```json
{
  "inner_kits": [
    {
      "name": "//drivers/interface/audio/v6_0:libaudio_proxy_6.0",
      "header": {
        "header_files": [],
        "header_base": "//drivers/interface/audio"
      }
    }
  ]
}
```

---

## 驱动入口结构

### HdfDriverEntry 定义

编译生成的驱动模板包含以下结构：

```cpp
// 编译时生成的结构
struct HdfDriverEntry {
    const char *moduleName;    // 必须与 BUILD.gn 中的 module_name 匹配
    int (*Bind)(struct HdfDeviceObject *deviceObject);
    int (*Init)(struct HdfDeviceObject *deviceObject);
    void (*Release)(struct HdfDeviceObject *deviceObject);
};

#ifndef __cplusplus
// C 风格导出
HdfDriverEntry *GetDriverEntry(void);
#endif
```

### 服务实例化接口

```cpp
#ifdef __cplusplus
extern "C" {
#endif

// 服务构造/析构函数
IModuleInterface *ModuleInterfaceServiceConstruct();
void ModuleInterfaceServiceRelease(IModuleInterface *obj);

#endif
```

---

## 序列化机制

### Parcelable 类型

支持跨进程序列化的类型需要声明：

```idl
// IAllocator.idl
sequenceable OHOS.HDI.Display.BufferManager.AllocateMemResult;
sequenceable OHOS.HDI.Display.BufferManager.BufferHandle;
```

### BufferHandle 结构

Display 模块使用 `BufferHandle` 进行内存共享：

```cpp
struct BufferHandle {
    int fd;                    // 文件描述符
    void *virAddr;            // 虚拟地址
    int size;                  // 缓冲区大小
    int width;                 // 宽度
    int height;                // 高度
    // ...
};
```

---

## 常见模式

### 1. 单例服务模式

```cpp
class ModuleService : public IModuleInterface {
public:
    static ModuleService *GetInstance();
    // ...
};
```

### 2. 回调注册模式

```cpp
// 注册
int32_t Register(const sptr<IModuleCallback>& callback);

// 注销
int32_t Unregister();
```

### 3. 事件通知模式

```idl
[callback] interface IModuleCallback {
    void OnEvent(EventType type, EventData data);
};

interface IModuleInterface {
    void Subscribe(EventType type, [in] IModuleCallback callback);
    void Unsubscribe(EventType type);
};
```

---

## 资源管理最佳实践

### 1. 及时释放资源

```cpp
// GOOD
void Process() {
    auto buffer = AllocBuffer();
    // 使用...
    FreeBuffer(buffer);  // 及时释放
}

// BAD
void Process() {
    auto buffer = AllocBuffer();
    // 使用...
    // 忘记释放
}
```

### 2. 使用智能指针

```cpp
// GOOD
auto module = IModule::Get();
module->DoSomething();

// GOOD - 自定义 deleter
std::unique_ptr<Handle, decltype(&FreeHandle)> handle(AllocHandle(), FreeHandle);
```

### 3. 错误处理

```cpp
int32_t ret = module->Method(param);
if (ret != HDF_SUCCESS) {
    // 错误处理
    HDF_LOGE("Method failed: %{public}d", ret);
    return ret;
}
```
