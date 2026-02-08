# 架构说明 - Security Component Manager

> 目的：理解 Security Component Manager 的三层架构、数据流与线程模型

---

## 适用范围

本文档适用于：
- 需要理解系统架构的开发者
- 需要调试跨进程通信的开发者
- 需要优化性能的系统集成者

---

## 关键结论

1. **三层架构**：Ace UI 层（Layer 1）→ Security Component Service（Layer 2）→ Permission Manager（Layer 3）
2. **IPC 通信**：层间通过 OpenHarmony IPC 框架（Binder）通信
3. **临时权限机制**：点击组件时授予，应用后台时撤销
4. **异步任务处理**：使用 ffrt（Foundations Framework for Runtime）处理延迟任务
5. **增强框架**：可插拔的厂商定制能力，通过 dlopen 动态加载

---

## 三层架构图

```mermaid
graph TB
    subgraph "Layer 1: Ace Engine（arkui_ace_engine）"
        A1[PasteButton<br/>ArkTS 组件]
        A2[SaveButton<br/>ArkTS 组件]
        A3[LocationButton<br/>ArkTS 组件]
    end

    subgraph "Layer 2: Security Component Manager（本项目）"
        B1[SecCompService<br/>System Ability<br/>SA ID: 3506]
        B2[SecCompManager<br/>核心业务管理器]
        B3[SecCompPermManager<br/>权限管理器]
        B4[SecCompEntity<br/>组件实体]
        B5[AppStateObserver<br/>应用状态监听]
    end

    subgraph "Layer 3: Permission Manager<br/>（独立应用）"
        C1[Permission Manager 应用<br/>首次使用对话框]
        C2[AccessToken 服务<br/>权限授予/撤销]
    end

    %% 客户端到服务端的通信
    A1 -.->|IPC: 注册/更新/注销<br/>点击事件| B1
    A2 -.->|IPC: 注册/更新/注销<br/>点击事件| B1
    A3 -.->|IPC: 注册/更新/注销<br/>点击事件| B1

    %% 服务端内部通信
    B1 -->|IPC 处理分发| B2
    B2 -->|组件管理| B4
    B2 -->|权限管理| B3
    B2 -->|生命周期监听| B5

    %% 服务端到 Permission Manager 的通信
    B3 -.->|IPC: 授予/撤销权限| C2
    B1 -.->|Ability: 启动对话框| C1

    %% 用户交互
    C1 -->|用户确认| B3

    %% 应用状态监听
    B5 -.->|前台/后台事件| B3

    classDef clientAPI fill:#e1f5fe,stroke:#333,stroke-width:2px
    classDef serviceImpl fill:#fff4e6,stroke:#333,stroke-width:2px
    classDef externalApp fill:#90caf9,stroke:#333,stroke-width:2px

    class A1,A2,A3 clientAPI
    class B1,B2,B3,B4,B5 serviceImpl
    class C1,C2 externalApp
```

---

## 组件注册流程

```mermaid
sequenceDiagram
    participant App as 应用（ArkTS）
    participant Ace as Ace Engine
    participant SDK as SecCompKit
    participant Client as SecCompClient
    participant Service as SecCompService
    participant Manager as SecCompManager
    participant Entity as SecCompEntity

    App->>Ace: 1. 初始化 PasteButton
    Ace->>SDK: 2. 调用 RegisterSecurityComponent()
    SDK->>Client: 3. 获取 Proxy
    Client->>Service: 4. IPC: RegisterSecurityComponent
    Service->>Manager: 5. RegisterSecurityComponent()
    Manager->>Manager: 6. 创建 scId
    Manager->>Entity: 7. 创建 SecCompEntity
    Entity->>Entity: 8. 验证组件信息<br/>（位置、尺寸、类型）
    Entity-->>Manager: 9. 返回验证结果
    Manager-->>Service: 10. 返回 scId
    Service-->>Client: 11. 返回 scId（rawdata）
    Client-->>SDK: 12. 返回 scId
    SDK-->>Ace: 13. 返回 scId
    Ace-->>App: 14. 组件注册成功，显示按钮

    Note over Service,Manager: 存储到 componentMap_<br/>key: pid<br/>value: ProcessCompInfos
```

**关键验证点**：
- **类型验证**：组件类型是否有效（`IsComponentTypeValid`）
- **尺寸验证**：字体大小、图标大小、padding 是否符合最小值
- **信息完整性**：JSON 解析是否成功

**证据路径**：
- 注册入口：`services/security_component_service/sa/sa_main/sec_comp_service.h:48`
- 验证逻辑：`services/security_component_service/sa/sa_main/sec_comp_entity.cpp:124`

---

## 点击事件与权限授予流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant App as 应用
    participant SDK as SecCompKit
    participant Service as SecCompService
    participant Manager as SecCompManager
    participant PermMgr as SecCompPermManager
    participant Access as AccessToken 服务
    participant Dialog as Permission Manager 对话框

    User->>App: 1. 点击安全组件
    App->>SDK: 2. 调用 ReportSecurityComponentClickEvent()
    SDK->>Service: 3. IPC: ReportSecurityComponentClickEvent
    Service->>Manager: 4. ReportSecurityComponentClickEvent()
    Manager->>Manager: 5. 获取 SecCompEntity
    Manager->>Manager: 6. CheckClickSecurityComponentInfo()

    alt 点击事件验证
        Manager->>Manager: 6a. 检查窗口覆盖
        Manager->>Manager: 6b. 检查坐标范围
        Manager->>Manager: 6c. 检查时间戳
        Manager->>Manager: 6d. 检查键盘事件
    else 验证失败
        Manager-->>Service: 返回错误码
        Service-->>SDK: 返回错误码
        SDK-->>App: 返回错误码
    end

    par SaveButton 首次使用
        Manager->>Dialog: 7. 启动首次使用对话框
        Dialog->>User: 显示授权确认
        User->>Dialog: 8. 确认/拒绝
        Dialog-->>Manager: 9. OnDialogClosed(result)
    end

    Manager->>PermMgr: 10. GrantTempPermission()

    alt LocationButton / PasteButton
        PermMgr->>Access: 10a. GrantPermission(LOCATION)
        Access-->>PermMgr: 授予成功
    else SaveButton
        PermMgr->>PermMgr: 10b. 增加引用计数
        PermMgr->>PermMgr: 10c. 启动 60 秒延迟撤销
    end

    PermMgr-->>Manager: 11. 授予成功
    Manager-->>Service: 12. 返回成功
    Service-->>SDK: 13. 返回成功
    SDK-->>App: 14. 返回成功
    App->>App: 15. 访问敏感数据（位置/粘贴/保存）
```

**点击事件验证细节**：
1. **窗口覆盖检查**：`window_info_helper.cpp:WindowInfoHelper::CheckWindowCover()`
2. **坐标范围检查**：`sec_comp_entity.cpp:SecCompEntity::CheckPointEvent()`
3. **时间戳检查**：点击事件时间戳在 5000ms 内
4. **键盘事件检查**：仅允许 SPACE (2050)、ENTER (2054)、NUMPAD_ENTER (2119)
5. **增强数据验证**：`sec_comp_enhance_adapter.cpp:CheckExtraInfo()`

**证据路径**：
- 点击验证：`services/security_component_service/sa/sa_main/sec_comp_entity.cpp:124-174`
- 权限授予：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:279-323`

---

## 权限撤销流程

```mermaid
sequenceDiagram
    participant App as 应用
    participant Observer as AppStateObserver
    participant PermMgr as SecCompPermManager
    participant Handler as SecEventHandler
    participant Access as AccessToken 服务

    Note over App,Handler: 应用从前台切换到后台

    App->>Observer: 1. 应用进入后台
    Observer->>PermMgr: 2. NotifyProcessBackground(pid)
    PermMgr->>Handler: 3. RevokeAppPermisionsDelayed()
    Handler->>Handler: 4. Post delayed task (10s)

    Note over Handler: 延迟 10 秒

    Handler->>PermMgr: 5. RevokeAppPermisionsImmediately()
    PermMgr->>Access: 6. RevokePermission()
    Access-->>PermMgr: 撤销成功
    PermMgr->>PermMgr: 7. 清空 applySaveCountMap_
    PermMgr-->>App: 8. 权限已撤销

    Note over App,Handler: 应用从后台切换回前台

    App->>Observer: 1. 应用进入前台
    Observer->>PermMgr: 2. NotifyProcessForeground(pid)
    PermMgr->>Handler: 3. CancelAppRevokingPermisions()
    Handler->>Handler: 4. Remove delayed task
    Handler-->>App: 5. 撤销已取消
```

**撤销时机对比**：

| 组件类型 | 撤销触发 | 延迟时间 |
|---------|----------|---------|
| LocationButton | 应用进入后台 | 10 秒 |
| PasteButton | 应用进入后台 | 10 秒 |
| SaveButton | 单次操作完成 | 60 秒 |

**证据路径**：
- 应用状态监听：`services/security_component_service/sa/sa_main/app_state_observer.cpp`
- 延迟撤销：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:43-59`

---

## 模块依赖关系

```mermaid
graph LR
    subgraph "客户端 SDK"
        A[sec_comp_kit]
        B[sec_comp_client]
        C[sec_comp_caller_authorization]
    end

    subgraph "框架层"
        D[sec_comp_base<br/>paste_button<br/>save_button<br/>location_button]
        E[sec_comp_enhance_adapter]
        F[sec_comp_enhance_kit]
    end

    subgraph "服务端"
        G[sec_comp_service]
        H[sec_comp_manager]
        I[sec_comp_perm_manager]
        J[sec_comp_entity]
        K[app_state_observer]
    end

    A -->|调用| B
    A -->|使用| D
    B -->|鉴权| C
    A -->|增强| F
    F -->|加载| E

    G -->|分发| H
    H -->|管理| J
    H -->|调用| I
    H -->|监听| K

    classDef sdk fill:#e1f5fe,stroke:#333
    classDef framework fill:#fff4e6,stroke:#333
    classDef service fill:#ff9800,stroke:#333

    class A,B,C sdk
    class D,E,F framework
    class G,H,I,J,K service
```

**依赖方向**（避免环）：
- SDK → Framework → Service（单向依赖）
- Service 内部：Service → Manager → PermManager
- 无循环依赖

---

## 线程模型

### 事件处理线程

**SecEventHandler** (`services/security_component_service/sa/sa_main/sec_event_handler.cpp`)：
- 基于 `AppExecFwk::EventRunner`
- 单线程事件处理器
- 处理延迟任务（权限撤销、对话框等待）

**证据路径**：
- 事件处理器：`services/security_component_service/sa/sa_main/sec_event_handler.h:27`
- 初始化：`services/security_component_service/sa/sa_main/sec_comp_manager.cpp:98-99`

### FFRT 任务队列

**Foundations Framework for Runtime (ffrt)**：
- 异步任务执行
- 支持延迟任务
- 用于权限撤销、对话框回调

**证据路径**：
- ffrt 使用：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:17`
- 延迟任务：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:47-58`

### 线程安全

**互斥锁保护**：
- `componentInfoLock_` - `ffrt::shared_mutex`（组件信息）
- `scIdMtx_` - `std::mutex`（scId 分配）
- `mediaLibMutex_` - `std::mutex`（媒体库 Token）
- `mutex_` - `std::mutex`（权限管理器）
- `grantMtx_` - `std::mutex`（权限映射）

**证据路径**：
- 锁定义：`services/security_component_service/sa/sa_main/sec_comp_manager.h:90-95`

---

## 资源生命周期

### 组件生命周期

```
创建（RegisterSecurityComponent）
    ↓
存储到 componentMap_（key: pid, value: ProcessCompInfos）
    ↓
使用（点击事件）
    ↓
注销（UnregisterSecurityComponent）
    ↓
从 componentMap_ 删除
```

### 权限生命周期

```
授予（GrantTempPermission）
    ↓
存储到 applySaveCountMap_ / saveTaskDequeMap_
    ↓
应用使用
    ↓
应用进入后台
    ↓
启动延迟撤销任务（10s 或 60s）
    ↓
撤销（RevokeAppPermisionsImmediately）
    ↓
从 applySaveCountMap_ / saveTaskDequeMap_ 清空
```

### 服务生命周期

```
加载（按需加载：LoadSystemAbility）
    ↓
启动（OnStart）
    ↓
添加应用状态观察者
    ↓
启动增强服务（StartEnhanceService）
    ↓
无活动组件 + 延迟退出 → ExitSaProcess
    ↓
停止（OnStop）
    ↓
移除应用状态观察者
    ↓
退出增强服务（ExitEnhanceService）
```

---

## 增强框架机制

```mermaid
graph TB
    subgraph "客户端"
        C1[SecCompEnhanceKit]
        C2[SecCompEnhanceAdapter]
        C3[libsecurity_component_client_enhance.z.so]
    end

    subgraph "服务端"
        S1[SecCompService]
        S2[SecCompEnhanceAdapter]
        S3[libsecurity_component_service_enhance.z.so]
    end

    C1 -->|InitClientEnhance| C2
    C2 -->|dlopen + dlsym| C3
    C3 -.->|实现厂商定制| C2

    S1 -->|启动增强服务| S2
    S2 -->|dlopen + dlsym| S3
    S3 -.->|实现厂商定制| S2

    classDef client fill:#e1f5fe,stroke:#333
    classDef service fill:#ff9800,stroke:#333
    classDef vendorLib fill:#90caf9,stroke:#333,stroke-dasharray: 5 5

    class C1,C2 client
    class S1,S2 service
    class C3,S3 vendorLib
```

**增强接口类型**：

| 接口类型 | 加载的库 | 主要功能 |
|---------|----------|---------|
| `SEC_COMP_ENHANCE_INPUT_INTERFACE` | `libsecurity_component_client_enhance.z.so` | 输入事件增强（点击事件 HMAC） |
| `SEC_COMP_ENHANCE_SRV_INTERFACE` | `libsecurity_component_service_enhance.z.so` | 服务端增强（组件验证、Challenge 检查） |
| `SEC_COMP_ENHANCE_CLIENT_INTERFACE` | `libsecurity_component_client_enhance.z.so` | 客户端增强（地址随机化、调用者验证） |

**证据路径**：
- 动态加载：`frameworks/enhance_adapter/src/sec_comp_enhance_adapter.cpp:49-91`

---

## 相关跳转

- [对外 API](./03_Public_APIs.md) - 查看 C++ SDK API
- [内部 API](./04_Internal_APIs.md) - 查看内部模块接口
- [目录结构](./01_Directory_Structure.md) - 了解文件组织

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)
