# 90_FAQ - 常见问题

## 构建问题

### Q1: 如何单独构建 auth_widget 模块？

**问题**: 想只编译 auth_widget 而不编译整个系统

**解决方案**: 使用 `--build-target` 参数指定构建目标

**证据**: `README.md` 行 34-35

```bash
./build.sh --product-name rk3568 --ccache --build-target auth_widget
```

### Q2: 构建失败，提示找不到 `ohos_hap`

**问题**: GN 模板 `ohos_hap` 未定义

**解决方案**: 确保在 OpenHarmony 源码根目录执行构建命令

**说明**: `ohos_h` 模板是 OpenHarmony 构建系统的组成部分

### Q3: 编译产物 `.hap` 的位置？

**问题**: 想知道编译产物的输出路径

**解决方案**: 产物位于 out 目录下的 packages 文件夹

**证据**: `BUILD.gn` 行 46

```gn
module_install_dir = "app/com.ohos.useriam.authwidget"
```

**预期路径**: `out/{product}/packages/system/app/com.ohos.useriam.authwidget/`

## 运行问题

### Q4: 认证对话框不显示

**问题**: 调用认证 API 后没有弹出对话框

**可能原因**:
1. 权限未配置
2. Want 参数不完整
3. 认证类型不支持

**排查步骤**:
```typescript
// 检查权限配置
// module.json 中应包含:
"requestPermissions": [
  "ohos.permission.ACCESS_PIN_AUTH",
  "ohos.permission.ACCESS_BIOMETRIC",
  "ohos.permission.SUPPORT_USER_AUTH"
]
```

### Q5: 指纹认证选项不显示

**问题**: PIN 和人脸可以，但指纹认证没有出现

**可能原因**:
1. 设备不支持指纹
2. `FINGERPRINT` 未在 authType 中
3. 指纹传感器信息缺失

**排查步骤**:
```typescript
// Index.ets:39 - 检查认证类型
@State authType: Array<userAuth.UserAuthType> = [userAuth.UserAuthType.PIN];

// 检查 cmdData 中是否包含 fingerprint 类型
```

### Q6: 横屏模式下认证界面异常

**问题**: 横屏设备上显示提示信息而非认证界面

**证据**: `Index.ets` 行 262-282

**说明**: 这是设计行为，横屏+屏下指纹时显示提示而非认证界面

```typescript
if (this.isLandscape && this.underFingerPrint) {
  // 显示提示：请在竖屏模式验证
}
```

## 调试问题

### Q7: 如何查看认证组件的日志？

**解决方案**: 使用 `hilog` 工具查看日志

```bash
hilog | grep "useriam_auth_widget"
```

**证据**: `LogUtils.ts` 行 31

```typescript
const TAG = 'useriam_auth_widget';
```

### Q8: 如何调试 PIN 输入流程？

**说明**: PIN 输入涉及 `PasswordAuth` 组件和 `PINAuth` 的交互

**证据**: `PasswordAuth.ets` 行 102-110

```typescript
pinAuthManager = new account_osAccount.PINAuth();
pinAuthManager.registerInputer({
  onGetData: (authSubType, callback) => {
    const uint8PW = FuncUtils.getUint8PW(pinData);
    callback.onSetData(authSubType, uint8PW);
  }
});
```

### Q9: 如何追踪 user_auth_framework 的通信？

**说明**: 通过 `userAuthWidgetMgr.on('command')` 接收指令

**证据**: `Index.ets` 行 176-191

```typescript
userAuthWidgetMgr.on('command', {
  sendCommand: (result) => {
    const cmdDataObj: WidgetCommand = JSON.parse(result || '{}');
    // 处理认证指令
  }
});
```

## 功能问题

### Q10: 如何自定义认证对话框的标题？

**说明**: 通过 `WantParams.title` 传递自定义标题

**证据**: `Constants.ts` 行 175

```typescript
interface WantParams {
  title: string;  // 对话框标题
}
```

### Q11: 如何支持多种认证类型切换？

**说明**: `WantParams.type` 数组定义支持的认证类型

**证据**: `Constants.ts` 行 174

```typescript
interface WantParams {
  type: string[];  // 认证类型列表
}
```

### Q12: 认证失败后如何显示剩余尝试次数？

**说明**: 通过 `CmdData.remainAttempts` 获取剩余次数

**证据**: `Constants.ts` 行 151

```typescript
interface CmdData {
  remainAttempts: number;  // 剩余尝试次数
}
```

### Q13: 如何处理认证锁定状态？

**说明**: 根据 `lockoutDuration` 显示倒计时

**证据**: `PasswordAuth.ets` 行 80-85

```typescript
if (payload.remainAttempts === 0 && payload.lockoutDuration) {
  this.countTime(payload.lockoutDuration);
  this.isEdit = false;
  this.textValue = '';
}
```

## 其他问题

### Q14: 这个模块和 user_auth_framework 的关系？

**说明**: auth_widget 是 user_auth_framework 的 UI 层

**证据**: `README.md` 行 5-8

> The Authentication Widget works with the User Authentication Framework to provide a user authentication interaction interface.

### Q15: 为什么模块 RAM 占用为 0KB？

**说明**: RAM 统计的是静态加载占用

**证据**: `bundle.json` 行 20

```json
"ram": "0KB"
```

**解释**: 作为 UI Extension 组件，运行时内存由系统管理，按需分配

## 相关文档

- [概览](00_Overview.md) - 项目定位与运行环境
- [架构说明](01_Architecture.md) - 组件图与数据流
- [认证框架 API](02_UserAuth_API.md) - API 详细说明
- [安全风险评审](20_Security_Review.md) - 安全相关问题
