# 目录结构

## 2.1 顶层目录

```
/foundation/communication/ipc/
├── interfaces/          # 对外接口存放目录
│   ├── innerkits/       # 对内部子系统暴露的头文件
│   ├── kits/            # 对应用/开发者暴露的接口
│   └── ...
├── ipc/                 # IPC 框架代码
│   ├── native/          # Native 实现
│   └── ...
├── dbinder/            # DBinder（分布式 Binder）实现
├── utils/              # 工具类
├── example/            # 示例代码
├── figures/            # 架构图等文档资源
├── dl_deps/            # 动态依赖管理
├── test/               # 测试目录（不计入业务证据）
├── config/             # 构建配置
├── BUILD.gn            # 根构建入口
├── bundle.json         # 组件包配置
├── config.gni          # GN 配置
└── README_zh.md        # 项目说明
```

## 2.2 核心目录详解

### interfaces/innerkits/ （Inner API）

| 目录 | 职责 | 关键头文件 |
|------|------|------------|
| `ipc_core/` | 核心 IPC 接口 | `ipc_skeleton.h`, `iremote_object.h` |
| `ipc_single/` | 单进程 IPC 库 | `message_parcel.h`, `message_option.h` |
| `ipc_napi_common/` | NAPI 公共实现 | `napi_remote_object.h` |
| `libdbinder/` | DBinder 接口 | `dbinder_service.h` |
| `c_api/` | C API 接口 | `ipc_cparcel.h`, `ipc_kit.h` |
| `cj/` | Cangjie FFI 绑定 | `ipc_ffi.h` |
| `rust/` | Rust 绑定 | `ipc_rust.h` |

### interfaces/kits/ （对外 Kit）

| 目录 | 职责 | 产物 |
|------|------|------|
| `js/napi/` | JS NAPI RPC | `librpc.z.so` |
| `ndk/` | NDK 头文件库 | `./ndk/IPCKit/` |

### ipc/native/src/ （Native 实现）

| 目录 | 职责 |
|------|------|
| `core/` | IPC 核心框架 |
| ├── `framework/` | 框架层（Proxy/Stub） |
| ├── `invoker/` | 调用层（Binder/DBinder） |
| └── `dbinder/` | DBinder 实现 |
| `napi/` | NAPI 胶水层 |
| `ani/` | ANI（Ark Native Interface） |
| `taihe/` | Taihe（现代 JS 运行时） |
| `c/` | C/Lite 系统适配 |

## 2.3 目录职责划分

### 按功能分类

| 功能区域 | 主要目录 | 职责 |
|----------|----------|------|
| **对外接口** | `interfaces/kits/` | JS NAPI、NDK 头文件 |
| **Inner API** | `interfaces/innerkits/` | Native 内部接口 |
| **核心实现** | `ipc/native/src/core/` | Binder/DBinder 框架 |
| **JS 绑定** | `ipc/native/src/napi/` | N-API 胶水代码 |
| **构建配置** | `config/` | GN 配置与 config |
| **分布式 IPC** | `dbinder/` | 跨设备通信 |

### 按稳定性分类

| 稳定性 | 目录 | 说明 |
|--------|------|------|
| **公开 API** | `interfaces/kits/`, `interfaces/innerkits/ipc_core/` | 稳定接口 |
| **内部实现** | `ipc/native/src/` | 可能变更 |
| **测试代码** | `test/` | 不保证稳定性 |

## 2.4 关键文件清单

### 头文件（公开 API）

| 文件 | 职责 | 行号 |
|------|------|------|
| `ipc_skeleton.h` | IPC 骨架类 | `interfaces/innerkits/ipc_core/include/:22` |
| `iremote_object.h` | 远程对象接口 | `interfaces/innerkits/ipc_core/include/:34` |
| `iremote_proxy.h` | 远程代理模板 | `interfaces/innerkits/ipc_core/include/:24` |
| `iremote_stub.h` | 远程存根模板 | `interfaces/innerkits/ipc_core/include/:24` |
| `message_parcel.h` | 消息数据包 | `interfaces/innerkits/ipc_core/include/:26` |
| `message_option.h` | 消息选项配置 | `interfaces/innerkits/ipc_core/include/:21` |
| `dbinder_service.h` | DBinder 服务 | `interfaces/innerkits/libdbinder/include/:126` |

### N-API 实现

| 文件 | 职责 | 行号 |
|------|------|------|
| `napi_rpc_native_module.cpp` | N-API 模块注册 | `ipc/native/src/napi/src/:48` |
| `napi_remote_object.cpp` | RemoteObject 导出 | `ipc/native/src/napi_common/source/:1731` |
| `napi_ipc_skeleton.cpp` | IPCSkeleton 导出 | `ipc/native/src/napi/src/:486` |

### 构建配置

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | 根构建入口 |
| `config.gni` | GN 配置变量 |
| `config/BUILD.gn` | 公共配置目标 |

---

*证据来源*:
- `README_zh.md:25-38` - 官方目录结构说明
- `bundle.json:59-96` - 子组件与 inner_kits 配置
- `BUILD.gn:20-44` - 构建目标定义
- Phase 1 全局扫描结果
