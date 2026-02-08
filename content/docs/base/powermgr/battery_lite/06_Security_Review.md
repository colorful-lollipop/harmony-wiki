# 安全风险评审

> **目的**: 分析 battery_lite 项目的安全风险，识别攻击面和潜在漏洞。  
> **适用范围**: 安全审计、代码审查、开发安全实践。  
> **关键结论**: 项目整体风险等级为 **低**，主要风险来自输入验证不足和接口访问控制缺失，建议增强参数校验和权限控制。

---

## 1. 威胁模型

### 信任边界

```
┌──────────────────────────────────────────────────────────────┐
│                      可信区域                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              battery_lite 服务进程                       │  │
│  │  - BatteryService (服务实现)                            │  │
│  │  - BatteryFeature (特征实现)                            │  │
│  │  - 模拟电池数据 (battInfo)                              │  │
│  └────────────────────────────────────────────────────────┘  │
│                            │                                 │
│                    SAMgr IPC 边界                            │
│                            │                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              应用进程区域                                │  │
│  │  - Native 应用 (调用框架 API)                           │  │
│  │  - JS 应用 (调用 JS API)                                │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
         │                           │                           │
         ▼                           ▼                           ▼
   ┌──────────┐              ┌──────────┐              ┌──────────┐
   │  JS API  │              │ Native   │              │  硬件    │
   │  接口    │              │  API     │              │  HAL     │
   └──────────┘              └──────────┘              └──────────┘
```

### 数据流

| 数据流 | 方向 | 信任级别 | 说明 |
|--------|------|----------|------|
| JS API 调用 | 应用 → 服务 | 中 | 回调模式，异步返回 |
| Native API 调用 | 应用 → 服务 | 中 | 同步调用，返回枚举/整数 |
| 电池数据更新 | HAL → 服务 | 高 | 模拟数据，实际应为硬件读取 |
| LED 控制 | 服务 → HAL | 高 | 需验证调用来源 |

---

## 2. 攻击面清单

### 2.1 N-API / JS 接口

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| 参数校验缺失 | `battery_module.cpp:48-50` | **中** | 未校验回调对象是否为有效 JS 对象 |
| 字符串处理 | `battery_module.cpp:135` | **低** | `GetBatTechnology()` 返回指针未释放 |
| 回调执行 | `battery_module.cpp:23-41` | **低** | SuccessCallBack 未处理异常 |

**证据**:
- `frameworks/js/builtin/src/battery_module.cpp:48-50` 参数检查仅验证 `args[0]` 是否为 undefined
- `frameworks/js/builtin/src/battery_module.cpp:135` 设置字符串属性后未释放 `technology` 内存

### 2.2 Native 接口

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| 空指针解引用 | `battery_framework.c:51-52` | **中** | 虽有检查但 `intf->GetBatSocFunc` 可能为 NULL |
| 字符串缓冲区 | `services/src/battery_device.c:93` | **低** | `GetTechnologyImpl()` 返回内部静态缓冲区 |
| 全局状态竞争 | `battery_framework.c:22-23` | **低** | 互斥锁保护，但 `g_intf` 非原子操作 |

**证据**:
- `frameworks/native/src/mini/battery_framework.c:51-52` 未检查 `intf->GetBatSocFunc` 是否为 NULL
- `services/src/battery_device.c:93` 直接返回 `battInfo.BatTechnology` 静态数组

### 2.3 IPC / SAMgr

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| 服务注册 | `battery_device.c:184-191` | **低** | 未验证注册结果可能导致服务失效 |
| 接口获取 | `battery_manage_feature.c:50-55` | **中** | 每次调用都创建新实例，存在资源泄漏风险 |

**证据**:
- `services/src/battery_device.c:184-191` 未检查 `RegisterService` 和 `RegisterDefaultFeatureApi` 返回值

### 2.4 数据结构

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| BatInfo 复制 | `battery_device.c:127-142` | **中** | `strcpy_s` 成功但未检查源数据有效性 |
| 枚举越界 | 多个实现文件 | **低** | 枚举检查依赖调用方正确性 |

**证据**:
- `services/src/battery_device.c:132-134` `strcpy_s` 返回值被忽略

---

## 3. 可利用点详细分析

### 风险 1: 参数校验不足 - 中风险

**证据位置**: `frameworks/js/builtin/src/battery_module.cpp:44-50`

```cpp
JSIValue BatteryModule::GetBatterySOC(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum)
{
    JSIValue undefValue = JSI::CreateUndefined();
    if ((args == nullptr) || (argsNum == 0) || JSI::ValueIsUndefined(args[0])) {
        return undefValue;  // 仅检查是否为 undefined
    }
    // ... 继续处理
}
```

**问题**: 仅检查 `args[0]` 是否为 `undefined`，未验证：
- 是否为有效的对象类型
- 是否包含 `success` 回调
- 回调是否为函数类型

**触发方式**: 传入非对象参数或缺少必要属性的对象

**影响**:
- JS 运行时可能抛出类型错误
- 应用无法收到回调通知

**修复建议**:
```cpp
if ((args == nullptr) || (argsNum == 0) || 
    JSI::ValueIsUndefined(args[0]) || 
    !JSI::ValueIsObject(args[0])) {
    return undefValue;
}
```

---

### 风险 2: 字符串缓冲区暴露 - 低风险

**证据位置**: `services/src/battery_device.c:91-94`

```c
char *GetTechnologyImpl(void)
{
    return battInfo.BatTechnology;  // 返回内部静态缓冲区指针
}
```

**问题**: 返回指向内部静态数组的指针，调用者可以直接修改内容

**触发方式**:
```c
char* tech = GetTechnologyImpl();
strcpy(tech, "HACKED");  // 修改内部状态
```

**影响**:
- 意外修改电池技术信息
- 可能导致后续读取不一致

**修复建议**: 返回 const 指针或复制字符串
```c
const char *GetTechnologyImpl(void)
{
    return battInfo.BatTechnology;
}
```

---

### 风险 3: 函数指针空解引用 - 中风险

**证据位置**: `frameworks/native/src/mini/battery_framework.c:47-55`

```c
int32_t GetBatSoc(void)
{
    int32_t ret = EC_FAILURE;
    BatteryInterface *intf = GetBatteryInterface();
    if ((intf != NULL) && (intf->GetBatSocFunc != NULL)) {  // 检查了
        ret = intf->GetBatSocFunc((IUnknown *)intf);
    }
    return ret;
}
```

**问题**: 虽然有检查，但 `GetBatSocFunc` 是必需接口，任何 NULL 检查失败都表明系统状态异常

**触发方式**: SAMgr 返回损坏的接口对象

**影响**: 
- API 返回错误码
- 应用无法获取电池信息

**修复建议**: 添加详细日志记录异常情况

---

### 风险 4: 资源泄漏 - 低风险

**证据位置**: `services/src/battery_manage_feature.c:47-56`

```c
int32_t BatterySocImpl(IUnknown *iUnknown)
{
    int32_t soc = BATT_INT_VALUE;
    g_batteryDevice = NewBatterInterfaceInstance();  // 每次调用都获取
    if (g_batteryDevice == NULL) {
        return soc;
    }
    soc = g_batteryDevice->GetSoc();
    return soc;  // 未释放接口
}
```

**问题**: 每次调用 `BatterySocImpl` 都通过 `NewBatterInterfaceInstance()` 获取新实例，但从未释放

**触发方式**: 高频调用电池查询 API

**影响**: 
- 内存资源泄漏（如果接口实例分配内存）
- 文件描述符泄漏（如果 IPC 连接分配描述符）

**修复建议**: 实现缓存机制或使用单例模式
```c
static IBattery *g_cachedDevice = NULL;

IBattery *GetCachedBatteryDevice(void)
{
    if (g_cachedDevice == NULL) {
        g_cachedDevice = NewBatterInterfaceInstance();
    }
    return g_cachedDevice;
}
```

---

### 风险 5: LED 控制无访问控制 - 低风险

**证据位置**: `services/include/battery_device.h:55-58`

```c
int (*TurnOnLed)(int red, int green, int blue);
int (*TurnOffLed)(void);
int (*SetLedColor)(int red, int green, int blue);
int (*GetLedColor)(int *red, int *green, int *blue);
```

**问题**: LED 控制接口未设置任何访问权限，任何应用都可以调用

**触发方式**: 任意应用调用 LED 控制 API

**影响**:
- 恶意应用可能通过 LED 闪烁进行拒绝服务攻击
- 可能干扰用户对设备状态的判断

**修复建议**:
```c
// 建议添加权限检查机制（需要系统支持）
if (!CheckCallingPermission("ohos.permission.CONTROL_LED")) {
    return BATTERY_ERROR_PERMISSION_DENIED;
}
```

---

## 4. 内存安全分析

### 静态分配

| 数据结构 | 大小 | 说明 |
|----------|------|------|
| `BatInfo` | ~96 字节 | 包含整数和 64 字节字符串 |
| `g_intf` | 指针大小 | 接口指针缓存 |
| `g_mutex` | 互斥锁结构 | 线程同步 |

### 动态分配

| 位置 | 分配类型 | 释放位置 |
|------|----------|----------|
| `NewBatterInterfaceInstance()` | 可能动态分配 | `FreeBatterInterfaceInstance()` (未调用) |
| JSIValue 对象 | JSI 运行时分配 | `JSI::ReleaseValue()` |

**发现**: `FreeBatterInterfaceInstance()` 已定义但从未被调用，存在资源泄漏风险。

**证据**: `services/include/ibattery.h:62` 声明了释放函数，`services/src/battery_device.c:230-233` 提供了空实现。

---

## 5. 竞态条件分析

### 已发现的竞态

| 位置 | 描述 | 当前保护 |
|------|------|----------|
| `GetBatteryInterface()` | `g_intf` 读取/写入 | 互斥锁 `g_mutex` |
| SAMgr 注册 | 多线程注册竞争 | 系统保证单线程初始化 |

### 潜在竞态

| 场景 | 风险 | 建议 |
|------|------|------|
| `battInfo` 更新与读取 | 中 | 添加读写锁保护 |

---

## 6. 信息泄露风险

### 已识别风险

| 风险 | 等级 | 说明 |
|------|------|------|
| 返回内部缓冲区指针 | 低 | `GetTechnologyImpl()` 返回静态数组 |
| 错误信息泄露 | 低 | 暂无详细错误信息返回 |

### 建议措施

- 将敏感信息（如电池技术）通过只读接口暴露
- 添加错误码定义，避免返回原始错误字符串

---

## 7. 安全建议汇总

### 高优先级

| 建议 | 影响 | 难度 |
|------|------|------|
| 增强 JS API 参数校验 | 防止异常参数导致的运行时错误 | 低 |
| 实现接口实例缓存 | 防止资源泄漏 | 中 |

### 中优先级

| 建议 | 影响 | 难度 |
|------|------|------|
| 添加详细日志记录 | 便于安全审计和故障排查 | 低 |
| LED 控制添加权限检查 | 防止未授权访问 | 高 (需系统支持) |

### 低优先级

| 建议 | 影响 | 难度 |
|------|------|------|
| 返回 const char* | 防止意外修改内部状态 | 低 |
| 统一错误码处理 | 改善错误反馈 | 中 |

---

## 8. 检查范围局限性

### 已检查范围

- ✅ N-API 绑定层 (`battery_module.cpp`)
- ✅ Native 框架层 (`battery_framework.c`)
- ✅ 服务层实现 (`battery_device.c`, `battery_manage_feature.c`)
- ✅ 头文件接口定义
- ✅ BUILD.gn 构建配置

### 未检查范围

- ❌ HAL 层实现（当前为模拟数据）
- ❌ 系统 IPC 基础设施（SAMgr, IPC）
- ❌ JS 运行时安全性（JSI 内部）
- ❌ 硬件相关安全问题（需硬件文档）

### 结论

基于已检查的代码范围，battery_lite 项目整体安全状况良好。主要风险来自：
1. 输入验证不够严格
2. 缺少资源释放机制
3. LED 控制无权限检查

建议在后续迭代中逐步解决高优先级安全问题。

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [N-API 接口](03_N_API.md) | API 安全考量 |
| [内部 API](04_Inner_API.md) | 接口安全说明 |
| [架构说明](02_Architecture.md) | 信任边界和数据流 |
