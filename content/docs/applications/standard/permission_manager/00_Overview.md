# PermissionManager 项目概览

> **目的**: 介绍PermissionManager项目的定位、边界、核心能力和运行环境  
> **适用范围**: OpenHarmony系统开发者、安全研究人员  
> **最后更新**: 2026-02-05

---

## 项目定位

PermissionManager是OpenHarmony系统的**预置权限管理应用**，提供以下核心功能：

1. **权限请求对话框** - 当应用请求用户授权权限时显示弹窗
2. **权限管理设置** - 通过 Settings → Privacy → Permission Manager 访问
3. **权限使用记录** - 查看应用权限使用历史
4. **全局开关管理** - 管理系统级权限开关状态

### 项目边界

| 范围内 | 范围外 |
|--------|--------|
| 权限UI展示和交互 | 权限策略决策逻辑（AT权限服务） |
| 用户授权/拒绝操作 | 权限状态持久化存储 |
| 应用包信息展示 | 应用安装/卸载逻辑 |
| 权限使用记录展示 | 权限使用记录收集 |

### 相关仓库

- **security_access_token**: 底层权限访问令牌管理服务
- **applications/standard/permission_manager**: 本项目（UI层）

---

## 核心能力

### 1. 权限请求对话框 (ServiceExtAbility)

当应用调用 `abilityAccessCtrl.requestPermissionsFromUser()` 时触发：

```typescript
// 应用侧调用示例
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
let atManager = abilityAccessCtrl.createAtManager();
atManager.requestPermissionsFromUser(this.context, ['ohos.permission.CAMERA'], (err, data) => {
    console.info('授权结果:' + data.authResults);
});
```

**处理流程**:
1. 系统通过IPC调用启动 `ServiceExtAbility`
2. 解析调用方应用信息和请求的权限列表
3. 根据权限类型分组，加载对应的策略
4. 显示权限请求对话框
5. 用户选择后，通过IPC返回授权结果

### 2. 权限管理设置 (MainAbility)

提供两个维度的权限管理视图：

**按权限维度 (Permission维度)**:
- 展示所有用户授权权限(user_grant)
- 查看每个权限被哪些应用使用
- 管理每个应用的权限状态

**按应用维度 (Application维度)**:
- 展示所有已安装应用
- 查看每个应用请求的权限
- 管理应用每个权限的状态
- 查看权限使用记录

### 3. 权限组策略管理

支持22个权限组，采用**策略模式**实现：

| 权限组 | 包含权限 | 策略类 |
|--------|----------|--------|
| 位置信息 | LOCATION_IN_BACKGROUND, APPROXIMATELY_LOCATION, LOCATION | LocationStrategy |
| 相机 | CAMERA | CameraStrategy |
| 麦克风 | MICROPHONE | MicrophoneStrategy |
| 通讯录 | READ_CONTACTS, WRITE_CONTACTS | ContactsStrategy |
| 日历 | READ_CALENDAR, WRITE_CALENDAR, ... | CalendarStrategy |
| 图片和视频 | READ_IMAGEVIDEO, WRITE_IMAGEVIDEO, MEDIA_LOCATION | ImageAndVideosStrategy |
| 音频 | READ_AUDIO, WRITE_AUDIO | AudioStrategy |
| 文件 | READ_DOCUMENT, WRITE_DOCUMENT, ... | DocumentsStrategy |
| 等等... | ... | ... |

### 4. 安全组件对话框 (SecurityExtAbility)

为安全组件提供授权对话框：
- 相机组件
- 麦克风组件
- 位置组件
- 等等...

---

## 运行环境

### 系统要求

- **OS**: OpenHarmony 3.2+
- **设备类型**: phone, tablet, wearable, 2in1
- **API级别**: 9+

### 安装路径

```
/system/app/com.ohos.permissionmanager/permission_manager.hap
```

### 运行权限

应用自身声明了12个系统权限，包括：
- `ohos.permission.GET_SENSITIVE_PERMISSIONS` - 获取敏感权限信息
- `ohos.permission.GRANT_SENSITIVE_PERMISSIONS` - 授予敏感权限
- `ohos.permission.REVOKE_SENSITIVE_PERMISSIONS` - 撤销敏感权限
- `ohos.permission.PERMISSION_USED_STATS` - 访问权限使用统计
- 等等...

---

## 关键概念

### User_Grant权限

用户授权权限，应用必须在运行时请求用户明确授权。PermissionManager负责：
- 显示授权对话框
- 收集用户选择
- 通知权限服务更新状态

### 权限组 (Permission Group)

将逻辑相关的权限分组，减少弹窗次数：
- **位置组**: 包含模糊位置、精确位置、后台位置
- **通讯录组**: 包含读取和写入通讯录
- **日历组**: 包含读取和写入日历

### 全局开关 (Global Switch)

系统级权限开关：
- 当全局开关关闭时，所有应用无法使用该权限
- PermissionManager提供全局开关启用对话框

### 安全组件 (Security Component)

带权限的UI组件：
- 首次使用时显示授权对话框
- 授权后在短期内自动获得权限
- 无需重复弹窗

---

## 技术栈

| 层级 | 技术 |
|------|------|
| UI框架 | ArkUI (声明式UI) |
| 开发语言 | ArkTS (TypeScript超集) |
| 应用模型 | Stage模型 |
| 构建工具 | GN + Ninja |
| 包格式 | .hap (HarmonyOS Ability Package) |

---

## 代码统计

- **主模块源文件**: 64个 `.ets` 文件
- **策略实现类**: 20个策略 + 1个管理器
- **Ability**: 6个 (1 UIAbility + 3 ServiceExtAbility + 2 UIExtensionAbility)
- **页面**: 15个页面组件
- **代码行数**: 约8000+ 行（主模块）

---

## 相关文档

- [目录结构](01_Directory_Structure.md) - 详细源码组织说明
- [架构说明](02_Architecture.md) - 组件图和数据流
- [安全风险评审](06_Security_Analysis.md) - 安全威胁分析

---

*返回 [README](README.md) | 下一篇: [目录结构](01_Directory_Structure.md)*
