# 问题定位

> 本章档汇总常见构建/运行/调试问题及解决路径。

## 构建问题

### 问题 1: SDK 版本不匹配

**错误表现**:
```
error: compileSdkVersion 9 is not compatible with current environment.
```

**原因**: 本地 SDK 版本与项目要求不符

**解决步骤**:

```bash
# 1. 检查当前 SDK 版本
hvigor --version

# 2. 下载匹配的 SDK
# 通过 DevEco Studio SDK Manager 下载 SDK 9

# 3. 设置 SDK 路径
export SDK_PATH=/path/to/sdk
```

**证据**: `build-profile.json5:3-4`

```json5
{
  "compileSdkVersion": 9,
  "compatibleSdkVersion": 9
}
```

### 问题 2: Node.js 版本过低

**错误表现**:
```
TypeError: Cannot read property 'hapModuleTask' of undefined
```

**原因**: Node.js 版本低于 14，不支持某些 ES6+ 特性

**解决步骤**:

```bash
# 1. 检查 Node.js 版本
node --version

# 2. 升级 Node.js (推荐 16+ LTS)
nvm install 16
nvm use 16
```

### 问题 3: hvigor 找不到

**错误表现**:
```
 hvigor: command not found
```

**原因**: hvigor 未安装或未配置环境变量

**解决步骤**:

```bash
# 1. 全局安装 hvigor
npm install -g @ohos/hvigor

# 2. 或使用项目本地 hvigor
./node_modules/.bin/hvigor assembleHap
```

### 问题 4: 模块路径错误

**错误表现**:
```
error: module path not found: ./product/phone
```

**原因**: 模块路径配置错误

**解决步骤**:

```bash
# 1. 验证目录存在
ls -la product/phone/

# 2. 检查 build-profile.json5 配置
# 确认 srcPath 正确
```

**证据**: `build-profile.json5:15`

```json5
{
  "srcPath": "./product/phone"  // 路径必须存在
}
```

## 运行问题

### 问题 1: 壁纸窗口不显示

**现象**: 壁纸扩展能力启动后，屏幕显示空白或默认壁纸

**可能原因**:
1. 窗口创建失败
2. 页面加载失败
3. 壁纸数据获取失败

**诊断步骤**:

```typescript
// 1. 查看日志确认窗口创建
// Log tag: 'ExtWallpaper : '

// 2. 检查 onCreated 是否被调用
console.info(MODULE_TAG + 'ability on created start');

// 3. 检查 windowManager.create 结果
windowManager.create(this.context, "wallpaper", 2000).then((win) => {
    console.info(MODULE_TAG, "wallpaper window loadContent success!")
}, (error) => {
    Log.showError(TAG, name + " window createFailed, error.code = " + error.code)
})

// 4. 检查壁纸数据获取
wallPaper.getPixelMap(0, (err, data) => {
    console.info(MODULE_TAG + 'ability get pixel map, err: ' + JSON.stringify(err) +
    " data: " + JSON.stringify(data));
});
```

**证据**: `product/phone/src/main/ets/WallpaperExtAbility/WallpaperExtAbility.ts:24-73`

### 问题 2: 权限被拒绝

**现象**: 壁纸数据获取失败，提示权限错误

**错误日志**:
```
Permission denied: ohos.permission.GET_WALLPAPER
```

**解决步骤**:

```bash
# 1. 检查 module.json5 是否声明权限
# product/phone/src/main/module.json5

# 2. 检查权限是否在 grace period 内
# 新权限需要用户授权

# 3. 对于系统应用，权限通常自动授予
```

**证据**: `product/phone/src/main/module.json5:15-22`

### 问题 3: AppStorage 数据不更新

**现象**: 壁纸图片不更新或显示空白

**可能原因**:
1. `slPixelData` 键名不匹配
2. @StorageLink 绑定错误
3. 数据类型不匹配

**诊断步骤**:

```typescript
// 1. 检查 AppStorage 设置
AppStorage.SetOrCreate('slPixelData', data);  // 键名: 'slPixelData'

// 2. 检查 @StorageLink 绑定
@StorageLink('slPixelData') pixelData: any = [];  // 必须匹配键名

// 3. 检查数据类型
// data 应该是 PixelMap 类型
```

**证据**:
- 设置: `WallpaperExtAbility.ts:71`
- 绑定: `pages/index.ets:21`

### 问题 4: 窗口全屏设置失败

**现象**: 壁纸窗口无法全屏显示

**错误日志**:
```
setFullScreen failed, error.code = ...
```

**解决步骤**:

```typescript
// 检查窗口状态
win.setFullScreen(true).then(() => {
    console.info(MODULE_TAG, "set full");
}).catch((error) => {
    console.error(MODULE_TAG, "setFullScreen failed: " + error.code);
});
```

**证据**: `WallpaperExtAbility.ts:30-32`

## 调试方法

### 日志查看

```bash
# 通过 hdc 查看日志
hdc shell
hidumper -s WallpaperExtAbility

# 或通过 DevEco Studio Log 窗口
# 搜索 Tag: 'ExtWallpaper : '
```

### 关键日志位置

| 文件 | 行号 | 日志 Tag |
|------|------|----------|
| WallpaperExtAbility.ts | 25 | `ability on created start` |
| WallpaperExtAbility.ts | 28 | `wallpaper window loadContent in then!` |
| WallpaperExtAbility.ts | 37 | `window createFailed` |
| WallpaperExtAbility.ts | 45 | `ability on wallpaper changed start` |
| WallpaperExtAbility.ts | 69 | `ability get pixel map` |

### 断点调试

1. **在 DevEco Studio 中打开项目**
2. **设置断点**:
   - `WallpaperExtAbility.onCreated()` - 入口断点
   - `windowManager.create().then()` - 窗口创建断点
   - `wallPaper.getPixelMap()` callback - 数据获取断点
3. **启动调试**: 点击 Debug 按钮

### 常见日志模式

| 模式 | 含义 | 处理建议 |
|------|------|----------|
| `ability on created start` | 扩展能力启动 | 正常流程 |
| `wallpaper window loadContent in then!` | 页面加载成功 | 正常流程 |
| `window createFailed` | 窗口创建失败 | 检查窗口配置 |
| `ability get pixel map, err: {}` | 壁纸获取成功 | 正常流程 |
| `ability get pixel map, err: {...}` | 壁纸获取失败 | 检查权限和服务 |

## 性能问题

### 壁纸加载慢

**现象**: 壁纸显示延迟或卡顿

**可能原因**:
1. 壁纸图片过大
2. 网络壁纸下载慢
3. 解码性能问题

**优化建议**:

```typescript
// 1. 预加载壁纸
initWallpaperImage() {
    this.sendPixelMapData();
}

// 2. 异步加载
wallPaper.getPixelMap(0, (err, data) => {
    // 异步回调，不阻塞主线程
    AppStorage.SetOrCreate('slPixelData', data);
});
```

### 内存占用高

**监控方法**:

```bash
# 查看内存使用
hdc shell
cat /proc/meminfo
```

**优化建议**:
- 及时释放不需要的 PixelMap
- 避免在 AppStorage 中存储大对象

## 相关文档

- [架构说明](03_Architecture.md)
- [安全评审](08_Security.md)
- [构建配置](06_Build.md)
- [编译产物](07_Artifacts.md)
