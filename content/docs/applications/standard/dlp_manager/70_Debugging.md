# 调试指南

## 目的

本文档提供DLP Manager的调试方法、日志抓取指南和常见问题定位路径，帮助开发者快速定位和解决问题。

## 适用范围

- 进行问题定位的开发人员
- 需要调试DLP功能的测试人员
- 处理用户反馈的运维人员

---

## 日志系统

### HiLog日志

DLP Manager使用OpenHarmony的HiLog系统进行日志输出。

**日志工具位置**: `entry/src/main/ets/common/HiLog.ets`

### 日志级别

| 级别 | 方法 | 用途 |
|------|------|------|
| DEBUG | `HiLog.debug()` | 详细调试信息 |
| INFO | `HiLog.info()` | 关键流程信息 |
| WARN | `HiLog.warn()` | 警告信息 |
| ERROR | `HiLog.error()` | 错误信息 |
| ERROR(with error) | `HiLog.wrapError()` | 包装错误信息 |

### 日志TAG定义

| TAG | 用途 | 代码位置 |
|-----|------|----------|
| `MainEx` | MainAbilityEx日志 | `entry/src/main/ets/Ability/MainAbilityEx.ets:49` |
| `View` | ViewAbility日志 | `entry/src/main/ets/Ability/ViewAbility.ets:29` |
| `Utils` | 文件工具日志 | `entry/src/main/ets/common/FileUtils/utils.ets:41` |
| `OpenDlpFileManager` | DLP文件管理日志 | `entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets:24` |
| `OpenDlpFileProcessor` | 文件处理流程日志 | `entry/src/main/ets/OpenDlpFile/ViewProcessor/ViewProcessor.ets:38` |
| `CredConnectService` | RPC连接日志 | `entry/src/main/ets/rpc/CredConnectService.ets:23` |
| `HuksCipherUtils` | 加密工具日志 | `entry/src/main/ets/common/huks/HuksCipherUtil.ets:20` |

---

## 日志抓取方法

### 方法1: 使用hilog命令

```bash
# 1. 关闭PID过滤（抓取所有日志）
hdc shell hilog -Q pidoff

# 2. 设置日志级别为Debug
hdc shell hilog -b D

# 3. 清除旧日志
hdc shell hilog -r

# 4. 实时查看日志
hdc hilog | grep -E "(MainEx|View|OpenDlpFile|DLP)"

# 5. 保存日志到文件
hdc hilog > dlp_log.txt
```

### 方法2: 抓取特定TAG日志

```bash
# 抓取MainAbilityEx日志
hdc hilog | grep "MainEx"

# 抓取ViewAbility日志
hdc hilog | grep "View"

# 抓取所有DLP相关日志
hdc hilog | grep -i "dlp"
```

### 方法3: 按日志级别过滤

```bash
# 只抓取Error级别以上日志
hdc shell hilog -b E
hdc hilog
```

---

## 常见问题定位

### 问题1: 无法启动DLP Manager

**现象**: 调用startAbility后无响应或报错

**定位路径**:

1. **检查Ability名称**
   ```typescript
   // 确认bundleName和abilityName正确
   bundleName: 'com.ohos.dlpmanager'
   abilityName: 'MainAbilityEx'  // 或 ViewAbility
   ```

2. **检查权限声明**
   ```bash
   # 查看应用权限
   hdc shell bm dump -n com.ohos.dlpmanager | grep permission
   ```

3. **查看启动日志**
   ```bash
   hdc hilog | grep -E "(MainEx|ability)"
   ```

**常见原因**:
- 权限未声明（`ohos.permission.ACCESS_DLP_FILE`）
- Ability名称错误
- HAP包未正确安装

**代码参考**: `entry/src/main/ets/Ability/MainAbilityEx.ets:98-119`

### 问题2: DLP文件无法打开

**现象**: 点击DLP文件后报错或无任何反应

**定位路径**:

1. **检查参数完整性**
   ```bash
   # 查看ViewAbility日志
   hdc hilog | grep "View"
   ```
   
   查找: `need parameters in want`、`need fileName`等错误

2. **检查文件路径**
   ```bash
   # 查看路径验证日志
   hdc hilog | grep -E "(invalid uri|checkValidWant)"
   ```

3. **检查账号信息**
   ```bash
   # 查看账号相关日志
   hdc hilog | grep -E "(getOsAccountInfo|checkDomainAccountInfo)"
   ```

**常见原因**:
- Want参数缺失（`callerToken`、`callerBundleName`）
- URI格式错误（必须以`file://`开头）
- 当前用户未登录域账号
- DLP文件格式损坏

**代码参考**: `entry/src/main/ets/Ability/MainAbilityEx.ets:197-246`

### 问题3: 权限检查失败

**现象**: 提示"无权限访问"或权限级别错误

**定位路径**:

1. **查看权限计算日志**
   ```bash
   hdc hilog | grep "getAuthPerm"
   ```

2. **检查DLP属性**
   ```bash
   # 查看DLP属性相关日志
   hdc hilog | grep -E "(dlpProperty|authUserList)"
   ```

3. **检查账号匹配**
   ```bash
   # 查看账号信息日志
   hdc hilog | grep -E "(accountName|ownerAccount)"
   ```

**常见原因**:
- 当前用户不在授权用户列表
- DLP文件已过期
- 权限级别配置错误

**代码参考**: `entry/src/main/ets/common/FileUtils/utils.ets:173-189`

### 问题4: 沙箱启动失败

**现象**: DLP文件解密成功但无法打开应用

**定位路径**:

1. **查看沙箱启动日志**
   ```bash
   hdc hilog | grep -E "(StartSandboxHandler|startSandbox)"
   ```

2. **检查应用安装状态**
   ```bash
   # 查看沙箱应用安装
   hdc shell bm dump -n <sandbox_bundle_name>
   ```

3. **检查URI权限**
   ```bash
   hdc hilog | grep "grantUriPermission"
   ```

**常见原因**:
- 目标应用未安装
- 沙箱安装失败
- URI权限授予失败

**代码参考**: `entry/src/main/ets/OpenDlpFile/handler/StartSandboxHandler.ets`

### 问题5: 加密失败

**现象**: 设置权限后无法生成DLP文件

**定位路径**:

1. **查看加密页面日志**
   ```bash
   hdc hilog | grep "Encrypt"
   ```

2. **检查账号验证**
   ```bash
   hdc hilog | grep -E "(checkAccountInfo|AccountManager)"
   ```

3. **检查DLP服务调用**
   ```bash
   hdc hilog | grep -E "(generateDlpFile|dlpPermission)"
   ```

**常见原因**:
- 域账号未认证
- DLP Permission Service异常
- 磁盘空间不足

**代码参考**: `entry/src/main/ets/pages/encryptionProtection.ets`

---

## 调试技巧

### 技巧1: 增加调试日志

在需要调试的位置添加日志：

```typescript
import { HiLog } from '../common/HiLog';

const TAG = 'Debug';

// 记录变量值
HiLog.info(TAG, `variable value: ${JSON.stringify(variable)}`);

// 记录流程进入
HiLog.info(TAG, 'Enter function XXX');

// 记录错误详情
HiLog.wrapError(TAG, error, 'Operation failed');
```

### 技巧2: 使用GlobalContext传递调试信息

```typescript
import GlobalContext from '../common/GlobalContext';

// 存储调试信息
GlobalContext.store('debugInfo', { 
  timestamp: Date.now(),
  param: paramValue 
});

// 在其他位置获取
const debugInfo = GlobalContext.load('debugInfo');
HiLog.info(TAG, `Debug info: ${JSON.stringify(debugInfo)}`);
```

### 技巧3: 检查AppStorage状态

```typescript
// 在调试位置打印AppStorage
HiLog.info(TAG, `AppStorage keys: ${AppStorage.keys()}`);
HiLog.info(TAG, `authPerm: ${AppStorage.get('authPerm')}`);
HiLog.info(TAG, `accountDomain: ${AppStorage.get('accountDomain')}`);
```

### 技巧4: 使用hitrace跟踪性能

```typescript
import hiTraceMeter from '@ohos.hiTraceMeter';

// 开始跟踪
hiTraceMeter.startTrace('DlpOpenFileJs', startId);

// 执行操作
await processOpenDlpFile();

// 结束跟踪
hiTraceMeter.finishTrace('DlpOpenFileJs', startId);
```

---

## 错误码速查

### 常见错误码对照表

| 错误码 | 常量 | 含义 | 常见原因 |
|--------|------|------|----------|
| 0 | SUCCESS | 成功 | - |
| 1 | ERR_JS_APP_INSIDE_ERROR | 应用内部错误 | 未预期的异常 |
| 2 | ERR_JS_GET_ACCOUNT_ERROR | 获取账号失败 | 域账号未登录 |
| 4 | ERR_JS_APP_PARAM_ERROR | 参数错误 | Want参数缺失或格式错误 |
| 6 | ERR_JS_APP_OPEN_REJECTED | 打开被拒绝 | 文件正在其他地方打开 |
| 11 | ERR_JS_APP_CANNOT_OPEN | 无法打开 | 文件格式不支持 |
| 19100013 | ERR_JS_USER_NO_PERMISSION | 用户无权限 | 不在授权列表 |
| 19100014 | ERR_JS_ACCOUNT_NOT_LOGIN | 账号未登录 | 域账号未认证 |
| 19100019 | ERR_JS_FILE_EXPIRATION | 文件已过期 | 超过有效期 |
| -1 | ERR_CODE_OPEN_FILE_ERROR | 打开文件错误 | 文件不存在或无权限 |
| -2 | ERR_CODE_PARAMS_CHECK_ERROR | 参数检查错误 | 参数格式错误 |

---

## 测试验证

### 验证DLP文件打开流程

```bash
# 1. 清理日志
hdc shell hilog -r

# 2. 点击DLP文件

# 3. 抓取日志
hdc hilog | grep -E "(View|OpenDlpFile|process)" > open_dlp_log.txt

# 4. 检查关键流程
# - ViewAbility onRequest
# - OpenDlpFileProcessor.process
# - checkAndSetWantParams
# - parseFile
# - handleAccount
# - decryptAndInstall
# - startSandbox
```

### 验证权限检查流程

```bash
# 1. 打开DLP设置页面

# 2. 抓取日志
hdc hilog | grep -E "(MainEx|getAuthPerm|checkAccount)" > auth_log.txt

# 3. 检查关键流程
# - MainAbilityEx.onSessionCreate
# - checkValidWantAndAccount
# - getAuthPerm
```

---

## 相关命令汇总

### HDC常用命令

```bash
# 安装HAP包
hdc install dlp_manager.hap

# 卸载应用
hdc shell bm uninstall -n com.ohos.dlpmanager

# 查看应用信息
hdc shell bm dump -n com.ohos.dlpmanager

# 清除应用数据
hdc shell bm clean -n com.ohos.dlpmanager

# 查看日志
hdc hilog

# 清除日志
hdc shell hilog -r

# 设置日志级别
hdc shell hilog -b D  # Debug
hdc shell hilog -b I  # Info
hdc shell hilog -b W  # Warn
hdc shell hilog -b E  # Error

# 关闭PID过滤
hdc shell hilog -Q pidoff
```

### 文件操作命令

```bash
# 查看DLP文件
hdc shell ls -la /data/storage/el2/base/files/*.dlp

# 拉取文件到本地
hdc file recv /data/storage/el2/base/files/test.doc.dlp ./

# 推送文件到设备
hdc file send ./test.doc /data/storage/el2/base/files/
```

---

## 关键结论

1. **日志TAG**: 使用特定TAG过滤日志可快速定位问题
2. **参数检查**: 大部分问题源于Want参数不完整或格式错误
3. **账号认证**: 域账号认证是常见失败点
4. **权限检查**: 权限计算逻辑需仔细核对
5. **沙箱启动**: 沙箱启动失败通常与目标应用有关

---

## 相关链接

- [安全风险](60_Security_Analysis.md) - 了解安全相关日志
- [对外接口](30_Public_API.md) - 查看接口参数要求
- [架构设计](10_Architecture.md) - 了解系统流程
