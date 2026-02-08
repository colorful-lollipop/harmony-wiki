# 目录结构

## 顶层目录概览

UDMF 项目采用分层目录结构，将接口声明、实现代码、适配层和配置文件分离，便于维护和扩展。源码根目录为 `foundation/distributeddatamgr/udmf`，主要包含以下顶层目录：

```
udmf/
├── interfaces/           # 对外接口声明（JS/Native/InnerKit）
├── framework/            # 核心逻辑实现
├── adapter/             # ArkUI-X 跨平台适配
├── conf/                # 配置文件
├── figures/             # 架构图等资源
├── BUILD.gn             # 根构建配置
├── bundle.json          # 模块配置
└── README_zh.md         # 项目说明
```

根据 `BUILD.gn` 根构建文件（`BUILD.gn:16-31`），UDMF 的主要构建产物通过 `udmf_packages` 组进行组织，包括 `interfaces/components`、`interfaces/innerkits`、`interfaces/jskits`、`interfaces/ndk` 等模块的产物。

## interfaces/ 目录详解

`interfaces/` 目录包含所有对外接口声明，按接口类型分为多个子目录。

### interfaces/jskits/（N-API 接口）

N-API 接口层面向 JS/ArkTS 应用开发者，提供 JavaScript 绑定。该目录结构如下：

```
interfaces/jskits/
├── module/                          # 模块注册入口
│   ├── uniform_type_descriptor_napi_module.cpp   # data.uniformTypeDescriptor 模块
│   └── unified_data_channel_napi_module.cpp      # data.unifiedDataChannel 模块
├── data/                            # 数据类型 N-API 头文件
│   ├── unified_data_napi.h          # UnifiedData 绑定
│   ├── unified_data_channel_napi.h  # UnifiedDataChannel 绑定
│   ├── uniform_type_descriptor_napi.h
│   ├── unified_record_napi.h
│   ├── summary_napi.h
│   ├── get_data_params_napi.h       # 异步参数（含 threadsafe function）
│   ├── data_load_params_napi.h      # 数据加载参数
│   └── [record 类型: text_napi.h, image_napi.h, video_napi.h 等]
├── intelligence/                    # AI/ML N-API 头文件
│   ├── native_module_intelligence.h
│   ├── text_embedding_napi.h
│   └── image_embedding_napi.h
└── common/                          # 公共工具
    ├── napi_queue.h                 # 异步工作队列
    ├── napi_data_utils.h            # 类型转换
    └── napi_error_utils.h           # 错误处理
```

关键 N-API 模块注册点：
- `data.uniformTypeDescriptor`：`uniform_type_descriptor_napi_module.cpp:40-44`
- `data.unifiedDataChannel`：`unified_data_channel_napi_module.cpp:97-101`
- `data.intelligence`：`native_module_intelligence.cpp:43-46`

### interfaces/ndk/（NDK 接口）

NDK 接口层面向 Native 应用开发者，提供 C 语言 API。核心头文件包括：

```
interfaces/ndk/
├── BUILD.gn                         # 构建配置
└── data/
    ├── udmf.h                      # 主 UDMF API（数据操作）
    ├── utd.h                       # UTD 类型 API
    ├── uds.h                       # UDS 数据结构定义
    ├── udmf_meta.h                 # 类型常量（200+ 预定义类型）
    ├── udmf_err_code.h             # 错误码定义
    └── data_provider_impl.h        # 数据提供者实现
```

NDK 接口导出符号示例（`udmf.h`）：
- `OH_UdmfData_Create/Destroy`：数据创建与销毁
- `OH_UdmfRecord_AddGeneralEntry`：添加通用条目
- `OH_Udmf_GetUnifiedData/SetUnifiedData`：数据存取

### interfaces/innerkits/（InnerKit 接口）

InnerKit 接口层面向系统组件开发者，提供 C++ 接口：

```
interfaces/innerkits/
├── BUILD.gn                         # 构建配置（udmf_client、utd_client）
├── client/                          # 客户端接口
│   ├── udmf_client.h               # UdmfClient 单例接口
│   ├── utd_client.h               # UtdClient 单例接口
│   └── udmf_async_client.h         # 异步客户端
├── data/                           # 数据结构接口
│   ├── unified_data.h              # UnifiedData 定义
│   ├── unified_record.h            # UnifiedRecord 定义
│   ├── type_descriptor.h          # TypeDescriptor 定义
│   ├── unified_data_properties.h
│   └── [record 类型: text.h, file.h, image.h 等]
├── common/                         # 公共类型与工具
│   ├── unified_meta.h              # 核心枚举（UDType、Intention）
│   ├── unified_types.h             # 公共结构（Summary、Runtime）
│   ├── error_code.h               # 错误码
│   ├── visibility.h               # 符号可见性宏
│   └── async_task_params.h         # 异步任务参数
├── convert/                       # 类型转换
│   └── ndk_data_conversion.h
├── dynamic/                        # 动态类型
│   ├── pixelmap_loader.h
│   └── pixelmap_wrapper.h
└── aipcore/                       # AI 核心接口
    └── i_aip_core_manager.h
```

关键单例接口：
- `UdmfClient::GetInstance()`（`udmf_client.h:35`）：获取客户端实例
- `UtdClient::GetInstance()`（`utd_client.h:33`）：获取类型管理实例

### interfaces/taihe/（Taihe ANI/ETS 绑定）

Taihe 接口提供 ArkTS 原生语言绑定：

```
interfaces/taihe/
├── BUILD.gn                         # 构建配置
├── ets/BUILD.gn                    # ETS 组件
├── include/                         # 头文件
│   ├── data_intelligence_impl.h
│   ├── unified_data_taihe.h
│   ├── uniform_type_descriptor_taihe.h
│   └── [数据类型头文件: text_taihe.h, image_taihe.h 等]
└── [IDL 文件: ohos.data.*.ti]
```

### interfaces/cj/（Cangjie FFI 绑定）

Cangjie 语言 FFI 绑定：

```
interfaces/cj/
├── BUILD.gn                         # 构建配置
└── include/                         # 头文件
    ├── unified_data_ffi.h
    ├── uniform_type_descriptor_ffi.h
    ├── type_descriptor_impl.h
    └── [实现头文件]
```

### interfaces/components/（UI 组件）

JS UI 组件库：

```
interfaces/components/
├── BUILD.gn                         # 构建配置
└── udmfcomponents.abc               # 预编译字节码
```

## framework/ 目录详解

`framework/` 目录包含核心逻辑实现，按模块组织代码。

### framework/common/（公共工具）

公共基础设施代码：

```
framework/common/
├── udmf_utils.h/cpp                # 通用工具（ID 生成、令牌检查）
├── tlv_util.h/cpp                  # TLV 序列化
├── tlv_object.h/cpp                # TLV 对象封装
├── concurrent_map.h                 # 线程安全 Map
├── graph.h/cpp                      # 图数据结构
├── utd_graph.h/cpp                 # UTD 类型层次图
├── custom_utd_store.h/cpp          # 自定义 UTD 持久化
├── udmf_executor.h/cpp              # 线程池执行器
├── udmf_copy_file.h/cpp             # 文件复制工具
├── udmf_radar_reporter.h            # 性能监控
├── endian_converter.h               # 字节序转换
├── base32_utils.h                  # Base32 编码
├── itypes_util.h                    # IPC 类型工具
├── logger.h                        # 日志宏
└── test/                           # 测试目录（已排除）
```

关键工具类：
- `UtdGraph`（`utd_graph.h`）：UTD 类型层次关系图（单例）
- `ConcurrentMap`（`concurrent_map.h`）：线程安全映射表
- `TLVUtil`（`tlv_util.h`）：TLV 序列化工具

### framework/innerkitsimpl/（InnerKit 实现）

InnerKit 接口的具体实现：

```
framework/innerkitsimpl/
├── client/                          # 客户端实现
│   ├── udmf_client.cpp             # UdmfClient 实现
│   ├── utd_client.cpp              # UtdClient 实现
│   ├── udmf_async_client.cpp       # 异步客户端
│   └── getter_system.cpp           # Entry Getter 工厂
├── service/                        # 服务端实现
│   ├── udmf_service.h/cpp          # 服务接口
│   ├── udmf_service_client.h/cpp   # 服务客户端（单例）
│   ├── udmf_service_proxy.h/cpp    # IPC 代理
│   ├── udmf_notifier_stub.h/cpp    # 通知存根
│   ├── utd_service_client.h/cpp     # UTD 服务客户端
│   ├── utd_service_proxy.h/cpp      # UTD IPC 代理
│   ├── utd_notifier.h/cpp          # UTD 变更通知
│   ├── progress_callback.h/cpp     # 进度回调
│   └── distributeddata_udmf_ipc_interface_code.h  # IPC 接口码
└── data/                            # 数据结构实现
    ├── unified_data.cpp            # UnifiedData 实现
    ├── unified_record.cpp          # UnifiedRecord 实现
    ├── unified_data_helper.cpp     # 序列化辅助
    ├── unified_html_record_process.cpp
    ├── type_descriptor.cpp         # TypeDescriptor 实现
    ├── preset_type_descriptors.h/cpp  # 预设类型
    ├── flexible_type.cpp          # 灵活类型
    └── [record 类型实现: plain_text.cpp, file.cpp, image.cpp 等]
```

关键单例实现：
- `UdmfServiceClient`（`udmf_service_client.h`）：服务连接管理
- `UtdClient`（`utd_client.cpp`）：UTD 类型管理
- `UtdGraph`（`utd_graph.cpp`）：类型层次图

### framework/jskitsimpl/（N-API 实现）

N-API 接口的具体实现（与 `interfaces/jskits/` 镜像对应）：

```
framework/jskitsimpl/
├── module/                          # 模块注册实现
│   └── [对应 interfaces/jskits/module/ 的实现]
├── data/                            # 数据类型实现
│   ├── uniform_type_descriptor_napi.cpp
│   ├── unified_data_channel_napi.cpp
│   ├── unified_data_napi.cpp
│   ├── get_data_params_napi.cpp     # 含 threadsafe function 实现
│   ├── data_load_params_napi.cpp
│   └── [record 类型实现]
└── intelligence/                    # AI/ML 实现
    ├── native_module_intelligence.cpp
    ├── text_embedding_napi.cpp
    └── image_embedding_napi.cpp
```

### framework/ndkimpl/（NDK 实现）

NDK 接口的具体实现（与 `interfaces/ndk/` 镜像对应）：

```
framework/ndkimpl/
├── data/
│   ├── udmf.cpp                    # C API 实现
│   ├── uds.cpp                      # UDS 实现
│   ├── utd.cpp                      # UTD 实现
│   ├── data_provider_impl.cpp       # 数据提供者
│   └── [类型实现]
└── test/                           # 测试目录（已排除）
```

## adapter/ 目录详解

ArkUI-X 跨平台适配层：

```
adapter/
├── BUILD.gn                         # 构建配置
└── framework/                       # 跨平台源码集合
    ├── common/                      # 公共适配
    ├── innerkitsimpl/               # InnerKit 适配
    └── [适配代码]
```

## conf/ 目录详解

配置文件目录：

```
conf/
├── BUILD.gn                         # 构建配置
└── uniform_data_types.json          # UTD 类型定义配置
```

`uniform_data_types.json` 定义了 UDMF 支持的所有预定义数据类型，安装路径为 `system/etc/utd/conf/`。

## _work/ 目录说明

`_work/` 目录为 Wiki 生成过程的工作区，包含事实记录和任务计划：

```
_wrok/
├── NOTES.md                         # 事实记录（源码证据）
└── PLAN.md                          # 任务计划与进度
```

## 模块职责总结

| 目录 | 职责 |
|------|------|
| `interfaces/jskits/` | N-API 接口声明 |
| `interfaces/ndk/` | NDK 接口声明 |
| `interfaces/innerkits/` | InnerKit 接口声明 |
| `framework/common/` | 公共工具与基础设施 |
| `framework/innerkitsimpl/` | InnerKit 接口实现 |
| `framework/jskitsimpl/` | N-API 接口实现 |
| `adapter/` | ArkUI-X 跨平台适配 |
| `conf/` | UTD 类型配置 |

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述与核心能力
- [10_NAPI_Reference.md](./10_NAPI_Reference.md)：N-API 接口详解
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口详解
- [12_InnerKit_Reference.md](./12_InnerKit_Reference.md)：InnerKit 接口详解
- [20_Service_Layer.md](./20_Service_Layer.md)：服务层架构
