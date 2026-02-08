# 常见问题

## 构建问题

### Q1：编译失败，提示 "Sdk not configured"

**问题描述**：
```
Error: SDK not configured. Please configure the SDK in File → Settings → OpenHarmony SDK.
```

**解决方案**：
1. 打开 DevEco Studio
2. 进入 `File → Settings → OpenHarmony SDK`
3. 配置 SDK 路径（确保 SDK 版本为 23 或更高）
4. 点击 `Apply` 确认

**参考文档**：[如何替换 full-SDK](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/faqs/full-sdk-switch-guide.md)

---

### Q2：编译失败，提示签名验证失败

**问题描述**：
```
Error: Signature verification failed. Please check the signing configuration.
```

**解决方案**：
1. 检查 `build-profile.json5` 中的签名配置是否正确
2. 确认签名文件存在于 `signature/` 目录：
   - `OpenHarmony.p12`（密钥库）
   - `OpenHarmonyApplication.cer`（证书）
   - `privacyCenter.p7b`（Profile）
3. 如使用默认签名，确保 Profile 与包名匹配

**证据来源**：`build-profile.json5:18-30`

---

### Q3：编译时提示 "Module not found"

**问题描述**：
```
Error: Cannot find module '@ohos.xxx'.
```

**解决方案**：
1. 确认 full-SDK 已正确替换（见 Q1）
2. 在项目根目录执行 `Sync Project with Gradle Files`
3. 检查 `module.json5` 中的 `requestPermissions` 配置是否正确

---

### Q4：增量编译不生效

**问题描述**：
修改代码后重新编译，但修改未生效。

**解决方案**：
1. 执行 `Build → Clean Project`
2. 删除 `entry/build/` 目录
3. 重新编译

---

## 运行问题

### Q5：应用安装后无法启动

**问题描述**：
应用安装成功，但点击图标无响应或闪退。

**排查步骤**：
1. 检查日志：`hdc shell hilog | grep "EntryAbility"`
2. 确认入口配置正确：`module.json5:29-30`
3. 检查主页面路径：`EntryAbility.ets:37`
4. 验证页面是否在 `main_pages.json` 中注册

**证据来源**：
- `EntryAbility.ets:37`（加载 Index 页面）
- `resources/base/profile/main_pages.json`（页面配置）

---

### Q6：位置开关控制无反应

**问题描述**：
点击位置开关按钮，但系统位置状态未改变。

**排查步骤**：
1. 确认已授予 `ohos.permission.CONTROL_LOCATION_SWITCH` 权限
2. 检查 `LocationService` 是否正确初始化：
   ```typescript
   // LocationViewModel.ets:26-35
   initViewModel() {
     LocationService.registerListener(this);
     LocationService.startService();
     LocationService.getServiceState();
   }
   ```
3. 查看日志：`hdc shell hilog | grep "LocationService"`

**证据来源**：`LocationService.ets`、`LocationViewModel.ets`

---

### Q7：菜单列表为空或不完整

**问题描述**：
隐私菜单列表为空，或显示的菜单项不完整。

**排查步骤**：
1. 确认 BMS 中已注册菜单配置
2. 检查 RDB 是否损坏：
   ```typescript
   // AutoMenuModel.ets:45-52
   if (resultSet === undefined) {
     return menuInfoList;  // 返回空数组
   }
   ```
3. 手动刷新菜单：`Index.ets:58`（onPageShow 时刷新）
4. 查看日志：`hdc shell hilog | grep "AutoMenu"`

**证据来源**：`AutoMenuModel.ets:40-104`

---

### Q8：UIExtension 页面显示异常

**问题描述**：
跳转到 UiExtensionPage 后，第三方 UI 显示异常或空白。

**排查步骤**：
1. 确认目标应用已声明 UIExtensionAbility
2. 检查参数传递：
   ```typescript
   // UiExtensionPage.ets:31-36
   UIExtensionComponent({
     bundleName: this.dstBundleName,
     abilityName: this.dstAbilityName,
     parameters: {
       'ability.want.params.uiExtensionType': 'sys/commonUI',
     }
   })
   ```
3. 确认 `dstAbilityMode = 1`（DST_ABILITY_MODE）

**证据来源**：`UiExtensionPage.ets`、`AutoMenuModel.ets:132-143`

---

### Q9：页面跳转失败

**问题描述**：
点击菜单项后，跳转失败或提示 "page not found"。

**排查步骤**：
1. 检查 `dstAbilityMode`：
   - `0` = UIAbility 跳转（需 startAbility）
   - `2` = 页面跳转（需 router.pushUrl）
2. 确认路由模式：`RouterMode.Single` / `Standard`
3. 检查目标页面是否在 `main_pages.json` 中注册
4. 查看日志：`hdc shell hilog | grep "router"`

**证据来源**：`AutoMenuModel.ets:107-145`

---

### Q10：国际化资源不显示

**问题描述**：
多语言资源未正确加载，显示默认语言而非期望语言。

**排查步骤**：
1. 确认资源文件位置正确：
   ```
   resources/
   ├── base/element/string.json      # 默认
   ├── zh_CN/element/string.json     # 中文
   └── en_US/element/string.json     # 英文
   ```
2. 检查资源名称是否一致
3. 重启应用使语言设置生效
4. 查看日志：`hdc shell hilog | grep "ResourceUtil"`

**证据来源**：`ResourceUtil.ets`

---

## 调试技巧

### 查看应用日志

```bash
# 查看所有日志
hdc shell hilog

# 过滤特定标签
hdc shell hilog | grep "EntryAbility"
hdc shell hilog | grep "LocationService"
hdc shell hilog | grep "AutoMenu"

# 按优先级过滤
hdc shell hilog -s D       # Debug
hdc shell hilog -s I       # Info
hdc shell hilog -s W       # Warning
hdc shell hilog -s E       # Error
```

### 查看应用信息

```bash
# 查看已安装应用
hdc shell bm dump -n com.ohos.security.privacycenter

# 查看应用数据目录
hdc shell cd /data/app/el2/100/base/com.ohos.security.privacycenter/
```

### 调试页面渲染

```typescript
// 在代码中添加日志
Logger.info(TAG, 'Page onPageShow');
Logger.info(TAG, `Menu count: ${menuInfoList.length}`);
```

---

## 性能问题

### Q11：页面启动慢

**排查方向**：
1. 检查 `onCreate` 初始化逻辑（`EntryAbility.ets:25-27`）
2. 检查 `onWindowStageCreate` 加载内容（`EntryAbility.ets:33-43`）
3. 检查首页数据加载（`Index.ets:56-78`）

**优化建议**：
- 延迟加载非关键数据
- 使用懒加载优化列表渲染

---

### Q12：内存占用高

**排查方向**：
1. 检查 AppStorage 中存储的数据大小
2. 检查 RDB 查询是否返回过多数据
3. 检查是否存在内存泄漏（未取消的事件监听）

**证据来源**：
- `LocationService.ets:28-37`（位置变化监听）
- `AutoMenuModel.ets:86-104`（BMS 查询）

---

## 相关资源

| 资源 | 链接 |
|-----|------|
| DevEco Studio 下载 | https://developer.huawei.com/consumer/cn/deveco-studio/ |
| OpenHarmony 文档 | https://gitee.com/openharmony/docs |
| 应用接入指南 | https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/SecurityPrivacyCenter/auto-menu-guidelines.md |
| full-SDK 替换指南 | https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/faqs/full-sdk-switch-guide.md |

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [08_Security_Review.md](./08_Security_Review.md) → 安全评审
- [README.md](./README.md) → 文档说明
