# 目录结构与模块职责 - Security Component Manager

> 目的：了解 security_component_manager 项目的代码组织、模块职责与文件布局

---

## 适用范围

本文档适用于：
- 需要定位代码文件的开发者
- 需要理解模块边界的架构师
- 需要修改特定模块的维护者

---

## 关键结论

1. **三层目录结构**：`frameworks/`（框架层）、`interfaces/`（接口层）、`services/`（服务层）
2. **核心业务逻辑在 `services/security_component_service/sa/sa_main/`**
3. **公共 SDK 在 `frameworks/inner_api/security_component/`**
4. **增强适配器在 `frameworks/enhance_adapter/`**，支持动态加载
5. **忽略 `test/` 目录，本文档仅描述生产代码**

---

## 完整目录树

```
security_component_manager/
├── frameworks/                          # 框架层（核心实现）
│   ├── common/                        # 公共工具和数据定义
│   │   ├── include/
│   │   │   ├── sec_comp_log.h        # 日志接口（基于 HiLog）
│   │   │   ├── sec_comp_rawdata.h   # 原始数据封装（IPC 传输）
│   │   │   └── sec_comp_tool.h     # 工具函数
│   │   └── src/
│   │       └── sec_comp_tool.cpp    # 字符串处理、JSON 工具
│   │
│   ├── enhance_adapter/               # 增强适配器（扩展层）
│   │   ├── include/
│   │   │   └── sec_comp_enhance_adapter.h
│   │   └── src/
│   │       └── sec_comp_enhance_adapter.cpp  # 动态加载增强库
│   │
│   ├── inner_api/                   # 内部 API 实现
│   │   ├── enhance_kits/            # 增强功能 kit
│   │   │   ├── include/
│   │   │   │   └── sec_comp_enhance_kit.h
│   │   │   └── src/
│   │   │       └── sec_comp_enhance_kit.cpp
│   │   └── security_component/      # 安全组件客户端实现
│   │       ├── include/
│   │       │   ├── sec_comp_client.h          # IPC 客户端单例
│   │       │   ├── sec_comp_death_recipient.h # 服务死亡监听
│   │       │   ├── sec_comp_dialog_callback.h   # 对话框回调接口
│   │       │   ├── sec_comp_caller_authorization.h # 调用者鉴权
│   │       │   └── sec_comp_load_callback.h    # SA 加载回调
│   │       └── src/
│   │           ├── sec_comp_kit.cpp       # 主 Kit API 实现
│   │           ├── sec_comp_client.cpp    # IPC 客户端实现
│   │           ├── sec_comp_death_recipient.cpp
│   │           ├── sec_comp_dialog_callback.cpp
│   │           ├── sec_comp_caller_authorization.cpp
│   │           ├── sec_comp_ui_register.cpp
│   │           └── sec_comp_load_callback.cpp
│   │
│   └── security_component/           # 具体组件实现
│       ├── include/
│       │   ├── sec_comp_base.h               # 组件基类
│       │   ├── sec_comp_click_event_parcel.h # 点击事件封装
│       │   ├── paste_button.h               # 粘贴按钮
│       │   ├── save_button.h                # 保存按钮
│       │   └── location_button.h            # 位置按钮
│       └── src/
│           ├── sec_comp_base.cpp
│           ├── sec_comp_click_event_parcel.cpp
│           ├── paste_button.cpp
│           ├── save_button.cpp
│           └── location_button.cpp
│
├── interfaces/                          # 公共接口层（头文件）
│   └── inner_api/
│       ├── security_component/          # 公共头文件（对外暴露）
│       │   ├── sec_comp_kit.h           # 主 API 类
│       │   ├── sec_comp_base.h          # 组件基类
│       │   ├── sec_comp_info.h          # 数据结构定义
│       │   ├── sec_comp_err.h           # 错误码定义
│       │   ├── sec_comp_enhance_kit.h  # 增强 Kit API
│       │   ├── sec_comp_enhance_adapter.h
│       │   ├── sec_comp_ui_register.h
│       │   ├── sec_comp_enhance_kit_c.h # C 接口
│       │   ├── i_sec_comp_probe.h        # 探针接口
│       │   ├── location_button.h
│       │   ├── paste_button.h
│       │   ├── save_button.h
│       │   └── security_component_service_ipc_interface_code.h
│       └── security_component_common/    # 服务公共头文件
│           ├── sec_comp_info_helper.h
│           ├── delay_exit_task.h
│           └── sec_event_handler.h
│
├── services/                          # 服务层
│   └── security_component_service/sa
│       ├── sa_main/                    # System Ability 实现
│       │   ├── sec_comp_service.cpp/h     # 主 SystemAbility 类
│       │   ├── sec_comp_manager.cpp/h     # 核心业务管理器
│       │   ├── sec_comp_entity.cpp/h       # 组件实体
│       │   ├── sec_comp_perm_manager.cpp/h # 权限管理器
│       │   ├── app_state_observer.cpp/h    # 应用状态监听
│       │   ├── first_use_dialog.cpp/h      # 首次使用对话框
│       │   ├── window_info_helper.cpp/h     # 窗口信息辅助
│       │   ├── delay_exit_task.cpp/h       # 延迟退出任务
│       │   ├── sec_comp_dialog_callback_proxy.cpp/h  # 对话框回调代理
│       │   ├── sec_comp_malicious_apps.cpp/h        # 恶意应用管理
│       │   ├── app_mgr_death_recipient.cpp/h          # AppMgr 死亡监听
│       │   └── sec_event_handler.cpp/h              # 事件处理器
│       │
│       ├── sa_profile/                 # SA 配置文件
│       │   ├── 3506.json             # SA ID 3506 配置
│       │   └── BUILD.gn
│       ├── ISecCompService.idl         # IDL 接口定义
│       ├── security_component_service.cfg # 服务启动配置
│       └── BUILD.gn
│
├── config/                            # 配置
│   └── BUILD.gn
│
├── figures/                           # 文档图片
├── bundle.json                        # 包配置
├── BUILD.gn                          # 根构建文件
├── security_component.gni             # GN 配置（feature flags）
├── hisysevent.yaml                    # HiSysEvent 配置
└── README.md                         # 项目说明
```

---

## 模块职责

### 1. `frameworks/common/` - 公共工具层

**职责**：提供跨模块使用的工具函数和数据定义

| 文件 | 职责 |
|------|------|
| `sec_comp_log.h` | 日志接口封装（基于 OpenHarmony HiLog） |
| `sec_comp_rawdata.h` | 原始数据封装（用于 IPC 传输，带长度和安全拷贝） |
| `sec_comp_tool.h/cpp` | 工具函数（字符串处理、JSON 解析等） |

**依赖方向**：被所有其他模块依赖

---

### 2. `frameworks/enhance_adapter/` - 增强适配器

**职责**：提供增强框架的动态加载能力，供厂商定制

| 文件 | 职责 |
|------|------|
| `sec_comp_enhance_adapter.cpp/h` | 动态加载 `libsecurity_component_client_enhance.z.so` 和 `libsecurity_component_service_enhance.z.so` |

**关键功能**：
- 运行时加载增强库（`dlopen` + `dlsym`）
- 调用增强接口（地址随机化、Challenge 验证等）
- 支持三种接口类型：
  - `SEC_COMP_ENHANCE_INPUT_INTERFACE` - 输入事件增强
  - `SEC_COMP_ENHANCE_SRV_INTERFACE` - 服务端增强
  - `SEC_COMP_ENHANCE_CLIENT_INTERFACE` - 客户端增强

**依赖方向**：被 `inner_api/` 和 `services/` 依赖

---

### 3. `frameworks/inner_api/enhance_kits/` - 增强 Kit

**职责**：提供应用层使用增强功能的 API

| 文件 | 职责 |
|------|------|
| `sec_comp_enhance_kit.h` | 增强 Kit API 定义 |
| `sec_comp_enhance_kit.cpp` | 增强 Kit 实现 |

**API 方法**：
- `InitClientEnhance()` - 初始化客户端增强
- `SetEnhanceCfg(cfg, cfgLen)` - 设置增强配置
- `GetPointerEventEnhanceData(data, dataLen, enhanceData, enHancedataLen)` - 获取点击事件增强数据

**依赖方向**：依赖 `enhance_adapter/`

---

### 4. `frameworks/inner_api/security_component/` - 安全组件客户端

**职责**：提供客户端 SDK，实现与服务端的 IPC 通信

| 文件 | 职责 |
|------|------|
| `sec_comp_kit.cpp/h` | 主 Kit API 实现（暴露给应用） |
| `sec_comp_client.cpp/h` | IPC 客户端单例，管理 Proxy 连接 |
| `sec_comp_death_recipient.cpp/h` | 服务死亡监听与重连 |
| `sec_comp_load_callback.cpp/h` | SA 加载回调（按需加载） |
| `sec_comp_caller_authorization.cpp/h` | 调用者鉴权（Token/UID 校验） |
| `sec_comp_ui_register.cpp/h` | UI 组件注册接口 |
| `sec_comp_dialog_callback.cpp/h` | 对话框回调实现 |

**依赖方向**：
- 依赖 `enhance_adapter/` - 增强功能
- 依赖 `common/` - 公共工具
- 依赖 `security_component/` - 组件基类

---

### 5. `frameworks/security_component/` - 组件实现

**职责**：实现具体的安全组件（Location、Paste、Save）

| 文件 | 职责 |
|------|------|
| `sec_comp_base.cpp/h` | 组件基类（位置、尺寸、样式验证） |
| `sec_comp_click_event_parcel.cpp/h` | 点击事件序列化/反序列化 |
| `paste_button.cpp/h` | 粘贴按钮实现 |
| `save_button.cpp/h` | 保存按钮实现 |
| `location_button.cpp/h` | 位置按钮实现 |

**依赖方向**：被 `inner_api/` 和 `services/` 依赖

---

### 6. `interfaces/inner_api/security_component/` - 公共接口

**职责**：定义对外暴露的头文件和数据结构

**关键头文件**：
- `sec_comp_kit.h` - 主 API 类
- `sec_comp_info.h` - 数据结构（SecCompInfo、SecCompType、SecCompClickEvent）
- `sec_comp_err.h` - 错误码定义
- `sec_comp_enhance_kit.h` - 增强 Kit API
- `sec_comp_enhance_kit_c.h` - C 接口
- `i_sec_comp_probe.h` - 探针接口

**依赖方向**：无，仅被包含

---

### 7. `services/security_component_service/sa/sa_main/` - 服务端核心

**职责**：实现 System Ability 的核心业务逻辑

| 文件 | 职责 |
|------|------|
| `sec_comp_service.cpp/h` | 主 SystemAbility 类（SA 3506） |
| `sec_comp_manager.cpp/h` | 核心业务管理器（组件注册、注销、点击处理） |
| `sec_comp_entity.cpp/h` | 组件实体（存储组件信息，验证点击事件） |
| `sec_comp_perm_manager.cpp/h` | 权限管理器（授予、撤销临时权限） |
| `app_state_observer.cpp/h` | 应用状态监听（前台/后台转换） |
| `first_use_dialog.cpp/h` | 首次使用对话框管理 |
| `window_info_helper.cpp/h` | 窗口信息查询（覆盖检测） |
| `delay_exit_task.cpp/h` | 延迟退出任务（服务生命周期管理） |
| `sec_comp_dialog_callback_proxy.cpp/h` | 对话框回调代理 |
| `sec_comp_malicious_apps.cpp/h` | 恶意应用黑名单管理 |
| `app_mgr_death_recipient.cpp/h` | AppMgr 死亡监听 |
| `sec_event_handler.cpp/h` | 事件处理器（异步任务） |

**依赖方向**：
- 依赖 `security_component/` - 组件实现
- 依赖 `enhance_adapter/` - 增强功能
- 依赖 `inner_api/security_component_common/` - 公共头文件

---

## 依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│                  interfaces/inner_api/               │
│              （公共头文件，无依赖）                       │
└───────────────────┬─────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌──────────────────┐    ┌──────────────────┐
│ frameworks/     │    │  services/      │
│ common/        │    │  security_      │
│（公共工具）     │    │  component_     │
└──────┬─────────┘    │  service/      │
       │              │  sa/sa_main/   │
       │              │（服务端核心）   │
       │              └──────────────────┘
       │                       │
       │                       ▼
       │              ┌──────────────────┐
       │              │  frameworks/     │
       │              │  enhance_       │
       │              │  adapter/       │
       │              │  （增强框架）     │
       │              └──────────────────┘
       │                       │
       └───────────────────┬───────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │  frameworks/     │
                  │  inner_api/     │
                  │  security_       │
                  │  component/     │
                  │  （客户端 SDK）    │
                  └──────────────────┘
```

---

## 相关跳转

- [架构说明](./02_Architecture.md) - 深入理解三层架构与数据流
- [对外 API](./03_Public_APIs.md) - 查看 C++ SDK API 参考

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)
