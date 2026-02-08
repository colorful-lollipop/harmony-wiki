# Wi-Fi Aware 安全风险评审

## 评审范围

| 维度 | 范围 |
|-----|------|
| **代码范围** | `frameworks/source/wifiaware.c`, `interfaces/kits/wifiaware.h`, `hals/hal_wifiaware.h` |
| **排除范围** | 测试代码 (`test/`, `*_test.*`), HAL 实现（板级） |
| **评审深度** | 静态代码分析 |

## 攻击面分析

### 已识别攻击面

| 攻击面 | 类型 | 说明 |
|-------|------|-----|
| **C API 输入** | 参数校验 | 9 个 C 函数的参数校验 |
| **回调函数** | 控制流 | 用户提供的 RecvCallback |
| **服务名** | 数据 | SHA256 哈希前的明文服务名 |
| **MAC 地址** | 数据 | 对端设备 MAC 地址 |
| **功率配置** | 配置 | 全局变量 g_power |
| **HAL 接口** | 边界 | 框架层与 HAL 层接口 |

### 未涉及的攻击面

| 攻击面 | 原因 |
|-------|------|
| **IPC** | 本模块不涉及 IPC（lite 版本） |
| **文件系统** | 无文件读写操作 |
| **网络** | 无 TCP/UDP Socket 操作 |
| **权限校验** | 无权限检查代码 |
| **签名验证** | 无签名相关逻辑 |
| **动态加载** | 无 dlopen/dlsym 等 |

## 信任边界

```
┌──────────────────────────────────────────────────────────────┐
│  Application Layer (JavaScript/Native App)                   │
│  - 权限校验在此层（或 IPC 层）                                 │
│  - 调用 C API                                                 │
└──────────────────────────────────────────────────────────────┘
                              │ 调用
                              ▼
┌──────────────────────────────────────────────────────────────┐
│  Framework Layer (wifiaware.c)                               │
│  - 参数校验                                                   │
│  - SHA256 服务名哈希                                          │
│  - 热点状态验证                                              │
└──────────────────────────────────────────────────────────────┘
                              │ 调用
                              ▼
┌──────────────────────────────────────────────────────────────┐
│  HAL Layer (hal_wifiaware.h - 抽象接口)                       │
│  - 板级实现（Hi3861 HAL）                                     │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│  Hardware Layer (Wi-Fi 芯片)                                 │
└──────────────────────────────────────────────────────────────┘
```

## 安全风险清单

### 🔴 高风险

#### 1. 全局变量线程安全问题

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:26` |
| **证据** | `static signed char g_power = WIFIAWARE_TX_LOW_POWER;` |
| **风险** | `SetPower()`/`SetLowPower()` 读写全局变量，非线程安全 |
| **触发** | 多线程并发调用功耗控制函数 |
| **影响** | 数据竞争导致功率设置错误，可能影响通信稳定性 |
| **修复建议** | 使用 `pthread_mutex_t` 保护，或移至 HAL 层管理 |

---

#### 2. 回调函数类型安全

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:71` |
| **证据** | `(HalRecvCallback)recvCB` |
| **风险** | 直接类型转换，假设 RecvCallback 与 HalRecvCallback 完全兼容 |
| **触发** | 回调签名不匹配（虽然目前两者相同） |
| **影响** | 未定义行为 |
| **修复建议** | 添加运行时断言或使用 `static_assert` 验证类型大小一致 |

---

#### 3. 热点状态竞争条件

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:33-39` |
| **证据** | `InitNAN()` 中检查热点状态后立即使用 |
| **风险** | 检查与使用之间存在时间窗口，热点可能关闭 |
| **触发** | 热点在 `IsHotspotActive()` 后、`HalWifiSdpInit()` 前关闭 |
| **影响** | 在不正确的状态下初始化 HAL，可能导致未定义行为 |
| **修复建议** | 在 HAL 层添加状态验证，或使用同步机制确保热点持续 |

---

### 🟡 中风险

#### 4. 缺乏参数范围验证

| 项目 | 说明 |
|-----|------|
| **位置** | 多个 API 函数 |
| **证据** | `SubscribeService()`: `localHandle` 预期 1-255，无显式检查 |
| **触发** | 传入超出范围的值（如 0, 256, 负数） |
| **影响** | HAL 层可能接收非法值 |
| **修复建议** | 添加参数范围检查，返回 `WIFIAWARE_FAIL` |

**代码证据**: `interfaces/kits/wifiaware.h:150-151`
```c
* @param localHandle Indicates the instance ID of the local device associated with the service name. The value ranges
 * from 1 to 255.
```

---

#### 5. SendData 缺乏输入验证

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:78-86` |
| **证据** | `SendData()` 直接传递 `macAddr`, `peerHandle`, `msg`, `len` 到 HAL |
| **风险** | - `macAddr` 可能 NULL<br>- `len` 可能超出 255<br>- `peerHandle` 范围未检查 |
| **触发** | 调用者传入无效参数 |
| **影响** | HAL 层崩溃或未定义行为 |
| **修复建议** | 添加参数验证 |

**文档说明**: `interfaces/kits/wifiaware.h:166-167`
> "As this function does not check whether the MAC address and instance ID are valid, the caller should ensure their validity."

---

#### 6. 长度整数溢出风险

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:62` |
| **证据** | `svcNameLen = strlen(svcName);` |
| **风险** | 超长字符串可能导致整数溢出或缓冲区问题 |
| **触发** | 服务名超过 `size_t` 范围 |
| **影响** | SHA256 计算异常 |
| **修复建议** | 添加长度限制检查 |

---

### 🟢 低风险 / 良好实践

#### 7. SHA256 服务名哈希（安全加固）

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:67` |
| **证据** | `HalCipherHashSha256(svcName, svcNameLen, shaSvcName, sizeof(shaSvcName))` |
| **评估** | ✅ 良好实践 - 服务名哈希后传递给 HAL，防止明文泄露 |
| **建议** | 考虑使用 HMAC 增加密钥混淆 |

---

#### 8. 错误码统一

| 项目 | 说明 |
|-----|------|
| **位置** | `interfaces/kits/wifiaware.h:70-75` |
| **证据** | `WIFIAWARE_SUCCESS` (0), `WIFIAWARE_FAIL` (-1) |
| **评估** | ✅ 良好实践 - 统一错误码模式 |
| **建议** | 可考虑扩展错误码以区分具体失败原因 |

---

#### 9. 无缓冲区重用（安全）

| 项目 | 说明 |
|-----|------|
| **位置** | `frameworks/source/wifiaware.c:56-57` |
| **证据** | `shaSvcName` 局部变量使用后释放 |
| **评估** | ✅ 良好实践 - 无缓冲区重用，减少悬空指针风险 |

---

## 安全总结

| 风险等级 | 数量 | 说明 |
|---------|------|------|
| 🔴 高 | 3 | 需优先修复 |
| 🟡 中 | 3 | 建议修复 |
| 🟢 低/良好 | 3 | 继续保持 |

## 总体评估

**模块安全性**: ⚠️ 中等

**主要原因**:
1. 线程安全问题（全局变量）
2. 缺乏输入参数验证
3. 文档已知限制（调用者保证）

**缓解因素**:
1. 本模块运行在受信任的 HAL 层之上
2. SHA256 服务名哈希提供基本混淆
3. 权限检查在更高层（IPC/SoftBus）处理

## 后续修复建议

### 短期（高优先级）

1. **添加互斥锁保护全局变量**
   ```c
   static pthread_mutex_t g_power_mutex = PTHREAD_MUTEX_INITIALIZER;
   ```

2. **添加参数范围检查**
   ```c
   if (localHandle == 0 || localHandle > 255) {
       return WIFIAWARE_FAIL;
   }
   ```

3. **在 SendData 添加参数验证**
   ```c
   if (macAddr == NULL || len > 255 || len < 0) {
       return WIFIAWARE_FAIL;
   }
   ```

### 中期

1. 文档化线程模型
2. 添加单元测试覆盖边界条件
3. 考虑扩展错误码

### 长期

1. 评估是否需要内核级锁机制
2. 安全审计 HAL 层实现
