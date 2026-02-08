# 对外 API

> 本章档说明本项目的对外接口。本项目为纯 ETS 前端项目，**无 N-API 暴露**。

## N-API 清单

### 结论

**本项目不涉及任何 N-API 对外接口。**

| 检查项 | 结果 |
|--------|------|
| N-API 导出 | ❌ 无 |
| NAPI_MODULE | ❌ 无 |
| napi_define_properties | ❌ 无 |
| native C/C++ 代码 | ❌ 无 |

### 证据

```typescript
// 项目源码均为 ArkTS/ETS，无 C/C++ 代码
// product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts
// product/phone/src/main/ets/MainAbility/MainAbility.ts
// product/phone/src/main/ets/Application/AbilityStage.ts
// product/phone/src/main/ets/pages/index.ets
```

## 系统 API 依赖

本项目使用 OpenHarmony 系统提供的 JavaScript API：

### @ohos.wallpaper (壁纸服务)

| API | 类型 | 说明 |
|-----|------|------|
| `getPixelMap(wallpaperType, callback)` | 异步回调 | 获取壁纸像素数据 |

**参数说明**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `wallpaperType` | number | 是 | 壁纸类型：0-锁屏壁纸，1-桌面壁纸 |
| `callback` | Function | 是 | 回调函数，参数 (err, data) |

**代码位置**: `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:68`

```typescript
wallPaper.getPixelMap(0, (err, data) => {
    console.info(MODULE_TAG + 'ability get pixel map, err: ' + JSON.stringify(err) +
    " data: " + JSON.stringify(data));
    AppStorage.SetOrCreate('slPixelData', data);
});
```

### @ohos.window (窗口管理)

| API | 类型 | 说明 |
|-----|------|------|
| `create(context, name, priority)` | Promise | 创建窗口 |
| `loadContent(path)` | Promise | 加载页面内容 |
| `show()` | Promise | 显示窗口 |
| `setFullScreen(isFull)` | Promise | 设置全屏 |

**代码位置**: `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:26-34`

### @ohos.WallpaperExtension (壁纸扩展基类)

| 基类方法 | 调用时机 | 说明 |
|----------|----------|------|
| `onCreated(want)` | 扩展能力创建 | 壁纸窗口初始化入口 |
| `onWallpaperChanged(type)` | 壁纸变化 | 响应壁纸设置变更 |
| `onDestroy()` | 扩展能力销毁 | 资源清理 |

### @ohos.application.Ability (能力基类)

| 基类方法 | 调用时机 | 说明 |
|----------|----------|------|
| `onCreate(want, launchParam)` | 能力创建 | 初始化 |
| `onDestroy()` | 能力销毁 | 资源释放 |
| `onWindowStageCreate(windowStage)` | 窗口创建 | UI 初始化 |
| `onWindowStageDestroy()` | 窗口销毁 | UI 清理 |
| `onForeground()` | 转到前台 | 恢复状态 |
| `onBackground()` | 转到后台 | 保存状态 |

**代码位置**: `product/phone/src/main/ets/MainAbility/MainAbility.ts:18-46`

### @ohos.application.AbilityStage (能力舞台基类)

| 基类方法 | 调用时机 | 说明 |
|----------|----------|------|
| `onCreate()` | 应用启动 | 应用级初始化 |

**代码位置**: `product/phone/src/main/ets/Application/AbilityStage.ts:18-21`

## JS API 导出清单

### 常量/配置

| 名称 | 值 | 用途 |
|------|-----|------|
| `bundleName` | `com.ohos.wallpaper` | 应用包名 |
| `moduleName` | `phone-wallpaper` / `pad-wallpaper` | 模块名 |

### 权限清单

| 权限 | 用途 | 必需 |
|------|------|------|
| `ohos.permission.GET_WALLPAPER` | 获取壁纸 | ✅ 必需 |
| `ohos.permission.READ_USER_STORAGE` | 读取用户存储 | ✅ 必需 |

**代码位置**: `product/phone/src/main/module.json5:15-22`

## 错误码说明

### wallpaper API 错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| 无 | 本项目未处理错误 | 错误通过 callback 的 err 参数返回 |

### window API 错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|----------|
| 无 | 本项目未处理错误 | 错误通过 Promise reject 返回 |

## 相关文档

- [内部 API](05_Inner_API.md)
- [架构说明](03_Architecture.md)
- [安全评审](08_Security.md)
