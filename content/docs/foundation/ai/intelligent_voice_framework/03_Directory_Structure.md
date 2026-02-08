# 目录结构

> **目的**: 帮助开发者熟悉 Intelligent Voice Framework 的代码组织方式和模块职责  
> **适用范围**: 代码阅读、模块定位、功能开发  
> **最后更新**: 2026-02-06

---

## 1. 顶层目录结构

```
intelligent_voice_framework/
├── frameworks/                    # 框架层
│   ├── native/                   # Native API 实现 (C++)
│   ├── js/                       # N-API 绑定层 (JS 接口)
│   └── taihe/                     # Taihe 模块 (TODO: 待分析)
├── interfaces/                   # 接口层
│   ├── inner_api/                # 内部 API (Native)
│   └── kits/                     # 外部 API (JS TypeScript)
├── services/                     # 服务层
│   ├── intell_voice_service/     # 智能语音主服务 (SA)
│   ├── intell_voice_engine/      # 语音引擎模块
│   ├── intell_voice_trigger/     # 语音触发模块
│   └── etc/                      # 配置文件
├── sa_profile/                  # SA 配置文件
├── utils/                        # 公共工具库
├── tests/                        # 测试代码 (忽略)
├── llt/                          # 长期测试 (忽略)
├── figures/                      # 文档图片
├── wiki/                         # Wiki 文档
│   ├── README.md
│   ├── SUMMARY.md
│   ├── 01_Overview.md
│   ├── 02_Architecture.md
│   ├── 03_Directory_Structure.md
│   ├── 04_NAPI_Reference.md
│   ├── 05_Inner_API.md
│   ├── 06_Build_System.md
│   ├── 07_Artifacts.md
│   ├── 08_Security_Review.md
│   └── appendix/
├── bundle.json                   # 组件配置
├── intell_voice_service.gni      # 构建配置
├── BUILD.gn                       # 根构建文件
└── README.md                      # 项目说明
```

---

## 2. frameworks/ 目录详解

### 2.1 frameworks/native/ (Native API)

**职责**: 提供 C++ 级别的对外接口实现

```
frameworks/native/
├── BUILD.gn                       # 构建配置
├── enroll_intell_voice_engine.cpp # 注册引擎实现
├── intell_voice_manager.cpp      # 管理器实现
└── wakeup_intell_voice_engine.cpp # 唤醒引擎实现
```

**对外导出**: `intellvoice_native` (NDK 库)

### 2.2 frameworks/js/ (N-API)

**职责**: 提供 JS/TS 到 C++ 的接口绑定

```
frameworks/js/
├── BUILD.gn                       # 构建配置
└── napi/
    ├── intell_voice_manager_napi.cpp/h      # 管理器 N-API
    ├── enroll_intell_voice_engine_napi.cpp/h # 注册引擎 N-API
    ├── wakeup_intell_voice_engine_napi.cpp/h # 唤醒引擎 N-API
    ├── wakeup_manager_napi.cpp/h             # 唤醒管理器 N-API
    ├── intell_voice_common_napi.cpp/h        # 公共 N-API 工具
    ├── intell_voice_napi_util.cpp/h         # N-API 工具函数
    ├── intell_voice_napi_queue.cpp/h        # N-API 队列
    ├── engine_event_callback_napi.cpp/h     # 引擎事件回调
    ├── enroll_intell_voice_engine_callback_napi.cpp/h # 注册回调
    ├── intell_voice_update_callback_napi.cpp/h      # 更新回调
    ├── service_change_callback_napi.cpp/h   # 服务状态回调
    └── napi/                               # N-API 头文件
```

**对外导出**: `intelligentvoice` (JS 模块)

**证据来源**:
- `frameworks/js/BUILD.gn`
- `frameworks/native/BUILD.gn`

---

## 3. interfaces/ 目录详解

### 3.1 interfaces/kits/ (外部 API)

**职责**: 提供 JS/TS 类型声明

```
interfaces/kits/js/
└── @ohos.ai.intelligentVoice.d.ts  # TypeScript 声明文件
```

**证据来源**: `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts`

### 3.2 interfaces/inner_api/ (内部 API)

**职责**: 提供 Native 级别的内部接口

```
interfaces/inner_api/native/
├── intell_voice_manager.h           # 管理器接口
├── enroll_intell_voice_engine.h     # 注册引擎接口
├── wakeup_intell_voice_engine.h     # 唤醒引擎接口
├── i_headset_wakeup.h                # 耳机唤醒接口
└── intell_voice_info.h              # 信息结构体
```

**证据来源**: `interfaces/inner_api/native/`

---

## 4. services/ 目录详解

### 4.1 services/intell_voice_service/ (主服务)

**职责**: SA 服务管理、引擎协调、系统事件处理

```
services/intell_voice_service/
├── BUILD.gn                         # 构建配置
├── inc/
│   ├── i_intell_voice_service.h    # SA 接口定义
│   ├── i_intell_voice_engine.h     # 引擎接口
│   ├── intell_voice_definitions.h  # 常量定义
│   ├── i_intell_voice_engine_callback.h # 回调接口
│   └── i_intell_voice_update_callback.h # 更新回调
└── server/
    ├── sa/
    │   ├── intell_voice_service.cpp/h      # SA 服务实现
    │   ├── intell_voice_service_stub.cpp/h # SA 存根
    │   ├── intell_voice_service_manager.cpp/h # 服务管理器
    │   ├── intell_voice_engine_registrar.cpp/h # 引擎注册器
    │   └── intell_voice_trigger_registrar.cpp/h # 触发器注册器
    └── utils/
        ├── switch_observer.cpp/h   # 开关观察者
        ├── switch_provider.cpp/h   # 开关提供者
        └── system_event_observer.cpp/h # 系统事件观察者
```

**对外导出**: `intell_voice_server` (.so)

### 4.2 services/intell_voice_engine/ (引擎模块)

**职责**: 语音引擎核心逻辑

```
services/intell_voice_engine/
├── BUILD.gn                         # 构建配置
├── inc/
│   ├── engine_base.h               # 引擎基类
│   ├── intell_voice_engine_manager.h # 引擎管理器
│   ├── intell_voice_engine_stub.h  # 引擎存根
│   └── update_state.h              # 更新状态
└── server/
    ├── base/                       # 基础组件
    │   ├── engine_base.cpp
    │   ├── engine_factory.cpp
    │   ├── intell_voice_engine_stub.cpp
    │   └── audio_source.cpp
    ├── enroll/                     # 注册引擎
    │   └── enroll_engine.cpp
    ├── wakeup/                     # 唤醒引擎
    │   ├── wakeup_engine.cpp
    │   ├── wakeup_engine_impl.cpp
    │   └── headset/wakeup_wrapper.cpp
    ├── update/                     # 更新引擎
    │   └── update_engine.cpp
    └── manager/
        └── intell_voice_engine_manager.cpp
```

**对外导出**: `intelligentvoice_engine` (.so)

### 4.3 services/intell_voice_trigger/ (触发模块)

**职责**: 声音触发检测、DSP 协调

```
services/intell_voice_trigger/
├── BUILD.gn                         # 构建配置
├── inc/
│   └── trigger_service.h           # 触发服务接口
└── server/
    ├── trigger_service.cpp          # 触发服务
    ├── trigger_manager.cpp          # 触发管理器
    ├── trigger_detector.cpp        # 触发检测器
    ├── trigger_connector.cpp       # 触发连接器
    └── connector_mgr/
        ├── trigger_connector.cpp
        ├── trigger_connector_mgr.cpp
        └── trigger_host_manager.cpp
```

**对外导出**: `intelligentvoice_trigger` (.so)

### 4.4 services/etc/ (配置文件)

```
services/etc/
├── BUILD.gn                         # 构建配置
└── intell_voice_service.cfg        # 服务权限配置
```

**证据来源**:
- `services/intell_voice_service/BUILD.gn`
- `services/intell_voice_engine/BUILD.gn`
- `services/intell_voice_trigger/BUILD.gn`

---

## 5. sa_profile/ 目录

**职责**: SA 配置文件

```
sa_profile/
├── BUILD.gn                         # 构建配置
└── intell_voice_service.json       # SA 配置 (SA ID 312)
```

**证据来源**: `sa_profile/intell_voice_service.json`

---

## 6. utils/ 目录

**职责**: 公共工具库

```
utils/
├── BUILD.gn                         # 构建配置
├── array_buffer_util.cpp           # 数组缓冲区工具
├── base_thread.cpp                 # 基础线程
├── history_info_mgr.cpp            # 历史信息管理
├── id_allocator.cpp                # ID 分配器
├── intell_voice_util.cpp           # 通用工具
├── memory_guard.cpp                # 内存保护
├── message_queue.cpp               # 消息队列
├── msg_handle_thread.cpp           # 消息处理线程
├── service_db_helper.cpp           # 数据库助手
├── state_manager.cpp               # 状态管理
├── string_util.cpp                 # 字符串工具
├── task_executor.cpp               # 任务执行器
├── thread_wrapper.cpp              # 线程包装
├── time_util.cpp                  # 时间工具
└── timer_mgr.cpp                  # 定时器管理
```

**对外导出**: `intell_voice_utils` (.so)

**证据来源**: `utils/BUILD.gn`

---

## 7. 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层                                    │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     frameworks/js (N-API)                        │
│                              │                                   │
│                              ▼                                   │
│                   frameworks/native (Native)                     │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     services/intell_voice_service               │
│           │                    │                    │            │
│           ▼                    ▼                    ▼            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌───────────────┐   │
│  │    utils/       │  │intell_voice_    │  │intell_voice_ │   │
│  │                 │  │   engine/       │  │  trigger/    │   │
│  └─────────────────┘  └─────────────────┘  └───────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**依赖方向**: `js` → `native` → `service` → `engine/trigger/utils`

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Overview.md) | 定位和核心能力 |
| [架构设计](./02_Architecture.md) | 组件和数据流 |
| [N-API 参考](./04_NAPI_Reference.md) | API 调用 |
| [构建系统](./06_Build_System.md) | 构建配置 |
