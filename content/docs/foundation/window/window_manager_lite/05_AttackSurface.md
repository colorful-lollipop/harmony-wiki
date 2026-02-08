# 攻击面分析

**文档定位**: 安全研究员专用 - 识别所有外部输入入口和信任边界

**分析范围**: `services/wms/`, `services/ims/`, `frameworks/`

**分析日期**: 2024-02-07

---

## 1. 攻击面概述

### 1.1 信任边界图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        不可信区域                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  用户空间应用（任意第三方代码）                              │   │
│  │  • 可发送任意 IPC 消息                                       │   │
│  │  • 可尝试构造畸形数据                                        │   │
│  │  • 可尝试权限绕过                                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────┬──────────────────────────────────────────┘
                         │ IPC (Samgr Lite)
                         │ 风险：消息伪造、重放、拦截
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        信任边界 ⚠️                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  wms_server（特权进程，system 用户）                         │   │
│  │                                                             │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │ LiteWMS (IPC 请求处理)                              │   │   │
│  │  │ • WMSRequestHandle() - 15 个操作码入口               │   │   │
│  │  │ • GetCallingPid()/GetCallingUid() - 身份获取         │   │   │
│  │  │ • CheckPermission() - 仅 Screenshot 使用            │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  │                                                             │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │ InputEventClientProxy (IMS 客户端管理)              │   │   │
│  │  │ • AddListener() - 客户端注册入口                     │   │   │
│  │  │ • RemoveListener() - 客户端注销                      │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────┬──────────────────────────────────────────┘
                         │ HAL 接口
                         │ 风险：驱动层攻击
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        硬件抽象层                                    │
│  ┌──────────────────────┐  ┌──────────────────────────────────────┐ │
│  │    Display HAL       │  │     Input HAL (HDI)                  │ │
│  │    (Surface/Buffer)  │  │     (RawEvent from kernel)           │ │
│  └──────────────────────┘  └──────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击向量总结

| 攻击向量 | 入口点 | 风险等级 | 说明 |
|----------|--------|----------|------|
| IPC 消息攻击 | WMSRequestHandle() | 🔴 高 | 15 个操作码，主要攻击面 |
| IPC 消息攻击 | ClientRequestHandle() | 🟡 中 | IMS 客户端管理 |
| 输入事件注入 | EventCallback() | 🟡 中 | 来自 HDI 的 RawEvent |
| 权限绕过 | GetCallingUid() | 🟡 中 | 身份验证依赖 |
| 资源耗尽 | CreateWindow() | 🟡 中 | 窗口/客户端数量限制 |

---

## 2. IPC 接口攻击面（详细）

### 2.1 WMS IPC 接口清单

**入口函数**: `LiteWMS::WMSRequestHandle()`  
**文件位置**: `services/wms/lite_wms.cpp:31-80`

| 操作码 | 函数 | 输入参数 | 敏感操作 | 权限检查 | 风险等级 |
|--------|------|----------|----------|----------|----------|
| 0 | GetSurface | windowId (int32) | 返回 SvcIdentity | ❌ 无 | 🟡 中 |
| 1 | Show | windowId (int32) | 修改窗口状态 | ❌ 无 | 🟢 低 |
| 2 | Hide | windowId (int32) | 修改窗口状态 | ❌ 无 | 🟢 低 |
| 3 | RaiseToTop | windowId (int32) | 修改 Z 序 | ❌ 无 | 🟡 中 |
| 4 | LowerToBottom | windowId (int32) | 修改 Z 序 | ❌ 无 | 🟡 中 |
| 5 | MoveTo | windowId + X/Y (uint32×3) | 修改位置 | ❌ 无 | 🟡 中 |
| 6 | Resize | windowId + W/H (uint32×3) | 重新分配 buffer | ❌ 无 | 🔴 高 |
| 7 | Update | windowId (int32) | 触发重绘 | ❌ 无 | 🟢 低 |
| 8 | CreateWindow | LiteWinConfig (struct) | 创建窗口对象 | ❌ 无 | 🔴 高 |
| 9 | RemoveWindow | windowId (int32) | 销毁窗口 | ❌ 无 | 🟡 中 |
| 10 | GetEventData | 无 | 返回 DeviceData | ❌ 无 | 🟢 低 |
| 11 | Screenshot | Surface 对象 | 屏幕捕获 | ✅ 有 | 🟡 中 |
| 12 | ClientRegister | SvcIdentity | 注册死亡回调 | ❌ 无 | 🟢 低 |
| 13 | GetLayerInfo | 无 | 返回显示信息 | ❌ 无 | 🟢 低 |

### 2.2 关键入口点代码

#### CreateWindow（高风险）

```cpp
// services/wms/lite_wms.cpp:186-199
void LiteWMS::CreateWindow(IpcIo* req, IpcIo* reply)
{
    GRAPHIC_LOGI("CreateWindow");
    // 读取外部输入：LiteWinConfig 结构体
    LiteWinConfig* config = static_cast<LiteWinConfig*>(
        ReadRawData(req, sizeof(LiteWinConfig)));
    
    if (config != nullptr) {
        // 获取调用者身份
        pid_t pid = GetCallingPid();
        // 创建窗口，无权限检查！
        LiteWindow* window = LiteWM::GetInstance()->CreateWindow(*config, pid);
        if (window != nullptr) {
            WriteInt32(reply, window->GetWindowId());
            return;
        }
    }
    WriteInt32(reply, INVALID_WINDOW_ID);
}
```

**攻击面**:
- `config` 结构体完全由调用者控制
- 可创建全屏窗口 (`rect` 覆盖整个屏幕)
- 可创建模态窗口 (`isModal = true`)
- 可指定任意合成模式 (`COPY`/`BLEND`)
- **无权限校验**，任意应用可调用

#### Resize（中高风险）

```cpp
// services/wms/lite_wms.cpp:166-176
void LiteWMS::Resize(IpcIo* req, IpcIo* reply)
{
    GRAPHIC_LOGI("Resize");
    int32_t id;
    ReadInt32(req, &id);
    uint32_t width;   // 无符号，可为任意大值
    ReadUint32(req, &width);
    uint32_t height;  // 无符号，可为任意大值
    ReadUint32(req, &height);
    
    // 直接传递，无边界检查！
    LiteWM::GetInstance()->Resize(id, width, height);
}
```

**攻击面**:
- `width`/`height` 为 `uint32_t`，可传入极大值
- 可能导致整数溢出或内存分配失败
- 未验证窗口所有权

#### Screenshot（有权限检查）

```cpp
// services/wms/lite_wms.cpp:216-228
void LiteWMS::Screenshot(IpcIo* req, IpcIo* reply)
{
    const char *writeMediaImagePermissionName = 
        "ohos.permission.WRITE_MEDIA_IMAGES";
    pid_t uid = GetCallingUid();
    
    // 权限检查
    if (CheckPermission(uid, writeMediaImagePermissionName) != GRANTED) {
        GRAPHIC_LOGE("permission denied");
        WriteInt32(reply, LiteWMS_EUNKNOWN);
        return;
    }
    // ... 截图逻辑
}
```

**攻击面**:
- 依赖 `GetCallingUid()`，可能被伪造
- 仅检查单一权限，无其他验证

---

## 3. IMS IPC 接口攻击面

### 3.1 IMS 接口清单

**入口函数**: `InputEventClientProxy::ClientRequestHandle()`  
**文件位置**: `services/ims/input_event_client_proxy.cpp:28-43`

| 操作码 | 函数 | 输入参数 | 敏感操作 | 权限检查 | 风险等级 |
|--------|------|----------|----------|----------|----------|
| 0 | AddListener | SvcIdentity + bool | 注册客户端 | ❌ 无 | 🟡 中 |
| 1 | RemoveListener | 隐式（通过 PID） | 注销客户端 | ❌ 无 | 🟢 低 |

### 3.2 关键入口点代码

#### AddListener

```cpp
// services/ims/input_event_client_proxy.cpp:45-72
void InputEventClientProxy::AddListener(const void* origin, 
                                        IpcIo* req, IpcIo* reply)
{
    // 检查最大客户端数
    pthread_mutex_lock(&lock_);
    if (clientInfoMap_.size() >= MAX_CLIENT_SIZE) {
        pthread_mutex_unlock(&lock_);
        GRAPHIC_LOGE("Exceeded the maximum number!");
        return;
    }
    pthread_mutex_unlock(&lock_);
    
    pid_t pid = GetCallingPid();
    SvcIdentity svc = {0};
    bool ret = ReadRemoteObject(req, &svc);
    bool alwaysInvoke;
    ReadBool(req, &alwaysInvoke);
    
    // 注册死亡回调
    uint32_t cbId = 0;
    if (AddDeathRecipient(svc, DeathCallback, nullptr, &cbId) != 0) {
        GRAPHIC_LOGE("Register death callback failed!");
        return;
    }
    
    struct ClientInfo clientInfo = { svc, cbId, alwaysInvoke };
    pthread_mutex_lock(&lock_);
    clientInfoMap_.insert(std::make_pair(pid, clientInfo));
    pthread_mutex_unlock(&lock_);
}
```

**攻击面**:
- 客户端数量有限制（`MAX_CLIENT_SIZE`）
- 可注册任意 `SvcIdentity`
- `alwaysInvoke` 标志可控制事件分发行为

---

## 4. 外部输入数据清单

### 4.1 IPC 输入数据

| 数据类型 | 来源 | 验证情况 | 风险 |
|----------|------|----------|------|
| `int32_t windowId` | IPC | 仅检查 null，不验证所有权 | 可操作其他应用窗口 |
| `uint32_t x, y` | IPC | ❌ 无验证 | 坐标越界 |
| `uint32_t width, height` | IPC | ❌ 无验证 | 整数溢出 |
| `LiteWinConfig` | IPC | ❌ 无验证 | 任意窗口配置 |
| `SvcIdentity` | IPC | ❌ 无验证 | 可能伪造 |

### 4.2 输入设备数据

| 数据类型 | 来源 | 验证情况 | 风险 |
|----------|------|----------|------|
| `RawEvent` | HDI/驱动 | ❌ 无验证 | 事件注入 |

**RawEvent 结构**:
```cpp
struct RawEvent {
    uint32_t type;      // 事件类型（鼠标/触摸/按键）
    uint32_t code;      // 事件代码
    int32_t  value;     // 事件值
    int16_t  x;         // X 坐标
    int16_t  y;         // Y 坐标
    uint64_t time;      // 时间戳
    uint16_t state;     // 按键状态
};
```

**来源**: `frameworks/ims/input_event_listener_proxy.cpp:59`
```cpp
RawEvent* eventTemp = static_cast<RawEvent*>(
    ReadRawData(io, sizeof(RawEvent)));
```

---

## 5. 敏感操作清单

### 5.1 系统服务调用

| 操作 | 位置 | 说明 |
|------|------|------|
| `CheckPermission()` | `lite_wms.cpp:220` | 权限校验（仅 Screenshot） |
| `GetCallingPid()` | `lite_wms.cpp:191,232` | 获取调用者 PID |
| `GetCallingUid()` | `lite_wms.cpp:219` | 获取调用者 UID |
| `AddDeathRecipient()` | `lite_wms.cpp:243` | 注册进程死亡监听 |
| `WriteRemoteObject()` | `lite_wms.cpp:119` | 返回服务对象 |

### 5.2 内存/图形操作

| 操作 | 位置 | 说明 |
|------|------|------|
| `new LiteWindow()` | `lite_wm.cpp:334` | 窗口对象分配 |
| `Surface::CreateSurface()` | `lite_win.cpp:80` | Surface 创建 |
| `memcpy_s()` | `lite_win.cpp:142` | 缓冲区拷贝 |
| `memset_s()` | `lite_wm.cpp:582` | 背景填充 |
| `LcdFlush()` | `lite_wm.cpp` | 屏幕刷新 |

### 5.3 资源限制

| 资源 | 限制值 | 位置 |
|------|--------|------|
| 最大窗口数 | 32 | `lite_wm.cpp:95` |
| 最大客户端数 | MAX_CLIENT_SIZE | `input_event_client_proxy.cpp` |
| 最大更新区域 | 8 | `lite_wm.h:32` |

---

## 6. 信任边界跨越点

### 6.1 边界跨越分析

```
应用层（不可信）
    │
    ├── IPC 调用 ──► 服务层（信任边界）
    │                  ├── 窗口管理操作
    │                  ├── 图形内存访问
    │                  └── 输入事件读取 ◄── 驱动层
    │
    └── 数据流向
        应用 ──► 服务：配置参数、控制命令
        服务 ──► 应用：窗口句柄、事件数据
```

### 6.2 关键跨越点

| 跨越点 | 方向 | 数据 | 风险 |
|--------|------|------|------|
| `CreateWindow` | 应用→服务 | LiteWinConfig | 配置注入 |
| `GetSurface` | 服务→应用 | SvcIdentity | 句柄泄露 |
| `GetEventData` | 服务→应用 | DeviceData | 信息泄露 |
| `OnRawEvent` | 驱动→服务 | RawEvent | 事件注入 |

---

## 7. 攻击场景示例

### 场景 1：窗口创建滥用

```
攻击步骤：
1. 恶意应用调用 CreateWindow
2. 构造全屏模态窗口配置
   - rect = {0, 0, screenWidth, screenHeight}
   - isModal = true
3. 窗口覆盖所有其他应用
4. 实现"锁屏"或钓鱼界面
```

**利用代码路径**:  
`LiteProxyWindow::CreateWindow()` → `LiteWMRequestor::CreateWindow()` → IPC → `LiteWMS::CreateWindow()` → `LiteWM::CreateWindow()`

### 场景 2：资源耗尽攻击

```
攻击步骤：
1. 循环调用 CreateWindow 创建 32 个窗口
2. 每个窗口分配大尺寸 Surface
3. 系统无法为正常应用创建窗口
```

**限制**: 最大 32 窗口，但有 32 个窗口的 DoS

### 场景 3：Resize 整数溢出

```
攻击步骤：
1. 创建正常窗口
2. 调用 Resize(width=0xFFFFFFFF, height=0xFFFFFFFF)
3. 可能导致内存分配异常
```

**相关代码**: `lite_wms.cpp:166-176`

---

## 8. 检查清单（安全审计用）

### IPC 入口检查
- [ ] 所有 IPC 处理函数都检查了吗？
- [ ] 输入参数都有边界验证吗？
- [ ] 窗口所有权都验证了吗？
- [ ] 资源限制都正确实施了吗？

### 权限检查
- [ ] 敏感操作都有权限检查吗？
- [ ] 身份验证足够强健吗？
- [ ] 权限绕过可能吗？

### 内存安全
- [ ] 缓冲区操作都使用安全函数了吗？
- [ ] 整数溢出都防护了吗？
- [ ] 资源泄漏都处理了吗？

---

## 9. 附录：关键代码位置索引

| 功能 | 文件 | 行号范围 |
|------|------|----------|
| IPC 请求处理 | `services/wms/lite_wms.cpp` | 31-80 |
| CreateWindow | `services/wms/lite_wms.cpp` | 186-199 |
| Resize | `services/wms/lite_wms.cpp` | 166-176 |
| MoveTo | `services/wms/lite_wms.cpp` | 154-164 |
| Screenshot | `services/wms/lite_wms.cpp` | 216-228 |
| IMS 客户端管理 | `services/ims/input_event_client_proxy.cpp` | 28-108 |
| 窗口创建实现 | `services/wms/lite_wm.cpp` | 328-345 |
| 目标窗口查找 | `services/wms/lite_wm.cpp` | 621-661 |
| 事件接收 | `frameworks/ims/input_event_listener_proxy.cpp` | 54-67 |

