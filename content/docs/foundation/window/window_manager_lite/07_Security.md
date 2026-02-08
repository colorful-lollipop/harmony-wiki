# 安全风险评审

## 评审概述

本章节对 window_manager_lite 进行安全风险评估，基于代码分析识别潜在攻击面和可被利用点。

**评审范围**: `frameworks/`, `services/`, `interfaces/` (不含测试代码)

**评审日期**: 2024-02-06

## 攻击面分析

### 1. IPC 接口攻击面

| 接口 | 风险等级 | 说明 |
|------|----------|------|
| `CreateWindow` | 中 | 接收外部配置，可能存在边界问题 |
| `Resize` | 低 | 接收尺寸参数 |
| `MoveTo` | 低 | 接收坐标参数 |
| `Screenshot` | 高 | 涉及权限校验 |
| `GetEventData` | 低 | 读取事件数据 |

**证据**: `lite_wms.cpp:31-80` - WMS 请求处理分发器

### 2. 输入设备攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| RawEvent 注入 | 中 | HDI 回调可能接收恶意事件 |
| 事件队列溢出 | 低 | 队列有大小限制 |

### 3. 文件/内存攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| Surface 缓冲区 | 中 | 共享内存，可能被恶意访问 |
| 窗口配置结构 | 低 | 栈上分配 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        不可信区域                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  用户空间应用 (任意第三方代码)                      │   │
│  └─────────────────────────────────────────────────────┘   │
└────────────────────────┬──────────────────────────────────┘
                         │ IPC 调用
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  wms_server (特权进程)                             │   │
│  │  ├── LiteWMS (IPC 处理)                           │   │
│  │  ├── LiteWM (窗口管理)                            │   │
│  │  └── InputManagerService (输入管理)               │   │
│  └─────────────────────────────────────────────────────┘   │
└────────────────────────┬──────────────────────────────────┘
                         │ HAL 调用
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                        硬件抽象层                            │
│  ┌──────────────────────┐  ┌──────────────────────────┐  │
│  │    Display HAL       │  │     Input HAL            │  │
│  └──────────────────────┘  └──────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 可被利用点 (CVE/风险)

### 高风险项

#### 1. Screenshot 权限绕过风险

| 属性 | 值 |
|------|-----|
| **风险等级** | 高 |
| **位置** | `lite_wms.cpp:216-228` |
| **触发条件** | 未正确校验权限 |

**代码分析**:

```cpp
void LiteWMS::Screenshot(IpcIo* req, IpcIo* reply)
{
    const char* writeMediaImagePermissionName = "ohos.permission.WRITE_MEDIA_IMAGES";
    pid_t uid = GetCallingUid();
    if (CheckPermission(uid, writeMediaImagePermissionName) != GRANTED) {
        GRAPHIC_LOGE("permission denied");
        WriteInt32(reply, LiteWMS_EUNKNOWN);
        return;
    }
    // 执行截图...
}
```

**问题描述**:
- `GetCallingUid()` 可能被伪造（取决于 IPC 实现）
- 无调用方身份强校验
- 权限校验失败后仅记录日志，无其他保护措施

**修复建议**:
1. 增加 token 校验而非仅 UID
2. 记录详细的审计日志
3. 考虑添加 CAPTURE_SCREEN 权限

#### 2. 窗口创建无权限校验

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 |
| **位置** | `lite_wms.cpp:186-199` |
| **触发条件** | 任意应用可创建窗口 |

**代码分析**:

```cpp
void LiteWMS::CreateWindow(IpcIo* req, IpcIo* reply)
{
    LiteWinConfig* config = static_cast<LiteWinConfig*>(ReadRawData(req, sizeof(LiteWinConfig)));
    if (config != nullptr) {
        pid_t pid = GetCallingPid();  // 仅获取 PID，无权限校验
        LiteWindow* window = LiteWM::GetInstance()->CreateWindow(*config, pid);
        // ...
    }
}
```

**问题描述**:
- 任意应用可创建窗口
- 可创建全屏/模态窗口
- 可覆盖其他应用窗口

**修复建议**:
1. 添加 `ohos.permission.CREATE_WINDOW` 权限校验
2. 限制非系统应用创建模态窗口
3. 限制窗口尺寸和位置范围

### 中风险项

#### 3. 窗口 ID 验证不严格

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 |
| **位置** | `lite_wms.cpp:96-105`, `lite_wms.cpp:122-136` |
| **触发条件** | 使用无效窗口 ID |

**代码分析**:

```cpp
void LiteWMS::Show(IpcIo* req, IpcIo* reply)
{
    int32_t id;
    ReadInt32(req, &id);
    LiteWM::GetInstance()->Show(id);  // 未验证 ID 有效性
}
```

**问题描述**:
- 无效 ID 导致空指针操作
- 可能访问未授权窗口

**修复建议**:
1. 在入口处验证 ID 范围
2. 增加 ID 存在性检查

#### 4. Resize 参数边界未校验

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 |
| **位置** | `lite_wms.cpp:166-176` |

**代码分析**:

```cpp
void LiteWMS::Resize(IpcIo* req, IpcIo* reply)
{
    int32_t id;
    ReadInt32(req, &id);
    uint32_t width, height;
    ReadUint32(req, &width);
    ReadUint32(req, &height);
    LiteWM::GetInstance()->Resize(id, width, height);  // 未校验尺寸范围
}
```

**问题描述**:
- 宽度/高度可为任意正值
- 可能导致整数溢出或内存分配过大

**修复建议**:
1. 添加最大尺寸限制（如屏幕尺寸）
2. 添加最小尺寸限制

#### 5. MoveTo 坐标越界

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 |
| **位置** | `lite_wms.cpp:154-164` |

**代码分析**:

```cpp
void LiteWMS::MoveTo(IpcIo* req, IpcIo* reply)
{
    int32_t id;
    ReadInt32(req, &id);
    uint32_t x, y;
    ReadUint32(req, &x);
    ReadUint32(req, &y);
    LiteWM::GetInstance()->MoveTo(id, x, y);  // 未校验坐标范围
}
```

### 低风险项

#### 6. ClientRegister 回调管理

| 属性 | 值 |
|------|-----|
| **风险等级** | 低 |
| **位置** | `lite_wms.cpp:230-246` |

**代码分析**:

```cpp
void LiteWMS::ClientRegister(IpcIo* req, IpcIo* reply)
{
    pid_t pid = GetCallingPid();
    SvcIdentity sid;
    bool ret = ReadRemoteObject(req, &sid);
    // ...
    uint32_t cbId = -1;
    if (AddDeathRecipient(arg->sid, DeathCallback, arg, &cbId) != 0) {
        GRAPHIC_LOGE("AddDeathRecipient failed!");
    }
}
```

**问题描述**:
- 无回调数量限制
- 可能被恶意应用耗尽资源

#### 7. 输入事件处理

| 属性 | 值 |
|------|-----|
| **风险等级** | 低 |
| **位置** | `lite_wm.cpp:663-697` |

**问题描述**:
- RawEvent 数据未经严格校验
- 可能导致异常行为

## 内存安全问题

### 已使用安全函数

| 函数 | 用途 | 文件:行号 |
|------|------|-----------|
| `memcpy_s` | 内存拷贝 | `lite_wm.cpp:742` |
| `memset_s` | 内存设置 | `lite_wm.cpp:582` |

### 潜在问题

| 问题 | 位置 | 说明 |
|------|------|------|
| 空指针解引用 | `lite_wms.cpp:102-105` | `window == nullptr` 检查后直接返回 |
| 内存泄漏 | `lite_wms.cpp:106-108` | `objectStub` 分配但异常路径可能泄漏 |
| 整数溢出 | `lite_wm.cpp:729-730` | 尺寸比较可能溢出 |

## 安全建议总结

### 紧急修复（高风险）

1. **Screenshot 权限增强**
   - 添加 token/sid 校验
   - 记录审计日志
   - 考虑多因素验证

2. **窗口创建权限**
   - 添加 `CREATE_WINDOW` 权限
   - 限制非系统应用窗口能力

### 重要修复（中风险）

3. **参数边界校验**
   - 窗口 ID 有效性验证
   - 尺寸/坐标范围限制
   - Surface 大小限制

4. **资源限制**
   - 回调数量限制
   - 窗口数量限制（已有 32 上限）

### 常规优化（低风险）

5. **日志增强**
   - 安全相关操作记录详细日志
   - 失败操作记录上下文

6. **输入验证**
   - RawEvent 字段校验
   - 异常事件过滤

## 检查局限性声明

本次安全评审存在以下局限性：

| 局限性 | 说明 |
|--------|------|
| 未进行动态测试 | 仅基于静态代码分析 |
| 未审查依赖库 | SAMGR、IPC 等框架安全性未评估 |
| 未审查内核交互 | HDI 调用安全性未评估 |
| 未进行模糊测试 | 输入边界未充分测试 |

建议在生产环境部署前进行：
1. 完整的渗透测试
2. 模糊测试（Fuzz Testing）
3. 依赖库安全审计
