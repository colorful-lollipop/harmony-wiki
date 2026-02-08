# 外部调用接口

## 1. 概述

本章节详细说明图库应用的**外部调用接口**，包括：
- Ability 入口定义
- URI Scheme 参数规范
- Want 参数完整清单
- 调用返回结果格式

> **注意**: 本应用**无 N-API**，所有外部接口均为 Ability 形式的 OpenHarmony 标准接口。

---

## 2. Ability 入口定义

### 2.1 MainAbility（主入口）

| 属性 | 值 | 证据 |
|------|-----|------|
| **Name** | `com.ohos.photos.MainAbility` | `module.json5:60` |
| **Type** | Page | `module.json5:60` |
| **LaunchType** | singleton | `module.json5:66` |
| **Visible** | true | `module.json5:65` |
| **Icon** | `$media:ohos_gallery` | `module.json5:63` |
| **Label** | `$string:app_name` | `module.json5:64` |

**入口文件**: `product/phone/src/main/ets/MainAbility/MainAbility.ts`  
**描述**: 处理所有外部调用，根据参数路由到不同页面

---

### 2.2 FormAbility（FA 卡片）

| 属性 | 值 | 证据 |
|------|-----|------|
| **Name** | `com.ohos.photos.FormAbility` | `module.json5:116` |
| **Type** | form | `module.json5:120` |
| **Description** | `$string:app_name` | `module.json5:119` |

**入口文件**: `product/phone/src/main/ets/FormAbility/FormAbility.ts`

---

### 2.3 ServiceExtAbility（服务扩展）

| 属性 | 值 | 证据 |
|------|-----|------|
| **Name** | `com.ohos.photos.ServiceExtAbility` | `module.json5:129` |
| **Type** | service | `module.json5:133` |
| **Visible** | true | `module.json5:134` |

**入口文件**: `product/phone/src/main/ets/ServiceExt/ServiceExtAbility.ts`

---

### 2.4 PickerUIExtensionAbility（选择器 UI）

| 属性 | 值 | 证据 |
|------|-----|------|
| **Name** | `PickerUIExtensionAbility` | `module.json5:149` |
| **Type** | sysPicker/photoPicker | `module.json5:151` |
| **Exported** | true | `module.json5:152` |

**入口文件**: `product/phone/src/main/ets/picker/PickerUIExtensionAbility.ets`

---

### 2.5 DeleteUIExtensionAbility（删除对话框）

| 属性 | 值 | 证据 |
|------|-----|------|
| **Name** | `DeleteUIExtensionAbility` | `module.json5:156` |
| **Type** | sysDialog/common | `module.json5:158` |
| **Exported** | true | `module.json5:159` |

**入口文件**: `product/phone/src/main/ets/DeleteAbility/DeleteUIExtensionAbility.ets`

---

### 2.6 SaveUIExtensionAbility（保存对话框）

| 属性 | 值 | 证据 |
|------|-----|------|
| **Name** | `SaveUIExtensionAbility` | `module.json5:163` |
| **Type** | sysDialog/common | `module.json5:165` |
| **Exported** | true | `module.json5:166` |

**入口文件**: `product/phone/src/main/ets/SaveAbility/SaveUIExtensionAbility.ets`

---

## 3. Skill 配置

### 3.1 MainAbility Skills

```json
{
  "skills": [
    {
      "entities": ["entity.system.home"],
      "actions": [
        "action.system.home",
        "ohos.want.action.viewData",
        "ohos.want.action.photoPicker"
      ],
      "uris": [
        { "type": "image/*", "scheme": "file" },
        { "type": "video/*", "scheme": "file" },
        { "type": "multipleselect", "scheme": "file" },
        { "type": "singleselect", "scheme": "file" },
        { "type": "image/*" },
        { "type": "video/*" },
        { "type": "multipleselect" },
        { "type": "singleselect" }
      ]
    }
  ]
}
```

| URI Pattern | 用途 |
|-------------|------|
| `file://image/*` | 文件协议图片 |
| `file://video/*` | 文件协议视频 |
| `file://singleselect` | 单选模式 |
| `file://multipleselect` | 多选模式 |

> **证据**: `module.json5:67-108`

---

## 4. URI Scheme 参数规范

### 4.1 调用方式

```typescript
// 方式1: startAbility
context.startAbility({
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: { ... }
});

// 方式2: startAbilityForResult (需要返回结果)
context.startAbilityForResult({
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: { ... }
}).then((result) => {
  // 处理返回结果
});
```

---

### 4.2 参数清单

#### 4.2.1 `uri=photodetail` - 相机照片浏览

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `uri` | string | ✅ | 值: `"photodetail"` |

**用途**: 相机拍摄后浏览刚拍照片

**路由目标**: `pages/PhotoBrowser`

**示例**:
```typescript
let want = {
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: {
    uri: "photodetail"
  }
};
context.startAbility(want);
```

> **证据**: `MainAbility.ts:99-112`

---

#### 4.2.2 `uri=singleselect` - 单选图片

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `uri` | string | ✅ | - | 值: `"singleselect"` |
| `callerBundleName` | string | ✅ | - | 调用方包名 |
| `filterMediaType` | string | ❌ | `"all"` | 过滤类型 |
| `isPhotoTakingSupported` | boolean | ❌ | `true` | 是否允许拍照 |
| `isEditSupported` | boolean | ❌ | `true` | 是否允许编辑 |

**路由目标**: `pages/ThirdSelectPhotoGridPage`

**返回结果**:
```typescript
{
  want: {
    parameters: {
      "select-item-list": ["file://..."]  // 选中的 URI 列表
    }
  }
}
```

**示例**:
```typescript
let want = {
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: {
    uri: "singleselect",
    callerBundleName: "com.example.myapp",
    filterMediaType: "image",
    isPhotoTakingSupported: true,
    isEditSupported: true
  }
};
context.startAbilityForResult(want);
```

> **证据**: `MainAbility.ts:113-121`

---

#### 4.2.3 `uri=multipleselect` - 多选图片

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `uri` | string | ✅ | - | 值: `"multipleselect"` |
| `callerBundleName` | string | ✅ | - | 调用方包名 |
| `maxSelectCount` | number | ✅ | - | 最大选择数 |
| `filterMediaType` | string | ❌ | `"all"` | 过滤类型 |
| `preselectedUris` | string[] | ❌ | - | 预选中的 URI |
| `isPhotoTakingSupported` | boolean | ❌ | `true` | 是否允许拍照 |
| `isEditSupported` | boolean | ❌ | `true` | 是否允许编辑 |

**路由目标**: `pages/ThirdSelectPhotoGridPage`

**返回结果**:
```typescript
{
  want: {
    parameters: {
      "select-item-list": ["file://...", "file://..."]
    }
  }
}
```

**示例**:
```typescript
let want = {
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: {
    uri: "multipleselect",
    callerBundleName: "com.example.myapp",
    maxSelectCount: 9,
    filterMediaType: "image",
    preselectedUris: ["file://preset1", "file://preset2"]
  }
};
context.startAbilityForResult(want);
```

> **证据**: `MainAbility.ts:123-131`

---

#### 4.2.4 `uri=form` - FA 卡片浏览

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `uri` | string | ✅ | 值: `"form"` |
| `albumUri` | string | ❌ | 相册 URI |
| `currentUri` | string | ❌ | 当前照片 URI |
| `currentIndex` | number | ❌ | 当前索引 |
| `displayName` | string | ❌ | 显示名称 |

**路由目标**:
- `pages/PhotoBrowser` (有完整 URI 参数)
- `pages/DefaultPhotoPage` (无参数时)

**示例**:
```typescript
let want = {
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: {
    uri: "form",
    albumUri: "album://...",
    currentUri: "file://...",
    currentIndex: 0,
    displayName: "photo.jpg"
  }
};
```

> **证据**: `MainAbility.ts:132-141`

---

#### 4.2.5 `uri=formNone` - FA 默认页

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `uri` | string | ✅ | 值: `"formNone"` |

**路由目标**: `pages/DefaultPhotoPage`

> **证据**: `MainAbility.ts:162-163`

---

#### 4.2.6 `formId` - FA 卡片编辑

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `formId` | string | ✅ | FA 卡片 ID |

**路由目标**: `pages/FormEditorPage`

> **证据**: `MainAbility.ts:141-143`

---

#### 4.2.7 `action=action.viewData` - 外部数据查看

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `action` | string | ✅ | 值: `"ohos.want.action.viewData"` |
| `uri` | string | ❌ | 数据 URI |
| `isShowMenu` | boolean | ❌ | 是否显示菜单 |
| `albumUri` | string | ❌ | 关联相册 URI |
| `viewIndex` | number | ❌ | 视图索引 |

**路由目标**: `pages/PhotoBrowser`

**示例**:
```typescript
let want = {
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  action: "ohos.want.action.viewData",
  uri: "file://...",
  parameters: {
    isShowMenu: true,
    albumUri: "album://...",
    viewIndex: 0
  }
};
```

> **证据**: `MainAbility.ts:144-161`

---

## 5. 返回结果格式

### 5.1 选择结果

```typescript
interface SelectResult {
  resultCode: number;      // 0 = 成功
  want: {
    bundleName: string;
    abilityName: string;
    parameters: {
      "select-item-list": string[];  // 选中的 URI 列表
    };
  };
}
```

### 5.2 示例

```typescript
context.startAbilityForResult(want).then((result) => {
  if (result.resultCode === 0) {
    let selectedUris = result.want.parameters["select-item-list"];
    console.info(`Selected: ${JSON.stringify(selectedUris)}`);
  }
});
```

---

## 6. 错误处理

### 6.1 常见错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| -1 | 参数无效 | 检查参数完整性 |
| 1 | 内部错误 | 重试或查看日志 |
| 16000001 | 能力不存在 | 检查 abilityName |

### 6.2 错误处理示例

```typescript
context.startAbilityForResult(want).then((result) => {
  if (result.resultCode !== 0) {
    console.error(`Select failed: ${result.resultCode}`);
    return;
  }
  // 处理成功结果
}).catch((error) => {
  console.error(`Start ability failed: ${JSON.stringify(error)}`);
});
```

---

## 7. 权限要求

### 7.1 调用方权限

第三方应用调用图库**无需额外权限**，只需：
1. 知道目标 bundleName 和 abilityName
2. 构造正确的 want 参数

### 7.2 图库所需权限

图库运行时会检查以下权限：

| 权限 | 用途 | 触发条件 |
|------|------|----------|
| `ohos.permission.READ_IMAGEVIDEO` | 读取媒体 | FA 卡片访问 |
| `ohos.permission.WRITE_IMAGEVIDEO` | 写入媒体 | FA 卡片编辑 |
| `ohos.permission.MEDIA_LOCATION` | 位置信息 | 媒体位置访问 |

> **证据**: `module.json5:21-55`

---

## 8. 调用链追踪

```
第三方应用
    │
    │ startAbility / startAbilityForResult
    ▼
┌──────────────────────────────────────────────────────────────┐
│                    MainAbility                                │
│  ┌────────────────────────────────────────────────────────┐   │
│  │ onCreate(want, param)                                │   │
│  │   └─> parseWantParameter() [MainAbility.ts:92-166]   │   │
│  └────────────────────────────────────────────────────────┘   │
│                              │                                │
│                              ▼                                │
│              ┌─────────────────────────────┐                   │
│              │   want.parameters.uri      │                   │
│              │   决定路由目标              │                   │
│              └─────────────────────────────┘                   │
│                              │                                │
│         ┌────────┬─────────┼─────────┬────────┐             │
│         ▼        ▼         ▼         ▼        ▼             │
│    photodetail  single    multiple   form   viewData        │
│         │        │         │         │        │              │
│         ▼        ▼         ▼         ▼        ▼             │
│    PhotoBrowser  ThirdSelectPhotoGridPage                      │
│                                                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 9. 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview.md](00_Overview.md) | 项目概览 |
| [01_Architecture.md](01_Architecture.md) | 架构与数据流 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 问题排查 |
