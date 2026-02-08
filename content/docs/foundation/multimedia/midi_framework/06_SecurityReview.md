# 安全风险评估 (SecurityReview)

## 1. 评估方法

本评估基于以下维度分析 midi_framework 的安全风险：
- **输入验证缺陷**: 类型、长度、范围、null、编码、路径遍历
- **内存安全问题**: 缓冲区、Use-After-Free、双重释放
- **权限与鉴权**: 权限校验、身份验证、访问控制
- **并发安全**: 竞态条件、TOCTOU、线程安全
- **逻辑漏洞**: 错误处理、资源耗尽、信息泄露

---

## 2. 输入验证缺陷

### R1: BLE MAC 地址解析漏洞

**风险等级**: MEDIUM

**位置**: `services/server/src/midi_device_ble.cpp:180-220`

**代码证据**:
```cpp
static bool ParseMac(const std::string &mac, BdAddr &out) {
    CHECK_AND_RETURN_RET(mac.size() == MAC_STR_LENGTH, false);  // 17 chars
    int32_t  bi = 0;
    for (size_t i = 0; i < mac.size();) {
        CHECK_AND_RETURN_RET(i + 1 < mac.size(), false);
        char c1 = mac[i];
        char c2 = mac[i + 1];
        auto hexVal = [](char c)->int {
            if (c >= '0' && c <= '9') return c - '0';
            if (c >= 'A' && c <= 'F') return HEX_VAL_OFFSET + (c - 'A');
            if (c >= 'a' && c <= 'f') return HEX_VAL_OFFSET + (c - 'a');
            return -1;  // 无效字符
        };
        CHECK_AND_RETURN_RET(hexVal(c1) >= 0 && hexVal(c2) >= 0, false);
        // ...
    }
}
```

**分析**:
- ✅ 长度检查: 17 字符 (AA:BB:CC:DD:EE:FF)
- ✅ 字符集验证: 十六进制数字和冒号
- ⚠️ 冒号位置未严格验证

**触发路径**:
```
OH_MIDIOpenBleDevice("GG:HH:II:JJ:KK:LL", ...) 
→ 在 hexVal 返回 -1 处被拒绝
→ 返回 false，连接失败
```

**影响**: 低 - 输入被正确拒绝

**修复建议**: 当前实现已足够，可添加冒号位置检查增强健壮性

---

### R2: 共享内存 FD 验证

**风险等级**: MEDIUM

**位置**: `services/common/src/midi_shared_ring.cpp:196-205`

**代码证据**:
```cpp
int minfd = 2; // ignore stdout, stdin and stderr
CHECK_AND_RETURN_RET_LOG(fd > minfd, nullptr, 
    "CreateFromRemote failed: invalid fd: %{public}d", fd);

off_t actualSize = lseek(fd, 0, SEEK_END);
CHECK_AND_RETURN_RET_LOG((actualSize == (off_t)size) && size != 0, nullptr, ...);
```

**分析**:
- ✅ FD 范围检查: > 2 (排除 stdin/stdout/stderr)
- ✅ 文件大小验证: 与预期大小匹配
- ✅ 非零大小检查

**触发路径**:
```
恶意 IPC 调用传递 fd=1 (stdout)
→ minfd 检查失败
→ 返回 nullptr，拒绝映射
```

**影响**: 中 - 防止特殊 FD 注入攻击

**修复建议**: 当前实现良好

---

### R3: UMP 事件长度验证

**风险等级**: MEDIUM

**位置**: `services/common/src/midi_shared_ring.cpp:541-546`

**代码证据**:
```cpp
if (event.length > (std::numeric_limits<size_t>::max() / sizeof(uint32_t))) {
    return false;  // 整数溢出检查
}
const size_t payloadBytes = event.length * sizeof(uint32_t);
const size_t maxLeftBytes = static_cast<size_t>(capacity_) - 1u - sizeof(ShmMidiEventHeader);
CHECK_AND_RETURN_RET_LOG(payloadBytes <= maxLeftBytes, false, "event length overflow");
```

**分析**:
- ✅ 整数溢出检查: 乘法前验证
- ✅ 缓冲区边界检查: payload <= 剩余空间
- ✅ 安全类型转换

**影响**: 低 - 已实施完整边界保护

---

## 3. 内存安全问题

### R4: 共享内存映射验证

**风险等级**: LOW

**位置**: `services/common/src/midi_shared_ring.cpp:128-130`

**代码证据**:
```cpp
void *addr = mmap(nullptr, size_, PROT_READ | PROT_WRITE, MAP_SHARED, fd_, 0);
CHECK_AND_RETURN_RET_LOG(addr != MAP_FAILED, MIDI_STATUS_UNKNOWN_ERROR, ...);
```

**分析**:
- ✅ mmap 返回值检查
- ✅ MAP_SHARED 确保进程间可见
- ⚠️ 未设置 MAP_LOCKED (可能被交换)

**影响**: 低 - 标准做法

---

### R5: 死亡通知回调安全

**风险等级**: LOW

**位置**: `services/server/src/midi_service_controller.cpp:147-155`

**代码证据**:
```cpp
sptr<MidiServiceDeathRecipient> deathRecipient_ = 
    new (std::nothrow) MidiServiceDeathRecipient(clientId);
deathRecipient_->SetNotifyCb([weakSelf](uint32_t clientId) {
    auto self = weakSelf.lock();
    CHECK_AND_RETURN_LOG(self != nullptr, "MidiServiceController destroyed");
    self->DestroyMidiClient(clientId);
});
object->AddDeathRecipient(deathRecipient_);
```

**分析**:
- ✅ nothrow new 防止异常
- ✅ weak_ptr 防止悬空引用
- ✅ nullptr 检查

**影响**: 低 - 实现安全

---

### R6: 未发现 UAF/Double-Free

**评估结果**: 经代码审查，未发现明显的 Use-After-Free 或 Double-Free 漏洞

**检查范围**:
- 服务端对象生命周期管理
- 共享内存引用计数
- 客户端连接清理

---

## 4. 权限与鉴权

### R7: 权限检查缺失

**风险等级**: HIGH

**发现**:
```cpp
// 搜索 services/ 和 frameworks/native/ 目录
// 未发现: VerifyPermission, CheckPermission, CheckSystemPermission
```

**分析**:
- ❌ 服务端代码**未显式校验**调用者权限
- ✅ 依赖 IPC 机制隐式校验 (OpenHarmony Binder 自带 UID 校验)
- ⚠️ BLE 权限仅在文档标注 (`native_midi.h:134`)

**代码证据**:
```cpp
/**
 * @permission ohos.permission.ACCESS_BLUETOOTH
 * @param client Target client handle.
 * @param deviceAddr The MAC address of the BLE device
 */
OH_MIDIStatusCode OH_MIDIOpenBleDevice(...);
```

**触发路径**:
```
无蓝牙权限的应用
→ 调用 OH_MIDIOpenBleDevice()
→ 通过 IPC 到达服务端
→ 服务端未校验权限
→ 可能建立 BLE 连接
```

**影响**: 高 - 权限绕过风险

**修复建议**:
```cpp
// 在服务端增加权限检查
int32_t MidiServiceController::OpenBleDevice(...) {
    // 添加权限校验
    if (!VerifyPermission("ohos.permission.ACCESS_BLUETOOTH")) {
        return MIDI_STATUS_PERMISSION_DENIED;
    }
    // ... 原逻辑
}
```

---

## 5. 并发安全

### R8: 锁保护完整性

**风险等级**: LOW

**位置**: `services/server/src/midi_service_controller.cpp` (多处)

**代码证据**:
```cpp
int32_t MidiServiceController::OpenInputPort(...) {
    std::lock_guard lock(lock_);  // 全程持有锁
    CHECK_AND_RETURN_RET_LOG(clients_.find(clientId) != clients_.end(), ...);
    // ... 后续操作
}
```

**分析**:
- ✅ 使用 `std::lock_guard` RAII 模式
- ✅ 关键操作全程锁保护
- ✅ 原子操作用于 clientId 生成

**代码证据**:
```cpp
// services/server/src/midi_service_controller.cpp:37
static std::atomic<uint32_t> MidiServiceController::currentClientId_ = 0;
```

**影响**: 低 - 线程安全良好

---

### R9: 共享内存竞态条件

**风险等级**: MEDIUM

**位置**: `services/common/include/midi_shared_ring.h:34-40`

**代码证据**:
```cpp
struct alignas(64) ControlHeader {
    std::atomic<uint32_t> readPosition;
    std::atomic<uint32_t> writePosition;
    uint32_t capacity;
    std::atomic<uint32_t> futexObj;
    uint32_t flags;
};
```

**分析**:
- ✅ 位置指针使用 `std::atomic`
- ✅ 64 字节对齐 (cache line)
- ⚠️ `capacity` 非原子 (初始化后只读)
- ⚠️ `flags` 非原子

**影响**: 低 - 设计合理

---

## 6. 逻辑漏洞

### R10: 自动卸载定时器

**风险等级**: LOW

**位置**: `services/server/src/midi_service_controller.cpp:98-125`

**代码证据**:
```cpp
void MidiServiceController::ScheduleUnloadTask() {
    unloadThread_ = std::thread([this]() {
        std::unique_lock<std::mutex> lk(unloadMutex_);
        if (unloadCv_.wait_for(lk, std::chrono::milliseconds(unloadDelayTime_)) 
            == std::cv_status::timeout) {
            samgr->UnloadSystemAbility(MIDI_SERVICE_ID);
        }
    });
}
```

**分析**:
- ✅ 可取消机制 (CancelUnloadTask)
- ✅ 条件变量等待
- ⚠️ 卸载延迟默认 60 秒

**潜在风险**: 快速创建/销毁客户端可能触发频繁启动/停止

**影响**: 低 - 资源消耗而非安全

---

### R11: 日志信息泄露

**风险等级**: LOW

**发现**:
```cpp
// services/server/src/midi_service_controller.cpp:218
MIDI_INFO_LOG("OpenBleDevice: clientId=%{public}u, device=%{public}s", 
    clientId, GetEncryptStr(address).c_str());
```

**分析**:
- ✅ BLE 地址使用 `GetEncryptStr` 加密
- ⚠️ 部分日志可能包含敏感信息

**影响**: 低 - 已采取保护措施

---

## 7. 风险汇总

| 风险 ID | 类别 | 等级 | 状态 | 修复建议 |
|---------|------|------|------|----------|
| R1 | 输入验证 | MEDIUM | ✅ 已缓解 | 添加冒号位置验证 |
| R2 | 输入验证 | MEDIUM | ✅ 已缓解 | 保持当前实现 |
| R3 | 输入验证 | MEDIUM | ✅ 已缓解 | 保持当前实现 |
| R4 | 内存安全 | LOW | ✅ 已缓解 | 可选: MAP_LOCKED |
| R5 | 内存安全 | LOW | ✅ 已缓解 | 保持当前实现 |
| R6 | 内存安全 | - | ✅ 未发现 | 持续监控 |
| R7 | 权限鉴权 | **HIGH** | ❌ **待修复** | **添加权限检查** |
| R8 | 并发安全 | LOW | ✅ 已缓解 | 保持当前实现 |
| R9 | 并发安全 | MEDIUM | ✅ 已缓解 | 保持当前实现 |
| R10 | 逻辑漏洞 | LOW | ✅ 已缓解 | 保持当前实现 |
| R11 | 逻辑漏洞 | LOW | ✅ 已缓解 | 审计日志内容 |

---

## 8. 修复优先级

### 立即修复 (P0)
- **R7**: 添加服务端权限校验

### 建议修复 (P1)
- **R1**: 增强 MAC 地址格式验证

### 观察项 (P2)
- 持续 fuzz 测试共享内存操作
- 监控 USB/BLE 驱动安全更新

---

## 9. 参考 CVE

基于外部研究，类似系统的历史漏洞:

| CVE | 组件 | 类型 | 与本项目关联 |
|-----|------|------|-------------|
| CVE-2025-37891 | ALSA UMP | SysEx 缓冲区溢出 | UMP 处理需验证 |
| CVE-2016-2384 | USB MIDI | Double-Free | 死亡通知机制 |
| CVE-2025-67326 | MIDI Parser | OOB Read | 输入验证 |

---

*文档结束*
