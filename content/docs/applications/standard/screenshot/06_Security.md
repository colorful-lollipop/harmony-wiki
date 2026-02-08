# 安全风险评审

## 概述

本文档对 ScreenShot 应用进行安全风险评审，识别潜在攻击面和可被利用点，并提供修复建议。

**评审范围**: `product/phone/`, `features/screenshot/`, `common/`
**评审时间**: 基于代码仓库当前版本

## 攻击面分析

### 1. 能力入口

| 攻击面 | 类型 | 描述 |
|-------|------|------|
| ServiceExtAbility | IPC | 接收 TOGGLE action 触发截屏 |
| DialogAbility | UI Extension | 接收用户交互 |
| 外部应用调用 | Intent | startAbility 启动其他应用 |

**证据**: `product/phone/src/main/module.json5:16-48`

### 2. 系统 API 调用

| API | 风险等级 | 用途 |
|-----|---------|------|
| `ScreenshotManager.save()` | 中 | 截取系统屏幕 |
| `windowManager.createWindow()` | 低 | 创建浮动窗口 |
| `userFileManager.createPhotoAsset()` | 中 | 写入用户相册 |
| `startAbility()` | 中 | 启动外部应用 |

### 3. 数据存储

| 存储类型 | 路径 | 敏感级别 |
|---------|------|---------|
| 图片文件 | `Screenshots/` | 中 |
| AppStorage | 内存 | 低 |

### 4. 权限暴露

| 权限 | 敏感级别 | 必要性 |
|-----|---------|------|
| `ohos.permission.CAPTURE_SCREEN` | 高 | 必须 |
| `ohos.permission.WRITE_IMAGEVIDEO` | 高 | 必须 |
| `ohos.permission.MEDIA_LOCATION` | 中 | 可选 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 中 | 可选 |

## 信任边界

```
┌─────────────────────────────────────────┐
│           外部触发源                       │
│  System UI / 其他应用                     │
└──────────────┬──────────────────────────┘
               │ IPC (Want)
               ▼
┌─────────────────────────────────────────┐
│         ServiceExtAbility                │
│  权限: CAPTURE_SCREEN                   │
│  边界: 接收外部请求，验证权限               │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         ScreenShotModel                  │
│  边界: 执行截屏，保存图片                   │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         系统服务                          │
│  @ohos.screenshot / @ohos.filemanager   │
│  边界: 底层系统调用                       │
└─────────────────────────────────────────┘
```

## 可被利用点

### 风险 1：全局变量未初始化验证

**严重程度**: 中

**证据**:
```
文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:66
```

```typescript
const context = globalThis.shotScreenContext as common.ServiceExtensionContext;
```

**问题描述**:
- `globalThis.shotScreenContext` 在 ServiceExtAbility.onCreate 中设置
- 但在 ScreenShotModel 中使用前未验证是否已初始化
- 如果调用顺序异常，可能导致空指针异常

**触发条件**:
1. 并发调用 ScreenShotModel 方法
2. ServiceExtAbility 未正常初始化

**潜在影响**:
- 应用崩溃 (NullPointerException)
- 拒绝服务

**修复建议**:
```typescript
const context = globalThis.shotScreenContext as common.ServiceExtensionContext;
if (!context) {
    Log.showError(TAG, 'shotScreenContext is undefined');
    return;
}
```

---

### 风险 2：文件名时间戳注入

**严重程度**: 低

**证据**:
```
文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ecs:69
```

```typescript
this.imageFileName = SCREENSHOT_PREFIX + '_' + (new Date()).getTime() + PICTURE_TYPE;
```

**问题描述**:
- 使用时间戳作为文件名
- 虽然不是直接的用户输入，但时间戳可被预测
- 理论上可能用于文件名枚举

**触发条件**:
- 需要预测文件名保存位置
- 攻击者需要知道确切时间

**潜在影响**:
- 信息泄露 (可预测文件名)
- 低风险，因为存储在用户私有目录

**修复建议**:
```typescript
// 使用 UUID 或安全随机数
import { crypto } from '@ohos.security';
const randomName = crypto.generateRandomUUID();
this.imageFileName = SCREENSHOT_PREFIX + '_' + randomName + PICTURE_TYPE;
```

---

### 风险 3：Want 参数未校验

**严重程度**: 中

**证据**:
```
文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:122-124
```

```typescript
openAbility(wantData: Want) {
    globalThis.shotScreenContext.startAbility(wantData);
}
```

**问题描述**:
- `openAbility` 接收外部传入的 Want 参数
- 未验证 want.bundleName 和 want.abilityName 的合法性
- 可能被利用启动恶意应用

**触发条件**:
1. 恶意应用传入伪造的 bundleName/abilityName
2. 用户确认后启动目标应用

**潜在影响**:
- 启动恶意应用
- 权限提升 (通过相册应用)

**修复建议**:
```typescript
openAbility(wantData: Want) {
    // 白名单验证
    const ALLOWED_BUNDLES = ['com.ohos.photos'];
    if (!wantData.bundleName || !ALLOWED_BUNDLES.includes(wantData.bundleName)) {
        Log.showError(TAG, 'Invalid bundleName');
        return;
    }
    globalThis.shotScreenContext.startAbility(wantData);
}
```

---

### 风险 4：截图数据未加密存储

**严重程度**: 中

**证据**:
```
文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets:62-98
```

```typescript
const packedImg = await packer.packing(pixelMap, options);
await file.write(fd, packedImg);
```

**问题描述**:
- 截图以 JPEG 格式明文存储
- 未启用加密存储选项
- 恶意应用可能通过 root 权限访问

**触发条件**:
- 设备已 root
- 恶意应用获取 root 权限

**潜在影响**:
- 截图内容泄露
- 隐私数据暴露

**修复建议**:
```typescript
// 使用加密存储选项 (如果系统支持)
const createOption: userFileManager.PhotoCreateOptions = {
    subType: userFileManager.PhotoSubType.SCREENSHOT,
    // encrypted: true,  // 如果支持
};
```

---

### 风险 5：对话框应用标签注入

**严重程度**: 低

**证据**:
```
文件: product/phone/src/main/ets/PrivacyDialog/DialogPage.ets:178-179
```

```typescript
if (this.want.parameters?.callingLabel) {
    this.appLabel = (this.want.parameters?.appLabel).toString();
}
```

**问题描述**:
- `appLabel` 来源于外部传入的 Want parameters
- 直接转换为字符串显示
- 可能用于显示欺骗 (Phishing)

**触发条件**:
1. 恶意应用伪造 callingLabel
2. 显示在隐私确认对话框中

**潜在影响**:
- UI 欺骗
- 用户误判应用来源

**修复建议**:
```typescript
// 只使用系统返回的 callingLabel，不信任外部传入的 appLabel
this.appLabel = this.want.parameters?.callingLabel || '';
```

---

### 风险 6：超时时间硬编码

**严重程度**: 低

**证据**:
```
文件: product/phone/src/main/ets/pages/index.ets:46
```

```typescript
setTimeout(ViewModel.CloseShotScreen, Constants.interval);
```

**问题描述**:
- 自动关闭时间硬编码为 5000ms
- 可能被利用进行 UI 欺骗

**触发条件**:
- 用户来不及确认截图内容
- 恶意使用场景有限

**潜在影响**:
- 用户体验问题
- 非安全相关

**修复建议**:
- 考虑提供配置项，允许用户调整

---

## 权限使用分析

### 权限必要性

| 权限 | 必要性 | 理由 |
|-----|-------|------|
| CAPTURE_SCREEN | 必须 | 核心功能 |
| WRITE_IMAGEVIDEO | 必须 | 保存截图 |
| MEDIA_LOCATION | 可选 | 非必须 |
| START_ABILITIES_FROM_BACKGROUND | 可选 | 非必须 |

### 权限使用时机

| 权限 | 使用组件 | 使用方法 |
|-----|---------|---------|
| CAPTURE_SREEN | ServiceExtAbility | 声明权限 |
| WRITE_IMAGEVIDEO | ScreenShotModel | createPhotoAsset |

## 安全建议汇总

| 优先级 | 风险 | 建议 |
|-------|------|------|
| 高 | Want 参数未校验 | 添加白名单验证 |
| 中 | 全局变量未验证 | 增加空值检查 |
| 中 | 截图未加密 | 使用加密存储 |
| 低 | 文件名可预测 | 使用随机 UUID |
| 低 | 应用标签来源 | 验证参数来源 |

## 检查局限性

1. **范围限制**: 仅检查了应用代码，未检查系统框架
2. **动态分析**: 静态代码分析，无法验证运行时行为
3. **Root 权限**: 无法防御 root 后的恶意行为
4. **新版本 API**: 需持续跟踪 OpenHarmony API 安全更新

## 相关文档

- OpenHarmony 安全指南
- ArkUI 安全编码规范
- OpenHarmony 权限管理
