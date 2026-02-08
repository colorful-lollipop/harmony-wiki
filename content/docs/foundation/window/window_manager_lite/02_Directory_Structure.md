# 目录结构

## 顶层结构

```
window_manager_lite/
├── frameworks/          # 客户端代码 (C++)
│   ├── ims/           # 输入管理客户端
│   └── wms/           # 窗口管理客户端
├── interfaces/         # 接口定义
│   └── innerkits/     # 模块间内部接口
├── services/          # 服务端代码 (C++)
│   ├── ims/           # 输入管理服务
│   └── wms/           # 窗口管理服务
├── wiki/              # 本文档目录
├── BUILD.gn           # GN 构建入口
├── bundle.json        # 组件配置
└── README*.md         # 项目说明
```

## 模块职责说明

### 1. frameworks/ - 客户端

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `frameworks/wms/` | WMS 客户端代理 | `lite_wms_client.h/cpp` - IPC 客户端<br>`lite_proxy_window.h/cpp` - 窗口代理<br>`lite_wm_requestor.h/cpp` - 请求发送 |
| `frameworks/ims/` | IMS 客户端代理 | `input_event_listener_proxy.h/cpp`<br>`input_event_client_proxy.h/cpp` |

**证据**: `BUILD.gn:38-54` - wms_client sources 列表

### 2. interfaces/innerkits/ - 模块间接口

| 文件 | 职责 |
|------|------|
| `iwindows_manager.h` | 窗口管理器抽象接口 |
| `iwindow.h` | 窗口抽象接口 |
| `isurface.h` | Surface 抽象接口 |
| `input_event_listener_proxy.h` | 输入事件监听器接口 |
| `lite_wm_type.h` | 共用类型定义 |

### 3. services/ - 服务端

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `services/wms/` | WMS 服务端 | `lite_wms.h/cpp` - 请求处理分发<br>`lite_wm.h/cpp` - 窗口管理器实现<br>`lite_win.h/cpp` - 窗口实现<br>`samgr_wms.cpp` - SAMGR 服务注册 |
| `services/ims/` | IMS 服务端 | `input_manager_service.h/cpp` - 输入管理服务<br>`input_event_distributer.h/cpp` - 事件分发器<br>`input_event_hub.h/cpp` - 输入设备中心<br>`samgr_ims.cpp` - SAMGR 服务注册 |

## 文件清单（非测试代码）

### 客户端文件 (frameworks/)

```
frameworks/ims/
└── input_event_listener_proxy.cpp

frameworks/wms/
├── iwindows_manager.cpp
├── lite_proxy_surface.cpp
├── lite_proxy_surface.h
├── lite_proxy_window.cpp
├── lite_proxy_window.h
├── lite_proxy_windows_manager.cpp
├── lite_proxy_windows_manager.h
├── lite_win_requestor.cpp
├── lite_win_requestor.h
├── lite_wm_requestor.cpp
├── lite_wm_requestor.h
├── lite_wms_client.cpp
└── lite_wms_client.h
```

### 接口文件 (interfaces/)

```
interfaces/innerkits/
├── input_event_listener_proxy.h
├── isurface.h
├── iwindow.h
├── iwindows_manager.h
└── lite_wm_type.h
```

### 服务端文件 (services/)

```
services/ims/
├── input_event_client_proxy.cpp
├── input_event_distributer.cpp
├── input_event_distributer.h
├── input_event_hub.cpp
├── input_event_hub.h
├── input_manager_service.cpp
├── input_manager_service.h
└── samgr_ims.cpp

services/wms/
├── lite_win.cpp
├── lite_win.h
├── lite_wm.cpp
├── lite_wm.h
├── lite_wms.cpp
├── lite_wms.h
├── samgr_wms.cpp
└── wms.cpp
```

## 测试目录 (不纳入本文档范围)

```
test/
├── BUILD.gn
└── sample_window/  # 示例代码（不计入业务分析）
```

**注意**: 根据要求，本文不涉及测试代码分析。所有测试相关目录 (`test/`, `*_test.*`, `fuzz/`) 均已排除。
