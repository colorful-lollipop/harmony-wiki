# 07. 编译产物

## 目的

本文档介绍 Neural Network Runtime 的编译产物，包括产物清单、安装路径和运行时加载关系。

## 适用范围

- 系统集成工程师
- 发布工程师

## 产物清单

### 核心库

| 产物名称 | 类型 | 说明 | 安装路径 |
|----------|------|------|----------|
| `libneural_network_core.so` | 共享库 | Core 抽象层 | /system/lib/ |
| `libneural_network_runtime.so` | 共享库 | Runtime 实现层 | /system/lib/ |

### 示例驱动（可选）

| 产物名称 | 类型 | 说明 | 安装路径 |
|----------|------|------|----------|
| `libmindspore-lite.so` | 预构建共享库 | MindSpore Lite 库 | /vendor/lib/ |
| `libnnrt_device_service_2.0.so` | 共享库 | HDI 设备服务 | /vendor/lib/ |
| `libnnrt_driver.so` | 共享库 | HDI 驱动入口 | /vendor/lib/ |

### 头文件

| 产物名称 | 类型 | 说明 | 安装路径 |
|----------|------|------|----------|
| `neural_network_runtime_type.h` | 头文件 | 类型定义 | SDK 头文件目录 |
| `neural_network_runtime.h` | 头文件 | 模型构建 API | SDK 头文件目录 |
| `neural_network_core.h` | 头文件 | 编译执行 API | SDK 头文件目录 |

## 产物详细信息

### libneural_network_core.so

**来源**: `frameworks/native/neural_network_core/BUILD.gn`

**源文件** (6 个):
- `backend_manager.cpp` - 后端管理器
- `backend_registrar.cpp` - 后端注册器
- `neural_network_core.cpp` - Core 初始化
- `nnrt_client.cpp` - NNRT 客户端
- `tensor_desc.cpp` - 张量描述符
- `utils.cpp` - 工具函数
- `validation.cpp` - 参数验证

**依赖**:
- `c_utils:utils`
- `hilog:libhilog`
- `openssl:libcrypto_shared`

**证据**: `frameworks/native/neural_network_core/BUILD.gn`

### libneural_network_runtime.so

**来源**: `frameworks/native/neural_network_runtime/BUILD.gn`

**源文件** (138 个):
- Core 源文件 (28 个): 设备适配、模型管理、编译、执行、内存管理等
- 算子源文件 (110 个): 各种算子的构建器实现

**核心组件**:
- HDI 设备适配 (V1.0/V2.0/V2.1)
- LiteGraph 转换
- 算子注册表
- 内存管理器
- 编译缓存

**依赖**:
- `libneural_network_core.so`
- `drivers_interface_nnrt:libnnrt_proxy_1.0/2.0/2.1`
- `hdf_core:libhdf_utils`
- `hilog:libhilog`
- `hitrace:libhitracechain`
- `init:libbegetutil`
- `ipc:ipc_core`
- `json:nlohmann_json_static`
- `mindspore:mindir_lib`
- `eventhandler:libeventhandler`

**证据**: `frameworks/native/neural_network_runtime/BUILD.gn`

## 安装路径

### 系统镜像 (system.img)

```
/system/lib/
    ├── libneural_network_core.so
    └── libneural_network_runtime.so

/system/include/
    └── neural_network_runtime/
        ├── neural_network_runtime_type.h
        ├── neural_network_runtime.h
        └── neural_network_core.h
```

### 芯片基线镜像 (chipset_base_dir)

```
/vendor/lib/
    ├── libmindspore-lite.so
    ├── libnnrt_device_service_2.0.so
    └── libnnrt_driver.so

/vendor/etc/init/
    └── nnrt_host.cfg  # 驱动服务配置
```

## 运行时加载关系

### 进程加载关系

```
应用进程 (Application Process)
    └── 加载 libneural_network_runtime.so
        ├── 依赖 libneural_network_core.so
        ├── 依赖 libhilog.so
        ├── 依赖 libipc_core.so
        ├── 依赖 libnnrt_proxy_2.1.so (HDI 代理)
        │   └── 依赖 libhdf_ipc_adapter.so
        └── ...

驱动进程 nnrt_host (独立进程)
    └── 加载 libnnrt_driver.so
        ├── 依赖 libnnrt_device_service_2.0.so
        ├── 依赖 libnnrt_stub_2.0.so (HDI 存根)
        ├── 依赖 libhdf_host.so
        ├── 依赖 libipc_core.so
        └── 依赖 libmindspore-lite.so (示例驱动)
```

### 库加载时序

```
1. 应用启动
   └── 加载 libneural_network_runtime.so
       ├── 自动加载 libneural_network_core.so
       └── 调用 BackendRegistrar 构造函数
           └── 注册 NNBackend 到 BackendManager

2. 第一次调用 NNRt API
   └── BackendManager::GetInstance() 初始化单例
       └── 扫描并加载已注册的后端

3. 设备发现
   └── HDI 代理库加载
       └── 与 nnrt_host 进程建立 IPC 连接

4. 模型编译/执行
   └── 通过 IPC 调用 nnrt_host 中的服务
```

## 运行时依赖

### 必需依赖

| 依赖库 | 说明 | 加载时机 |
|--------|------|----------|
| libc.so | C 标准库 | 启动时 |
| libm.so | 数学库 | 启动时 |
| libdl.so | 动态链接库 | 启动时 |
| libhilog.so | 日志库 | 启动时 |
| libipc_core.so | IPC 核心 | 启动时 |
| libneural_network_core.so | Core 层 | 启动时 |

### 可选依赖

| 依赖库 | 说明 | 加载时机 |
|--------|------|----------|
| libnnrt_proxy_1.0.so | HDI v1.0 代理 | 发现 v1.0 设备时 |
| libnnrt_proxy_2.0.so | HDI v2.0 代理 | 发现 v2.0 设备时 |
| libnnrt_proxy_2.1.so | HDI v2.1 代理 | 发现 v2.1 设备时 |
| libhitracechain.so | 性能跟踪 | 启用跟踪时 |
| libeventhandler.so | 事件处理 | 创建 Executor 时 |

## 产物大小

### 库文件大小（估算）

| 产物 | 大小（估算） | 说明 |
|------|-------------|------|
| libneural_network_core.so | ~100 KB | Core 抽象层 |
| libneural_network_runtime.so | ~2 MB | Runtime 实现 + 110+ 算子 |
| libnnrt_device_service_2.0.so | ~500 KB | HDI 服务实现 |
| libnnrt_driver.so | ~50 KB | 驱动入口 |

### 资源占用

**ROM**: 1024 KB（由 bundle.json 声明）
**RAM**: 2048 KB（由 bundle.json 声明）

**证据**: `bundle.json:22-23`

## 版本信息

### 库版本

| 产物 | 版本 | 说明 |
|------|------|------|
| libneural_network_core.so | 4.0 | 与组件版本一致 |
| libneural_network_runtime.so | 4.0 | 与组件版本一致 |

### 兼容性

- **API 版本**: 9 (基础), 11 (扩展)
- **HDI 版本**: 1.0, 2.0, 2.1 (同时支持)

## 调试符号

### 符号文件

编译时生成以下调试符号文件：

```
out/rk3568/symbols/
    └── system/lib/
        ├── libneural_network_core.so
        └── libneural_network_runtime.so
```

### 剥离符号

发布版本会剥离调试符号以减小体积：

```bash
# 带符号版本（用于调试）
libneural_network_runtime.so

# 剥离符号版本（发布）
libneural_network_runtime.so (stripped)
```

## 相关跳转

- [GN 构建目标](06_GN_Targets.md)
- [架构说明](02_Architecture.md)
- [安全风险评审](08_Security_Review.md)
