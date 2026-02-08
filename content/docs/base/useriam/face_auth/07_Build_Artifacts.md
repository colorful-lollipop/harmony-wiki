# 编译产物

## 产物清单

### 运行时产物

| 产物 | 类型 | Target | 安装路径 | 加载时机 |
|------|------|--------|----------|----------|
| `libfaceauth.so` | 动态库 | `faceauth` | `system/lib/module/useriam/` | 应用首次加载 N-API |
| `libfaceauth_framework.so` | 动态库 | `faceauth_framework` | `system/lib/` | SA 启动时 |
| `libfaceauthservice.so` | 动态库 | `faceauthservice` | `system/lib/` | SA 启动时 |
| `libfaceauthservice_ex.so` | 动态库 | `faceauthservice_ex` | `system/lib/` | SA 启动时 |
| `libfaceauth_ani.so` | 动态库 | `faceauth_ani` | `system/lib/` | ETS 应用加载 |
| `ohos.userIAM.faceAuth.abc` | ABC 字节码 | `face_auth_taihe_abc` | `system/framework/` | ETS 运行时 |

### SA 配置产物

| 产物 | 类型 | Target | 安装路径 |
|------|------|--------|----------|
| `942.json` | JSON | `faceauth_sa_profile` | `system/sa_profile/` |

## 安装路径详情

```
system/
├── lib/
│   ├── module/
│   │   └── useriam/
│   │       └── libfaceauth.so          # JS N-API
│   │
│   ├── libfaceauth_framework.so        # IPC 框架
│   ├── libfaceauthservice.so           # SA 主服务
│   ├── libfaceauthservice_ex.so        # SA 扩展
│   └── libfaceauth_ani.so              # ETS ANI
│
├── framework/
│   └── ohos.userIAM.faceAuth.abc       # ETS 字节码
│
└── sa_profile/
    └── 942.json                        # SA 配置
```

## 运行时加载关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                         应用启动加载顺序                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. 应用进程启动                                                      │
│     │                                                               │
│     ▼                                                               │
│  2. 加载 libfaceauth.so (JS/TS 调用 @ohos/faceAuth 时)               │
│     │                                                               │
│     ├── 依赖: libfaceauth_framework.so                              │
│     │                                                               │
│     ▼                                                               │
│  3. FaceAuthClient::GetInstance()                                    │
│     │                                                               │
│     ├── SAMgr 获取 SA 942                                            │
│     │                                                               │
│     ▼                                                               │
│  4. 加载 libfaceauthservice.so                                       │
│     │                                                               │
│     ├── 依赖: libfaceauth_framework.so                               │
│     │                                                               │
│     ▼                                                               │
│  5. libfaceauthservice_ex.so                                         │
│     │                                                               │
│     └── libfaceauthservice.so 依赖加载                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## SA 加载机制

**SA 配置** (`sa_profile/942.json`):

```json
{
    "process": "useriam",
    "systemability": [
        {
            "name": 942,
            "libpath": "libfaceauthservice.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

| 配置项 | 值 | 说明 |
|--------|------|------|
| `process` | `useriam` | 运行进程名 |
| `libpath` | `libfaceauthservice.z.so` | SA 主库路径 |
| `run-on-create` | `true` | 首次请求时启动 |
| `distributed` | `false` | 非分布式 SA |

## HDI 依赖

| 依赖组件 | 用途 | 最小版本 |
|----------|------|----------|
| `libface_auth_proxy_2.0.z.so` | HDI 接口桩 | 2.0 |

## 资源占用

**bundle.json 配置**:

```json
{
    "rom": "1024KB",
    "ram": "1306KB"
}
```

## CFI 边界

**文件**: `cfi_blocklist.txt`

列出禁用 CFI 检查的函数/文件：

```
# Copyright (c) 2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
```

## 条件编译

### FACE_USE_DISPLAY_MANAGER_COMPONENT

**定义位置**: `services_ex/BUILD.gn`

**条件**: `face_use_display_manager_component == true`

**依赖组件**:

```gn
if (face_use_display_manager_component) {
  external_deps += [
    "display_manager:displaymgr",
    "ipc:ipc_core",
  ]
  defines += [ "FACE_USE_DISPLAY_MANAGER_COMPONENT" ]
}
```

### FACE_USE_SENSOR_COMPONENT

**定义位置**: `services_ex/BUILD.gn`

**条件**: `face_use_sensor_component == true`

**依赖组件**:

```gn
if (defined(face_use_sensor_component)) {
  external_deps += [ "sensor:sensor_interface_native" ]
  defines += [ "FACE_USE_SENSOR_COMPONENT" ]
}
```
