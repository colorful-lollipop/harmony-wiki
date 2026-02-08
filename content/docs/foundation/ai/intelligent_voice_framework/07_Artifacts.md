# 编译产物

> **目的**: 详细说明 Intelligent Voice Framework 的所有编译产物及其安装路径  
> **适用范围**: 系统集成、测试、发布  
> **最后更新**: 2026-02-06

---

## 1. 产物概览

### 1.1 主要产物

| 产物名 | 类型 | 描述 | 路径模式 |
|--------|------|------|---------|
| `libintell_voice_server.z.so` | .so | 主服务进程库 | system/lib64/ |
| `libintelligentvoice_engine.z.so` | .so | 语音引擎库 | system/lib64/ |
| `libintelligentvoice_trigger.z.so` | .so | 触发器库 | system/lib64/ |
| `libintelligentvoice.z.so` | .so | N-API 模块 | hmos/module/ai/ |
| `libintellvoice_native.z.so` | .so | Native API 库 | hmos/module/ndk/ |
| `libintell_voice_utils.z.so` | .so | 公共工具库 | system/lib64/ |

### 1.2 配置文件

| 产物名 | 类型 | 描述 | 路径 |
|--------|------|------|------|
| `intell_voice_service.cfg` | .cfg | 服务权限配置 | system/etc/ |
| `intell_voice_service.sa` | .sa | SA 配置文件 | sa_profile/ |

### 1.3 声明文件

| 产物名 | 类型 | 描述 | 路径 |
|--------|------|------|------|
| `@ohos.ai.intelligentVoice.d.ts` | .d.ts | TypeScript 声明 | hmos/module/ai/ |

---

## 2. 详细产物说明

### 2.1 主服务库 (intell_voice_server)

| 属性 | 值 |
|------|-----|
| **产物名** | `libintell_voice_server.z.so` |
| **构建 Target** | `services/intell_voice_service:intell_voice_server` |
| **类型** | shared_library |
| **SA ID** | 312 |
| **安装路径** | `/system/lib64/` |
| **加载方式** | dlopen (由 SA 框架加载) |

**包含模块**:
```
- IntellVoiceService (SA 主服务)
- ServiceManager (服务管理)
- EngineRegistrar (引擎注册)
- TriggerRegistrar (触发器注册)
- SystemEventObserver (系统事件监听)
```

**依赖**:
```
libintelligentvoice_engine.z.so
libintelligentvoice_trigger.z.so
libintell_voice_utils.z.so
libhilog.z.so
```

### 2.2 引擎库 (intelligentvoice_engine)

| 属性 | 值 |
|------|-----|
| **产物名** | `libintelligentvoice_engine.z.so` |
| **构建 Target** | `services/intell_voice_engine:intelligentvoice_engine` |
| **类型** | shared_library |
| **安装路径** | `/system/lib64/` |
| **加载方式** | dlopen (由服务加载) |

**条件产物名**:
| 配置 | 产物名 |
|------|-------|
| 完整引擎 | `libintelligentvoice_engine.z.so` |
| 仅第一阶段 | `libintelligentvoice_only_first_engine.z.so` |
| 虚拟引擎 | `libintelligentvoice_dummy_engine.z.so` |

**包含模块**:
```
- EnrollEngine (注册引擎)
- WakeupEngine (唤醒引擎)
- UpdateEngine (更新引擎)
- EngineManager (引擎管理)
- HDI Adapter (驱动适配)
```

**依赖**:
```
libintell_voice_utils.z.so
libintell_voice_engine_proxy.hdi (HDI)
libaudio_*.so (音频框架)
libhukssdk.so (密钥管理)
```

### 2.3 触发器库 (intelligentvoice_trigger)

| 属性 | 值 |
|------|-----|
| **产物名** | `libintelligentvoice_trigger.z.so` |
| **构建 Target** | `services/intell_voice_trigger:intelligentvoice_trigger` |
| **类型** | shared_library |
| **安装路径** | `/system/lib64/` |
| **加载方式** | dlopen (由服务加载) |

**条件产物名**:
| 配置 | 产物名 |
|------|-------|
| 触发启用 | `libintelligentvoice_trigger.z.so` |
| 触发禁用 | `libintelligentvoice_dummy_trigger.z.so` |

**包含模块**:
```
- TriggerService (触发服务)
- TriggerManager (触发管理)
- TriggerDetector (触发检测)
- DSP Adapter (DSP 适配)
```

### 2.4 N-API 模块 (intelligentvoice)

| 属性 | 值 |
|------|-----|
| **产物名** | `libintelligentvoice.z.so` |
| **构建 Target** | `frameworks/js:intelligentvoice` |
| **类型** | shared_library |
| **安装路径** | `/hmos/module/ai/` |
| **相对安装目录** | `module/ai` |
| **JS 模块名** | `@ohos.ai.intelligentVoice` |

**包含模块**:
```
- IntellVoiceManagerNapi (管理器)
- EnrollIntelligentVoiceEngineNapi (注册引擎)
- WakeupIntelligentVoiceEngineNapi (唤醒引擎)
- WakeupManagerNapi (唤醒管理器)
- 回调处理器
```

**依赖**:
```
libintellvoice_native.z.so
libintell_voice_proxy.z.so
libace_napi.so (N-API 框架)
libipc_core.so
```

### 2.5 Native API 库 (intellvoice_native)

| 属性 | 值 |
|------|-----|
| **产物名** | `libintellvoice_native.z.so` |
| **构建 Target** | `frameworks/native:intellvoice_native` |
| **类型** | shared_library |
| **安装路径** | `/hmos/module/ndk/` |
| **NDK 标签** | `["ndk"]` |
| **头文件基础路径** | `interfaces/inner_api/native` |

**导出头文件**:
```
intell_voice_manager.h
i_headset_wakeup.h
wakeup_intell_voice_engine.h
enroll_intell_voice_engine.h
```

**依赖**:
```
libintell_voice_proxy.z.so
libintell_voice_engine_proxy_*.so (HDI)
```

### 2.6 工具库 (intell_voice_utils)

| 属性 | 值 |
|------|-----|
| **产物名** | `libintell_voice_utils.z.so` |
| **构建 Target** | `utils:intell_voice_utils` |
| **类型** | shared_library |
| **安装路径** | `/system/lib64/` |

**包含模块**:
```
- ArrayBufferUtil (缓冲区工具)
- MessageQueue (消息队列)
- TaskExecutor (任务执行)
- ServiceDbHelper (数据库助手)
- TimeUtil (时间工具)
- HuksAesAdapter (密钥适配，条件编译)
```

---

## 3. 运行时加载关系

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用进程                                 │
│  @ohos.ai.intelligentVoice (JS 模块)                            │
│              │                                                   │
│              ▼                                                   │
│  libintelligentvoice.z.so (N-API)                               │
│              │                                                   │
│              ▼                                                   │
│  libintellvoice_native.z.so (Native API)                         │
└────────────────────────────┬────────────────────────────────────┘
                           │ IPC (Binder)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      服务进程 (intell_voice_service)              │
│  libintell_voice_server.z.so (SA 312)                            │
│         │                    │                    │            │
│         ▼                    ▼                    ▼            │
│  ┌─────────────┐    ┌─────────────────┐    ┌───────────────┐   │
│  │libint_voice_│    │libintelligent  │    │libintelligent │   │
│  │utils.z.so   │    │voice_engine    │    │voice_trigger │   │
│  └─────────────┘    │.z.so           │    │.z.so          │   │
│                     └─────────────────┘    └───────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                       驱动进程 (HDI)                             │
│  libintell_voice_engine_proxy_*.so (HDI 代理)                    │
│              │                                                   │
│              ▼                                                   │
│                    DSP 驱动 / 硬件                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. 安装路径汇总

| 产物 | 目标路径 | 备注 |
|------|---------|------|
| `libintell_voice_server.z.so` | `/system/lib64/` | 系统库 |
| `libintelligentvoice_engine.z.so` | `/system/lib64/` | 系统库 |
| `libintelligentvoice_trigger.z.so` | `/system/lib64/` | 系统库 |
| `libintell_voice_utils.z.so` | `/system/lib64/` | 系统库 |
| `libintelligentvoice.z.so` | `/hmos/module/ai/` | 应用模块 |
| `libintellvoice_native.z.so` | `/hmos/module/ndk/` | NDK 模块 |
| `@ohos.ai.intelligentVoice.d.ts` | `/hmos/module/ai/` | 声明文件 |
| `intell_voice_service.cfg` | `/system/etc/` | 配置文件 |

---

## 5. 相关文档

| 文档 | 说明 |
|------|------|
| [构建系统](./06_Build_System.md) | 构建配置 |
| [配置开关](./appendix/Config_Flags.md) | Feature Flags |
| [架构设计](./02_Architecture.md) | 组件关系 |
