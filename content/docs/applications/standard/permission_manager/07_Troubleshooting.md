# 常见问题排查

> **目的**: 记录PermissionManager构建、运行、调试中的常见问题及定位方法  
> **适用范围**: 开发者、测试人员、维护人员  
> **最后更新**: 2026-02-05

---

## 构建问题

### 1. 编译失败：找不到模块

**错误信息**:
```
ERROR: Cannot find module '@ohos.abilityAccessCtrl'
ERROR: Cannot find module '@ohos.bundle.bundleManager'
```

**原因**: 
- SDK未正确配置
- 导入路径错误

**解决方案**:
1. 检查 `BUILD.gn` 中的SDK配置:
   ```gn
   sdk_home = "//prebuilts/ohos-sdk/linux"
   ```

2. 确认SDK存在:
   ```bash
   ls -la prebuilts/ohos-sdk/linux/
   ```

3. 检查导入语句:
   ```typescript
   // 正确
   import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
   import { bundleManager } from '@kit.AbilityKit';
   
   // 错误
   import abilityAccessCtrl from '@ohos.abilityAccessCtrl.js';  // 不需要.js后缀
   ```

**相关代码**: 所有使用系统API的源文件

---

### 2. 签名失败

**错误信息**:
```
ERROR: Sign hap failed
ERROR: Certificate not found: signature/pm.p7b
```

**原因**:
- 签名证书不存在
- 证书过期
- 签名工具路径错误

**解决方案**:
1. 检查证书文件:
   ```bash
   ls -la applications/standard/permission_manager/signature/pm.p7b
   ```

2. 检查签名配置:
   ```gn
   # BUILD.gn
   certificate_profile = "signature/pm.p7b"
   ```

3. 使用调试签名（开发环境）:
   ```bash
   ./build.sh --product-name rk3568 --build-target permission_manager \
       --sign-hap-py-path {sdk_path}/sign_hap.py
   ```

**相关文件**: `BUILD.gn:22`, `signature/pm.p7b`

---

### 3. 资源编译失败

**错误信息**:
```
ERROR: Resource compilation failed
ERROR: Invalid resource file: resources/base/element/string.json
```

**原因**:
- JSON格式错误
- 资源ID冲突
- 缺少必要资源

**解决方案**:
1. 验证JSON格式:
   ```bash
   cat permissionmanager/src/main/resources/base/element/string.json | python -m json.tool
   ```

2. 检查资源引用:
   ```typescript
   // 确保引用的资源存在
   $r('app.string.permission_manager')  // 检查string.json中是否有定义
   $r('app.media.app_icon')             // 检查media目录是否存在
   ```

3. 检查 `module.json` 中的pages配置:
   ```json
   {
       "pages": "$profile:main_pages"
   }
   ```

**相关文件**: `permissionmanager/src/main/resources/`, `permissionmanager/src/main/module.json`

---

## 运行问题

### 1. Ability启动失败

**错误信息**:
```
[PermissionManager_Log] MainAbility onCreate failed
[PermissionManager_Log] permission status is denied.
```

**原因**:
- 缺少必要权限
- Ability配置错误

**解决方案**:
1. 检查权限声明 (`module.json`):
   ```json
   {
       "requestPermissions": [
           {
               "name": "ohos.permission.GET_INSTALLED_BUNDLE_LIST"
           }
       ]
   }
   ```

2. 检查权限获取代码:
   ```typescript
   // MainAbility.ts:141-157
   private permissionCheck(): boolean {
       let atManager = abilityAccessCtrl.createAtManager();
       let status = atManager.verifyAccessTokenSync(
           bundleInfo.appInfo.accessTokenId, 
           'ohos.permission.GET_INSTALLED_BUNDLE_LIST'
       );
       if (status === abilityAccessCtrl.GrantStatus.PERMISSION_DENIED) {
           return false;
       }
       return true;
   }
   ```

3. 如果是系统应用，确认已正确签名

**相关代码**: `MainAbility.ts:141-157`, `module.json:93-132`

---

### 2. 权限请求对话框不显示

**错误信息**:
```
[PermissionManager] ServiceExtensionAbility terminateSelf
```

**原因**:
- 设备类型不支持（wearable跳过）
- 窗口创建失败
- 参数解析失败

**解决方案**:
1. 检查设备类型:
   ```typescript
   // ServiceExtAbility.ets:40-44
   if (deviceInfo.deviceType === 'wearable') {
       this.context.terminateSelf();
       return;
   }
   ```

2. 检查窗口创建日志:
   ```bash
   hdc shell hilog | grep "createWindow"
   ```

3. 检查参数解析:
   ```typescript
   // GrantDialogModel.ets:56-88
   // 确保want.parameters包含必要字段
   ```

**相关代码**: `ServiceExtAbility.ets:40-44`, `GrantDialogModel.ets:225-254`

---

### 3. 权限授权失败

**错误信息**:
```
grant permission faild, permission: ohos.permission.CAMERA, code: xxx, message: xxx
```

**原因**:
- Token无效
- 权限不存在
- 系统服务异常

**解决方案**:
1. 检查Token有效性:
   ```typescript
   // GrantDialogModel.ets:426-443
   // 确保tokenId > 0
   ```

2. 检查权限名称:
   ```typescript
   // 使用definition.ets中定义的枚举
   Permission.CAMERA  // 'ohos.permission.CAMERA'
   ```

3. 查看详细错误:
   ```bash
   hdc shell hilog | grep -i "permission"
   ```

**相关代码**: `GrantDialogModel.ets:426-469`

---

### 4. IPC通信失败

**错误信息**:
```
write result failed: {...}
terminateWithResult faild, code: xxx, message: xxx
```

**原因**:
- 回调对象无效
- IPC服务未启动
- 序列化失败

**解决方案**:
1. 检查回调对象:
   ```typescript
   // GrantDialogModel.ets:66-67
   let callback: Property = want.parameters['ohos.ability.params.callback'] as Property;
   let proxy: rpc.RemoteObject = callback.value as rpc.RemoteObject;
   // 确保proxy有效
   ```

2. 检查IPC调用:
   ```typescript
   // GrantDialogModel.ets:496-522
   // 确保使用try-catch-finally
   ```

3. 验证系统服务状态:
   ```bash
   hdc shell ps -ef | grep accesstoken
   ```

**相关代码**: `GrantDialogModel.ets:496-522`

---

## 调试方法

### 1. 日志查看

**启用详细日志**:
```bash
# 关闭PID过滤
hdc shell hilog -Q pidoff

# 设置日志级别
hdc shell hilog -b D

# 清除日志缓冲区
hdc shell hilog -r

# 抓取日志
hdc hilog > log.txt
```

**过滤PermissionManager日志**:
```bash
hdc shell hilog | grep -i "permissionmanager"
```

**关键日志TAG**:
- `PermissionManager_Log` - MainAbility日志
- `PermissionManager` - 其他模块日志

**相关文档**: `README.md:77-89`

---

### 2. 代码中添加日志

```typescript
// 在关键位置添加日志
const TAG = "PermissionManager_Debug";

// 在Ability生命周期
console.info(TAG + ' onCreate: ' + JSON.stringify(want));

// 在权限操作
console.info(TAG + ' grant permission: ' + permission);

// 在错误处理
console.error(TAG + ' error: ' + JSON.stringify(error));
```

---

### 3. 使用DevEco Studio调试

1. **配置项目**:
   - 打开项目根目录
   - 等待项目同步完成

2. **设置断点**:
   - 在 `ServiceExtAbility.ets` 的 `onRequest` 方法
   - 在 `GrantDialogModel.ets` 的 `createWindow` 方法
   - 在 `dialogPlus.ets` 的点击事件处理

3. **连接设备**:
   - 确保设备已连接
   - hdc设备列表: `hdc list targets`

4. **启动调试**:
   - 选择调试配置
   - 点击调试按钮

---

### 4. 常见问题定位路径

```
问题: 权限请求对话框不显示
    │
    ├── 检查ServiceExtAbility是否启动
    │   └── hdc shell hilog | grep "ServiceExtensionAbility onRequest"
    │
    ├── 检查设备类型
    │   └── ServiceExtAbility.ets:40-44 (是否被wearable跳过)
    │
    ├── 检查窗口创建
    │   └── GrantDialogModel.ets:225-254 (createWindow日志)
    │
    └── 检查页面加载
        └── dialogPlus.ets (是否执行build)

问题: 权限授权失败
    │
    ├── 检查tokenId
    │   └── GrantDialogModel.ets:78 (是否有效)
    │
    ├── 检查权限名称
    │   └── definition.ets (是否正确)
    │
    ├── 检查授权调用
    │   └── GrantDialogModel.ets:430 (grantUserGrantedPermission)
    │
    └── 查看错误码
        └── hilog日志

问题: MainAbility无法显示应用列表
    │
    ├── 检查权限
    │   └── MainAbility.ets:141-157 (permissionCheck)
    │
    ├── 检查bundle查询
    │   └── MainAbility.ets:169 (getAllBundleInfo)
    │
    └── 检查日志
        └── hdc shell hilog | grep "MainAbility"
```

---

## 性能问题

### 1. 应用列表加载慢

**原因**:
- 应用数量多
- 同步查询Ability信息

**优化建议**:
```typescript
// MainAbility.ets:175-193
// 当前代码同步查询每个应用的Ability信息
for (let i = 0; i < bundleInfos.length; i++) {
    await bundleManager.queryAbilityInfo(...);  // 同步调用
}

// 建议: 使用异步并行查询
const promises = bundleInfos.map(info => 
    bundleManager.queryAbilityInfo(...).catch(() => null)
);
const results = await Promise.all(promises);
```

**相关代码**: `MainAbility.ets:175-193`

---

### 2. 内存泄漏

**可能位置**:
- 窗口未正确销毁
- 事件监听器未移除
- GlobalContext未清理

**检查方法**:
```bash
# 监控内存使用
hdc shell hidumper -s WindowManagerService -a -a

# 查看进程内存
hdc shell ps -ef | grep permissionmanager
```

**修复建议**:
```typescript
// 确保在onDestroy中清理
onDestroy(): void {
    // 移除事件监听
    bundleMonitor.off('add');
    bundleMonitor.off('remove');
    
    // 清理全局状态
    GlobalContext.store('dialogSet', null);
}
```

**相关代码**: `MainAbility.ets:118-127`, `ServiceExtAbility.ets:64-66`

---

## 设备兼容性问题

### 1. Wearable设备不支持

**说明**: 某些功能在wearable设备上被禁用

**代码处理**:
```typescript
// ServiceExtAbility.ets:40-44
if (deviceInfo.deviceType === 'wearable') {
    this.context.terminateSelf();
    return;
}

// PermissionStateSheetAbility.ets:36-40
if (deviceInfo.deviceType === 'wearable') {
    this.context.terminateSelf();
    return;
}
```

**解决方案**: 这是设计行为，wearable设备使用其他权限管理方式

---

### 2. 不同屏幕尺寸适配

**问题**: UI在不同屏幕尺寸上显示异常

**解决方案**:
```typescript
// 获取屏幕尺寸
let dis = display.getDefaultDisplaySync();
let width = dis.width;
let height = dis.height;

// 根据尺寸调整UI
// dialogPlus.ets, authority-management.ets 等页面
```

**相关代码**: `ServiceExtAbility.ets:47-53`, `SecurityExtAbility.ets:42-54`

---

## 调试技巧

### 1. 模拟权限请求

```typescript
// 在测试应用中添加
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';

async function testPermissionRequest(): Promise<void> {
    let atManager = abilityAccessCtrl.createAtManager();
    atManager.requestPermissionsFromUser(
        getContext(),
        ['ohos.permission.CAMERA', 'ohos.permission.MICROPHONE']
    ).then((data) => {
        console.info('Result: ' + JSON.stringify(data));
    });
}
```

### 2. 手动触发Ability

```bash
# 启动MainAbility
hdc shell am start -a action.access.privacy.center

# 启动GrantAbility（需要正确参数，仅测试用）
hdc shell am start -n com.ohos.permissionmanager/com.ohos.permissionmanager.GrantAbility
```

### 3. 查看运行时状态

```bash
# 查看Ability状态
hdc shell aa dump -a

# 查看窗口状态
hdc shell wm dump

# 查看权限状态
hdc shell accesstoken_dump
```

---

## 问题反馈模板

报告问题时，请提供以下信息:

```markdown
## 问题描述
简要描述问题

## 复现步骤
1. 步骤1
2. 步骤2
3. ...

## 期望行为
描述期望的行为

## 实际行为
描述实际的行为

## 日志信息
```
[粘贴相关日志]
```

## 环境信息
- OpenHarmony版本: 
- 设备类型: 
- PermissionManager版本: 

## 附加信息
截图、录屏等
```

---

*上一篇: [安全风险评审](06_Security_Analysis.md) | 返回 [README](README.md) | [附录：调用链](appendix/Callgraphs.md)*
