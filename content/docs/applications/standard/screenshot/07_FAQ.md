# 常见问题与定位指南

## 构建问题

### Q1: hvigor 命令未找到

**问题描述**:
```
hvigor: command not found
```

**解决方案**:
```bash
# 安装 hvigor
npm install -g @ohos/hvigor

# 或在项目目录下
npm install
```

**参考**: `oh-package.json5`

---

### Q2: 编译 SDK 版本不匹配

**问题描述**:
```
error: incompatible SDK version
```

**解决方案**:
1. 检查 `build-profile.json5` 配置
2. 确认 DevEco Studio SDK 版本
3. 更新 compileSdkVersion 至匹配版本

**参考配置**:
```json5
{
  "compileSdkVersion": 23,
  "compatibleSdkVersion": 23,
  "targetSdkVersion": 23
}
```

---

### Q3: 模块依赖错误

**问题描述**:
```
error: module not found
```

**解决方案**:
1. 检查模块路径是否正确
2. 确认模块已在 build-profile.json5 中声明
3. 检查模块构建产物是否存在

---

## 运行问题

### Q4: 截屏功能无响应

**问题描述**: 触发截屏后没有反应

**排查步骤**:

1. **检查日志**
   ```bash
   # 使用 hdc 查看日志
   hdc shell hidumper
   # 或
   hdc shell log
   ```

2. **检查权限**
   - 确认应用已声明 `CAPTURE_SCREEN` 权限
   - 确认权限已授予

3. **检查 ServiceExtAbility**
   ```typescript
   // 文件: product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets
   // 检查 onCreate 是否正常执行
   ```

4. **检查窗口创建**
   ```typescript
   // 检查 window.create 是否成功
   windowManager.createWindow(...).then((win) => {
       // 确认进入此回调
   }).catch((error) => {
       // 检查错误信息
   });
   ```

---

### Q5: 截图保存失败

**问题描述**: 截屏成功但保存失败

**排查步骤**:

1. **检查权限**
   ```json5
   // module.json5 中应声明
   {
     "name": "ohos.permission.WRITE_IMAGEVIDEO"
   }
   ```

2. **检查相册路径**
   ```typescript
   // 文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets
   const SCREEN_SHOT_PATH = 'Screenshots/';
   ```

3. **检查文件操作**
   ```typescript
   // 确认文件句柄正确打开和关闭
   const fd = await fileAsset.open('w');
   await file.write(fd, packedImg);
   await file.fsync(fd);
   await fileAsset.close(fd);
   ```

4. **检查错误日志**
   ```typescript
   } catch (error) {
       Log.showInfo(TAG, `SaveImage failed, cause: ${error}`);
   }
   ```

---

### Q6: 预览窗口显示异常

**问题描述**: 浮动窗口位置或大小不正确

**排查步骤**:

1. **检查窗口配置**
   ```typescript
   // 文件: product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets
   const ZOOM_RATIO = 0.4;  // 40% 缩放
   const WINDOW_Y = 300;    // Y 位置
   ```

2. **检查显示信息获取**
   ```typescript
   const dis = display.getDefaultDisplaySync();
   win.resize(dis.width * ZOOM_RATIO, dis.height * ZOOM_RATIO);
   ```

3. **检查 UI 配置**
   ```typescript
   // 文件: product/phone/src/main/ets/common/constants.ets
   public static FULL_CONTAINER_WIDTH = '100%';
   public static FULL_CONTAINER_HEIGHT = '100%';
   ```

---

### Q7: 自动关闭功能失效

**问题描述**: 5秒后窗口未自动关闭

**排查步骤**:

1. **检查定时器**
   ```typescript
   // 文件: product/phone/src/main/ets/pages/index.ets
   setTimeout(ViewModel.CloseShotScreen, Constants.interval);
   // Constants.interval = 5000
   ```

2. **检查关闭逻辑**
   ```typescript
   // 文件: product/phone/src/main/ets/vm/ViewModel.ets
   CloseShotScreen(): void {
       ShotScreenModel.dismiss();
   }
   ```

3. **检查 dismiss 实现**
   ```typescript
   // 文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets
   dismiss(): void {
       globalThis.shotScreenContext.terminateSelf(...);
       WindowMar.find(Constants.WIN_NAME).then((win) => {
           win.destroy(...);
       });
   }
   ```

---

### Q8: 无法打开相册

**问题描述**: 点击截图后未打开相册

**排查步骤**:

1. **检查 Want 配置**
   ```typescript
   // 文件: product/phone/src/main/ets/vm/ViewModel.ets
   const wantData: Want = {
       bundleName: 'com.ohos.photos',
       abilityName: 'com.ohos.photos.MainAbility',
       parameters: { uri: imageFileName }
   };
   ```

2. **检查相册应用**
   - 确认相册应用已安装
   - 确认 bundleName 正确

3. **检查 URI 传递**
   ```typescript
   // 文件: features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets
   AppStorage.setOrCreate('imageUri', fileAsset.uri);
   ```

---

## 调试方法

### 日志查看

```bash
# 查看应用日志
hdc shell log | grep -i screenshot

# 或使用 HiLog
hdc shell hidumper -s AppSpawner
```

### 窗口调试

```typescript
// 添加调试日志
WindowMar.find(Constants.WIN_NAME).then((win) => {
    Log.showInfo(TAG, `Window found: ${JSON.stringify(win)}`);
    win.show(...);
});
```

### 性能分析

```bash
# 使用 DevEco Studio Profiler
# 1. 打开 Profiler
# 2. 选择设备和应用
# 3. 录制性能数据
```

---

## 日志标签说明

| 标签 | 模块 | 用途 |
|-----|------|------|
| `ScreenShot-ScreenShotModel` | ScreenShotModel | 核心逻辑日志 |
| `ScreenShot-Index` | Index 页面 | UI 日志 |
| `ScreenShot-ViewModel` | ViewModel | 页面逻辑日志 |
| `ScreenShot-ScreenShotServiceAbility` | ServiceExtAbility | 服务日志 |
| `ScreenShot-Default` | Log 工具 | 默认日志 |
| `AVScreenCapture-DiaLogPage` | DialogPage | 对话框日志 |
| `AVScreenCapture-DiaLogUIExtensionAbility` | DialogAbility | 对话框能力日志 |

**日志域**: `0x55EE`
**日志前缀**: `[Screenshot]`

---

## 相关资源

### 调试命令

| 命令 | 用途 |
|-----|------|
| `hdc shell log` | 查看日志 |
| `hdc shell hidumper` | 系统信息 |
| `hdc install` | 安装应用 |
| `hdc uninstall` | 卸载应用 |

### 日志级别

| 级别 | 方法 | 说明 |
|-----|------|------|
| INFO | `Log.showInfo()` | 普通信息 |
| DEBUG | `Log.showDebug()` | 调试信息 |
| ERROR | `Log.showError()` | 错误信息 |

### 关键文件路径

| 用途 | 路径 |
|-----|------|
| ServiceExtAbility | `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ets` |
| ScreenShotModel | `features/screenshot/src/main/ets/com/ohos/model/screenShotModel.ets` |
| ViewModel | `product/phone/src/main/ets/vm/ViewModel.ets` |
| 模块配置 | `product/phone/src/main/module.json5` |
| 构建配置 | `build-profile.json5` |
