# 安全风险评审

## 综述

本文档基于代码审查，识别 Camera 应用的安全风险点。Camera 作为系统相机应用，涉及相机硬件访问、媒体文件存储、地理位置等敏感操作，需要严格的安全控制。

## 攻击面分析

### 1. 外部输入攻击面

| 攻击面 | 入口 | 风险等级 |
|--------|------|----------|
| **第三方应用调用** | MainAbility (action.imageCapture/videoCapture) | 高 |
| **卡片调用** | FormAbility | 中 |
| **多机位协同** | ExtensionPickerAbility | 高 |
| **文件分享** | ThirdPreviewView | 中 |
| **设置加载** | settinglist.json, settingdetaillist.json | 低 |

### 2. 系统能力攻击面

| 攻击面 | 系统能力 | 风险等级 |
|--------|----------|----------|
| **相机访问** | @ohos.multimedia.camera | 高 |
| **媒体读写** | 文件系统 + 相册 | 高 |
| **位置信息** | @ohos.geoLocationManager | 中 |
| **网络访问** | ohos.permission.INTERNET | 中 |
| **分布式能力** | ohos.permission.DISTRIBUTED_DATASYNC | 中 |

### 3. 数据存储攻击面

| 攻击面 | 存储方式 | 风险等级 |
|--------|----------|----------|
| **应用设置** | Preferences | 低 |
| **相机设置** | RDB | 低 |
| **拍摄媒体** | 系统相册 | 中 |
| **缩略图缓存** | 内存/文件缓存 | 低 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不信任区域                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │ 第三方应用 │  │ 恶意网页  │  │ 系统其他组件│                    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                      │
│       │             │             │                             │
│       └─────────────┴─────────────┘                             │
│                   │                                             │
│                   ▼                                             │
│         Intent / Want 传递                                      │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼ 信任边界
┌─────────────────────────────────────────────────────────────────┐
│                        受信任区域                                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   Camera Application                      │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │  │
│  │  │MainAbility │  │UIAbility  │  │Extension │              │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘              │  │
│  │       │             │             │                      │  │
│  │       └─────────────┴─────────────┘                      │  │
│  │                   │                                      │  │
│  │                   ▼                                      │  │
│  │           CameraService                                  │  │
│  │                   │                                      │  │
│  │                   ▼                                      │  │
│  │  ┌──────────────────────────────────────────────────┐   │  │
│  │  │              OpenHarmony System                   │   │  │
│  │  │  (多媒体服务、文件系统、位置服务等)                │   │  │
│  │  └──────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**关键边界**:
1. **Intent 边界**: 外部应用通过 Want/Intent 进入应用
2. **权限边界**: 系统权限检查点
3. **IPC 边界**: 与系统服务通信

## 风险点详细分析

### 风险 1: 第三方调用参数未充分校验

**风险等级**: 🔴 高

**证据位置**: 
- `product/phone/src/main/ets/MainAbility/MainAbility.ts:97-115`
- `product/phone/src/main/ets/pages/index.ets:619-638`
- `product/phone/src/main/ets/pages/PreviewArea.ets:503-509`

**问题描述**:
应用支持通过 Intent 被第三方应用调用，处理 `ACTION_IMAGE_CAPTURE` 和 `ACTION_VIDEO_CAPTURE` action。代码中通过 `launchWant.parameters` 获取参数，但缺乏充分的输入校验。

```typescript
// MainAbility.ts:97-115
if (this.launchWant?.action === wantConstant.Action.ACTION_IMAGE_CAPTURE ||
    this.launchWant?.parameters?.action === wantConstant.Action.ACTION_IMAGE_CAPTURE) {
  GlobalContext.get().setCameraFormParam({
    action: 'capture',
    mode: this.launchWant?.parameters?.mode || 'PHOTO',
    // ...
  });
}
```

```typescript
// PreviewArea.ets:503-509
let from: string = "";
if (GlobalContext.get().getCameraAbilityWant()?.parameters?.from) {
  from = GlobalContext.get().getCameraAbilityWant()?.parameters?.from as string;
}
```

**攻击路径**:
1. 恶意应用构造携带特殊参数的 Intent
2. 调用 Camera 应用的 capture 功能
3. 可能导致非预期的相机行为或信息泄露

**影响**:
- 潜在的逻辑绕过
- 非预期的相机启动模式
- 可能配合其他漏洞利用

**修复建议**:
1. 对 `launchWant.parameters` 进行白名单校验
2. 验证 `mode`, `action` 等参数的合法性
3. 对字符串参数进行长度和格式检查
4. 不信任外部传入的 `from` 参数用于安全决策

```typescript
// 建议的校验代码
const validModes = ['PHOTO', 'VIDEO', 'MULTI'];
const mode = this.launchWant?.parameters?.mode;
if (mode && !validModes.includes(mode)) {
  Log.error(`Invalid mode: ${mode}`);
  return;
}
```

---

### 风险 2: 权限过度申请

**风险等级**: 🟡 中

**证据位置**: `product/phone/src/main/module.json5:35-131`

**问题描述**:
应用申请了 13 项权限，部分权限的必要性存疑：

| 权限 | 用途 | 必要性评估 |
|------|------|-----------|
| `ohos.permission.INTERNET` | 网络访问 | ❓ 相机核心功能可能不需要 |
| `ohos.permission.MODIFY_AUDIO_SETTINGS` | 修改音频设置 | ✅ 录像需要 |
| `ohos.permission.GET_BUNDLE_INFO` | 获取 Bundle 信息 | ❓ 用途不明确 |
| `ohos.permission.ACCESS_SERVICE_DM` | 设备管理 | ✅ 多机位需要 |
| `ohos.permission.PROXY_AUTHORIZATION_URI` | URI 代理授权 | ❓ 需确认使用场景 |
| `ohos.permission.LOCATION_IN_BACKGROUND` | 后台定位 | ❓ 需确认使用场景 |

**攻击路径**:
- 过度权限可能被恶意利用
- 如果应用被攻破，攻击者可利用多余权限

**修复建议**:
1. 审查每项权限的必要性
2. 移除不必要的 `INTERNET` 权限（如确实不需要）
3. 明确记录每项权限的使用场景
4. 遵循最小权限原则

---

### 风险 3: 动态权限申请后的状态竞态

**风险等级**: 🟡 中

**证据位置**: `product/phone/src/main/ets/pages/index.ets:170-200`

**问题描述**:
动态权限申请使用回调方式，如果用户在权限对话框显示期间切换应用或系统状态变化，可能导致竞态条件。

```typescript
// 权限申请代码
abilityAccessCtrl.requestPermissionsFromUser(
  GlobalContext.get().getCameraAbilityContext(),
  permissionList,
  (err, result) => {
    // 回调处理
  }
);
```

**攻击路径**:
1. 触发权限申请对话框
2. 在对话框显示期间切换应用
3. 可能导致权限状态与 UI 状态不一致

**修复建议**:
1. 在回调中重新验证权限状态
2. 使用 try-catch 包裹权限操作
3. 在 onForeground 中重新检查权限

---

### 风险 4: 地理位置信息泄露风险

**风险等级**: 🟡 中

**证据位置**: 
- `common/src/main/ets/default/featurecommon/geolocation/GeoLocation.ts`
- `product/phone/src/main/module.json5:100-124`

**问题描述**:
应用申请了精确定位、后台定位、模糊定位三种位置权限。拍摄的照片会包含地理位置标签（EXIF）。

**问题点**:
1. 后台定位权限 (`LOCATION_IN_BACKGROUND`) 的必要性需确认
2. 位置数据存储和传输需加密保护
3. 用户可能不了解照片包含位置信息

**攻击路径**:
1. 获取照片文件
2. 提取 EXIF 地理位置信息
3. 用户位置隐私泄露

**修复建议**:
1. 提供设置项允许用户关闭地理位置标签
2. 审查后台定位的使用场景
3. 在隐私政策中明确告知位置信息使用
4. 考虑使用模糊定位替代精确定位

---

### 风险 5: 文件分享路径遍历风险

**风险等级**: 🟡 中

**证据位置**: `product/phone/src/main/ets/pages/ThirdPreviewView.ets:17`

**问题描述**:
ThirdPreviewView 使用 `fileshare` API 分享文件，如果传入的路径参数被恶意构造，可能导致路径遍历。

```typescript
import fileshare from '@ohos.fileshare';

// 可能的代码模式
fileshare.share({
  path: somePath,  // 如果 somePath 来自外部输入
  // ...
});
```

**攻击路径**:
1. 构造包含 `../` 的路径参数
2. 诱导应用分享非预期文件
3. 信息泄露

**修复建议**:
1. 对文件路径进行规范化处理
2. 验证路径在白名单内
3. 使用系统提供的安全文件分享机制

```typescript
// 建议的校验
function validatePath(path: string): boolean {
  const normalized = path.normalize();
  const allowedPrefix = '/storage/media/';
  return normalized.startsWith(allowedPrefix);
}
```

---

### 风险 6: 缩略图和缓存数据未清理

**风险等级**: 🟢 低

**证据位置**: `common/src/main/ets/default/camera/ThumbnailGetter.ts`

**问题描述**:
应用生成缩略图用于显示，但可能存在缓存数据未及时清理的问题。

**修复建议**:
1. 在 Ability 销毁时清理临时缩略图
2. 限制缓存大小
3. 定期清理过期缓存

---

### 风险 7: 日志敏感信息泄露

**风险等级**: 🟢 低

**证据位置**: 多处使用 `Log.info/error` 打印对象

**问题描述**:
代码中使用 `JSON.stringify` 打印对象到日志，可能包含敏感信息。

```typescript
// 示例
Log.info(`${this.TAG} want: ${JSON.stringify(this.launchWant)}`);
```

**修复建议**:
1. 审查日志输出，避免打印敏感信息
2. 生产构建移除详细日志
3. 对日志进行分级管理

---

### 风险 8: 分布式协同的安全边界

**风险等级**: 🟡 中

**证据位置**: 
- `product/phone/src/main/ets/MainAbility/ExtensionPickerAbility.ts`
- `product/phone/src/main/module.json5:91-97`

**问题描述**:
应用支持多机位协同 (`DISTRIBUTED_DATASYNC` 权限)，涉及分布式设备间的通信。

**潜在风险**:
1. 分布式通信的身份验证
2. 数据传输加密
3. 设备信任关系建立

**修复建议**:
1. 确保使用系统提供的安全分布式通道
2. 验证对端设备身份
3. 敏感数据端到端加密

---

### 风险 9: 系统签名密钥管理

**风险等级**: 🟡 中

**证据位置**: `build-profile.json5:27-38`

**问题描述**:
构建配置中包含加密的签名密码，但如果密钥泄露，攻击者可伪造应用。

```json
{
  "storePassword": "0000001C59E804AFC3B94CF72E6B32DC3921677DD07049135EA15C603E0782B8DF215F14497B25CC3DCC2D78",
  "keyPassword": "0000001C45BB3DEC4311ACA6F306211039D56C9DC4DB08A3B777A135E8D369557041FC175B0EAD1D95DC2814"
}
```

**修复建议**:
1. 使用环境变量或密钥管理服务
2. 定期轮换签名密钥
3. 限制签名密钥的访问权限
4. 生产环境使用硬件安全模块 (HSM)

---

### 风险 10: 配置文件注入风险

**风险等级**: 🟢 低

**证据位置**: 
- `product/phone/src/main/resources/rawfile/settinglist.json`
- `product/phone/src/main/resources/rawfile/settingdetaillist.json`

**问题描述**:
应用从 JSON 配置文件加载设置项，如果文件被篡改，可能影响应用行为。

**修复建议**:
1. 对配置文件进行签名验证
2. 验证 JSON 数据结构
3. 使用资源文件哈希校验

## 安全建议总结

### 立即修复 (高优先级)

1. ✅ **加强第三方调用参数校验**
   - 对 `launchWant.parameters` 进行白名单校验
   - 验证所有外部输入参数

2. ✅ **审查权限申请**
   - 移除不必要的 `INTERNET` 权限（如不需要）
   - 明确每项权限的使用场景

### 短期修复 (中优先级)

3. ✅ **完善权限申请状态管理**
   - 处理权限回调的竞态条件
   - 重新验证权限状态

4. ✅ **加强文件操作安全**
   - 路径遍历防护
   - 文件分享安全校验

5. ✅ **优化位置隐私保护**
   - 提供关闭地理位置标签的选项
   - 审查后台定位使用

### 长期改进 (低优先级)

6. ✅ **日志安全管理**
   - 移除敏感信息日志
   - 生产环境日志分级

7. ✅ **缓存清理机制**
   - 缩略图缓存管理
   - 定期清理临时文件

8. ✅ **签名密钥管理**
   - 使用密钥管理服务
   - 定期轮换密钥

## 安全检查范围与局限性

### 已检查范围

✅ 权限声明与使用 (`module.json5`)
✅ 第三方调用入口 (`MainAbility`, `ExtensionPickerAbility`)
✅ 文件操作代码路径
✅ 位置服务使用
✅ 日志输出
✅ 签名配置
✅ 动态权限申请流程

### 未检查范围 (需人工审查)

❓ 后端服务通信 (如有)
❓ 完整的分布式协同实现
❓ 详细的 EXIF 数据处理
❓ 完整的设置项验证

### 建议的进一步行动

1. **渗透测试**: 对第三方调用入口进行 fuzz 测试
2. **代码审计**: 人工审查分布式协同相关代码
3. **隐私评估**: 完整的隐私影响评估 (PIA)
4. **合规检查**: 确保符合数据保护法规要求
