# 安全风险评审

> 攻击面、信任边界与修复建议

## 概述

本文档对 Launcher 应用进行安全风险评审，基于代码证据识别潜在的攻击面和可利用点。

## 评审范围

### 代码范围

| 目录 | 扫描内容 |
|------|---------|
| `product/*/` | MainAbility、页面组件 |
| `feature/*/` | 功能模块 |
| `common/` | 公共能力 |

### 排除范围

| 目录 | 排除原因 |
|------|----------|
| `test/` | 测试代码 |
| `docs/` | 文档目录 |
| `signature/` | 签名文件 |

## 信任边界

### 外部输入

| 输入源 | 说明 |
|--------|------|
| 系统 API (`Want` 参数) | 从系统收到的意图对象 |
| 用户输入 | 触摸、手势、按键 |
| 配置文件 | JSON 资源配置 |
| 数据库 | RDB 存储数据 |

### 内部组件

```
┌─────────────────────────────────────────────────────────────┐
│                      Launcher 进程                          │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  MainAbility (ServiceExtension)                     │  │
│  │  - 接收系统 onCreate/onRequest/onDestroy           │  │
│  └─────────────────────────────────────────────────────┘  │
│                          │                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  各功能模块 (feature)                                │  │
│  │  - ViewModel 更新                                  │  │
│  │  - UI 渲染                                         │  │
│  └─────────────────────────────────────────────────────┘  │
│                          │                                 │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  数据存储 (Rdb/Preferences)                        │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 攻击面清单

| 攻击面 | 说明 |
|--------|------|
| N-API | ❌ 无（纯 ArkTS 应用） |
| IPC | ✅ 与 AMS/BMS 通信 |
| 文件操作 | ✅ 数据库、首选项存储 |
| 网络 | ❌ 本地应用，无网络 API |
| 系统能力 | ✅ 权限范围内的系统 API |
| 用户输入 | ✅ 触摸、手势、按键 |

## 权限清单

### 已声明权限

Launcher 申请了以下系统权限（共 **11 项**）：

> **证据**: `product/phone/src/main/module.json5:46-80`

| 权限名 | 敏感级别 | 用途 |
|--------|----------|------|
| `ohos.permission.INJECT_INPUT_EVENT` | 高 | 注入输入事件 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 高 | 获取特权包信息 |
| `ohos.permission.INSTALL_BUNDLE` | 高 | 安装包 |
| `ohos.permission.LISTEN_BUNDLE_CHANGE` | 中 | 监听包变化 |
| `ohos.permission.MANAGE_MISSIONS` | 高 | 管理任务 |
| `ohos.permission.REQUIRE_FORM` | 中 | 请求卡片 |
| `ohos.permission.INPUT_MONITORING` | 高 | 输入监控 |
| `ohos.permission.NOTIFICATION_CONTROLLER` | 中 | 通知控制 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 高 | 管理安全设置 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 高 | 后台启动 |
| `ohos.permission.GET_WALLPAPER` | 低 | 获取壁纸 |

## 风险清单

### 风险 1：输入验证不足

**证据位置**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:47-51`

```typescript
onCreate(want: Want): void {
    this.context.area = 0;
    this.initLauncher();
}
```

**问题**: `want` 参数直接使用，未验证 `want` 中的字段

| 属性 | 说明 |
|------|------|
| 风险等级 | 中 |
| 影响 | 恶意 `want` 可能导致异常行为 |
| 修复建议 | 对 `want` 参数进行字段验证 |

---

### 风险 2：全局变量暴露

**证据位置**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:55`

```typescript
globalThis.desktopContext = this.context;
```

**问题**: `desktopContext` 暴露在全局，可能被其他模块滥用

| 属性 | 说明 |
|------|------|
| 风险等级 | 中 |
| 影响 | 其他模块可能访问敏感上下文 |
| 修复建议 | 限制访问范围，使用受控的接口 |

---

### 风险 3：卡片参数未校验

**证据位置**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:172-177`

```typescript
onRequest(want: Want, startId: number): void {
    if(want.action === FormConstants.ACTION_PUBLISH_FORM) {
        PageDesktopViewModel.getInstance().publishCardToDesktop(want.parameters);
    }
}
```

**问题**: `want.parameters` 直接传递给 `publishCardToDesktop`

| 属性 | 说明 |
|------|------|
| 风险等级 | 中 |
| 影响 | 恶意参数可能导致卡片功能异常 |
| 修复建议 | 验证参数格式、来源 |

---

### 风险 4：数据库查询参数

**证据**: `RdbStoreManager` 操作数据库时使用外部参数

| 属性 | 说明 |
|------|------|
| 风险等级 | 低 |
| 影响 | SQL 注入风险（如果使用 rawQuery） |
| 修复建议 | 使用参数化查询 |

---

### 风险 5：本地存储敏感信息

**证据**: `PreferencesHelper` 存储首选项

```typescript
PreferencesHelper.getInstance().initPreference(this.context);
```

| 属性 | 说明 |
|------|------|
| 风险等级 | 低 |
| 影响 | 首选项可能包含敏感配置 |
| 修复建议 | 敏感信息加密存储 |

---

## 安全机制评估

### 已实现的安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| 权限控制 | `module.json5` | 声明式权限 |
| 组件隔离 | Stage 模型 | ServiceExtension 隔离 |
| 输入过滤 | Log 类 | 日志输出过滤 |

### 缺失的安全机制

| 机制 | 优先级 | 说明 |
|------|--------|------|
| 参数校验 | 高 | Want、参数验证 |
| 访问控制 | 中 | globalThis 访问控制 |
| 数据加密 | 低 | 敏感数据加密 |

## 修复建议

### 高优先级

1. **Want 参数验证**
   ```typescript
   onCreate(want: Want): void {
       if (!want || !want.bundleName) {
           Log.showError(TAG, 'Invalid want parameters');
           return;
       }
       // ...
   }
   ```

2. **卡片参数校验**
   ```typescript
   publishCardToDesktop(params) {
       if (!params || !params.formId) {
           Log.showError(TAG, 'Invalid card parameters');
           return;
       }
       // ...
   }
   ```

### 中优先级

3. **globalThis 访问控制**
   - 考虑使用闭包或模块作用域替代全局变量
   - 提供受控的 getter/setter 接口

4. **敏感数据加密**
   - 首选项中的敏感信息加密存储
   - 使用系统级安全存储

## 相关文档

| 文档 | 链接 |
|------|------|
| 架构说明 | [02_Architecture.md](02_Architecture.md) |
| 构建配置 | [05_Build.md](05_Build.md) |
| OpenHarmony 安全指南 | [开发者官网](https://developer.harmonyos.com/cn/) |
