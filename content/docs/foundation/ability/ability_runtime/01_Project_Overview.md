# 项目定位与核心能力

## 项目定位

**ability_runtime** 是 OpenHarmony 元能力子系统的核心运行时组件，位于应用框架层，为上层应用提供 Ability 组件的运行环境和管理能力。

### 核心定位

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                          │
├─────────────────────────────────────────────────────────────┤
│  应用层                                                      │
│  └── 开发者编写的 Ability 应用（UIAbility, ExtensionAbility）│
├─────────────────────────────────────────────────────────────┤
│  框架层（ability_runtime）                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  运行时环境：Ability 生命周期管理、组件调度           │   │
│  │  语言绑定：JS/NAPI, ETS/ArkTS, CJ FFI               │   │
│  │  系统服务：AbilityManager, AppManager, UriPermMgr   │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  系统服务层                                                  │
│  ├── BundleManagerService（包管理）                         │
│  ├── AppManagerService（应用管理，由 ability_runtime 实现） │
│  ├── IPC/RPC（进程间通信）                                  │
│  └── ...                                                   │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

### 1. Ability 生命周期管理

**能力描述**：统一管理所有 Ability 组件的创建、启动、暂停、恢复、停止等生命周期状态转换。

**支持的能力类型**：
- **UIAbility**：具有用户界面的能力组件
- **ExtensionAbility**：扩展能力组件，支持多种场景
  - ServiceExtensionAbility：后台服务扩展
  - UIExtensionAbility：UI 扩展组件
  - FormExtensionAbility：卡片扩展
  - DataShareExtensionAbility：数据共享扩展
  - PhotoEditorExtensionAbility：图片编辑扩展
  - ...

**代码证据**：
- 生命周期调度：`services/abilitymgr/src/lifecycle_deal/`
- 组件类型定义：`interfaces/kits/native/ability/native/` 目录下各扩展头文件

### 2. 组件调度

**能力描述**：协调各 Ability 之间的运行关系，支持跨应用进程间和同一进程内的组件调用。

**核心功能**：
- 启动/停止 Ability
- Ability 连接管理（ConnectAbility）
- 任务栈管理（MissionStack）
- 能力隐式启动

**代码证据**：
- 启动调度：`services/abilitymgr/src/start_ability_handler/`
- 连接管理：`services/abilitymgr/src/ability_connect_manager.cpp`
- 任务管理：`services/abilitymgr/src/mission/`

### 3. 进程管理

**能力描述**：与应用管理服务协作，管理应用进程的创建、销毁和状态转换。

**核心功能**：
- 应用进程生命周期
- 进程优先级管理
- 父子进程通信

**代码证据**：
- 进程管理：`services/appmgr/src/`
- 子进程支持：`frameworks/native/child_process/`

### 4. 多语言运行时支持

**能力描述**：为不同编程语言提供 Ability 开发接口。

| 语言/运行时 | 接口类型 | 代码位置 |
|------------|---------|---------|
| JavaScript | N-API | `frameworks/js/napi/` |
| ArkTS | ANI | `frameworks/ets/ani/` |
| CJ (云语言) | FFI | `frameworks/cj/` |
| C/C++ | Native API | `frameworks/native/` |

## 关键概念

### 1. Want

Want 是 OpenHarmony 中用于描述操作意图的对象。

```cpp
// want.h 定义
struct Want {
    std::string deviceId;      // 设备 ID
    std::string bundleName;   // Bundle 名称
    std::string abilityName;  // Ability 名称
    std::string uri;          // URI
    std::string action;       // 动作
    // ... 更多字段
};
```

**代码证据**：`interfaces/kits/native/ability/ability_runtime/want.h`

### 2. Ability 生命周期状态

| 状态 | 说明 | 触发时机 |
|------|------|---------|
| INITIAL | 初始状态 | 能力对象创建 |
| INACTIVE | 不可交互 | 刚启动或后台 |
| ACTIVATED | 可交互 | 获得焦点 |
| BACKGROUND | 后台运行 | 切换到后台 |
| TERMINATED | 已终止 | 能力销毁 |

**代码证据**：
- 状态枚举：`frameworks/native/ability/native/include/ability_lifecycle.h`

### 3. CallBack 与 Promise

OpenHarmony N-API 支持两种异步模式：

```typescript
// Callback 模式
abilityManager.startAbility(want, (err) => {
    if (err) {
        console.error('启动失败');
    } else {
        console.log('启动成功');
    }
});

// Promise 模式（推荐）
try {
    await abilityManager.startAbility(want);
    console.log('启动成功');
} catch (err) {
    console.error('启动失败:', err);
}
```

## 系统服务

### AbilityManagerService

**服务 ID**：`ABILITY_MGR_SERVICE_ID` (3701)

**主要职责**：
- 管理 Ability 生命周期
- 处理组件启动/停止请求
- 管理组件连接
- 任务栈管理

**代码证据**：
- 服务定义：`services/abilitymgr/include/ability_manager_service.h`
- 服务实现：`services/abilitymgr/src/ability_manager_service.cpp`

### AppManagerService

**服务 ID**：`APP_MGR_SERVICE_ID` (1201)

**主要职责**：
- 应用进程生命周期管理
- 应用状态跟踪
- 与 AppSpawn 协作应用孵化

**代码证据**：
- 服务定义：`services/appmgr/include/app_mgr_service.h`
- 服务实现：`services/appmgr/src/app_mgr_service.cpp`

## 特性开关

ability_runtime 支持多种特性开关，可在编译时配置：

| 特性开关 | 默认值 | 说明 |
|---------|--------|------|
| `ability_runtime_auto_fill` | true | 自动填充扩展支持 |
| `ability_runtime_child_process` | true | 子进程支持 |
| `ability_runtime_graphics` | true | 图形依赖特性 |
| `ability_runtime_power` | true | 电源管理特性 |
| `ability_runtime_ui_service_extension` | true | UI 服务扩展 |
| `ability_runtime_check_internet_permission` | false | 网络权限检查 |

**代码证据**：`ability_runtime.gni` 第 93-260 行

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [N-API 参考](04_NAPI_Reference.md)
