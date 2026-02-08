# 附录

## 关键调用链

### 1. 用户认证调用链

```
用户认证测试用例
    │
    ▼
UserAuthInterfaceService (实现类)
    │
    ├── Init(deviceUdid) ──────────────► HDI 框架初始化
    │
    ├── AddExecutor(info, ...) ─────────► Executor 注册
    │       │
    │       └── 返回 publicKey, templateIds
    │
    ├── BeginEnrollment(authToken, ...) ─► 开始录入流程
    │       │
    │       └── 触发生物特征采集回调
    │
    ├── BeginAuthentication(...) ──────► 开始认证流程
    │       │
    │       └── 验证凭证，返回认证结果
    │
    └── DeleteUser(userId, authToken) ─► 删除用户数据
            │
            └── 清除所有生物特征模板
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/useriam/userauth/src/user_auth_hdi.cpp`

---

### 2. HDI 服务调用链

```
测试用例 (GoogleTest)
    │
    ▼
IXXXInterface::Get() ─────────────────► 获取接口单例
    │
    ▼
Register(callback) ───────────────────► 注册回调
    │
    ▼
Service Method (async) ───────────────► 发送异步请求
    │
    ├──► MessageParcel::Write... ───────► 序列化参数
    │
    ├──► SendRequest(code, data, reply) ► Binder IPC 发送
    │
    └──► Callback::OnEvent() ──────────► 接收异步回调
            │
            └── 验证回调数据
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/manager/managerServiceTest/service_manager_hdi_test.cpp:102-186`

---

### 3. 服务死亡监控调用链

```
DeathTest 测试用例
    │
    ▼
IDisplayBuffer::Get() ─────────────────► 获取 Buffer 服务
    │
    ▼
AddDeathRecipient(recipient) ─────────► 注册死亡回调
    │
    ▼
system("killall allocator_host") ─────► 模拟服务崩溃
    │
    ▼
BufferDiedRecipient::OnRemoteDied() ─► 触发死亡回调
    │
    └──► 验证死亡通知正确接收
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/display/buffer/death/death_test.cpp`

---

### 4. 系统调用测试调用链

```
Syscall 测试用例 (HWTEST)
    │
    ├──► syscall(params) ──────────────► 调用系统调用
    │       │
    │       └──► 内核处理
    │
    └──► EXPECT_EQ(...) ───────────────► 验证返回值/副作用
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/kernel/syscalls/user/UserApiTest.cpp`

---

## 配置参数说明

### 特性开关

| 参数 | 类型 | 默认值 | 位置 | 说明 |
|------|------|--------|------|------|
| `hats_rich` | bool | false | build.gni | 启用富功能测试集 |
| `hats_nnrt` | bool | false | build.gni | 启用神经网络运行时测试 |
| `hats_drivers_peripheral_power_wakeup_cause_path` | bool | false | build.gni | 电源唤醒路径测试 |
| `hats_drivers_peripheral_battery_pc_macro_isolation` | bool | false | build.gni | 电池宏隔离测试 |

### 环境变量

| 变量 | 用途 | 示例 |
|------|------|------|
| `XTS_SUITENAME` | 指定测试套件 | `export XTS_SUITENAME=hats` |
| `target_subsystem` | 指定子系统 | `export target_subsystem=hdf` |
| `product_name` | 指定产品 | `product_name=hispark_taurus_standard` |

### 构建参数

| 参数 | 用途 | 示例 |
|------|------|------|
| `suite` | 指定测试套件 | `suite=hats` |
| `system_size` | 系统大小 | `system_size=standard` |
| `hats_nnrt` | NNRT 测试 | `hats_nnrt=true` |

---

## Test.json 配置

### 配置文件结构

```json
{
  "description": "测试套件描述",
  "test_cases": [
    {
      "name": "测试用例名称",
      "level": 1,
      "type": "Function",
      "timeout": 30000
    }
  ],
  "dependencies": [
    "驱动接口",
    "系统服务"
  ]
}
```

### 示例：电池测试配置

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/powermgr/battery/hdi_battery_config/Test.json`

```json
{
  "description": "Battery HDI Test Configuration",
  "test_framework": "GoogleTest",
  "subsystem": "powermgr",
  "part": "hats"
}
```

---

## HDI 版本历史

### 各模块版本支持

| 模块 | V1_0 | V1_1 | V1_2 | V1_3 | V2_0 | V2_1 | V2_2 |
|------|------|------|------|------|------|------|------|
| Audio | ✅ | - | - | - | - | - | - |
| Battery | ✅ | - | - | - | ✅ | - | - |
| Camera | ✅ | ✅ | - | - | - | - | - |
| Display | ✅ | ✅ | ✅ | - | - | - | - |
| FacialAuth | ✅ | - | - | - | - | - | - |
| FingerprintAuth | ✅ | - | - | - | - | - | - |
| Light | ✅ | - | - | - | - | - | - |
| Motion | ✅ | - | - | - | - | - | - |
| Nnrt | ✅ | - | - | - | ✅ | - | - |
| Power | ✅ | ✅ | ✅ | ✅ | - | - | - |
| Ril | ✅ | - | - | - | - | ✅ | - |
| Sensor | ✅ | ✅ | - | - | - | - | ✅ |
| Thermal | ✅ | ✅ | - | - | - | - | - |
| Usb | ✅ | - | - | - | ✅ | ✅ | - |
| Vibrator | ✅ | - | - | - | ✅ | - | - |

---

## 用例命名约定

### 命名格式

| 类型 | 格式 | 示例 |
|------|------|------|
| **HDF 驱动** | `SUB_Driver_{Module}_{Feature}_{Number}` | `SUB_Driver_Display_Buffer_Death_0100` |
| **用户认证** | `Security_IAM_{Module}_HDI_FUNC_{Number}` | `Security_IAM_UserAuth_HDI_FUNC_0001` |
| **系统调用** | `{ApiName}{Validation}_{Number}` | `GetgidReturnActualGroupIDSuccess_0004` |
| **AI 推理** | `SUB_AI_NNRt_Func_{Category}_{Feature}_{Number}` | `SUB_AI_NNRt_Func_South_Device_DeviceInfo_0001` |
| **电源管理** | `SUB_Driver_Power_{Feature}_{Number}` | `SUB_Driver_Power_Suspend_0100` |

### 等级标记

| 标记 | 含义 | 使用场景 |
|------|------|----------|
| `Level0` | 冒烟测试 | 基本功能验证 |
| `Level1` | 基本测试 | 常见输入验证 |
| `Level2` | 重要测试 | 常规+异常测试 |
| `Level3` | 一般测试 | 全部功能测试 |
| `Level4` | 生僻测试 | 极端条件测试 |

---

## 测试类型标记

| 标记 | 含义 | 示例 |
|------|------|------|
| `Function` | 功能测试 | 验证 API 功能正确性 |
| `Performance` | 性能测试 | 验证响应时间/吞吐量 |
| `Power` | 功耗测试 | 验证能耗 |
| `Reliability` | 可靠性测试 | 长时间运行/压力测试 |
| `Security` | 安全测试 | 权限/认证/加密 |
| `Compatibility` | 兼容性测试 | 硬件/软件兼容性 |

---

## 术语表

| 术语 | 全称 | 说明 |
|------|------|------|
| **HATS** | Hardware Abstract Test Suite | 硬件抽象测试套件 |
| **HDI** | Hardware Driver Interface | 硬件驱动接口 |
| **HDF** | Hardware Driver Foundation | 硬件驱动框架 |
| **HAL** | Hardware Abstraction Layer | 硬件抽象层 |
| **XTS** | X Test Suite | 认证测试套件集合 |
| **SA** | System Ability | 系统能力 |
| **SAMgr** | System Ability Manager | 系统能力管理器 |
| **NNRT** | Neural Network Runtime | 神经网络运行时 |
| **RIL** | Radio Interface Layer | 无线接口层 |
| **IPC** | Inter-Process Communication | 进程间通信 |

---

## 相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| README | [README.md](./README.md) | 文档说明 |
| 导航 | [SUMMARY.md](./SUMMARY.md) | 完整导航 |
| 概览 | [00_Overview.md](./00_Overview.md) | 项目定位 |
| 架构 | [01_Architecture.md](./01_Architecture.md) | 系统架构 |
| 模块 | [02_Modules.md](./02_Modules.md) | 子系统详解 |
| N-API | [03_N-API.md](./03_N-API.md) | HDI 接口清单 |
| 构建 | [04_Build.md](./04_Build.md) | GN 构建配置 |
| 安全 | [05_Security.md](./05_Security.md) | 安全风险评审 |
