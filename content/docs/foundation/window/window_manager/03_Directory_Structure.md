# 目录结构

## 目的

本文档详细说明 Window Manager 仓库的目录组织、模块职责和关键文件位置。

## 顶层目录结构

```
foundation/window/window_manager/
├── bundle.json                    # 组件配置
├── windowmanager_aafwk.gni        # GN 构建变量定义
├── scene_board_enable.gni         # Scene Board 特性开关
├── hisysevent.yaml                # HiSysEvent 日志配置
├── LICENSE                        # Apache 2.0 许可证
├── README.md / README_zh.md       # 项目说明
│
├── dm/                            # Display Manager Client
├── dm_lite/                       # 轻量级 Display Manager
├── dmserver/                      # Display Manager Server
├── wm/                            # Window Manager Client
├── wmserver/                      # Window Manager Server
├── window_scene/                  # Scene Board 架构
│
├── interfaces/                    # 对外接口
│   ├── innerkits/                 # Native API
│   └── kits/                      # JS API / N-API
│
├── extension/                     # 扩展功能
├── utils/                         # 工具类
├── resources/                     # 资源文件
├── sa_profile/                    # System Ability 配置
├── snapshot/                      # 截图命令行工具
├── setresolution/                 # 设置分辨率工具
├── previewer/                     # IDE 预览器支持
├── product/                       # 产品配置
├── etc/                           # 配置文件
└── test/                          # 测试代码（本 Wiki 不覆盖）
```

## 模块详细说明

### 1. wm/ - Window Manager Client

**职责**：窗口管理客户端，提供应用层窗口操作接口

```
wm/
├── BUILD.gn                       # 构建: libwm.so, libwm_lite.so
├── include/                       # 头文件
│   ├── window.h                   # Window 类定义
│   ├── window_manager.h           # WindowManager 类
│   ├── window_option.h            # 窗口配置选项
│   ├── window_scene.h             # 窗口场景
│   ├── window_accessibility_controller.h  # 无障碍控制
│   └── zidl/                      # IPC 接口定义
│       ├── window_interface.h
│       ├── window_manager_interface.h
│       └── ...
└── src/                           # 实现
    ├── window.cpp
    ├── window_impl.cpp
    ├── window_manager.cpp
    ├── window_scene.cpp
    ├── picture_in_picture_controller.cpp  # PIP 控制
    ├── floating_ball_controller.cpp       # 悬浮球控制
    └── zidl/                      # IPC Proxy/Stub 实现
        ├── window_proxy.cpp
        ├── window_stub.cpp
        └── ...
```

**关键产物**：
- `libwm.so` - 完整窗口管理客户端库
- `libwm_lite.so` - 轻量级版本
- `libnative_window_manager.so` - NDK 接口

### 2. dm/ - Display Manager Client

**职责**：显示管理客户端，提供显示信息查询和屏幕控制

```
dm/
├── BUILD.gn                       # 构建: libdm.so
├── include/                       # 头文件
│   ├── display.h                  # Display 类
│   ├── display_manager.h          # DisplayManager 类
│   ├── screen.h                   # Screen 类
│   ├── screen_manager.h           # ScreenManager 类
│   └── zidl/                      # IPC 接口
└── src/                           # 实现
    ├── display.cpp
    ├── display_manager.cpp
    ├── display_manager_adapter.cpp
    ├── screen.cpp
    ├── screen_manager.cpp
    └── zidl/
```

**关键产物**：
- `libdm.so` - 显示管理客户端库
- `libnative_display_manager.so` - NDK 接口

### 3. wmserver/ - Window Manager Server

**职责**：窗口管理服务，处理窗口布局、Z序、生命周期

```
wmserver/
├── BUILD.gn                       # 构建: libwms.so
├── include/                       # 头文件
│   ├── window_manager_service.h   # WMS 服务
│   ├── window_controller.h        # 窗口控制
│   ├── window_root.h              # 窗口树根
│   ├── window_layout_policy.h     # 布局策略
│   ├── window_node.h              # 窗口节点
│   └── zidl/                      # IPC 接口
│       └── IWindowManager.idl     # IDL 接口定义
└── src/                           # 实现
    ├── window_manager_service.cpp
    ├── window_controller.cpp
    ├── window_root.cpp
    ├── window_layout_policy.cpp
    ├── window_layout_policy_cascade.cpp   # 层叠布局
    ├── window_layout_policy_tile.cpp      # 平铺布局
    ├── window_node.cpp
    ├── window_node_container.cpp
    ├── drag_controller.cpp        # 拖拽控制
    ├── snapshot_controller.cpp    # 截图控制
    └── zidl/
        └── window_manager_stub.cpp
```

**关键产物**：
- `libwms.so` - 窗口管理服务（System Ability）

### 4. dmserver/ - Display Manager Server

**职责**：显示管理服务，管理显示硬件、截图、亮灭屏

```
dmserver/
├── BUILD.gn                       # 构建: libdms.so
├── IDisplayManager.idl            # IDL 接口定义
├── include/                       # 头文件
│   ├── display_manager_service.h
│   ├── abstract_display_controller.h
│   ├── abstract_screen_controller.h
│   └── ...
└── src/                           # 实现
    ├── display_manager_service.cpp
    ├── display_manager_service_inner.cpp
    ├── abstract_display_controller.cpp
    ├── abstract_screen_controller.cpp
    ├── screen_rotation_controller.cpp
    ├── display_power_controller.cpp
    └── ...
```

**关键产物**：
- `libdms.so` - 显示管理服务（System Ability）

### 5. window_scene/ - Scene Board 架构

**职责**：新一代窗口场景管理架构

```
window_scene/
├── BUILD.gn                       # 根构建文件
├── common/                        # 公共定义
│   ├── BUILD.gn
│   └── ...
├── session/                       # SceneSession 实现
│   ├── BUILD.gn                   # 构建: scene_session
│   ├── host/include/              # SceneSession 头文件
│   └── src/                       # 实现
├── session_manager/               # 会话管理
│   ├── BUILD.gn                   # 构建: session_manager, session_manager_lite
│   ├── include/                   # SessionManager 头文件
│   └── src/                       # 实现
├── screen_session_manager/        # 屏幕会话管理
│   ├── BUILD.gn                   # 构建: screen_session_manager
│   ├── include/
│   └── src/
├── screen_session_manager_client/ # 屏幕会话管理客户端
│   └── BUILD.gn                   # 构建: screen_session_manager_client
├── session_manager_service/       # 会话管理服务
│   └── BUILD.gn
├── intention_event/               # 意图事件处理
│   ├── service/                   # ANR 管理
│   └── framework/                 # ANR 处理
└── interfaces/                    # 对外接口
    ├── innerkits/                 # libwsutils
    └── kits/                      # N-API / ANI 接口
        ├── napi/
        └── ani/
```

**关键产物**：
- `libscene_session.z.so`
- `libscene_session_manager.z.so`
- `libscreen_session_manager.z.so`
- `libwsutils.z.so`

### 6. interfaces/ - 对外接口

**职责**：定义 Window Manager 对外暴露的所有 API

```
interfaces/
├── innerkits/                     # Native C++ API
│   ├── wm/                        # Window API
│   │   ├── window.h
│   │   ├── window_manager.h
│   │   └── wm_common.h
│   └── dm/                        # Display API
│       ├── display.h
│       └── display_manager.h
└── kits/                          # JS / N-API / NDK
    ├── napi/                      # N-API 实现
    │   ├── window_runtime/        # Window Stage N-API
    │   ├── picture_in_picture_napi/  # PIP N-API
    │   ├── floating_ball_napi/    # 悬浮球 N-API
    │   ├── screenshot/            # 截图 N-API
    │   ├── window_extension/      # 窗口扩展 N-API
    │   └── ...
    ├── ani/                       # ANI (ArkNative Interface)
    │   ├── window_runtime/
    │   └── scene_session_manager/
    ├── cj/                        # Cangjie FFI
    │   ├── window_runtime/
    │   └── display_runtime/
    └── ndk/                       # NDK C API
        └── wm/
            └── oh_window.h
```

### 7. extension/ - 扩展功能

**职责**：窗口扩展和系统 UI 扩展支持

```
extension/
├── window_extension/              # 窗口扩展
│   ├── BUILD.gn                   # 构建: libwindow_extension
│   ├── include/
│   └── src/
├── modal_system_ui_extension/     # 模态系统 UI 扩展
│   ├── BUILD.gn                   # 构建: libmodal_system_ui_extension_client
│   ├── include/
│   └── src/
└── extension_connection/          # 扩展连接
    ├── BUILD.gn                   # 构建: libwindow_extension_client
    ├── include/
    └── src/
```

### 8. utils/ - 工具类

**职责**：公共工具类和基础设施

```
utils/
├── BUILD.gn                       # 构建: libwmutil, libwmutil_base
├── include/                       # 头文件
│   ├── window_visibility_info.h
│   ├── singleton_container.h
│   └── ...
└── src/                           # 实现
    ├── window_visibility_info.cpp
    └── ...
```

**关键产物**：
- `libwmutil.so` / `libwmutil_base.so`

### 9. resources/ - 资源文件

```
resources/
├── abc/                           # ArkTS 字节码
├── config/                        # 配置文件
│   ├── window_manager_config.xml
│   └── build/                     # 构建配置
│       └── coverage_flags.gni
└── media/                         # 媒体资源
```

### 10. sa_profile/ - System Ability 配置

```
sa_profile/
├── 4606.json                      # WindowManagerService SA 配置
├── 4607.json                      # DisplayManagerService SA 配置
└── scene_board/                   # Scene Board SA 配置
    └── 4607.json
```

**SA 配置示例** (4606.json):
```json
{
    "process": "foundation",
    "systemability": [
        {
            "name": 4606,
            "libpath": "libwms.z.so",
            "run-on-create": true,
            "distributed": false,
            "bootphase": "CoreStartPhase"
        }
    ]
}
```

## 源代码统计

| 目录 | 类型 | 估算代码量 |
|------|------|-----------|
| wm/ | 客户端 | ~30K 行 |
| dm/ | 客户端 | ~10K 行 |
| wmserver/ | 服务端 | ~50K 行 |
| dmserver/ | 服务端 | ~20K 行 |
| window_scene/ | 新架构 | ~100K 行 |
| interfaces/ | 接口层 | ~20K 行 |
| 总计 | - | ~230K 行 |

## 关键文件速查表

| 功能 | 文件路径 |
|------|----------|
| 组件配置 | `bundle.json` |
| GN 变量 | `windowmanager_aafwk.gni` |
| Window 类 | `wm/include/window.h` |
| WindowManager | `wm/include/window_manager.h` |
| Display 类 | `dm/include/display.h` |
| WMS 服务 | `wmserver/include/window_manager_service.h` |
| DMS 服务 | `dmserver/include/display_manager_service.h` |
| SceneSession | `window_scene/session/host/include/scene_session.h` |
| SessionManager | `window_scene/session_manager/include/scene_session_manager.h` |

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [GN Targets](06_GN_Targets.md)
