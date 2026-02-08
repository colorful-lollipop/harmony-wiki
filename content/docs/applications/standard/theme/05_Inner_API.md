# 内部 API

> 本章档说明本项目内部的模块接口、依赖方向和稳定性标注。

## 模块接口

### WallpaperExtAbility 对外接口

| 方法/属性 | 访问级别 | 稳定性 | 说明 |
|-----------|----------|--------|------|
| `onCreated(want)` | public | 稳定 | 系统回调，窗口初始化 |
| `onWallpaperChanged(type)` | public | 稳定 | 系统回调，壁纸变化 |
| `onDestroy()` | public | 稳定 | 系统回调，资源清理 |
| `initWallpaperImage()` | private | 稳定 | 内部方法，壁纸初始化 |
| `sendPixelMapData()` | private | 稳定 | 内部方法，数据同步 |

**代码位置**: `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:23-76`

### MainAbility 对外接口

| 方法/属性 | 访问级别 | 稳定性 | 说明 |
|-----------|----------|--------|------|
| `onCreate(want, launchParam)` | public | 稳定 | 系统回调 |
| `onDestroy()` | public | 稳定 | 系统回调 |
| `onWindowStageCreate(stage)` | public | 稳定 | 系统回调 |
| `onWindowStageDestroy()` | public | 稳定 | 系统回调 |
| `onForeground()` | public | 稳定 | 系统回调 |
| `onBackground()` | public | 稳定 | 系统回调 |

**代码位置**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:18-46`

### AbilityStage 对外接口

| 方法/属性 | 访问级别 | 稳定性 | 说明 |
|-----------|----------|--------|------|
| `onCreate()` | public | 稳定 | 系统回调 |

**代码位置**: `product/phone/src/main/ets/Application/AbilityStage.ts:18-21`

## 依赖方向

### 依赖关系图

```mermaid
graph TD
    subgraph 系统 API
        WAPI[@ohos.wallpaper]
        WIN[@ohos.window]
        WEXT[@ohos.WallpaperExtension]
        AB[@ohos.application.Ability]
        AST[@ohos.application.AbilityStage]
    end

    subgraph 应用代码
        AS[AbilityStage]
        MA[MainAbility]
        WE[WallpaperExtAbility]
        PG[pages/index]
        ST[AppStorage]
    end

    WE --> WAPI
    WE --> WIN
    WE --> WEXT
    WE --> ST

    MA --> AB
    MA --> WIN

    AS --> AST

    PG --> ST
```

### 依赖矩阵

| 模块 | 依赖 | 被依赖 | 依赖类型 |
|------|------|--------|----------|
| **AbilityStage** | AbilityStage | MainAbility | 继承 |
| **MainAbility** | Ability, WindowManager | 无 | API 调用 |
| **WallpaperExtAbility** | WallpaperExtension, wallpaper, window, AppStorage | 无 | API 调用/继承 |
| **pages/index** | AppStorage | WallpaperExtAbility | 数据绑定 |

## 稳定性标注

### 稳定性等级定义

| 等级 | 定义 | 示例 |
|------|------|------|
| **稳定 (Stable)** | 系统 API 或标准实现 | @ohos.*, 基类方法 |
| **实验 (Experimental)** | 新增功能，可能变化 | 无 |
| **内部 (Internal)** | 仅项目内部使用 | 无 |

### 接口稳定性清单

| 接口 | 稳定性 | 依据 |
|------|--------|------|
| `wallPaper.getPixelMap()` | 稳定 | 系统 API |
| `windowManager.create()` | 稳定 | 系统 API |
| `AppStorage.SetOrCreate()` | 稳定 | 系统 API |
| `Extension.onCreated()` | 稳定 | 基类方法 |
| `Ability.onCreate()` | 稳定 | 基类方法 |
| `WallpaperExtAbility.initWallpaperImage()` | 稳定 | 私有方法，逻辑简单 |
| `WallpaperExtAbility.sendPixelMapData()` | 稳定 | 私有方法，逻辑简单 |

## 可替换点

### 潜在可替换组件

| 组件 | 可替换性 | 说明 |
|------|----------|------|
| **壁纸数据源** | 不可替换 | 必须使用系统 wallpaper 服务 |
| **窗口管理** | 不可替换 | 必须使用系统 window 服务 |
| **UI 组件** | 可替换 | 可替换为其他 UI 实现 |
| **数据存储** | 不可替换 | 必须使用 AppStorage 进行跨组件同步 |

### 扩展点

| 扩展点 | 位置 | 说明 |
|--------|------|------|
| **壁纸变化处理** | `onWallpaperChanged()` | 可添加自定义逻辑 |
| **窗口初始化** | `onCreated()` | 可添加自定义窗口配置 |
| **页面渲染** | `pages/index.ets` | 可自定义 UI 展示 |

## 错误传播机制

### 错误处理模式

```typescript
// 模式 1: Promise 错误处理（未使用）
windowManager.create(this.context, "wallpaper", 2000)
    .then((win) => {
        // 成功处理
    })
    .catch((error) => {
        // 错误处理（代码中未实现）
    })

// 模式 2: 回调错误参数
wallPaper.getPixelMap(0, (err, data) => {
    if (err) {
        console.info(MODULE_TAG + 'ability get pixel map, err: ' + JSON.stringify(err));
    }
    AppStorage.SetOrCreate('slPixelData', data);
});
```

### 错误日志

| 位置 | 日志级别 | 说明 |
|------|----------|------|
| `WallpaperExtAbility.ts:37` | error | 窗口创建失败 |
| `WallpaperExtAbility.ts:69` | info | 获取壁纸结果 |

## 相关文档

- [对外 API](04_API.md)
- [架构说明](03_Architecture.md)
- [安全评审](08_Security.md)
