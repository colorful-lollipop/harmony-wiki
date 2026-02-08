# 安全风险评审

> 本章档基于代码证据对 applications_theme 进行安全风险分析。

## 评审范围

| 检查项 | 覆盖范围 | 证据 |
|--------|----------|------|
| **代码范围** | `product/phone/src/main/ets/` | ETS 源码目录 |
| **配置范围** | `module.json5`, `app.json5` | 配置文件 |
| **签名配置** | `signature/` | 签名配置目录 |
| **测试代码** | ❌ 不包含 | 按规范排除 |

## 攻击面清单

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| **权限请求** | 权限滥用 | 低 | 请求壁纸读取权限 |
| **系统 API 调用** | API 滥用 | 低 | 调用 wallpaper/window 服务 |
| **数据存储** | 数据泄露 | 低 | AppStorage 存储壁纸数据 |
| **窗口管理** | 窗口劫持 | 低 | 创建系统壁纸窗口 |
| **外部输入** | 输入验证 | 无 | 本地服务回调，无外部输入 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                     信任边界图                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   可信区域 (Trust Zone)                     │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │           applications_theme                         │  │  │
│  │  │  - WallpaperExtAbility                              │  │  │
│  │  │  - MainAbility                                      │  │  │
│  │  │  - pages/index.ets                                  │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   边界 API (Boundary)                      │  │
│  │  - @ohos.wallpaper (系统壁纸服务)                         │  │
│  │  - @ohos.window (系统窗口服务)                            │  │
│  │  - @ohos.WallpaperExtension (系统扩展基类)                │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   不可信区域 (Untrusted)                    │  │
│  │  - 其他应用                                                │  │
│  │  - 网络输入                                                │  │
│  │  - 外部存储                                                │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 数据流与信任边界

| 数据流 | 源 | 目标 | 信任级别 |
|--------|-----|------|----------|
| 壁纸数据 | WallpaperService | WallpaperExtAbility | 高 (系统服务) |
| 窗口配置 | WallpaperExtAbility | WindowManager | 高 (系统 API) |
| 存储同步 | WallpaperExtAbility | AppStorage | 高 (系统存储) |
| UI 渲染 | AppStorage | pages/index | 高 (内部组件) |

## 可被利用点分析

### 风险 1: 壁纸权限过度使用

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-001 |
| **攻击面** | 权限请求 |
| **证据** | `product/phone/src/main/module.json5:15-22` |
| **触发条件** | 应用请求 `ohos.permission.GET_WALLPAPER` |
| **影响** | 获取用户壁纸数据，可能涉及隐私 |
| **风险等级** | 低 |
| **修复建议** | 权限使用符合最小必要原则，仅用于壁纸显示 |

**代码证据**:
```json5
// product/phone/src/main/module.json5
"requestPermissions": [
  {
    "name": "ohos.permission.GET_WALLPAPER"  // 权限名称
  },
  {
    "name": "ohos.permission.READ_USER_STORAGE"
  }
]
```

### 风险 2: 窗口创建失败未处理

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-002 |
| **攻击面** | 窗口管理 |
| **证据** | `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:36-38` |
| **触发条件** | `windowManager.create()` 失败 |
| **影响** | 壁纸窗口创建失败，功能降级 |
| **风险等级** | 低 |
| **修复建议** | 增加重试机制或错误恢复逻辑 |

**代码证据**:
```typescript
// product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts
windowManager.create(this.context, "wallpaper", 2000).then((win) => {
    // 成功处理
}, (error) => {
    Log.showError(TAG, name + " window createFailed, error.code = " + error.code)
    // 仅记录日志，无恢复逻辑
})
```

### 风险 3: 壁纸数据回调未验证

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-003 |
| **攻击面** | 数据处理 |
| **证据** | `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:68-73` |
| **触发条件** | `wallPaper.getPixelMap()` 返回异常数据 |
| **影响** | 异常数据可能导致 UI 异常或崩溃 |
| **风险等级** | 低 |
| **修复建议** | 增加数据有效性校验 |

**代码证据**:
```typescript
wallPaper.getPixelMap(0, (err, data) => {
    console.info(MODULE_TAG + 'ability get pixel map, err: ' + JSON.stringify(err) +
    " data: " + JSON.stringify(data));
    AppStorage.SetOrCreate('slPixelData', data);  // 未校验 data 有效性
});
```

### 风险 4: 全局变量潜在泄露

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-004 |
| **攻击面** | 数据存储 |
| **证据** | `product/phone/src/main/ets/MainAbility/MainAbility.ts:21` |
| **触发条件** | `globalThis.abilityWant` 被其他模块访问 |
| **影响** | 能力意图 (Want) 信息可能泄露 |
| **风险等级** | 低 |
| **修复建议** | 如非必要，移除 globalThis 使用 |

**代码证据**:
```typescript
// product/phone/src/main/ets/MainAbility/MainAbility.ts
onCreate(want, launchParam) {
    console.log("ExtWallpaper: MainAbility onCreate")
    globalThis.abilityWant = want;  // 可能泄露 want 信息
}
```

### 风险 5: 回调中异常处理缺失

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-005 |
| **攻击面** | 错误处理 |
| **证据** | `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:68-73` |
| **触发条件** | `getPixelMap` 回调中 err 不为空 |
| **影响** | 错误被静默忽略，可能导致状态不一致 |
| **风险等级** | 低 |
| **修复建议** | 增加 err 状态的错误处理逻辑 |

**代码证据**:
```typescript
wallPaper.getPixelMap(0, (err, data) => {
    console.info(MODULE_TAG + 'ability get pixel map, err: ' + JSON.stringify(err) +
    " data: " + JSON.stringify(data));
    // 未判断 err 是否为空，直接使用 data
    AppStorage.SetOrCreate('slPixelData', data);
});
```

## 风险汇总

| 风险 ID | 风险名称 | 风险等级 | 状态 |
|---------|----------|----------|------|
| SEC-001 | 壁纸权限过度使用 | 低 | 已知 |
| SEC-002 | 窗口创建失败未处理 | 低 | 已知 |
| SEC-003 | 壁纸数据回调未验证 | 低 | 已知 |
| SEC-004 | 全局变量潜在泄露 | 低 | 已知 |
| SEC-005 | 回调中异常处理缺失 | 低 | 已知 |

## 安全建议

### 短期改进

1. **增强错误处理**
   - 在 `windowManager.create()` 失败时增加重试机制
   - 在 `getPixelMap` 回调中增加 err 判断

2. **数据有效性校验**
   - 在设置 `AppStorage` 前校验 `data` 不为 null/undefined

3. **移除不必要的 globalThis 使用**
   - 评估 `globalThis.abilityWant` 的必要性
   - 如非必要，移除该行代码

### 长期改进

1. **权限最小化**
   - 定期审查权限请求的必要性
   - 遵循最小权限原则

2. **安全审计**
   - 定期进行代码安全审计
   - 关注 OpenHarmony 安全更新

## 安全检查清单

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 权限请求合理 | ✅ 通过 | 仅请求必要的壁纸权限 |
| API 调用安全 | ⚠️ 需改进 | 部分 API 错误处理不足 |
| 数据存储安全 | ⚠️ 需改进 | 缺少数据有效性校验 |
| 全局变量管理 | ⚠️ 需改进 | 存在不必要的 globalThis 使用 |
| 错误处理完整 | ⚠️ 需改进 | 部分场景缺少错误恢复 |

## 相关文档

- [对外 API](04_API.md)
- [内部 API](05_Inner_API.md)
- [问题定位](09_Troubleshooting.md)
