# 问题排查指南

## 1. 构建问题

### 1.1 编译失败

#### 问题 1：模块找不到

**现象**:
```
error: failed to resolve module '@ohos/common'
```

**原因**: HAR 模块未正确编译或引用

**排查步骤**:
```bash
# 1. 检查模块是否存在
ls -la ./common/src/main/ets/

# 2. 检查模块名
cat ./common/src/main/module.json5 | grep name

# 3. 检查导入路径
cat ./product/phone/src/main/ets/MainAbility/MainAbility.ts | grep "@ohos/common"
```

**解决方案**:
1. 确保执行了完整构建：
```bash
hvigor clean
hvigor assembleHap --product default
```

2. 检查模块配置：
```json5
// common/src/main/module.json5
{
  "name": "photos_common"
}
```

---

#### 问题 2：签名验证失败

**现象**:
```
error: signature verification failed
```

**原因**: 签名文件缺失或配置错误

**排查步骤**:
```bash
# 1. 检查签名文件是否存在
ls -la ./signature/

# 2. 检查签名配置
cat ./build-profile.json5 | grep -A 20 signingConfigs
```

**解决方案**:
1. 确认签名文件完整：
```
signature/
├── OpenHarmonyApplication.cer
├── OpenHarmony.p12
└── photos.p7b
```

2. 配置正确的签名密码

> **证据**: `README_zh.md:184-198`

---

### 1.2 构建产物问题

#### 问题 3：HAP 过大

**现象**:
构建产物超出预期大小

**排查步骤**:
```bash
# 查看产物大小
ls -lh ./build/outputs/hap/release/phone_photos/default/photos.hap

# 检查资源文件
du -sh ./product/phone/src/main/resources/
```

**解决方案**:
1. 检查是否有未使用的资源文件
2. 优化图片资源大小
3. 移除不必要的媒体文件

---

## 2. 运行问题

### 2.1 应用启动失败

#### 问题 4：MainAbility 崩溃

**现象**:
应用启动后立即闪退，logcat 显示：
```
fatal error: Cannot read property 'xxx' of undefined
```

**排查步骤**:
```bash
# 查看崩溃日志
hilog | grep -E "Photos|FATAL|Error"

# 过滤 MainAbility 日志
hilog | grep "MainAbility"
```

**常见原因**:
1. `AppStorage` 中必要数据未初始化
2. `UserFileManagerAccess` 初始化失败
3. 路由页面不存在

**证据定位**:
- `MainAbility.ts:60-79` - `onCreate()` 初始化
- `MainAbility.ts:181-209` - `onWindowStageCreate()`

**解决方案**:
```typescript
// 确保 AppStorage 初始化完整
AppStorage.setOrCreate('photosAbilityContext', this.context);
AppStorage.setOrCreate('formContext', this.context);
```

---

#### 问题 5：页面路由失败

**现象**:
```
error: router.replaceUrl failed, the page is not exists
```

**排查步骤**:
```bash
# 检查页面是否存在
ls -la ./product/phone/src/main/ets/pages/

# 检查路由配置
cat ./product/phone/src/main/ets/MainAbility/MainAbility.ts | grep router
```

**常见原因**:
1. 页面文件拼写错误
2. 路由页面路径错误
3. 模块未正确引用

**证据定位**:
- `MainAbility.ts:243` - `router.replaceUrl('pages/ThirdSelectPhotoGridPage')`

**解决方案**:
```typescript
// 检查页面路径
router.replaceUrl({
  url: 'pages/ThirdSelectPhotoGridPage',  // 确认路径正确
  params: { ... }
});
```

---

### 2.2 外部调用问题

#### 问题 6：startAbility 无响应

**现象**:
第三方应用调用图库无任何响应

**排查步骤**:
```bash
# 检查目标应用是否安装
hdc shell bm dump -a | grep photos

# 检查应用状态
hdc shell app run
```

**常见原因**:
1. 图库应用未安装
2. Want 参数格式错误
3. URI scheme 不匹配

**排查代码**:
```typescript
// 检查 URI 参数
let wantParamUri = wantParam?.uri as string;
if (wantParamUri === Constants.WANT_PARAM_URI_DETAIL) {
  // 正确
}
```

> **证据**: `MainAbility.ts:99-166`

---

#### 问题 7：选择结果返回为空

**现象**:
`startAbilityForResult` 返回但 `select-item-list` 为空

**排查步骤**:
```bash
# 检查返回数据
hilog | grep -E "select-item-list|resultCode"
```

**常见原因**:
1. 用户未选择任何图片
2. 页面跳转逻辑错误
3. 返回结果处理异常

**排查代码**:
```typescript
// 检查返回处理
if (want != null && want != undefined) {
  let param = want['parameters'];
  if (param != null && param != undefined) {
    let uri = param['select-item-list'];
    // 确保 uri 有值
  }
}
```

> **证据**: `README_zh.md:147-154`

---

## 3. 性能问题

### 3.1 卡顿/掉帧

#### 问题 8：图片加载缓慢

**现象**:
图片宫格加载慢，滑动卡顿

**排查步骤**:
```bash
# 查看帧率
hdc shell ps -ef | grep photos

# 检查内存
hdc shell cat /proc/meminfo
```

**优化建议**:
1. 使用 LazyForEach 懒加载
2. 实现图片缓存
3. 优化缩略图质量

**证据定位**:
- `common/model/browser/photo/UriDataSource.ts`
- `common/model/browser/photo/FifoCache.ts`

---

#### 问题 9：内存占用过高

**现象**:
长时间使用后应用内存持续增长

**排查步骤**:
```bash
# 查看内存使用
hdc shell dumpsys meminfo com.ohos.photos
```

**排查方向**:
1. DataSource 是否正确释放
2. 观察者是否正确注销
3. 图片缓存是否无限增长

**证据定位**:
- `MainAbility.ts:169-179` - `onDestroy()` 资源释放
- `MainAbility.ts:176-177` - MediaObserver 注销

**解决方案**:
```typescript
onDestroy(): void {
  // 确保所有观察者注销
  MediaObserver.getInstance().unregisterForAllPhotos();
  MediaObserver.getInstance().unregisterForAllAlbums();
}
```

---

## 4. 日志查看

### 4.1 常用日志命令

```bash
# 查看图库所有日志
hilog | grep Photos

# 查看 MainAbility 日志
hilog | grep "MainAbility"

# 查看错误日志
hilog | grep -E "Error|FATAL|CRASH"

# 过滤特定标签
hilog | grep "TAG_NAME"

# 导出完整日志
hilog > hilog.log
```

### 4.2 日志标签

| 组件 | 标签 | 说明 |
|------|------|------|
| MainAbility | `MainAbility` | 主能力 |
| Common | `@ohos/common` | 通用模块 |

> **证据**: `MainAbility.ts:42` - `const TAG: string = 'MainAbility'`

---

## 5. 调试技巧

### 5.1 使用断点

在 DevEco Studio 中：
1. 打开 `.ets` 文件
2. 在代码行号左侧点击设置断点
3. 点击 Debug 按钮启动调试

### 5.2 打印调试

```typescript
import { Log } from '@ohos/common';

const TAG = 'Debug';
Log.info(TAG, `Variable value: ${JSON.stringify(data)}`);
```

### 5.3 性能分析

```bash
# 使用 DevEco Studio Profiler
# 1. 打开 Profiler
# 2. 选择 com.ohos.photos
# 3. 录制性能数据
```

---

## 6. 常见问题 FAQ

### Q1: 如何调试第三方调用？

A: 使用 `hdc shell am start` 命令模拟调用：
```bash
hdc shell am start -n com.ohos.photos/com.ohos.photos.MainAbility \
  -b com.ohos.photos \
  --ela "uri=singleselect" \
  --epc "callerBundleName=com.example.test"
```

### Q2: 如何查看页面栈？

A: 在 `onNewWant` 中添加日志：
```typescript
onNewWant(want: Want): void {
  Log.info(TAG, `onNewWant: ${JSON.stringify(want)}`);
}
```

### Q3: 如何确认 MediaObserver 正常工作？

A: 添加媒体文件后检查日志：
```bash
hilog | grep -E "MediaObserver|registerForAll"
```

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [05_Build_System.md](05_Build_System.md) | 构建系统 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物 |
| [README_zh.md](../README_zh.md) | 原始开发说明 |
