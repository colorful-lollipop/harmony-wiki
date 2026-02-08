# 09. 常见问题与调试

> 目的: 汇总常见问题和调试方法  
> 适用范围: 开发者、测试人员

---

## 1. 构建问题

### 1.1 构建失败

#### 问题: SDK版本不匹配

**症状**:
```
Error: compileSdkVersion 23 not found
```

**原因**: 本地安装的SDK版本与build-profile.json5中配置的不匹配

**解决**:
```bash
# 检查已安装的SDK版本
ohpm list

# 安装指定版本SDK
ohpm install @ohos/sdk@9

# 或修改build-profile.json5中的版本
{
  "app": {
    "products": [{
      "compileSdkVersion": 9,  // 修改为已安装版本
      "compatibleSdkVersion": 9
    }]
  }
}
```

---

#### 问题: 依赖找不到

**症状**:
```
Error: Cannot find module '@ohos/common'
```

**原因**: 模块依赖未正确配置

**解决**:
```bash
# 1. 清理并重新安装依赖
rm -rf node_modules
ohpm install

# 2. 检查oh-package.json5配置
# 确保模块名和路径正确
{
  "dependencies": {
    "@ohos/common": "file:./common"
  }
}
```

---

#### 问题: 签名失败

**症状**:
```
Error: Signature verification failed
```

**原因**: 签名配置错误或签名文件不存在

**解决**:
```bash
# 1. 检查签名文件存在
ls signature/systemui.p7b

# 2. 重新生成签名
# 使用OpenHarmony SDK的签名工具

# 3. 开发测试时可使用自动签名
# 在build-profile.json5中配置:
{
  "app": {
    "products": [{
      "signingConfig": "default"
    }]
  }
}
```

---

### 1.2 编译警告

#### 警告: TS类型错误

**症状**:
```
Warning: Type 'any' is not assignable to type 'string'
```

**解决**:
```typescript
// 添加类型声明或类型检查
function processData(data: unknown): string {
  if (typeof data === 'string') {
    return data;
  }
  return String(data);
}
```

---

## 2. 运行问题

### 2.1 应用无法启动

#### 问题: Ability找不到

**症状**: 应用安装成功但无法启动，日志显示Ability不存在

**原因**: module.json5中Ability配置错误

**解决**:
```json5
// 检查module.json5中的Ability配置
{
  "module": {
    "abilities": [
      {
        "srcEntry": "./ets/MainAbility/MainAbility.ts",  // 确保路径正确
        "name": "MainAbility"
      }
    ]
  }
}
```

**检查路径**:
- 文件路径大小写是否匹配
- 相对路径是否正确

---

#### 问题: 权限拒绝

**症状**:
```
Error: Permission denied: ohos.permission.ACCESS_SCREEN_LOCK_INNER
```

**原因**: 
1. 权限未在module.json5中声明
2. 系统未授予权限

**解决**:
```json5
// 1. 在module.json5中声明权限
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCESS_SCREEN_LOCK_INNER"
      }
    ]
  }
}
```

```bash
# 2. 检查权限是否授予
hdc shell bm dump -n com.ohos.systemui

# 3. 如果是系统应用，确保使用系统签名
```

---

### 2.2 锁屏不显示

#### 问题: 锁屏窗口未创建

**症状**: 屏幕关闭后再打开，无锁屏界面

**排查步骤**:

```bash
# 1. 检查ServiceExtAbility是否运行
hdc shell ps -ef | grep screenlock

# 2. 检查日志
hdc shell hilog | grep ScreenLock

# 3. 检查窗口是否存在
hdc shell wm dump
```

**常见原因**:

| 原因 | 检查方法 | 解决 |
|------|----------|------|
| ServiceExtAbility未启动 | 检查日志 | 重启设备或重新安装 |
| 窗口创建失败 | 查看WindowManager日志 | 检查窗口权限 |
| 系统服务异常 | 检查system日志 | 重启设备 |

---

#### 问题: 锁屏界面空白

**症状**: 锁屏窗口显示但内容空白

**排查**:
```typescript
// 检查ServiceExtAbility.ts中的窗口创建
private createWindow(name: string) {
    windowManager.create(this.context, name, windowManager.WindowType.TYPE_KEYGUARD)
        .then((win) => {
            // 检查这里是否执行
            Log.showInfo(TAG, "Window created successfully");
            return win.loadContent("pages/index");
        })
        .catch((error) => {
            // 检查错误日志
            Log.showError(TAG, `Window creation failed: ${JSON.stringify(error)}`);
        });
}
```

---

### 2.3 解锁问题

#### 问题: 密码验证失败

**症状**: 输入正确密码但提示错误

**排查步骤**:

```bash
# 1. 检查AccountsModel日志
hdc shell hilog | grep AccountsModel

# 2. 检查认证回调
hdc shell hilog | grep "authUser onResult"
```

**常见原因**:

| 原因 | 检查 | 解决 |
|------|------|------|
| 当前用户不正确 | 检查SwitchUserManager日志 | 确认当前用户 |
| 认证服务异常 | 检查account服务日志 | 重启设备 |
| 密码输入错误 | 检查输入处理 | 确认输入逻辑 |

---

#### 问题: 无法人脸解锁

**症状**: 人脸解锁功能不可用

**排查**:
```bash
# 1. 检查设备是否支持人脸
hdc shell param get const.product.faceauth

# 2. 检查人脸是否已录入
hdc shell bm dump -n com.ohos.settings

# 3. 检查日志
hdc shell hilog | grep -i face
```

---

## 3. 调试方法

### 3.1 日志调试

#### 日志级别

```typescript
import { Log } from '@ohos/common';

// 不同级别的日志
Log.showDebug(TAG, "Debug message");    // 调试信息
Log.showInfo(TAG, "Info message");      // 一般信息
Log.showWarn(TAG, "Warning message");   // 警告
Log.showError(TAG, "Error message");    // 错误
Log.showFatal(TAG, "Fatal message");    // 致命错误
```

#### 查看日志

```bash
# 实时查看日志
hdc shell hilog

# 过滤特定TAG
hdc shell hilog | grep ScreenLock

# 保存日志到文件
hdc shell hilog > screenlock.log

# 清除日志
hdc shell hilog -r
```

---

### 3.2 断点调试

#### DevEco Studio调试

1. 打开项目
2. 在代码中设置断点
3. 选择调试配置
4. 点击调试按钮

#### 支持的断点类型

- 行断点
- 条件断点
- 异常断点

---

### 3.3 性能分析

#### Trace跟踪

```typescript
import { Trace } from '@ohos/common';

// 开始跟踪
Trace.begin(Trace.CORE_METHOD_SHOW_LOCK_SCREEN);

// 执行业务逻辑
this.showLockScreen();

// 结束跟踪
Trace.end(Trace.CORE_METHOD_SHOW_LOCK_SCREEN);
```

#### 查看性能数据

```bash
# 使用hiTrace工具分析
hdc shell hitrace -b 10240 -t 10
```

---

### 3.4 内存分析

```bash
# 查看应用内存
hdc shell hidumper -s Memory -a com.ohos.systemui

# 查看PSS内存
hdc shell cat /proc/$(pidof com.ohos.systemui)/status
```

---

## 4. 常见问题速查

### 4.1 锁屏相关

| 问题 | 快速检查 | 快速解决 |
|------|----------|----------|
| 锁屏不显示 | `hdc shell ps \| grep screenlock` | 重启ServiceExtAbility |
| 无法解锁 | `hdc shell hilog \| grep auth` | 检查用户和密码 |
| 通知不显示 | `hdc shell hilog \| grep notification` | 检查通知权限 |
| 时间不对 | `hdc shell date` | 检查系统时间设置 |

### 4.2 构建相关

| 问题 | 快速检查 | 快速解决 |
|------|----------|----------|
| 构建失败 | `ohpm list` | 重新安装依赖 |
| 签名错误 | `ls signature/` | 重新配置签名 |
| 类型错误 | 检查TypeScript类型 | 添加类型声明 |

### 4.3 运行相关

| 问题 | 快速检查 | 快速解决 |
|------|----------|----------|
| 应用崩溃 | `hdc shell hilog \| grep -i error` | 查看崩溃堆栈 |
| 权限问题 | `hdc shell bm dump -n com.ohos.systemui` | 检查权限声明 |
| ANR | `hdc shell hilog \| grep ANR` | 检查主线程阻塞 |

---

## 5. 调试技巧

### 5.1 快速定位问题

```bash
# 1. 收集所有相关日志
hdc shell hilog | grep -E "(ScreenLock|ServiceExtAbility|accountsModel)" > debug.log

# 2. 检查应用状态
hdc shell bm dump -n com.ohos.systemui

# 3. 检查进程状态
hdc shell ps -ef | grep systemui

# 4. 检查窗口状态
hdc shell wm dump

# 5. 检查权限
hdc shell aa dump -a
```

### 5.2 模拟测试场景

```bash
# 模拟屏幕关闭
hdc shell power-shell timeout

# 模拟屏幕打开
hdc shell power-shell wakeup

# 发送测试通知
hdc shell aa start -a ohos.test.notification

# 切换用户
hdc shell aa start -a ohos.settings.user
```

### 5.3 热更新调试

```bash
# 1. 修改代码后快速编译
hvigor assemble

# 2. 推送到设备
hdc file send entry.hap /data/app/el1/100/base/com.ohos.systemui/

# 3. 重启应用
hdc shell aa force-stop com.ohos.systemui
hdc shell aa start -a com.ohos.systemui.MainAbility
```

---

## 6. 相关资源

### 6.1 日志TAG列表

| TAG | 模块 | 用途 |
|-----|------|------|
| ScreenLock-ServiceExtAbility | phone | ServiceExtAbility日志 |
| ScreenLockManager | common | 屏幕状态管理 |
| WindowManagerSc | common | 窗口管理 |
| ScreenLock-AccountsModel | screenlock | 账户模型 |
| ScreenLock-Service | screenlock | 锁屏服务 |
| NotificationManager | noticeitem | 通知管理 |

### 6.2 调试工具

| 工具 | 用途 |
|------|------|
| hdc | 设备连接和调试 |
| hilog | 日志查看 |
| hidumper | 系统信息导出 |
| hitrace | 性能跟踪 |

---

*提示: 遇到问题时，首先收集完整日志，然后根据日志TAG定位问题模块，最后查看对应代码。*
