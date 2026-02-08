# 目录结构

## 整体结构

```
//base/useriam/face_auth/
├── bundle.json                  # 模块描述文件
├── face_auth.gni                # GN 全局配置
├── cfi_blocklist.txt            # CFI 禁用列表
│
├── common/                      # 公共模块
│   ├── BUILD.gn                # 公共构建配置
│   ├── inc/                    # 公共头文件
│   │   └── face_auth_defines.h # 公共定义（错误码）
│   ├── utils/                  # 工具类
│   └── logs/                   # 日志配置
│       └── iam_logger.h        # 日志宏定义
│
├── frameworks/                  # 框架层
│   ├── js/napi/                # JS N-API 接口
│   │   ├── BUILD.gn
│   │   └── src/
│   │       └── face_auth_napi.cpp  # N-API 实现
│   │
│   ├── ets/ani/                # ETS ANI 接口
│   │   ├── BUILD.gn
│   │   ├── idl/
│   │   │   └── ohos.userIAM.faceAuth.taihe
│   │   └── src/
│   │       ├── ani_constructor.cpp
│   │       └── ohos.userIAM.faceAuth.impl.cpp
│   │
│   └── ipc/                     # IPC 通信模块
│       ├── BUILD.gn
│       ├── inc/
│       │   ├── iface_auth.h              # 接口定义
│       │   ├── iface_auth_ipc_interface_code.h  # 命令码
│       │   ├── face_auth_client.h        # 客户端接口
│       │   ├── face_auth_proxy.h         # 代理类
│       │   └── face_auth_stub.h          # 存根类
│       └── src/
│           ├── face_auth_client_impl.cpp # 客户端实现
│           ├── face_auth_proxy.cpp       # 代理实现
│           └── face_auth_stub.cpp       # 存根实现
│
├── interfaces/                  # API 暴露
│   └── inner_api/              # 内部 API（系统服务使用）
│
├── sa_profile/                 # SA 配置
│   ├── BUILD.gn
│   └── 942.json               # SA ID 942 配置
│
├── services/                   # SA 实现（核心服务）
│   ├── BUILD.gn
│   ├── inc/
│   │   ├── face_auth_service.h           # SA 主类
│   │   ├── face_auth_hdi.h               # HDI 类型别名
│   │   ├── face_auth_driver_hdi.h       # 驱动 HDI
│   │   ├── face_auth_all_in_one_executor_hdi.h  # Executor HDI
│   │   ├── face_auth_executor_callback_hdi.h     # HDI 回调
│   │   ├── face_auth_interface_adapter.h         # HDI 适配
│   │   ├── sa_command_manager.h         # SA 命令管理
│   │   ├── service_ex_manager.h         # 服务扩展管理
│   │   └── screen_brightness_manager.h  # 屏幕亮度管理
│   └── src/
│       ├── face_auth_service.cpp
│       ├── face_auth_driver_hdi.cpp
│       ├── face_auth_all_in_one_executor_hdi.cpp
│       ├── face_auth_executor_callback_hdi.cpp
│       ├── face_auth_interface_adapter.cpp
│       ├── sa_command_manager.cpp
│       ├── service_ex_manager.cpp
│       └── screen_brightness_manager.cpp
│
├── services_ex/                # 扩展服务实现
│   ├── BUILD.gn
│   ├── inc/
│   │   └── thread_handler.h
│   └── src/
│       ├── finite_state_machine_builder.cpp
│       ├── finite_state_machine_impl.cpp
│       ├── screen_brightness_task.cpp
│       └── thread_handler_impl.cpp
│
├── ui/                        # UI 模块
│   └── Settings_FaceAuth/     # 人脸设置界面
│
└── figures/                   # 文档图片
    └── faceauth_architecture.png
```

## 模块职责

| 目录 | 职责 | 稳定性 |
|------|------|--------|
| `common/` | 日志、工具类 | 稳定 |
| `frameworks/js/napi/` | JS API 绑定 | 稳定 |
| `frameworks/ets/ani/` | ETS API 绑定 | 稳定 |
| `frameworks/ipc/` | IPC 通信 | 稳定 |
| `services/` | SA 实现、HDI 适配 | 稳定 |
| `services_ex/` | 扩展功能（状态机、线程） | 稳定 |
| `sa_profile/` | SA 配置 | 稳定 |

## 依赖方向

```
common/                    # 无外部依赖
    ↓
frameworks/ipc/           # 依赖 common/
    ↓
frameworks/js/napi/       # 依赖 frameworks/ipc/
    ↓
services/                 # 依赖 frameworks/ipc/
    ↓
services_ex/              # 依赖 services/
```

## 排除目录

以下目录不计入架构分析（测试相关）：

- `test/` - 单元测试
- `test/unittest/` - 单元测试
- `test/fuzztest/` - 模糊测试
