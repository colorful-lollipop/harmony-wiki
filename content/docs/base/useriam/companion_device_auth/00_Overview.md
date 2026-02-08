# 00_Overview - 伴随设备认证项目概览

> 理解项目定位、边界与核心能力

---

## 1. 项目定位

**companion_device_auth** (伴随设备认证) 是OpenHarmony `useriam` 子系统的核心组件，SA ID为 **945**。

### 1.1 在OpenHarmony中的位置

```
OpenHarmony用户认证体系
├── useriam_user_auth_framework  (统一用户认证框架)
├── useriam_pin_auth             (PIN认证执行器)
├── useriam_face_auth            (人脸认证执行器)
├── useriam_fingerprint_auth     (指纹认证执行器)
└── companion_device_auth        (伴随设备认证执行器) ← 本项目
```

### 1.2 核心职责

根据 `README_ZH.md`，伴随设备认证提供：

1. **设备间可信关系建立**: 伴随设备与主设备通过用户显式操作建立可信绑定
2. **用户身份确认**: 伴随设备确认当前持有者身份（如手表佩戴检测）
3. **操作意图确认**: 证明主设备上的操作来自机主本人

---

## 2. 应用场景

### 场景1: 无感解锁
**手表作为伴随设备支持手机/PC无感认证用户身份后解锁**

- 用户佩戴已绑定的手表
- 手表持续认证用户身份（佩戴检测）
- 手机通过手表认证状态实现无感解锁

### 场景2: 分布式认证
**PC作为手机的伴随设备，通过PC本地的人脸/指纹认证用户身份后支持投屏手机解锁**

- 避免分布式场景下用户在设备间来回切换
- PC本地完成认证，远程解锁手机

### 场景3: 双因子融合
**手表与手机人脸进行双因子融合认证，提升安全性**

- 结合 possession factor (手表) + inherence factor (人脸)
- 提升单一认证方式的安全性

---

## 3. 目录结构

```
/base/useriam/companion_device_auth/
├── common/                              # 公共定义
│   ├── inc/
│   │   ├── common_defines.h            # 错误码、枚举定义
│   │   ├── iam_logger.h                # 日志宏
│   │   └── scope_guard.h               # RAII工具
│   └── src/
│
├── frameworks/                          # 接口层
│   ├── js/napi/                         # JS/N-API接口
│   │   ├── inc/                        # 头文件
│   │   └── src/
│   │       ├── companion_device_auth_entry.cpp      # N-API注册入口
│   │       ├── companion_device_auth_napi_impl.cpp  # 实现
│   │       ├── companion_device_auth_napi_helper.cpp # 工具函数
│   │       ├── status_monitor.cpp                   # StatusMonitor类
│   │       └── *_callback.cpp                       # 回调处理
│   │
│   ├── ets/ani/                         # ETS/ArkUI Native接口
│   │   └── src/
│   │
│   └── native/                          # Native C++接口
│       ├── client/                      # 客户端库
│       │   ├── inc/
│       │   └── src/
│       │       ├── companion_device_auth_client_impl.cpp
│       │       └── ipc_*_callback_service.cpp
│       │
│       └── ipc/                         # IPC通信
│           ├── idl/                     # IDL接口定义
│           │   ├── ICompanionDeviceAuth.idl
│           │   ├── CompanionDeviceAuthTypes.idl
│           │   └── IIpc*.idl
│           ├── inc/
│           └── stub/
│
├── services/                            # 服务实现
│   ├── service_entry/                   # 服务入口
│   │   ├── inc/
│   │   │   └── companion_device_auth_service.h
│   │   └── src/
│   │       └── companion_device_auth_service.cpp     # SA实现
│   │
│   ├── singleton/                       # 单例管理器
│   │   └── inc/
│   │       ├── companion/
│   │       ├── cross_device_comm/
│   │       └── security_agent/
│   │
│   ├── companion/                       # 伴随设备管理
│   │   └── src/companion_manager_impl.cpp
│   │
│   ├── host_binding/                    # 主设备绑定管理
│   │   └── src/host_binding_manager_impl.cpp
│   │
│   ├── cross_device_comm/               # 跨设备通信基础设施
│   │   └── src/
│   │       ├── channel_manager.cpp
│   │       ├── connection_manager.cpp
│   │       └── message_router.cpp
│   │
│   ├── cross_device_channels/           # 跨设备通道实现
│   │   └── soft_bus/                    # SoftBus通道
│   │       └── src/
│   │           ├── soft_bus_channel.cpp
│   │           └── soft_bus_connection.cpp
│   │
│   ├── cross_device_interaction/        # 跨设备业务处理
│   │   ├── add_companion/               # 添加伴随设备
│   │   ├── delegate_auth/               # 委托认证
│   │   ├── issue_token/                 # 签发token
│   │   ├── obtain_token/                # 获取token
│   │   ├── token_auth/                  # Token认证
│   │   ├── revoke_token/                # 吊销token
│   │   ├── mix_auth/                    # 混合认证
│   │   ├── sync_device_status/          # 同步设备状态
│   │   └── keep_alive/                  # 保活机制
│   │
│   ├── security_agent/                  # 安全代理层
│   │   ├── inc/security_agent_imp.h
│   │   └── src/security_agent_impl.cpp  # C++接口
│   │
│   ├── external_adapters/               # 外部服务适配器
│   │   ├── access_token/                # 权限检查适配
│   │   ├── security_command_adapter/    # 安全命令适配(Rust)
│   │   ├── user_auth/                   # UserAuth框架适配
│   │   ├── event_manager/               # 事件管理适配
│   │   └── samgr/                       # SA管理器适配
│   │
│   ├── request/                         # 请求生命周期
│   ├── fwk_comm/                        # UserIAM框架集成
│   ├── misc/                            # 杂项工具
│   └── utils/                           # 工具类
│
├── sa_profile/                          # 系统服务配置
│   ├── 945.json                         # SA 945配置
│   └── companiondeviceauth.cfg          # 服务启动配置
│
├── param/                               # 系统参数
│   ├── companion_device_auth.para
│   └── companion_device_auth.para.dac
│
└── test/                                # 测试代码 (本文档不覆盖)
```

---

## 4. 运行环境

### 4.1 系统类型

从 `bundle.json`:
```json
"adapted_system_type": ["standard"]
```

- 仅支持 **Standard** 系统类型
- 不适用于轻量/小型系统

### 4.2 依赖组件

从 `bundle.json`，共依赖 **33个** 系统组件：

**核心依赖:**
- `ability_runtime` - Ability生命周期管理
- `access_token` - 权限与访问令牌
- `ipc` - 进程间通信
- `samgr` - SystemAbility管理器
- `safwk` - SystemAbility框架

**分布式能力:**
- `device_manager` - 设备管理
- `dsoftbus` - 分布式软总线
- `bluetooth` - 蓝牙通信

**用户认证:**
- `user_auth_framework` - 统一用户认证框架
- `os_account` - 系统账号管理

**运行时支持:**
- `napi` - N-API运行时
- `ace_engine` - ArkUI引擎
- `ets_frontend` - ETS前端

**安全与存储:**
- `rust_cxx`, `rust_libc`, `rust_rust-openssl` - Rust安全核心

### 4.3 SystemAbility配置

从 `sa_profile/945.json`:
```json
{
    "process": "useriam",
    "systemability": [{
        "name": 945,
        "libpath": "libcompaniondeviceauthservice.z.so",
        "run-on-create": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

- **SA ID**: 945
- **进程**: useriam
- **库文件**: libcompaniondeviceauthservice.z.so
- **启动方式**: 开机自启动 (`run-on-create: true`)
- **分布式**: 不支持 (`distributed: false`)

---

## 5. 关键概念

### 5.1 设备角色

| 角色 | 说明 | 对应模块 |
|------|------|----------|
| **Host (主设备)** | 被认证的设备，如手机、PC | `host_binding/` |
| **Companion (伴随设备)** | 辅助认证的设备，如手表 | `companion/` |

### 5.2 核心数据结构

从 `frameworks/native/client/inc/companion_device_auth_common_defines.h`:

**DeviceKey** - 设备唯一标识
```cpp
struct ClientDeviceKey {
    int32_t deviceIdType;      // 设备ID类型
    std::string deviceId;      // 设备标识符
    int32_t deviceUserId;      // 设备上的用户ID
};
```

**TemplateStatus** - 绑定凭证状态
```cpp
struct ClientTemplateStatus {
    uint64_t templateId;       // 模板ID
    bool isConfirmed;          // 是否已确认
    bool isValid;              // 是否有效
    int32_t localUserId;       // 本地用户ID
    int64_t addedTime;         // 添加时间
    std::vector<int32_t> enabledBusinessIds; // 启用的业务ID
    ClientDeviceStatus deviceStatus; // 关联设备状态
};
```

### 5.3 枚举定义

从 `common/inc/common_defines.h`:

**BusinessId** - 业务标识
```cpp
enum class BusinessId : int32_t {
    DEFAULT = 0,
    VENDOR_BEGIN = 10000,
};
```

**DeviceIdType** - 设备ID类型
```cpp
enum class DeviceIdType : int32_t {
    UNKNOWN = 0,
    UNIFIED_DEVICE_ID = 1,
    VENDOR_BEGIN = 10000,
};
```

**SelectPurpose** - 设备选择目的
```cpp
enum class SelectPurpose : int32_t {
    SELECT_ADD_DEVICE = 1,      // 添加设备
    SELECT_AUTH_DEVICE = 2,     // 认证设备
    CHECK_OPERATION_INTENT = 3, // 确认操作意图
    VENDOR_BEGIN = 10000,
};
```

---

## 6. 对外接口概览

### 6.1 JS/ETS API

从 `README_ZH.md` 和代码分析:

| 接口 | 类型 | 说明 |
|------|------|------|
| `getStatusMonitor(localUserId)` | 同步 | 获取状态监视器实例 |
| `registerDeviceSelectCallback(callback)` | 同步 | 注册设备选择回调 |
| `unregisterDeviceSelectCallback()` | 同步 | 注销设备选择回调 |
| `updateEnabledBusinessIds(templateId, businessIds)` | Promise | 更新业务ID列表 |
| `StatusMonitor.getTemplateStatus()` | Promise | 获取模板状态 |
| `StatusMonitor.onTemplateChange(callback)` | 同步 | 监听模板变化 |
| `StatusMonitor.offTemplateChange(callback?)` | 同步 | 取消监听模板变化 |
| `StatusMonitor.onAvailableDeviceChange(callback)` | 同步 | 监听可用设备变化 |
| `StatusMonitor.offAvailableDeviceChange(callback?)` | 同步 | 取消监听可用设备 |
| `StatusMonitor.onContinuousAuthChange(param, callback)` | 同步 | 监听持续认证变化 |
| `StatusMonitor.offContinuousAuthChange(callback?)` | 同步 | 取消监听持续认证 |

### 6.2 权限要求

所有API都需要以下权限：

1. **ohos.permission.USE_USER_IDM** - 使用用户身份管理
   - 检查位置: `frameworks/js/napi/src/companion_device_auth_entry.cpp:40`
   
2. **系统应用权限** - 必须是系统应用
   - 检查位置: `frameworks/js/napi/src/companion_device_auth_entry.cpp:51`
   - 通过 `TokenIdKit::IsSystemAppByFullTokenID()` 验证

---

## 7. 相关仓库

| 仓库 | 说明 | 关系 |
|------|------|------|
| [useriam_user_auth_framework](https://gitcode.com/openharmony/useriam_user_auth_framework) | 统一用户认证框架 | 调用方/被调用方 |
| [useriam_pin_auth](https://gitcode.com/openharmony/useriam_pin_auth) | PIN认证执行器 | 同级执行器 |
| [useriam_face_auth](https://gitcode.com/openharmony/useriam_face_auth) | 人脸认证执行器 | 同级执行器 |
| [useriam_fingerprint_auth](https://gitcode.com/openharmony/useriam_fingerprint_auth) | 指纹认证执行器 | 同级执行器 |

---

## 8. 重要安全提示

### 8.1 纯软件实现限制

从 `README_ZH.md`:

> OpenHarmony开源架构内提供了伴随设备认证的纯软件实现，供开发者demo伴随设备认证功能，**纯软件实现部分并未包含伴随设备认证相关信息的安全存储能力**。

**含义**: 生产环境需要基于TEE/安全芯片实现安全存储。

### 8.2 安全代理层接口

从 `README_ZH.md`:

> 需在尽可能安全的环境中实现头文件 `services/singleton/inc/security_agent/security_agent.h` 中定义的接口，确保伴随设备认证结果的安全性。

**关键接口文件**: `services/security_agent/inc/security_agent_imp.h`

---

## 9. 参考资料

- [OpenHarmony用户认证架构设计](https://gitee.com/openharmony/docs)
- [N-API开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/napi)
- [SystemAbility开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/subsys-boot)

---

## 10. 项目评估摘要 (2026-02-07 更新)

### 10.1 项目类型判定

| 属性 | 值 |
|------|-----|
| **类型** | 系统服务 + 框架模块 + N-API 插件 |
| **运行域** | 系统服务(System) / 应用框架(Framework) |
| **SA ID** | 945 |
| **进程** | useriam |

### 10.2 对外暴露面

| 暴露类型 | 是否存在 | 位置 |
|----------|----------|------|
| N-API (JS/TS) | ✅ | `frameworks/js/napi/` |
| ANI (ArkTS) | ✅ | `frameworks/ets/ani/` |
| IPC 接口 | ✅ | `frameworks/native/ipc/idl/` |
| C++ Native API | ✅ | `frameworks/native/client/` |
| 系统服务 | ✅ | SA #945 |

### 10.3 安全相关性

**高安全相关性** 组件：
- 涉及用户身份认证（设备解锁）
- 跨设备通信（网络攻击面）
- 凭据管理（密钥安全存储）
- Rust + C++ 混合（FFI 边界风险）

### 10.4 文档覆盖范围

| 受众 | 重点文档 |
|------|----------|
| **新人学习者** | 00_Overview, 01_Architecture, 02_NAPI_Reference |
| **安全研究员** | 01_Architecture (信任边界), 05_Security |
| **系统开发者** | 03_Inner_API, 04_GN_Targets |

---

*文档生成时间: 2025-02-06*  
*更新时间: 2026-02-07 (补充项目评估信息)*
