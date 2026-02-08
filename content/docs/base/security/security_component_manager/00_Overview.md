# 项目概览 - Security Component Manager

> 目的：5 分钟了解 Security Component Manager 项目的核心概念、定位与边界

---

## 适用范围

本文档适用于：
- 新加入 OpenHarmony 安全子系统的开发者
- 需要集成安全组件的应用开发者
- 需要理解临时权限机制的系统集成者
- 进行安全审计的安全研究人员

---

## 关键结论

1. **Security Component Manager 是 OpenHarmony 的三层安全架构中间层**，负责管理安全组件的注册、验证和临时权限授予
2. **本项目是纯 C++ System Ability 服务（SA ID: 3506）**，无 N-API 绑定
3. **支持三种安全组件**：LocationButton（位置）、PasteButton（粘贴）、SaveButton（保存）
4. **核心机制**：用户点击安全组件 → 验证有效性 → 授予临时权限 → 应用访问敏感数据 → 应用后台撤销权限
5. **扩展性**：通过增强框架（Enhance Framework）供厂商定制安全能力

---

## 项目定位

### 在 OpenHarmony 生态中的位置

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 1: Ace Engine (arkui_ace_engine 仓库)          │
│ • PasteButton, SaveButton, LocationButton 组件            │
│ • ArkTS 绑定层（N-API 在此处）                    │
└─────────────────────────────────────────────────────────────┘
                         │ IPC 调用
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ Layer 2: Security Component Manager (本项目)              │
│ • System Ability (SA ID: 3506)                        │
│ • 组件注册、验证、临时权限管理                          │
│ • 增强框架（厂商定制）                               │
└─────────────────────────────────────────────────────────────┘
                         │ 对话框请求
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ Layer 3: Permission Manager (独立应用)                  │
│ • 首次使用确认对话框                                  │
│ • 用户授权确认/拒绝                                   │
└─────────────────────────────────────────────────────────────┘
```

### 与传统权限申请的对比

| 特性 | 传统权限申请 | 安全组件 |
|------|------------|---------|
| 授权时机 | 安装时/首次使用时弹窗 | 用户点击组件时 |
| 授权范围 | 永久授权 | 临时授权 |
| 用户打扰 | 多次弹窗 | 最小化打扰 |
| 粒度 | 应用级 | 组件级（按钮级） |
| 可撤销性 | 需用户手动撤销 | 应用后台自动撤销 |

---

## 核心能力

### 1. 安全组件管理

- **组件注册**：应用启动时注册安全组件信息（位置、尺寸、样式）
- **组件更新**：动态更新组件属性
- **组件注销**：应用销毁组件时清理
- **组件验证**：确保组件可见、可识别、未被覆盖

**证据路径**：
- 注册接口：`services/security_component_service/sa/sa_main/sec_comp_service.h:48`
- 管理器：`services/security_component_service/sa/sa_main/sec_comp_manager.h:50`

### 2. 临时权限授予

**位置按钮（LocationButton）**：
- 权限：`ohos.permission.LOCATION`、`ohos.permission.APPROXIMATELY_LOCATION`
- 授予时机：用户点击组件
- 撤销时机：应用进入后台 10 秒后

**粘贴按钮（PasteButton）**：
- 权限：`ohos.permission.SECURE_PASTE`
- 授予时机：用户点击组件
- 撤销时机：应用进入后台 10 秒后

**保存按钮（SaveButton）**：
- 权限：无标准权限（使用内部引用计数）
- 授予时机：首次使用时用户确认对话框
- 撤销时机：单次保存操作 60 秒后

**证据路径**：
- 权限管理器：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.h:29`
- 授予函数：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:279`

### 3. 点击事件验证

验证步骤：
1. **窗口覆盖检查**：确保组件未被其他窗口遮挡
2. **坐标范围检查**：触摸点在组件矩形内
3. **时间戳检查**：点击事件在有效时间内（5000ms）
4. **键盘事件检查**：仅允许 SPACE、ENTER、NUMPAD_ENTER
5. **增强数据验证**：Challenge 值/HMAC 校验（通过增强框架）

**证据路径**：
- 验证函数：`services/security_component_service/sa/sa_main/sec_comp_entity.cpp:124`

### 4. 应用生命周期监听

监听应用前台/后台转换：
- **前台 → 后台**：启动延迟撤销任务（10 秒后撤销位置/粘贴权限）
- **后台 → 前台**：取消延迟撤销任务（避免误撤销）
- **应用死亡**：清理所有组件和权限

**证据路径**：
- 观察者：`services/security_component_service/sa/sa_main/app_state_observer.h:18`

### 5. 增强框架（Enhance Framework）

**目的**：供厂商定制安全能力，防止恶意应用绕过

**可扩展能力**：
- 地址随机化
- Challenge 值验证
- UI 框架回调验证
- 调用者地址验证
- 组件防覆盖保护
- 真实点击事件验证

**加载方式**：运行时动态加载 `.z.so` 库
- 客户端：`libsecurity_component_client_enhance.z.so`
- 服务端：`libsecurity_component_service_enhance.z.so`

**证据路径**：
- 适配器：`frameworks/enhance_adapter/src/sec_comp_enhance_adapter.cpp:49`

---

## 运行环境

### 系统要求

| 要求 | 版本/配置 |
|------|----------|
| OpenHarmony 版本 | 标准系统（`is_standard_system = true`） |
| SAMGR | System Ability Manager（支持按需加载 SA） |
| IPC Core | OpenHarmony IPC 框架 |
| Access Token | OpenHarmony 权限管理框架 |
| Window Manager | 支持窗口信息查询 |
| FFRT | Foundations Framework for Runtime（异步任务） |

### 资源消耗

根据 `bundle.json` 配置：

| 资源 | 占用 |
|------|------|
| ROM | 2048 KB |
| RAM | 5102 KB |

---

## 关键概念

### System Ability (SA)

OpenHarmony 的系统服务机制，提供跨进程能力。

- **SA ID**: 3506
- **按需加载**：首次调用时启动，无活动组件时延迟退出
- **死亡通知**：服务异常死亡时，客户端接收通知并重连

**证据路径**：
- SA 注册：`services/security_component_service/sa/sa_main/sec_comp_service.cpp:45`

### 临时权限（Temporary Permission）

授予应用的短期权限，访问敏感数据后自动撤销。

| 组件类型 | 权限名称 | 有效期 |
|---------|----------|--------|
| LocationButton | `ohos.permission.LOCATION` | 前台 + 10 秒后台延迟 |
| LocationButton | `ohos.permission.APPROXIMATELY_LOCATION` | 前台 + 10 秒后台延迟 |
| PasteButton | `ohos.permission.SECURE_PASTE` | 前台 + 10 秒后台延迟 |
| SaveButton | 内部引用计数 | 单次操作 60 秒 |

### 安全组件（Security Component）

Ace Engine 提供的 ArkUI 组件，具有以下特点：

- **明确标识**：必须有图标和文字，用户可识别
- **可见性要求**：不能被覆盖、遮挡、或设置透明
- **点击验证**：点击事件必须真实有效
- **生命周期管理**：注册 → 显示 → 点击 → 注销

---

## 相关跳转

- [目录结构](./01_Directory_Structure.md) - 了解代码组织
- [架构说明](./02_Architecture.md) - 深入理解三层架构
- [对外 API](./03_Public_APIs.md) - 查看 C++ SDK API

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)
