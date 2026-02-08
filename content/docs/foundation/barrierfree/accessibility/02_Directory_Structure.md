# 目录结构与模块职责

## 目的

本文档详细说明 Accessibility 子系统的目录结构、各模块职责和关键文件清单。

## 适用范围

- 需要了解代码组织的开发者
- 需要快速定位文件的开发者
- 新项目成员

## 关键结论

### 目录组织原则

Accessibility 子系统采用**分层模块化**组织：
1. **Common 层** - 通用组件和 IPC 接口
2. **Frameworks 层** - 三个 Kit 实现
3. **Interfaces 层** - 对外和内部接口
4. **Services 层** - 系统服务实现

### 核心模块

| 目录 | 模块名 | 职责 |
|------|---------|------|
| `frameworks/aafwk` | AAkit | 无障碍辅助能力 Kit |
| `frameworks/acfwk` | ACkit | 无障碍功能设定 Kit |
| `frameworks/asacfwk` | ASACkit | 无障碍能力客户端 Kit |
| `services/aams` | AccessibilityService | 无障碍系统服务 |
| `interfaces/innerkits` | Inner Kits | C/C++ 内部接口 |
| `interfaces/kits` | Kits | 对外 TS/JS/ANI/CJ 接口 |

---

## 详细内容

### 完整目录树

```
foundation/barrierfree/accessibility/
├── common/                           # 通用组件
│   ├── interface/                    # IPC 接口定义
│   │   ├── include/                  # Proxy/Stub/Parcel 头文件
│   │   └── src/                      # IPC 实现
│   ├── log/                          # Hilog 日志适配
│   └── etc/                          # 通用配置
│
├── frameworks/                        # 框架实现层
│   ├── aafwk/                        # AAkit 实现
│   ├── acfwk/                        # ACkit 实现
│   ├── asacfwk/                      # ASACkit 实现
│   └── common/                       # 通用数据类型实现
│
├── interfaces/                       # 接口层
│   ├── innerkits/                    # C/C++ 内部接口
│   │   ├── aafwk/                    # AAkit 头文件
│   │   ├── acfwk/                    # ACkit 头文件
│   │   ├── asacfwk/                  # ASACkit 头文件
│   │   └── common/                   # 通用类型头文件
│   └── kits/                         # 对外接口
│       ├── js/                       # TypeScript 声明
│       ├── napi/                     # N-API 实现
│       ├── ani/                      # ANI 实现
│       └── cj/                       # Cangjie FFI
│
├── services/                         # 系统服务层
│   ├── aams/                         # 无障碍服务主实现
│   ├── aams_ext/                     # 服务扩展
│   ├── multiuser/                    # 多用户支持
│   └── etc/                          # 服务配置
│
├── sa_profile/                       # SA 配置
├── resources/                        # 资源文件
└── figures/                          # 文档图片
```

### 各目录职责

#### Common 层

| 子目录 | 职责 | 关键文件 |
|-------|------|----------|
| `common/interface/` | IPC 接口定义（Proxy/Stub/Parcel） | `iaccessible_ability_client.h`, `iaccessible_ability_channel.h` |
| `common/log/` | Hilog 日志包装 | `hilog_wrapper.h` |
| `common/etc/` | 通用配置 | API 事件上报配置 |

**证据**: `common/interface/include/` 下的 Proxy/Stub 头文件

#### Frameworks 层

##### AAkit (aafwk)

**职责**: 无障碍辅助能力开发套件，为无障碍扩展提供运行时环境。

**关键组件**:
- `AccessibleAbilityClient` - 客户端实现
- `AccessibleAbilityChannelClient` - 通道客户端
- `AccessibilityUiTestAbility` - UI 测试能力
- `AccessibilityElementOperatorCallback` - 元素操作回调

**证据**: `frameworks/aafwk/include/accessible_ability_client_impl.h`

##### ACkit (acfwk)

**职责**: 无障碍功能设定开发套件，提供无障碍配置设置能力。

**关键组件**:
- `AccessibilityConfig` - 配置实现

**证据**: `frameworks/acfwk/include/accessibility_config_impl.h`

##### ASACkit (asacfwk)

**职责**: 无障碍能力客户端开发套件，为普通应用提供使用无障碍辅助服务的能力。

**关键组件**:
- `AccessibilitySystemAbilityClient` - 系统能力客户端
- `AccessibilityElementOperator` - 元素操作实现
- 规则检查引擎 (`src/rules/`)

**证据**: `frameworks/asacfwk/include/accessibility_system_ability_client_impl.h`

#### Interfaces 层

##### Innerkits

| 子目录 | 说明 | 关键头文件 |
|-------|------|------------|
| `aafwk/` | AAkit 对外接口 | `accessible_ability_client.h`, `accessible_ability_listener.h` |
| `acfwk/` | ACkit 对外接口 | `accessibility_config.h` |
| `asacfwk/` | ASACkit 对外接口 | `accessibility_system_ability_client.h`, `accessibility_element_operator.h` |
| `common/` | 通用类型定义 | `accessibility_element_info.h`, `accessibility_event_info.h` |

**证据**: `interfaces/innerkits/*/include/` 下的头文件

##### Kits

| 子目录 | 技术栈 | 说明 |
|-------|---------|------|
| `js/` | TypeScript | .d.ts 类型定义 |
| `napi/` | C++ N-API | 旧接口实现 |
| `ani/` | ANI | 新接口（ArkTS Native Interface） |
| `cj/` | Cangjie | Cangjie 语言绑定 |

**证据**: `interfaces/kits/js/*.d.ts`, `interfaces/kits/napi/*/src/*.cpp`

#### Services 层

##### AAMS 核心模块

| 功能模块 | 关键文件 |
|----------|----------|
| 服务主入口 | `accessible_ability_manager_service.h/cpp` |
| 连接管理 | `accessible_ability_connection.h/cpp`, `accessible_ability_channel.h/cpp` |
| 窗口管理 | `accessibility_window_manager.h/cpp` |
| 手势/触摸 | `accessibility_touch_exploration.h`, 各种手势类 |
| 放大镜 | `magnification_manager.h/cpp`, 各种放大镜类 |
| 鼠标/键盘 | `accessibility_mouse_key.h/cpp`, `accessibility_keyevent_filter.h/cpp` |
| 设置/配置 | `accessibility_settings.h/cpp`, `accessibility_settings_config.h/cpp` |
| 快捷方式 | `accessibility_short_key.h/cpp` |
| 事件处理 | `accessibility_event_transmission.h/cpp` |

**证据**: `services/aams/include/` 下的头文件

##### AAMS_EXT

**职责**: 扩展服务（放大镜窗口、菜单等）

**关键文件**: `magnification_window.h/cpp`, `magnification_menu.h/cpp`

**证据**: `services/aams_ext/include/`

### 关键文件清单

#### 核心接口定义

| 文件路径 | 说明 |
|----------|------|
| `interfaces/innerkits/common/include/accessibility_def.h` | 基础类型定义 |
| `interfaces/innerkits/common/include/accessibility_element_info.h` | 元素信息结构 |
| `interfaces/innerkits/common/include/accessibility_event_info.h` | 事件信息结构 |
| `interfaces/innerkits/asacfwk/include/accessibility_system_ability_client.h` | 系统能力客户端接口 |
| `interfaces/innerkits/aafwk/include/accessible_ability_client.h` | 无障碍能力客户端接口 |
| `interfaces/innerkits/acfwk/include/accessibility_config.h` | 配置接口 |

**证据**: 以上文件均在对应路径存在

#### 服务核心

| 文件路径 | 说明 |
|----------|------|
| `services/aams/include/accessible_ability_manager_service.h` | 服务主类 |
| `services/aams/include/accessibility_window_manager.h` | 窗口管理 |
| `services/aams/include/magnification_manager.h` | 放大镜管理 |
| `services/aams/include/accessibility_touch_exploration.h` | 触摸探索 |

**证据**: 以上文件均在对应路径存在

#### IPC 接口

| 文件路径 | 说明 |
|----------|------|
| `common/interface/include/iaccessible_ability_client.h` | 客户端接口定义 |
| `common/interface/include/iaccessible_ability_channel.h` | 通道接口定义 |
| `common/interface/include/iaccessibility_element_operator.h` | 元素操作接口 |

**证据**: 以上文件均在 `common/interface/include/` 存在

---

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](03_Architecture.md)
- [内部 API](05_Inner_API.md)

---

最后更新: 2026-02-06
