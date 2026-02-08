# 编译产物说明

> **目的**: 详细说明 battery_manager 项目的所有编译产物、安装路径和运行时加载关系

**适用范围**: 共享库、可执行文件、配置文件

---

## 编译产物清单

### 共享库 (Shared Libraries)

| 库名 | Target | 产物名 | 安装路径 | 说明 |
|------|--------|----------|---------|------|
| libbatteryinfo.z.so | batteryinfo | libbatteryinfo.z.so | /system/lib/module/ | N-API batteryInfo 模块（13 个属性 + 3 个配置函数） |
| libbattery.z.so | battery | libbattery.z.so | /system/lib/module/ | N-API battery 模块（异步 getStatus 函数） |
| libcharger.z.so | charger | libcharger.z.so | /system/lib/module/ | N-API charger 模块（ChargeType 枚举） |
| libbatteryservice.z.so | batteryservice | libbatteryservice.z.so | /system/lib/ | 电池服务 SA 3302 主库 |
| libbatterysrv_client.z.so | batterysrv_client | libbatterysrv_client.z.so | /system/platformsdk/lib/ | 电池服务 IPC 客户端库 |
| libbattery_notification.z.so | battery_notification | libbattery_notification.z.so | /system/lib/ | 电池事件通知库 |
| libcj_battery_info_ffi.z.so | cj_battery_info_ffi | libcj_battery_info_ffi.z.so | /system/lib/ | CJ FFI 绑定库 |
| libohbattery_info.z.so | ohbattery_info | libohbattery_info.z.so | /system/lib/ndk/ | C API 库 |
| libbattery_hookmgr.z.so | battery_hookmgr | libbattery_hookmgr.z.so | /system/lib/ | Hook 管理器库 |

**证据**: 各 target 的 `relative_install_dir` 配置

### 可执行文件

| 可执行文件 | Target | 安装路径 | 说明 |
|----------|--------|---------|------|
| charger | charger | /system/bin/charger | 关机充电程序（可选功能） |

**证据**: `charger/BUILD.gn:install_enable=true`

---

## 安装路径映射

### 系统库路径

| 路径 | 类型 | 产物 |
|------|------|------|
| /system/lib/module/ | N-API 模块库 | libbatteryinfo.z.so, libbattery.z.so, libcharger.z.so |
| /system/lib/ | SA 和系统库 | libbatteryservice.z.so, libbattery_notification.z.so, libbattery_hookmgr.z.so |
| /system/platformsdk/lib/ | 平台 SDK 库 | libbatterysrv_client.z.so |
| /system/lib/ndk/ | NDK 库 | libohbattery_info.z.so |

**证据**: 各 target 的 `relative_install_dir` 和 `install_images` 配置

### 可执行文件路径

| 路径 | 产物 |
|------|------|
| /system/bin/ | charger 可执行文件 |

**证据**: `charger/BUILD.gn:install_enable=true`

---

## 运行时加载关系

### N-API 模块加载

```
JS 应用进程 (ARK TS Engine / N-API Runtime)
    ↓ dlopen()
加载 libbatteryinfo.z.so (同步）
    ↓
调用 napi_module_register()
    ↓
N-API 模块初始化 (BatteryInit)
    ↓
导出 JS 属性/方法
    ↓
JS 应用可调用 @ohos.batteryInfo

加载 libbattery.z.so (同步）
    ↓
调用 napi_module_register()
    ↓
N-API 模块初始化 (SystemBatteryInit)
    ↓
导出 JS 方法
    ↓
JS 应用可调用 @ohos.battery.getStatus

加载 libcharger.z.so (同步）
    ↓
调用 napi_module_register()
    ↓
N-API 模块初始化 (ChargeTypeInit)
    ↓
导出 JS 枚举类
    ↓
JS 应用可使用 @ohos.charger.ChargeType
```

**证据**: `frameworks/napi/src/battery_info.cpp:619-622`, `system_battery.cpp:289-292`, `charger.cpp:108-111`

### SA 启动

```
系统启动 (init 进程)
    ↓
SAMGR (System Ability Manager) 扫描 SA 配置文件
    ↓
读取 /system/profile/3302.json
    ↓
发现 BatteryService (SA 3302)
    ↓
根据 "run-on-create: true" 自动启动
    ↓
powermgr 进程加载 libbatteryservice.z.so
    ↓
调用 napi_module_register() (服务端）
    ↓
BatteryService OnStart() 执行
    ↓
注册 SA 3302
    ↓
服务就绪
```

**证据**: `sa_profile/3302.json:1-13`, `services/native/src/battery_service.cpp:78-84`

### IPC 连接

```
JS 应用进程
    ↓
N-API 模块调用
    ↓
BatterySrvClient::GetInstance()
    ↓
延迟初始化单例
    ↓
Connect() 方法执行
    ↓
通过 SAMGR 获取 SA 3302 远程对象
    ↓
iface_cast<IBatterySrv>(remoteObject)
    ↓
存储 proxy_ 供后续调用
    ↓
设置 DeathRecipient 监听服务死亡
```

**证据**: `frameworks/native/src/battery_srv_client.cpp:35-65`, `interfaces/inner_api/native/include/battery_srv_client.h:29`

### HDI 连接

```
BatteryService 进程 (SA 3302)
    ↓
OnStart() 执行
    ↓
RegisterBatteryHdiCallback()
    ↓
通过 HDI ServiceManager 获取电池驱动服务
    ↓
调用 IBatteryInterface::RegisterCallback()
    ↓
传递 BatteryCallback 对象
    ↓
注册 HDI 回调
    ↓
底层电池驱动事件通过回调上报到 BatteryService
```

**证据**: `services/native/include/battery_service.h:113-114`

---

## 依赖库加载顺序

### 初始化顺序

```
1. 系统基础库 (libc, libz, etc.)
2. SA 框架库 (libhilog, libsamgr_proxy, libipc_core)
3. HDI 框架库 (libhdi, libhdf_core)
4. 电池服务库 (libbatteryservice, libbattery_notification, libbattery_hookmgr)
5. 电池驱动库 (libbattery_proxy_2.0)
```

### 运行时符号解析

| 符号类型 | 来源 | 说明 |
|----------|------|------|
| N-API 符号 | libbatteryinfo.z.so, libbattery.z.so, libcharger.z.so | 供 JS 应用调用的符号 |
| IPC 符号 | libbatteryservice.z.so, libbatterysrv_client.z.so | ZIDL 生成的 Binder IPC 符号 |
| SA 符号 | libbatteryservice.z.so | SystemAbility 框架符号 |
| HDI 符号 | libbatteryservice.z.so | HDI 接口符号 |

**证据**: 各 target 的 `external_deps` 依赖配置

---

## 产物大小统计

### 预估大小（不含调试符号）

| Target | 预估大小 | 说明 |
|--------|---------|------|
| libbatteryinfo.z.so | ~200KB | 13 个属性 + 3 个配置函数 + 错误处理 |
| libbattery.z.so | ~50KB | 1 个异步函数 |
| libcharger.z.so | ~20KB | 1 个枚举类 |
| libbatteryservice.z.so | ~300KB | SA 主类 + 通知 + HDI 适配 |
| libbatterysrv_client.z.so | ~50KB | IPC 客户端代理 |
| libbattery_notification.z.so | ~150KB | 通知管理 |
| libcj_battery_info_ffi.z.so | ~50KB | 10 个 FFI 导出函数 |
| libohbattery_info.z.so | ~30KB | C API 客户端 |
| libbattery_hookmgr.z.so | ~30KB | Hook 管理器 |

**注意**: 实际大小以编译后实际输出为准

---

## 条件编译对产物的影响

### 特性开关影响

| 特性 | 关闭时产物 | 开启时新增产物 |
|------|-----------|---------|------|
| battery_manager_feature_enable_charger | 无 charger 模块产物 | charger 可执行文件和依赖库 |
| battery_manager_feature_enable_charging_sound | 无 charging_sound 库 | libcharging_sound.z.so |
| battery_manager_feature_support_notification | 无 notification 库和资源 | libbattery_notification.z.so 和多语言资源 |

**证据**: `bundle.json:21-27`, `services/BUILD.gn:195-221`

### 条件宏影响

| 宏 | 定义时影响 | 证据 |
|------|-----------|------|
| HAS_BATTERY_CONFIG_POLICY_PART | 添加 configpolicy_util 依赖 | `services/BUILD.gn:138-141` |
| HAS_SENSORS_MISCDEVICE_PART | 添加 light_interface_native 依赖 | `services/BUILD.gn:130-132` |

---

## 动态库与静态库

### 动态库 (Runtime Load)

所有 N-API 模块和 SA 库都是动态库，运行时通过 dlopen 加载。

### 静态库 (Compile-time Link)

- 无静态库依赖

**证据**: 所有 target 都是 `ohos_shared_library` 或 `ohos_executable` 类型

---

## 版本信息

### 库版本

| 库名 | 版本来源 | 说明 |
|------|---------|------|
| 所有 .z.so | 项目版本 | bundle.json 中定义的 version: "3.1" |

**证据**: `bundle.json:2-4`

### ABI 兼容性

- **目标架构**: OpenHarmony 标准系统（arm64-v8a 或 riscv64）
- **N-API 版本**: N-API 9.0+ (ace_napi)
- **HDI 版本**: 2.0 (libbattery_proxy_2.0)

---

## 相关跳转

- [GN Targets](06_GN_Targets.md)
- [系统架构](03_Architecture.md)

---

**返回**: [导航](SUMMARY.md)
